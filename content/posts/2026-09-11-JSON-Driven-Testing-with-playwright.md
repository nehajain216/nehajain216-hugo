---
title: "JSON-Driven Testing with Playwright, JUnit, and JSON Files"
date: 2026-09-11
description: "Learn how to build scalable, maintainable data-driven tests with Playwright, JUnit, and JSON files."
categories: ["QA"]
tags: ["QA", "Testing", "Playwright", "Java", "Maven", "JUnit", "Automation"]
image: "/images/playwright-json.png"
readTime: "5 min read"
---

## Introduction

Data-driven testing lets us run the same browser workflow with different inputs and expected results. Instead of creating a separate test method for every login scenario, we can keep the scenarios in a JSON file and let JUnit run the test once for each entry.

JSON is especially useful when test data needs named fields, strong mapping to Java objects or room to grow beyond a flat table. In this example, Playwright exercises the Bookstore login page, JUnit provides the parameterized test and Jackson converts the JSON data into Java records.

For the prerequisite project setup and background on data-driven Playwright tests, read the [CSV-driven Playwright testing article](../2026-09-09-CSV-Driven-Testing-with-playwright/). The examples below assume that `BaseTest` provides `page` and `BASE_URL`, and that `submitLogin()` opens the login page, fills the credentials and submits the form.

## Add the required dependencies

The test uses JUnit's parameterized-test support and Jackson for reading JSON. If they are not already present in the project, add `junit-jupiter-params` and `jackson-databind` as test dependencies in `pom.xml`:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>${junit.jupiter.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
    <scope>test</scope>
</dependency>
```

## Create the JSON test data

Create `login-data.json` inside `src/test/resources`:

```json
[
  {
    "username": "siva@gmail.com",
    "password": "siva",
    "expectedOutcome": "VALID",
    "expectedText": "Siva"
  },
  {
    "username": "siva1@gmail.com",
    "password": "siva1",
    "expectedOutcome": "INVALID",
    "expectedText": "Invalid email or password."
  },
  {
    "username": "",
    "password": "",
    "expectedOutcome": "BLANK",
    "expectedText": ""
  }
]
```

Each object describes one login scenario. The property names match the components of the Java `LoginTestData` record, allowing Jackson to map the values without custom parsing code.

Use accounts created only for local or test environments. Test-resource files should never contain personal or production credentials.

## Build the JSON-driven test

The parameterized test receives one `LoginTestData` record for every object in the JSON array:

```java
class JsonLoginPageTest extends BaseTest {

    @ParameterizedTest(name = "JSON login scenario {index}")
    @MethodSource("loginScenariosFromJson")
    void shouldValidateLoginUsingJson(LoginTestData testData) {
        submitLogin(testData.username(), testData.password());

        switch (testData.expectedOutcome()) {
            case "VALID" -> {
                assertThat(page).hasTitle("BookStore");
                assertThat(page.getByText(testData.expectedText())).isVisible();
            }
            case "INVALID" -> assertThat(page.getByRole(AriaRole.ALERT))
                    .hasText(testData.expectedText());
            case "BLANK" -> {
                assertThat(page.locator("#username:invalid")).isVisible();
                assertThat(page.locator("#password:invalid")).isVisible();
                assertThat(page).hasURL(BASE_URL + "/login");
            }
            default -> throw new IllegalArgumentException(
                    "Unknown expected login outcome: " + testData.expectedOutcome());
        }
    }

    static Stream<LoginTestData> loginScenariosFromJson() throws IOException {
        InputStream inputStream = JsonLoginPageTest.class
                .getResourceAsStream("/login-data.json");
        if (inputStream == null) {
            throw new IllegalStateException(
                    "Test resource not found: /login-data.json");
        }

        try (inputStream) {
            List<LoginTestData> scenarios = new ObjectMapper().readValue(
                    inputStream, new TypeReference<>() {
                    });
            return scenarios.stream();
        }
    }

    record LoginTestData(String username, String password,
                         String expectedOutcome, String expectedText) {
    }
}
```

* `@ParameterizedTest` tells JUnit to run the method multiple times. `@MethodSource` points to `loginScenariosFromJson()`, which loads the classpath resource and asks Jackson to deserialize the JSON array into a `List<LoginTestData>`. Returning the list as a stream supplies each record as a separate test invocation.
* The `TypeReference` preserves the list's element type during deserialization. Without it, Jackson would not know that every JSON object should become a `LoginTestData` record. The try-with-resources block also ensures that the input stream is closed after reading.
* If the resource is missing, the method fails with a clear message instead of passing a `null` stream to Jackson.

## Understand the expected outcomes

All scenarios reuse the same login action, but each outcome checks a different browser state:

* `VALID` verifies the page title and confirms that the expected user text is visible.
* `INVALID` checks the application's accessible alert text after incorrect credentials are submitted.
* `BLANK` checks the browser's HTML validation state and confirms that the page remains on the login URL.

The `default` branch is important because test data is external to the Java compiler. A typo such as `INVALD` fails explicitly rather than silently skipping the assertions.

As the suite grows, consider replacing outcome strings with an enum or validating the data while it is loaded. Keep the JSON file focused: unrelated workflows are easier to understand when they use separate test methods and resource files.

## JSON or CSV?

JSON provides descriptive property names and maps naturally to typed Java records. It also supports nested objects and arrays if a future scenario needs addresses, permissions or several expected messages.

For small, flat datasets, CSV may be more compact and easier for non-developers to edit. The [CSV-driven Playwright testing article](../2026-09-09-CSV-Driven-Testing-with-playwright/) demonstrates that approach. Choose the simplest format that keeps the test data readable.

| Testing scenario | Prefer | Why |
| --- | --- | --- |
| **Login with username, password and expected result** | CSV | Each scenario has the same simple columns. |
| **Search with keywords and expected result count** | CSV | The data is flat and easy to represent as rows. |
| **Form validation with field value and error message** | CSV | Simple input-and-output combinations fit naturally into a table. |
| **Checkout with customer, address and multiple products** | JSON | It contains nested objects and a variable-length product list. |
| **API request and response validation** | JSON | Request bodies and expected responses are usually structured JSON. |
| **User profiles with roles and permissions** | JSON | Each user may have multiple roles or permissions. |
| **Travel booking with passengers and journey segments** | JSON | Bookings can contain lists and nested details. |
| **Application configuration with optional settings** | JSON | Scenarios may have different fields and nested configuration values. |

> Use CSV when every scenario fits naturally into one row. Use JSON when a scenario contains nested objects, lists, optional fields or complex relationships.

## Conclusion

JSON-driven testing keeps test data separate from Playwright's browser actions. It is a practical choice when test scenarios contain structured data or need to grow beyond simple rows and columns.
