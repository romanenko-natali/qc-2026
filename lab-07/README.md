# Додаткові завдання (бонусні бали)
## Security та Performance тестування

**Максимальна оцінка:** 10 бонусних балів

> Ці завдання є необов'язковими та дають додаткові бали понад основну оцінку за лабораторну.

---

### Підготовка

Для цих завдань можна використовувати:
- **Ваш додаток** (локально або в Docker)
- **Безкоштовні онлайн-додатки** для практики (див. нижче)

---

### Завдання A: Performance тестування з k6 (5 балів)

Використайте **k6** для навантажувального тестування API.

<details>
<summary><strong>Встановлення k6</strong></summary>

**macOS:**
```bash
brew install k6
```

**Linux (Ubuntu/Debian):**
```bash
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D68
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6
```

**Docker:**
```bash
docker run --rm -i grafana/k6 run - <tests/performance/load-test.js
```

**Windows:**
```bash
winget install k6 --source winget
```

</details>

#### Цільові додатки для тестування

Можна тестувати ваш додаток **або** один із безкоштовних онлайн-сервісів:

| Додаток | URL | Опис |
|---------|-----|------|
| **k6 Test Site** | `https://test.k6.io` | Офіційний демо-сайт від k6 для практики load testing |
| **QuickPizza** | `https://quickpizza.grafana.com` | Демо-додаток Grafana з REST API |
| **Ваш додаток** | `http://localhost:8080` | Schedule API (локально або Docker) |

> ⚠️ **Важливо:** не навантажуйте онлайн-сервіси великою кількістю користувачів (до 20 VUs). Це демо-сервери для навчання.

#### Що потрібно зробити

1. Створіть файл `tests/performance/load-test.js`
2. Напишіть тест, що перевіряє **2-3 ендпоінти**
3. Визначте пороги (thresholds) для часу відповіді

<details>
<summary><strong>Приклад: тестування k6 Test Site</strong></summary>

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '10s', target: 5 },    // розігрів: 0 → 5 користувачів
    { duration: '30s', target: 10 },   // тримаємо 10 користувачів
    { duration: '10s', target: 0 },    // завершення
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],   // 95% запитів < 500ms
    http_req_failed: ['rate<0.05'],     // менше 5% помилок
  },
};

export default function () {
  // Тест 1: Головна сторінка
  const main = http.get('https://test.k6.io/');
  check(main, {
    'main: status 200': (r) => r.status === 200,
    'main: response < 300ms': (r) => r.timings.duration < 300,
  });

  // Тест 2: Контакти
  const contacts = http.get('https://test.k6.io/contacts.php');
  check(contacts, {
    'contacts: status 200': (r) => r.status === 200,
  });

  // Тест 3: Новини
  const news = http.get('https://test.k6.io/news.php');
  check(news, {
    'news: status 200': (r) => r.status === 200,
    'news: response < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);
}
```

</details>

<details>
<summary><strong>Приклад: тестування QuickPizza API</strong></summary>

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '10s', target: 5 },
    { duration: '30s', target: 10 },
    { duration: '10s', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<600'],
    http_req_failed: ['rate<0.05'],
  },
};

const BASE_URL = 'https://quickpizza.grafana.com';

export default function () {
  // Тест 1: API — список піц
  const pizzas = http.get(`${BASE_URL}/api/pizza`);
  check(pizzas, {
    'pizzas: status 200': (r) => r.status === 200,
    'pizzas: has data': (r) => JSON.parse(r.body).length > 0,
  });

  // Тест 2: API — рекомендація
  const recommend = http.post(
    `${BASE_URL}/api/pizza`,
    JSON.stringify({
      maxCaloriesPerSlice: 300,
      mustBeVegetarian: false,
      excludedIngredients: [],
      excludedTools: [],
      maxNumberOfToppings: 5,
      minNumberOfToppings: 2,
    }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  check(recommend, {
    'recommend: status 200': (r) => r.status === 200,
  });

  sleep(1);
}
```

</details>

<details>
<summary><strong>Приклад: тестування вашого додатку (Schedule API)</strong></summary>

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '10s', target: 10 },
    { duration: '30s', target: 10 },
    { duration: '10s', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.05'],
  },
};

