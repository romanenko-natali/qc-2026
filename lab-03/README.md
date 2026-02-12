# Лабораторна робота №3

## Unit-тестування

**Максимальна оцінка:** 20 балів

### Мета роботи

Навчитися писати unit-тести для перевірки окремих функцій та модулів програми.

### Предметна область

Ви продовжуєте працювати з системою управління розкладом.

Проєкт вже містить unit-тести:
- **Frontend (Jest):** `frontend/src/helper/*.test.js`
- **Backend (JUnit 5):** `src/test/java/com/softserve/service/*Test.java`, `src/test/java/com/softserve/util/*Test.java`

Ваше завдання — розширити існуюче тестове покриття.

### Вибір треку

Оберіть один трек:

| Трек | Технології | Що тестуємо |
|------|------------|-------------|
| **JavaScript** | Jest | Frontend-хелпери (`frontend/src/helper/`) |
| **Java** | JUnit 5, Spring Boot Test | Backend-сервіси (`src/test/java/com/softserve/`) |

---

### Підготовка

#### Трек JavaScript

Запуск тестів:
```bash
cd frontend
npm test
```

Запуск з покриттям:
```bash
npm test -- --coverage
```

#### Трек Java

Запуск тестів:
```bash
./gradlew test
```

Запуск з покриттям (JaCoCo):
```bash
./gradlew test jacocoTestReport
```

Звіт буде в `build/reports/jacoco/test/html/index.html`.

---

### Практична частина

#### Завдання 1: Написання та розширення тестів (10 балів)

Візьміть файли вашого варіанту, проаналізуйте що вже покрито тестами, та розширіть тестове покриття.

**Вимоги:**
- Мінімум 10 нових тестів
- Покрийте edge cases: пусті значення, null, граничні випадки
- Покрийте позитивні та негативні сценарії
- Групування тестів: `describe` (Jest) або `@Nested` (JUnit)
- Використовуйте AAA-патерн (Arrange → Act → Assert)

---

#### Завдання 2: Mutation Testing (5 балів)

Mutation testing — це техніка перевірки якості тестів. Інструмент автоматично створює "мутанти" — копії коду з маленькими змінами (наприклад, `>` → `<`, `true` → `false`, `===` → `!==`). Потім запускає тести на кожному мутанті. Якщо тести впали — мутант "вбитий" (тести якісні). Якщо тести пройшли — мутант "вижив" (тести не помітили зміну логіки).

##### Трек JavaScript (Stryker)

Встановіть Stryker:
```bash
cd frontend
npm install --save-dev @stryker-mutator/core @stryker-mutator/jest-runner
```

Створіть `stryker.config.mjs` у папці `frontend/`:
```javascript
/** @type {import('@stryker-mutator/api/core').PartialStrykerOptions} */
const config = {
  testRunner: "jest",
  jest: {
    configFile: "jest.config.js",
    config: {
      testMatch: ["**/src/helper/**/*.test.js"],
      coverageThreshold: undefined,
    }
  },
  mutate: [
    // Вкажіть файли вашого варіанту, наприклад:
    // "src/helper/disableComponent.js",
    // "src/helper/formHelper.js"
  ],
  mutator: {
    plugins: [],
    excludedMutations: []
  },
  reporters: ["html", "clear-text", "progress"],
  htmlReporter: {
    fileName: "reports/mutation/mutation.html"
  },
  coverageAnalysis: "perTest"
};
export default config;
```

Запуск:
```bash
npx stryker run
```

##### Трек Java (PIT)

Додайте в `build.gradle`:
```gradle
plugins {
    id 'info.solidsoft.pitest' version '1.15.0'
}

pitest {
    targetClasses = ['com.softserve.service.*', 'com.softserve.util.*']
    targetTests = ['com.softserve.service.*Test', 'com.softserve.util.*Test']
    junit5PluginVersion = '1.2.1'
    outputFormats = ['HTML']
    timestampedReports = false
}
```

Запуск:
```bash
./gradlew pitest
```

##### Що потрібно зробити

1. Налаштуйте mutation testing для файлів вашого варіанту
2. Запустіть та проаналізуйте звіт
3. Знайдіть вижилих мутантів — напишіть тести, щоб їх вбити
4. Якщо файл не покритий тестами — мутанти будуть мати статус "no coverage". Напишіть тести і перезапустіть
5. Перезапустіть mutation testing та порівняйте результати до/після

**Вимоги:**
- Скріншот mutation report до та після написання тестів
- Мінімум 3 вбитих мутанти, які раніше вижили
- Пояснення: які мутації вижили і чому тести їх не ловили

---

#### Завдання 3: Code Coverage (5 балів)

Запустіть тести з покриттям та проаналізуйте результати.

Створіть файл `docs/coverage-report.md`:

```markdown
# Coverage Report

## Загальне покриття
- Statements/Instructions: XX%
- Branches: XX%
- Functions/Methods: XX%
- Lines: XX%

## Аналіз
- Які функції/класи покриті найкраще?
- Які потребують додаткових тестів?
- Чому деякі branches не покриті?

## Скріншот
[Додайте скріншот coverage report]
```

**Вимоги:**
- Скріншот coverage report
- Аналіз результатів (не просто цифри, а пояснення)

---

### Варіанти завдань

---

#### Варіант 1

**Трек JS:**
- Завдання 1:
  - Розширити `disableComponent.test.js` — додати тести на непокриті гілки `setDisabledSaveButtonSemester` (семестр без id, порожні selectedGroups, видалення групи, порожній semester_groups)
  - Написати тести для функцій з `formHelper.js` — `setValueToTeacherForSiteHandler`, `setValueToSubjectForSiteHandler`. Для параметра `setValue` використайте `jest.fn()`. Покрийте: знайдений/не знайдений вчитель, id як рядок, порожній масив, неіснуючий id

