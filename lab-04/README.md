# Лабораторна робота №4
## API-тестування

**Максимальна оцінка:** 20 балів

### Мета роботи

Навчитися тестувати REST API вручну за допомогою Postman та автоматизувати API-тести за допомогою коду.

### Предметна область

Ви продовжуєте працювати з системою управління розкладом.

API-документація доступна за адресою: http://fmi-schedule.chnu.edu.ua/swagger-ui/index.html

### Вибір треку

| Трек | Postman | Автоматизація |
|------|---------|---------------|
| **JavaScript** | Postman + Newman | Jest + axios |
| **Java** | Postman + Newman | REST Assured + Gradle |

Postman-частина однакова для обох треків.

---

## Варіанти завдань

Кожен варіант має **основний ресурс** (повний CRUD) та **додатковий ресурс** (складніший, із залежностями від інших сутностей):

| Варіант | Основний ресурс | Додатковий ресурс |
|---------|----------------|-------------------|
| 1 | Class | Lesson |
| 2 | Department | Schedule |
| 3 | Group | Semester |
| 4 | RoomType | Lesson |
| 5 | Room | Schedule |
| 6 | Student | Semester |
| 7 | Subject | Lesson |
| 8 | Room | Schedule |

- **Основний ресурс** — простий CRUD без складних залежностей.
- **Додатковий ресурс** (Lesson, Schedule, Semester) — залежить від інших сутностей, тому потребує попереднього створення пов'язаних даних або використання вже існуючих.

---

## Частина 1: Postman (10 балів)

### Підготовка

1. Завантажте Postman: https://www.postman.com/downloads/
2. Створіть новий Workspace для проєкту
3. Створіть Environment `schedule-local`:
   - `baseUrl`: `http://localhost:8080`
   - Додаткові змінні для зберігання ID створених ресурсів (заповняться автоматично через скрипти)

### Завдання 1.1: Створення колекції (4 бали)

Створіть колекцію `Schedule API Tests` з такою структурою:

```
Schedule API Tests/
├── {Основний ресурс}/
│   ├── GET All
│   ├── GET by ID
│   ├── POST Create
│   ├── PUT Update
│   ├── DELETE
│   └── GET Non-existing (404)
├── {Додатковий ресурс}/
│   ├── GET All
│   ├── GET by ID
│   ├── POST Create
│   └── ...
└── ...
```

**Вимоги:**
- Мінімум 3 папки (entities)
- Мінімум 10 запитів загалом
- Використовуйте `{{baseUrl}}` замість хардкоду URL

<details>
<summary><strong>📋 Приклад: запити для Teachers</strong></summary>

**GET All Teachers:**
- Method: `GET`
- URL: `{{baseUrl}}/api/teachers`

**GET Teacher by ID:**
- Method: `GET`
- URL: `{{baseUrl}}/api/teachers/{{teacherId}}`

**POST Create Teacher:**
- Method: `POST`
- URL: `{{baseUrl}}/api/teachers`
- Body (JSON):
```json
{
    "name": "{{teacherName}}",
    "surname": "{{teacherSurname}}",
    "patronymic": "{{teacherPatronymic}}",
    "position": "{{teacherPosition}}"
}
```

**PUT Update Teacher:**
- Method: `PUT`
- URL: `{{baseUrl}}/api/teachers/{{teacherId}}`
- Body — аналогічний POST, але зі зміненими даними

**DELETE Teacher:**
- Method: `DELETE`
- URL: `{{baseUrl}}/api/teachers/{{teacherId}}`

</details>

### Завдання 1.2: Написання тестів у Postman (4 бали)

Додайте тести до кожного запиту у вкладці "Scripts → Post-response".

**Вимоги:**
- Кожен запит має мінімум 2 тести
- Використовуйте environment variables для передачі даних між запитами
- Включіть негативні тести (404, 400)

<details>
<summary><strong>📋 Приклад: тести для GET All</strong></summary>

```javascript
pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
});

pm.test("Response is an array", () => {
    const json = pm.response.json();
    pm.expect(json).to.be.an('array');
});

pm.test("Response time is less than 500ms", () => {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

pm.test("Each item has required fields", () => {
    const json = pm.response.json();
    if (json.length > 0) {
        pm.expect(json[0]).to.have.property('id');
        pm.expect(json[0]).to.have.property('name');
    }
});
```

</details>

<details>
<summary><strong>📋 Приклад: тести для POST Create</strong></summary>

```javascript
pm.test("Status code is 201", () => {
    pm.response.to.have.status(201);
});

pm.test("Response contains created resource", () => {
    const json = pm.response.json();
    pm.expect(json).to.have.property('id');
    pm.expect(json.name).to.eql(pm.environment.get("resourceName"));
});

// Зберігаємо ID для наступних запитів
if (pm.response.code === 201) {
    pm.environment.set("resourceId", pm.response.json().id);
}
```