const BASE_URL = 'http://localhost:8080/api';

export default function () {
  const groups = http.get(`${BASE_URL}/groups`);
  check(groups, {
    'groups: status 200': (r) => r.status === 200,
    'groups: response < 300ms': (r) => r.timings.duration < 300,
  });

  const subjects = http.get(`${BASE_URL}/subjects`);
  check(subjects, {
    'subjects: status 200': (r) => r.status === 200,
  });

  const schedule = http.get(`${BASE_URL}/schedules`);
  check(schedule, {
    'schedule: status 200': (r) => r.status === 200,
    'schedule: response < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);
}
```

</details>

#### Запуск та звіт

```bash
k6 run tests/performance/load-test.js
```

Створіть файл `docs/performance-report.md`:

```markdown
# Performance Report

## Цільовий додаток
- URL: (вкажіть що тестували)
- Дата: ...

## Параметри тесту
- Кількість віртуальних користувачів: до 10
- Тривалість: 50 секунд

## Результати
| Ендпоінт | Avg | P95 | Max | Помилки |
|----------|-----|-----|-----|---------|
| GET /... | XXms | XXms | XXms | 0% |
| GET /... | XXms | XXms | XXms | 0% |
| POST /... | XXms | XXms | XXms | 0% |

## Висновок
- Які ендпоінти найповільніші?
- Чи витримує додаток навантаження?
- Чи пройшли пороги (thresholds)?
```

<details>
<summary><strong>Бонус: додайте k6 в CI pipeline</strong></summary>

```yaml
  performance-tests:
    name: Performance Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run k6 tests against test.k6.io
        uses: grafana/k6-action@v0.3.1
        with:
          filename: tests/performance/load-test.js
```

Якщо тестуєте свій додаток:

```yaml
      - name: Start application
        run: docker-compose up -d
      
      - name: Wait for app to be ready
        run: |
          for i in {1..30}; do
            curl -s http://localhost:8080/api/groups && break
            sleep 2
          done
      
      - name: Run k6 tests
        uses: grafana/k6-action@v0.3.1
        with:
          filename: tests/performance/load-test.js
      
      - name: Stop application
        if: always()
        run: docker-compose down
```

</details>

---

### Завдання B: Security тестування (5 балів)

Перевірте базову безпеку веб-додатку.

#### Цільові додатки

| Додаток | URL | Як запустити |
|---------|-----|--------------|
| **OWASP Juice Shop** (онлайн) | `https://demo.owasp-juice.shop` | Вже працює, відкрийте в браузері |
| **OWASP Juice Shop** (Docker) | `http://localhost:3000` | `docker run -p 3000:3000 bkimminich/juice-shop` |
| **Ваш додаток** | `http://localhost:8080` | `docker-compose up -d` |

**OWASP Juice Shop** — спеціально створений вразливий додаток для навчання. Містить вразливості з OWASP Top 10: SQL Injection, XSS, Broken Authentication та інші. Має вбудований Score Board для відстеження прогресу.

#### Що потрібно зробити

Оберіть **один** з варіантів.

#### Варіант 1: ZAP сканування (рекомендовано)

Запустіть **OWASP ZAP** — безкоштовний сканер безпеки:

```bash
# Сканування Juice Shop (онлайн демо)
docker run --rm \
  -v $(pwd)/reports:/zap/wrk \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t https://demo.owasp-juice.shop \
  -r zap-report.html

# Або сканування Juice Shop (локальний Docker)
docker run --rm --network host \
  -v $(pwd)/reports:/zap/wrk \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://localhost:3000 \
  -r zap-report.html

# Або сканування вашого додатку
docker run --rm --network host \
  -v $(pwd)/reports:/zap/wrk \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://localhost:8080 \
  -r zap-report.html
```

#### Варіант 2: Ручні перевірки на Juice Shop

Виконайте мінімум **5 перевірок** з наведених нижче.

<details>
<summary><strong>SQL Injection</strong></summary>

На сторінці логіну (`https://demo.owasp-juice.shop/#/login`) спробуйте:

**Email:** `' OR 1=1--`
**Password:** будь-який

Чи вдалось увійти? Запишіть результат.

Також спробуйте в пошуку (`Search...` зверху):
```
'; DROP TABLE Products;--
```

</details>

<details>
<summary><strong>XSS (Cross-Site Scripting)</strong></summary>

В пошуку Juice Shop введіть:
```html
<iframe src="javascript:alert('XSS')">
```

Або:
```html
<img src=x onerror=alert('XSS')>
```

Чи виконався скрипт? Чи екранує додаток HTML?

</details>

<details>
<summary><strong>Broken Access Control</strong></summary>

1. Зареєструйте нового користувача
2. Відкрийте DevTools → Network
3. Знайдіть запит до API (наприклад `/api/Users`)
4. Спробуйте змінити URL, щоб отримати дані іншого користувача:

```
https://demo.owasp-juice.shop/api/Users/1
https://demo.owasp-juice.shop/api/Users/2
```

Чи вдається побачити чужі дані?

</details>

<details>
<summary><strong>Sensitive Data Exposure</strong></summary>

Перевірте, чи немає чутливих даних у відкритому доступі:

1. Відкрийте `https://demo.owasp-juice.shop/ftp`
2. Чи доступні якісь файли?
3. Перевірте `https://demo.owasp-juice.shop/api-docs` — чи є документація API?

</details>

<details>
<summary><strong>Перевірка заголовків безпеки</strong></summary>

```bash
curl -I https://demo.owasp-juice.shop
```

Перевірте наявність:
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY` або `SAMEORIGIN`
- `Content-Security-Policy`
- `Strict-Transport-Security`

Які заголовки відсутні?

</details>

<details>
<summary><strong>Перевірка CORS</strong></summary>

```bash
curl -H "Origin: http://evil.com" \
  -H "Access-Control-Request-Method: POST" \
  -X OPTIONS https://demo.owasp-juice.shop/api/Products
```

Чи `Access-Control-Allow-Origin` дорівнює `*`? Якщо так — це вразливість.

</details>

<details>
<summary><strong>Brute Force на логін</strong></summary>

Спробуйте залогінитися 10+ разів з неправильним паролем.

- Чи є обмеження на кількість спроб?
- Чи блокується акаунт?
- Чи є CAPTCHA?

Відсутність rate limiting — це вразливість.

</details>

#### Звіт

Створіть файл `docs/security-report.md`:

```markdown
# Security Report

## Цільовий додаток
- Назва: OWASP Juice Shop / Schedule API
- URL: ...
- Метод: ZAP Baseline Scan / Ручні перевірки

## Знайдені проблеми

| # | Рівень | Опис | Де знайдено |
|---|--------|------|-------------|
| 1 | High | SQL Injection на формі логіну | /login |
| 2 | Medium | Відсутній заголовок X-Frame-Options | Всі сторінки |
| 3 | Low | Cookie без флагу HttpOnly | Session cookie |
| ... | ... | ... | ... |

## Деталі

### Проблема 1: SQL Injection
- **Що зроблено:** ввів `' OR 1=1--` в поле Email
- **Результат:** успішний вхід без пароля
- **Ризик:** High — повний доступ до системи
- **Рекомендація:** використовувати параметризовані запити (prepared statements)

## Висновок
- Скільки проблем знайдено?
- Які найкритичніші?
- Що потрібно виправити в першу чергу?
```

<details>
<summary><strong>Бонус: додайте ZAP в CI pipeline</strong></summary>

```yaml
  security-tests:
    name: Security Scan (ZAP)
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: 'https://demo.owasp-juice.shop'
      
      - name: Upload ZAP report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: zap-report
          path: report_html.html
```

</details>

---

### Що має бути в результаті

```
schedule-testing/
├── tests/
│   └── performance/
│       └── load-test.js            ← k6 тест
├── docs/
│   ├── performance-report.md       ← Результати k6
│   └── security-report.md          ← Результати ZAP / ручних перевірок
└── reports/
    └── zap-report.html             ← HTML-звіт ZAP (якщо використовували)
```

### Критерії оцінювання

| Критерій | Бали |
|----------|------|
| k6 тест (2+ ендпоінти, thresholds, звіт) | 5 |
| Security scan (ZAP або ручні перевірки, звіт з 5+ знахідками) | 5 |
| **Разом (бонус)** | **10** |