**Трек Java:**
- Завдання 1: Розширити `GroupServiceTest` та `StudentServiceTest`

---

#### Варіант 2

**Трек JS:**
- Завдання 1:
  - Розширити `getScheduleType.test.js` — додати тести на `DEPARTMENT`, null/undefined значення, об'єкти без id, кілька заповнених полів одночасно (перевірка пріоритету)
  - Написати тести для функцій з `renderScheduleTable.js` — `checkSemesterEnd`, `isWeekOdd`, `getWeekParity`

**Трек Java:**
- Завдання 1: Розширити `TeacherServiceTest` та `SubjectServiceTest`

---

#### Варіант 3

**Трек JS:**
- Завдання 1:
  - Розширити `getHref.test.js` — додати тести на пустий link, null, link без протоколу, дуже довгий link, перевірка додаткових атрибутів елемента
  - Написати тести для функцій з `sheduleUtils.js` — `isNotReadySchedule`, `filterClassesArray`. Покрийте: порожній розклад з loading/без loading, непорожній розклад, null, масив з дублікатами, порожній масив, масив з унікальними елементами

**Трек Java:**
- Завдання 1: Розширити `DepartmentServiceTest` та `RoomServiceTest`

---

#### Варіант 4

**Трек JS:**
- Завдання 1:
  - Розширити `prepareLessonCell.test.js` — додати тести на відсутній room, різні place значення, порожні поля в card, різні формати lessonType
  - Написати тести для `sortRooms` з `sortRoom.js` та `sortGroups` з `sortGroup.js` — вставка на початок, вставка після елемента, неіснуючий afterId, пустий масив

**Трек Java:**
- Завдання 1: Розширити `LessonServiceTest` та `PeriodServiceTest`

---

#### Варіант 5

**Трек JS:**
- Завдання 1:
  - Розширити `disableComponent.test.js` — додати тести на непокриті гілки `setDisabledSaveButtonSemester` (null semester, undefined selectedGroups, порожні масиви, комбінації pristine/submitting з різними станами семестру)
  - Розширити тести для `handleFormSubmit` з `handleFormSubmit.js` та `cardObjectHandler` з `cardObjectHandler.js` — додати edge cases: id=0, id=null, id=undefined, строкові числа, пустий card

**Трек Java:**
- Завдання 1: Розширити `ScheduleServiceTest` та `SemesterServiceTest`

---

#### Варіант 6

**Трек JS:**
- Завдання 1:
  - Розширити `getScheduleType.test.js` — додати тести на `DEPARTMENT`, пустий об'єкт з id: 0, id: null, вкладені порожні об'єкти, усі поля заповнені
  - Розширити тести для `getColorByFullness` та `divideLessonsByOneHourLesson` з `schedule.js` — додати edge cases: виклик `getColorByFullness` без аргументів (default parameter), масив з одним елементом, три+ елементи зі зміною вчителя; `divideLessonsByOneHourLesson` з порожнім lessons, hours=0, всі items вже присутні

**Трек Java:**
- Завдання 1: Розширити `RoomTypeServiceTest` та `UserServiceTest`

---

#### Варіант 7

**Трек JS:**
- Завдання 1:
  - Розширити `getHref.test.js` — додати тести на пустий link, null, undefined, перевірка тексту посилання, спеціальні символи в URL
  - Розширити тести для `search` з `search.js` — додати edge cases: term з пробілами на початку/кінці, пошук по неіснуючому полі в arr, регістронезалежність, числові значення в полях, порожній масив items

**Трек Java:**
- Завдання 1: Розширити `GroupServiceTest` та `TranslatorTest` (`util/`)

---

#### Варіант 8

**Трек JS:**
- Завдання 1:
  - Розширити `prepareLessonCell.test.js` — додати тести на card з room без name, place як невідоме значення, lessonType у верхньому регістрі, card без teacher, card без subjectForSite
  - Розширити тести для `shortTitle.js`, `strings.js`, `sortArray.js` — додати edge cases: `getShortTitle` з title рівним MAX_LENGTH, порожній рядок, MAX_LENGTH=0; `firstStringLetterCapital` з порожнім рядком, рядком з великої літери, рядком що починається з цифри; `sortByName` з одним елементом, однаковими іменами, порожнім масивом

**Трек Java:**
- Завдання 1: Розширити `TeacherServiceTest` та `PasswordGeneratingUtilTest` (`util/`)

---

### Що має бути в результаті

**GitHub-репозиторій з:**
- Нові/розширені тести у відповідних директоріях проєкту
- Конфігурація mutation testing (`stryker.config.mjs` або зміни в `build.gradle`)
- `docs/coverage-report.md` з аналізом та скріншотами (code coverage + mutation testing)

---

### Критерії оцінювання

| Критерій | Бали |
|----------|------|
| Завдання 1: Написання та розширення тестів (10+ тестів, edge cases, групування) | 10 |
| Завдання 2: Mutation testing (налаштування, аналіз, вбивство мутантів) | 5 |
| Завдання 3: Coverage report та аналіз | 5 |
| **Разом** | **20** |

---

### Контрольні питання

1. Що таке AAA-патерн у тестуванні?
2. Яка різниця між `toBe` і `toEqual` в Jest?
3. Навіщо використовувати `@Nested` в JUnit?
4. Що таке code coverage і які його типи?
5. Чому 100% coverage не гарантує відсутність багів?
6. Як тестувати функції, що кидають виключення?
7. Що таке mutation testing і чим він відрізняється від code coverage?
8. Що означає "вижилий мутант" і як з ним боротися?