</details>

<details>
<summary><strong>📋 Приклад: тести для GET by ID, DELETE, негативних сценаріїв</strong></summary>

**GET by ID:**
```javascript
pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
});

pm.test("Returns correct resource", () => {
    const json = pm.response.json();
    const expectedId = parseInt(pm.environment.get("resourceId"));
    pm.expect(json.id).to.eql(expectedId);
});
```

**DELETE:**
```javascript
pm.test("Status code is 200 or 204", () => {
    pm.expect(pm.response.code).to.be.oneOf([200, 204]);
});
```

**Негативний тест — GET неіснуючого ресурсу:**
```javascript
pm.test("Status code is 404", () => {
    pm.response.to.have.status(404);
});
```

</details>

### Завдання 1.3: Pre-request Scripts та тестові дані (2 бали)

Використайте Pre-request Scripts для генерації тестових даних.

**Вимоги:**
- Налаштуйте Pre-request Script для POST-запитів
- Дані мають бути унікальними (використовуйте timestamp або random)

<details>
<summary><strong>📋 Приклад: Pre-request Script</strong></summary>

**Pre-request Script (вкладка "Scripts → Pre-request"):**
```javascript
const timestamp = Date.now();
pm.environment.set("teacherName", "Test");
pm.environment.set("teacherSurname", "Teacher_" + timestamp);
pm.environment.set("teacherPatronymic", "Testovich");
pm.environment.set("teacherPosition", "доцент");
```

**Body запиту:**
```json
{
    "name": "{{teacherName}}",
    "surname": "{{teacherSurname}}",
    "patronymic": "{{teacherPatronymic}}",
    "position": "{{teacherPosition}}"
}
```

</details>

### Експорт колекції

1. Експортуйте колекцію: Collection → Export → Collection v2.1
2. Експортуйте environment: Environment → Export
3. Збережіть файли в репозиторій:
   - `tests/api/postman/Schedule_API_Tests.postman_collection.json`
   - `tests/api/postman/schedule-local.postman_environment.json`

---

## Частина 2: Автоматизація (10 балів)

Автоматизуйте тести з Частини 1 за допомогою коду. Тести мають покривати ті ж ресурси, що й Postman-колекція.

### Завдання 2.1: CRUD-тести для основного ресурсу (5 балів)

Напишіть автоматизовані тести для повного CRUD-циклу **основного ресурсу** вашого варіанту:

- GET all — отримання списку
- GET by ID — отримання за ідентифікатором
- POST create — створення нового ресурсу
- PUT update — оновлення ресурсу
- DELETE — видалення ресурсу
- Негативні сценарії: 400 (невалідні дані), 404 (неіснуючий ресурс)

**Вимоги:**
- Мінімум 8 тестів
- Включіть позитивні та негативні сценарії
- Тести мають бути незалежними (або виконуватися в правильному порядку)
- Перевірте, що після DELETE ресурс дійсно повертає 404

### Завдання 2.2: Тести для додаткового ресурсу (3 бали)

Напишіть тести для **додаткового ресурсу** вашого варіанту. Оскільки додаткові ресурси (Lesson, Schedule, Semester) залежать від інших сутностей, тести мають враховувати створення пов'язаних даних.

**Вимоги:**
- Мінімум 5 тестів
- Покрийте основні endpoints (GET all, GET by ID, POST create)
- Перевірте валідацію зв'язків (наприклад, створення Lesson без вказаного Teacher)

### Завдання 2.3: Перевірка даних у БД (2 бали)

Додайте перевірку даних у базі даних після API-операцій.

<details>
<summary><strong>📋 Приклад: JavaScript (pg)</strong></summary>

```javascript
const { Pool } = require('pg');

const pool = new Pool({
    connectionString: process.env.DATABASE_URL
});

it('should save resource to database', async () => {
    const response = await api.post('/api/teachers', newTeacher);
    const teacherId = response.data.id;

    const result = await pool.query(
        'SELECT * FROM teachers WHERE id = $1',
        [teacherId]
    );

    expect(result.rows.length).toBe(1);
    expect(result.rows[0].name).toBe(newTeacher.name);
});
```

</details>

<details>
<summary><strong>📋 Приклад: Java (JdbcTemplate)</strong></summary>

```java
@Autowired
private JdbcTemplate jdbcTemplate;

@Test
void shouldSaveTeacherToDatabase() {
    Integer id = given()
        .contentType(ContentType.JSON)
        .body(teacherJson)
        .when()
            .post("/api/teachers")
        .then()
            .statusCode(201)
        .extract()
            .path("id");

    String name = jdbcTemplate.queryForObject(
        "SELECT name FROM teachers WHERE id = ?",
        String.class,
        id
    );

    assertEquals("Test", name);
}
```

