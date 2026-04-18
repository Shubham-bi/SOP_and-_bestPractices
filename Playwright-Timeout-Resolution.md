# Playwright Timeout Issue Resolution

## Issue Description
When executing the test script `farmerMilkCollectionFlow.spec.js` against the production environment (`https://hitechdairy.in/login`), the test consistently failed with a Playwright `TimeoutError` indicating that `30000ms exceeded` during `page.goto()`. However, the exact same URL worked perfectly fine when opened manually in a regular Chrome or Edge browser.

## Root Cause Analysis
The investigation revealed a combination of two networking and Playwright defaults that caused the failure:

### 1. The Strict 30-Second "Load" Timeout
By default, Playwright's `page.goto` function waits for the Javascript `load` event to fire. The `load` event only occurs after the initial HTML structure **AND** all external resources (such as fonts, images, scripts, and API calls) have finished downloading.
The `hitechdairy.in/login` page relies on multiple external scripts, including the Google Maps API, Fancybox, and Bootstrap widgets. If any single background resource is slow or hangs, Playwright will strictly enforce its maximum 30,000ms budget and hard crash the script. 

Regular web browsers do not have a fatal 30-second timeout; they simply continue to show the loading spinner while rendering the page content. This is why the page appeared "working perfectly fine" to the human eye in the browser, while Playwright considered it a failure. Furthermore, the test originally included `await page.waitForLoadState('networkidle');`, which was completely unreachable because these background resources never settled entirely to zero active connections within the timeframe.

### 2. IPv6 Routing Delays (NAT64 Prefix)
Internal network traces showed that the `hitechdairy.in` domain resolves to both IPv4 and IPv6 addresses. When executed programmatically, the underlying `dns.lookup` returned IPv6 results first. Because of potential ISP or local network routing configurations (such as NAT64 tunnels that blackhole specific IPv6 packets), Node.js spent nearly **21 seconds** waiting for the unresponsive IPv6 address to trigger a TCP timeout before falling back to IPv4. 

Standard browsers utilize the "Happy Eyeballs" protocol (RFC 8305) which simultaneously tries IPv4 if IPv6 doesn't respond within 300 milliseconds. Playwright automated scripts do not natively execute this fallback as cleanly, resulting in 21 seconds of the 30-second timeout budget being squandered before the page even began loading over IPv4.

## Solution

To solve this issue, the Playwright navigation step was modified to prioritize the DOM content rather than waiting for external network resources to fully settle.

**Before:**
```javascript
await page.goto('https://hitechdairy.in/login');
await page.waitForLoadState('networkidle');
```

**After:**
```javascript
// Use domcontentloaded to prevent timeouts on slow background resources
await page.goto('https://hitechdairy.in/login', { waitUntil: 'domcontentloaded' });
// Removed waitForLoadState('networkidle')
```

### Why this works:
Using `waitUntil: 'domcontentloaded'` instructs Playwright to consider the navigation successful as soon as the initial HTML document has been completely loaded and parsed, without rigidly waiting for all stylesheets, heavy third-party tracking APIs, mapping iframes, or IPv6-related resource lags to finish.

### Optional Alternative Solutions
If forced IPv6 fallbacks become a consistent issue across the suite, Chromium can be explicitly instructed to prioritize IPv4 in the Playwright `playwright.config.js` or browser launch options:
```javascript
const browser = await chromium.launch({ 
    args: ['--disable-ipv6']
});
```

### 3. Angular Rendering and DOM Readiness
**Issue:** After switching to `waitUntil: 'domcontentloaded'`, the initial navigation timeout was resolved. However, the script began immediately searching for the form inputs (like `username`) before the underlying Single-Page Application framework (Angular) had finished fully "hydrating" and rendering them into the interactive DOM. Since Playwright's default element assertions can sometimes be too brief during heavy framework boots, the test would crash when trying to `.fill()` an input that didn't exist yet.

**Solution:** Explicit element waiting was added directly before the interaction.
```javascript
// Wait explicitly for the Angular form to render the username input (Wait up to 15 seconds)
const usernameInput = page.locator("//input[@formcontrolname='username']");
await usernameInput.waitFor({ state: 'visible', timeout: 15000 });
```
This guarantees Playwright accurately waits for the internal framework to display the component, making the test far more robust against performance fluctuations.
