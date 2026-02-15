# Лабораторна робота №6
## CI/CD (GitHub Actions)

**Максимальна оцінка:** 15 балів

### Мета роботи

Навчитися налаштовувати автоматичний запуск тестів у CI/CD pipeline за допомогою GitHub Actions.

---

### Підготовка

1. Переконайтеся, що ваш репозиторій `schedule-testing` на GitHub
2. Створіть директорію `.github/workflows/` в корені репозиторію
3. Переконайтеся, що всі тести проходять локально

---

## Завдання

### Завдання 1: Базовий workflow для тестів (5 балів)

Створіть workflow, який запускає ваші тести при кожному push та pull request.

<details>
<summary><strong>JavaScript (Jest + Playwright)</strong></summary>

**.github/workflows/tests.yml:**
```yaml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm test -- --coverage
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: unit-tests
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npx playwright test
      
      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

</details>

<details>
<summary><strong>Java (Gradle)</strong></summary>

**.github/workflows/tests.yml:**
```yaml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'
      
      - name: Run unit tests
        run: ./gradlew test
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: build/reports/tests/

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: unit-tests
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'
      
      - name: Install Playwright
        run: ./gradlew playwright --args="install --with-deps"
      
      - name: Run E2E tests
        run: ./gradlew test -Dtest.include=**/e2e/**
```

</details>

<details>
<summary><strong>Java (Maven)</strong></summary>

**.github/workflows/tests.yml:**
```yaml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Run unit tests
        run: ./mvnw test -Dtest=*Test
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: target/surefire-reports/

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: unit-tests
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Install Playwright
        run: mvn exec:java -e -D exec.mainClass=com.microsoft.playwright.CLI -D exec.args="install --with-deps"
      
      - name: Run E2E tests
        run: ./mvnw test -Dtest=*E2ETest
```

</details>

<details>
<summary><strong>Python (pytest + Playwright)</strong></summary>

**.github/workflows/tests.yml:**
```yaml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run unit tests
        run: pytest tests/unit --cov=src --cov-report=html
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: htmlcov/

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: unit-tests
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest-playwright
          playwright install --with-deps
      
      - name: Run E2E tests
        run: pytest tests/e2e
```

</details>

---

### Завдання 2: API-тести в pipeline (4 бали)

Додайте job для запуску API-тестів.

<details>
<summary><strong>Варіант з Newman (Postman CLI)</strong></summary>

```yaml
  api-tests:
    name: API Tests (Newman)
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install Newman
        run: npm install -g newman newman-reporter-htmlextra
      
      - name: Run Postman collection
        run: |
          newman run tests/api/postman/Schedule_API_Tests.postman_collection.json \
            -e tests/api/postman/schedule-ci.postman_environment.json \
            -r htmlextra \
            --reporter-htmlextra-export newman-report.html
      
      - name: Upload Newman report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: newman-report
          path: newman-report.html
```

</details>

<details>
<summary><strong>Варіант з кодом (Jest / REST Assured / pytest)</strong></summary>

```yaml
  api-tests:
    name: API Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      # ... setup мови (Node.js / Java / Python) ...
      
      - name: Run API tests
        run: npm test -- tests/api/    # або відповідна команда
        env:
          API_URL: ${{ secrets.API_URL }}
```

</details>

---

### Завдання 3: Матриця тестування (3 бали)

Налаштуйте запуск тестів на різних версіях або браузерах. Оберіть один або кілька варіантів.

<details>
<summary><strong>Матриця версій — Node.js</strong></summary>

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      
      - run: npm ci
      - run: npm test
```

</details>

<details>
<summary><strong>Матриця версій — Java (Gradle)</strong></summary>

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: [17, 21]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java ${{ matrix.java-version }}
        uses: actions/setup-java@v4
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
          cache: 'gradle'
      
      - run: ./gradlew test
```

</details>

<details>
<summary><strong>Матриця версій — Java (Maven)</strong></summary>

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: [17, 21]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java ${{ matrix.java-version }}
        uses: actions/setup-java@v4
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
          cache: 'maven'
      
      - run: ./mvnw test
```

</details>

<details>
<summary><strong>Матриця версій — Python</strong></summary>

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      
      - run: pip install -r requirements.txt
      - run: pytest
```

</details>

<details>
<summary><strong>Матриця браузерів (Playwright)</strong></summary>

```yaml
jobs:
  e2e:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chromium, firefox, webkit]
    
    steps:
      - uses: actions/checkout@v4
      
      # ... setup мови ...
      
      - name: Install Playwright browser
        run: npx playwright install --with-deps ${{ matrix.browser }}
      
      - name: Run E2E tests on ${{ matrix.browser }}
        run: npx playwright test --project=${{ matrix.browser }}
```

</details>

<details>
<summary><strong>Матриця ОС</strong></summary>

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    
    steps:
      # ...
```

</details>

---

### Завдання 4: Звіти та нотифікації (3 бали)

#### Додайте статус badge до README.md

```markdown
# Schedule Testing

![Tests](https://github.com/USERNAME/schedule-testing/actions/workflows/tests.yml/badge.svg)

## Опис проєкту
...
```

<details>
<summary><strong>Публікація HTML-звітів на GitHub Pages (опційно)</strong></summary>

```yaml
      - name: Deploy Playwright Report to GitHub Pages
        if: always()
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./playwright-report
```

</details>

<details>
<summary><strong>Нотифікація в Slack/Discord (опційно)</strong></summary>

```yaml
      - name: Notify Slack on failure
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          fields: repo,message,commit,author
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

</details>

---

### Повний приклад workflow

Нижче наведено приклад повного pipeline для **JavaScript**-треку. Адаптуйте під свій стек.

<details>
<summary><strong>Повний .github/workflows/tests.yml (JavaScript)</strong></summary>

```yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'

jobs:
  # ============================================
  # Unit Tests
  # ============================================
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests with coverage
        run: npm test -- --coverage
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
          retention-days: 7

  # ============================================
  # API Tests
  # ============================================
  api-tests:
    name: API Tests
    runs-on: ubuntu-latest
    needs: unit-tests
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Newman
        run: npm install -g newman newman-reporter-htmlextra
      
      - name: Run API tests
        run: |
          newman run tests/api/postman/Schedule_API_Tests.postman_collection.json \
            -e tests/api/postman/schedule-ci.postman_environment.json \
            -r htmlextra \
            --reporter-htmlextra-export newman-report.html
        continue-on-error: true
      
      - name: Upload Newman report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: api-test-report
          path: newman-report.html

  # ============================================
  # E2E Tests
  # ============================================
  e2e-tests:
    name: E2E Tests (${{ matrix.browser }})
    runs-on: ubuntu-latest
    needs: api-tests
    strategy:
      fail-fast: false
      matrix:
        browser: [chromium, firefox]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browser
        run: npx playwright install --with-deps ${{ matrix.browser }}
      
      - name: Run E2E tests
        run: npx playwright test --project=${{ matrix.browser }}
      
      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report-${{ matrix.browser }}
          path: playwright-report/
          retention-days: 7

  # ============================================
  # Summary
  # ============================================
  test-summary:
    name: Test Summary
    runs-on: ubuntu-latest
    needs: [unit-tests, api-tests, e2e-tests]
    if: always()
    
    steps:
      - name: Check test results
        run: |
          echo "Unit Tests: ${{ needs.unit-tests.result }}"
          echo "API Tests: ${{ needs.api-tests.result }}"
          echo "E2E Tests: ${{ needs.e2e-tests.result }}"
          
          if [ "${{ needs.unit-tests.result }}" == "failure" ] || \
             [ "${{ needs.api-tests.result }}" == "failure" ] || \
             [ "${{ needs.e2e-tests.result }}" == "failure" ]; then
            echo "Some tests failed!"
            exit 1
          fi
          
          echo "All tests passed!"
```

</details>

---

### Що має бути в результаті

```
schedule-testing/
├── .github/
│   └── workflows/
│       └── tests.yml           ← CI/CD pipeline
├── tests/
│   ├── unit/
│   ├── api/
│   └── e2e/
├── README.md                   ← Зі status badge
└── ...
```

**GitHub:**
- Workflow запускається при push/PR
- Всі jobs проходять успішно (зелений статус)
- Артефакти (звіти) доступні для завантаження

---

### Критерії оцінювання

| Критерій | Бали |
|----------|------|
| Базовий workflow (unit + e2e тести) | 5 |
| API-тести в pipeline | 4 |
| Матриця тестування (браузери або версії) | 3 |
| Звіти, артефакти, badge в README | 3 |
| **Разом** | **15** |

### Контрольні питання

1. Що таке CI/CD і навіщо воно потрібне?
2. Яка різниця між `push` та `pull_request` тригерами?
3. Що таке job і як jobs можуть залежати один від одного?
4. Навіщо використовувати матрицю (strategy.matrix)?
5. Як зберігати секрети (API ключі, паролі) в GitHub Actions?
6. Що таке артефакти і навіщо їх зберігати?

### Рекомендовані джерела

1. GitHub Actions Documentation: https://docs.github.com/en/actions
2. Playwright CI: https://playwright.dev/docs/ci-intro
3. Newman GitHub Action: https://github.com/marketplace/actions/newman-action