</details>

---

### Підготовка та повний приклад за треками

<details>
<summary><strong>📋 Трек JavaScript: Jest + axios — підготовка та приклад тестів</strong></summary>

#### Підготовка

```bash
npm install --save-dev axios jest
```

#### Приклад файлу `tests/api/teachers.api.test.js`

```javascript
const axios = require('axios');

const API_URL = process.env.API_URL || 'http://localhost:8080';
const api = axios.create({
    baseURL: API_URL,
    timeout: 5000,
});

describe('Teachers API', () => {
    let createdTeacherId;

    describe('GET /api/teachers', () => {
        it('should return list of teachers', async () => {
            const response = await api.get('/api/teachers');

            expect(response.status).toBe(200);
            expect(Array.isArray(response.data)).toBe(true);
        });

        it('should return teachers with required fields', async () => {
            const response = await api.get('/api/teachers');

            if (response.data.length > 0) {
                expect(response.data[0]).toHaveProperty('id');
                expect(response.data[0]).toHaveProperty('name');
                expect(response.data[0]).toHaveProperty('surname');
            }
        });
    });

    describe('POST /api/teachers', () => {
        it('should create a new teacher', async () => {
            const newTeacher = {
                name: 'Test',
                surname: `Teacher_${Date.now()}`,
                patronymic: 'Testovich',
                position: 'доцент',
            };

            const response = await api.post('/api/teachers', newTeacher);

            expect(response.status).toBe(201);
            expect(response.data).toHaveProperty('id');
            expect(response.data.name).toBe(newTeacher.name);

            createdTeacherId = response.data.id;
        });

        it('should return 400 for invalid data', async () => {
            const invalidTeacher = { name: '' };

            await expect(api.post('/api/teachers', invalidTeacher))
                .rejects.toMatchObject({
                    response: { status: 400 }
                });
        });
    });

    describe('GET /api/teachers/:id', () => {
        it('should return teacher by id', async () => {
            const response = await api.get(`/api/teachers/${createdTeacherId}`);

            expect(response.status).toBe(200);
            expect(response.data.id).toBe(createdTeacherId);
        });

        it('should return 404 for non-existing teacher', async () => {
            await expect(api.get('/api/teachers/999999'))
                .rejects.toMatchObject({
                    response: { status: 404 }
                });
        });
    });

    describe('PUT /api/teachers/:id', () => {
        it('should update teacher', async () => {
            const updated = {
                name: 'Updated',
                surname: 'Teacher',
                patronymic: 'Testovich',
                position: 'професор',
            };

            const response = await api.put(
                `/api/teachers/${createdTeacherId}`, updated
            );

            expect(response.status).toBe(200);
            expect(response.data.position).toBe('професор');
        });
    });

    describe('DELETE /api/teachers/:id', () => {
        it('should delete teacher', async () => {
            const response = await api.delete(
                `/api/teachers/${createdTeacherId}`
            );
            expect([200, 204]).toContain(response.status);
        });

        it('should return 404 after deletion', async () => {
            await expect(api.get(`/api/teachers/${createdTeacherId}`))
                .rejects.toMatchObject({
                    response: { status: 404 }
                });
        });
    });
});
```

#### Запуск

```bash
npm test -- tests/api/
```

</details>

<details>
<summary><strong>📋 Трек Java: REST Assured + Gradle — підготовка та приклад тестів</strong></summary>

#### Підготовка

Додайте в `build.gradle`:

```gradle
dependencies {
    testImplementation 'io.rest-assured:rest-assured:5.3.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
}

test {
    useJUnitPlatform()
}
```

#### Приклад файлу `src/test/java/.../TeachersApiTest.java`

