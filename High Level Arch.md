High-Level Architecture
                ┌────────────────────┐
                │   Developer Code   │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │   CI/CD Pipeline   │ (GitHub Actions / Jenkins)
                └─────────┬──────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Unit Tests   │  │ API Tests    │  │ UI Tests     │
│ (Dev owned)  │  │ Playwright   │  │ Playwright   │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └────────────┬────┴────┬────────────┘
                    ▼         ▼
             ┌────────────────────┐
             │ Test Results Layer │
             └──────┬─────────────┘
                    │
        ┌───────────┼────────────┐
        ▼                        ▼
┌──────────────┐        ┌────────────────┐
│ PostgreSQL   │        │ Logs (JSON)    │
│ (Results DB) │        │ / Elastic (opt)│
└──────┬───────┘        └──────┬─────────┘
       │                       │
       ▼                       ▼
         ┌────────────────────────────┐
         │ Grafana Dashboard          │
         │ (Pass %, trends, failures) │
         └────────────────────────────┘
📁 2. Project Folder Structure (Playwright + TS)
test-automation-framework/
│
├── src/
│   ├── api/
│   │   ├── clients/
│   │   │   └── paymentClient.ts
│   │   ├── tests/
│   │   │   └── payment.spec.ts
│   │   └── schemas/
│   │       └── payment.schema.ts
│   │
│   ├── ui/
│   │   ├── pages/
│   │   │   └── LoginPage.ts
│   │   ├── tests/
│   │   │   └── login.spec.ts
│   │   └── locators/
│   │
│   ├── utils/
│   │   ├── logger.ts
│   │   ├── db.ts
│   │   ├── config.ts
│   │   └── testData.ts
│   │
│   ├── hooks/
│   │   ├── globalSetup.ts
│   │   ├── globalTeardown.ts
│   │   └── testHooks.ts
│
├── reports/
│   ├── playwright-report/
│   └── logs/
│
├── docker/
│   ├── docker-compose.yml
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
⚙️ 3. Core Components Explained
🧪 3.1 Playwright for UI + API

Playwright supports both → single framework advantage

Example API Test
import { test, expect } from '@playwright/test';

test('Create Payment API', async ({ request }) => {
  const response = await request.post('/api/payment', {
    data: { amount: 1000, currency: 'INR' }
  });

  expect(response.status()).toBe(201);
});
Example UI Test (POM)
import { test } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('Login Test', async ({ page }) => {
  const login = new LoginPage(page);
  await login.goto();
  await login.login('user', 'pass');
});
🗄️ 3.2 Logging + Store in DB (PostgreSQL)
DB Schema
CREATE TABLE test_results (
    id SERIAL PRIMARY KEY,
    test_name TEXT,
    status TEXT,
    execution_time INT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
DB Utility
import { Pool } from 'pg';

const pool = new Pool({
  user: 'postgres',
  host: 'localhost',
  database: 'testdb',
  password: 'password',
  port: 5432,
});

export const saveResult = async (testName: string, status: string, time: number) => {
  await pool.query(
    'INSERT INTO test_results(test_name, status, execution_time) VALUES($1,$2,$3)',
    [testName, status, time]
  );
};
📜 3.3 Custom Logger
import fs from 'fs';

export const log = (message: string) => {
  const logMessage = `${new Date().toISOString()} - ${message}`;
  fs.appendFileSync('reports/logs/test.log', logMessage + '\n');
};
🔁 3.4 Hooks (Capture Results Automatically)
import { test } from '@playwright/test';
import { saveResult } from '../utils/db';

test.afterEach(async ({}, testInfo) => {
  await saveResult(
    testInfo.title,
    testInfo.status,
    testInfo.duration
  );
});
📊 4. Grafana Integration
Step 1: Run via Docker
version: '3'
services:
  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
Step 2: Connect Grafana to PostgreSQL
Add Data Source → PostgreSQL
DB: testdb
Step 3: Dashboard Queries
Pass/Fail Rate
SELECT status, COUNT(*) FROM test_results GROUP BY status;
Execution Trend
SELECT timestamp, COUNT(*) 
FROM test_results 
GROUP BY timestamp;
🚀 5. CI/CD Pipeline (GitHub Actions)
name: Playwright Tests

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres
        ports: [5432:5432]
        env:
          POSTGRES_PASSWORD: password

    steps:
      - uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - run: npm install

      - run: npx playwright install

      - run: npm run test

      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
⚡ 6. Advanced Enhancements (Highly Recommended)
🔹 6.1 Parallel Execution
workers: 4
🔹 6.2 Retry Failed Tests
retries: 2
🔹 6.3 Environment Config
export const config = {
  baseURL: process.env.BASE_URL || 'https://dev-app.com'
};
🔹 6.4 Tagging Tests
test.describe('@smoke', () => { ... });

Run:

npx playwright test --grep @smoke
🔹 6.5 Add Allure Reporting (Optional)

Better than default reports