```java
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.*;
import java.util.List;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@DisplayName("Teachers API Tests")
class TeachersApiTest {

    private static Integer createdTeacherId;

    @BeforeAll
    static void setup() {
        RestAssured.baseURI = "http://localhost:8080";
    }

    @Nested
    @DisplayName("GET /api/teachers")
    class GetAllTeachers {

        @Test
        @DisplayName("Should return list of teachers")
        void shouldReturnListOfTeachers() {
            given()
                .when()
                    .get("/api/teachers")
                .then()
                    .statusCode(200)
                    .body("$", instanceOf(List.class));
        }

        @Test
        @DisplayName("Should return teachers with required fields")
        void shouldReturnTeachersWithRequiredFields() {
            given()
                .when()
                    .get("/api/teachers")
                .then()
                    .statusCode(200)
                    .body("[0].id", notNullValue())
                    .body("[0].name", notNullValue())
                    .body("[0].surname", notNullValue());
        }
    }

    @Nested
    @DisplayName("POST /api/teachers")
    @TestMethodOrder(MethodOrderer.OrderAnnotation.class)
    class CreateTeacher {

        @Test
        @Order(1)
        @DisplayName("Should create a new teacher")
        void shouldCreateNewTeacher() {
            String requestBody = """
                {
                    "name": "Test",
                    "surname": "Teacher_%d",
                    "patronymic": "Testovich",
                    "position": "доцент"
                }
                """.formatted(System.currentTimeMillis());

            createdTeacherId = given()
                .contentType(ContentType.JSON)
                .body(requestBody)
                .when()
                    .post("/api/teachers")
                .then()
                    .statusCode(201)
                    .body("id", notNullValue())
                    .body("name", equalTo("Test"))
                .extract()
                    .path("id");
        }

        @Test
        @DisplayName("Should return 400 for invalid data")
        void shouldReturn400ForInvalidData() {
            given()
                .contentType(ContentType.JSON)
                .body("""
                    { "name": "" }
                """)
                .when()
                    .post("/api/teachers")
                .then()
                    .statusCode(400);
        }
    }

    @Nested
    @DisplayName("GET /api/teachers/{id}")
    class GetTeacherById {

        @Test
        @DisplayName("Should return teacher by id")
        void shouldReturnTeacherById() {
            given()
                .when()
                    .get("/api/teachers/{id}", createdTeacherId)
                .then()
                    .statusCode(200)
                    .body("id", equalTo(createdTeacherId));
        }

        @Test
        @DisplayName("Should return 404 for non-existing teacher")
        void shouldReturn404ForNonExisting() {
            given()
                .when()
                    .get("/api/teachers/{id}", 999999)
                .then()
                    .statusCode(404);
        }
    }

    @Nested
    @DisplayName("PUT /api/teachers/{id}")
    class UpdateTeacher {

        @Test
        @DisplayName("Should update teacher")
        void shouldUpdateTeacher() {
            given()
                .contentType(ContentType.JSON)
                .body("""
                    {
                        "name": "Updated",
                        "surname": "Teacher",
                        "patronymic": "Testovich",
                        "position": "професор"
                    }
                """)
                .when()
                    .put("/api/teachers/{id}", createdTeacherId)
                .then()
                    .statusCode(200)
                    .body("position", equalTo("професор"));
        }
    }

    @Nested
    @DisplayName("DELETE /api/teachers/{id}")
    @TestMethodOrder(MethodOrderer.OrderAnnotation.class)
    class DeleteTeacher {

        @Test
        @Order(1)
        @DisplayName("Should delete teacher")
        void shouldDeleteTeacher() {
            given()
                .when()
                    .delete("/api/teachers/{id}", createdTeacherId)
                .then()
                    .statusCode(anyOf(equalTo(200), equalTo(204)));
        }

        @Test
        @Order(2)
        @DisplayName("Should return 404 after deletion")
        void shouldReturn404AfterDeletion() {
            given()
                .when()
                    .get("/api/teachers/{id}", createdTeacherId)
                .then()
                    .statusCode(404);
        }
    }
}
```

#### Запуск

```bash
./gradlew test --tests "*ApiTest"
```

</details>

---

## Що має бути в результаті

```
schedule-testing/
├── docs/
│   └── ...
├── tests/
│   ├── unit/
│   │   └── ...
│   └── api/
│       ├── postman/
│       │   ├── Schedule_API_Tests.postman_collection.json
│       │   └── schedule-local.postman_environment.json
│       ├── {resource1}.api.test.js     ← (JS трек)
│       └── {resource2}.api.test.js     ← (JS трек)
└── src/test/java/
    └── .../api/
        ├── {Resource1}ApiTest.java     ← (Java трек)
        └── {Resource2}ApiTest.java     ← (Java трек)
```

---

## Критерії оцінювання

| Критерій | Бали |
|----------|------|
| **Частина 1: Postman** | |
| Колекція з правильною структурою (10+ запитів) | 4 |
| Тести у Postman (assertions) | 4 |
| Pre-request Scripts та environment variables | 2 |
| **Частина 2: Автоматизація** | |
| CRUD-тести для основного ресурсу (8+ тестів) | 5 |
| Тести для додаткового ресурсу (5+ тестів) | 3 |
| Перевірка даних у БД | 2 |
| **Разом** | **20** |

---

## Контрольні питання

1. Яка різниця між PUT та PATCH?
2. Коли використовувати статус-код 201 vs 200?
3. Як передавати дані між запитами в Postman?
4. Що таке idempotent методи?
5. Як тестувати API, що вимагає авторизації?
6. Чому важливо перевіряти дані в БД, а не тільки відповідь API?
