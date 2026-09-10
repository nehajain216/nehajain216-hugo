---
title: "CSV-Driven Testing with Playwright, JUnit, and CSV Files"
date: 2026-09-09
description: "Learn how to build scalable, maintainable data-driven tests with Playwright, JUnit, and CSV files."
categories: ["QA"]
tags: ["QA", "Testing", "Playwright", "Java", "Maven", "JUnit", "Automation"]
image: "/images/playwright-csv.png"
readTime: "8 min read"
---

## Introduction

Automated tests often repeat the same workflow with different input values and expected results. We could create a separate test method for every scenario, but that approach duplicates browser interactions and makes the test suite harder to maintain as coverage grows.

Data-driven testing separates the test workflow from its data. The Java test contains the Playwright actions and assertions, while an external CSV file contains the input values and expected results. Adding another scenario then usually requires adding a row to the CSV file rather than copying an entire test method.

In this article, we will use JUnit 5 to read login scenarios from a CSV file and Playwright for Java to exercise the Bookstore login page. A single parameterized test will cover valid credentials, invalid credentials and blank fields.

## Prerequisites

We will use the Playwright test setup created in [Getting Started With Playwright Using Java, Maven and JUnit](../2026-08-05-getting-started-with-playwright-java-maven-junit/). Before continuing, make sure that:

* the [Bookstore GitHub repository](https://github.com/nehajain216/bookstore) is cloned locally;
* the Bookstore application is running at `http://localhost:8080`;
* Java, Maven, Playwright for Java and JUnit 5 are installed;
* the Playwright browsers are installed;
* the existing `BasePlaywrightTest` setup is available; and
* a disposable test user exists for the successful login scenario.

The examples assume that `BasePlaywrightTest` exposes `page` and `BASE_URL`, and that the test class has a helper named `submitLogin()` that opens the login page, fills the two fields and submits the form.

## Why store test data in CSV?

A CSV file is a good fit when multiple test scenarios share the same structure. Each row represents one scenario, and each column supplies a value to the test method. This provides several advantages:

* test data is separate from browser automation code;
* new scenarios can be added without duplicating Java test methods;
* reviewers can compare inputs and expected results in a compact table; and
* the same Playwright workflow is exercised consistently for every row.

CSV works best for small, flat datasets made up of strings, numbers and simple flags. It is less suitable for nested objects, dynamically generated values or scenarios that require substantially different setup.

## Add JUnit parameterized-test support

JUnit 5 provides parameterized tests through the `junit-jupiter-params` module. If the project does not already receive it through the aggregate `junit-jupiter` dependency, add it to `pom.xml`:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>${junit.jupiter.version}</version>
    <scope>test</scope>
</dependency>
```

`@ParameterizedTest` tells JUnit to invoke a test method once for every supplied dataset. `@CsvFileSource` loads those datasets from a CSV file on the test classpath.

## Create the login CSV file

Create `login-data.csv` inside `src/test/resources`:

```text
username,password,expectedOutcome,expectedText
siva@gmail.com,siva,VALID,Siva
siva1@gmail.com,siva1,INVALID,Invalid email or password.
"","",BLANK,""
```

The first line names the four columns. The remaining lines describe three scenarios: valid credentials, invalid credentials and blank credentials. The `expectedOutcome` column selects the assertions for each scenario. The quoted values in the final row explicitly represent empty strings.

Do not use personal or production credentials in test resources. Use accounts created only for local or test environments.

## Build the CSV-driven Playwright test

Add the following parameterized test to the Bookstore login test class:

```java
@ParameterizedTest(name = "Login scenario: username={0}, outcome={2}")
@CsvFileSource(resources = "/login-data.csv", numLinesToSkip = 1)
void shouldValidateLoginUsingCsv(String username, String password,
                                 String expectedOutcome, String expectedText) {
    submitLogin(username == null ? "" : username,
            password == null ? "" : password);

    switch (expectedOutcome) {
        case "VALID" -> {
            assertThat(page).hasTitle("BookStore");
            assertThat(page.getByText(expectedText)).isVisible();
        }
        case "INVALID" -> assertThat(page.getByRole(AriaRole.ALERT))
                .hasText(expectedText);
        case "BLANK" -> {
            assertThat(page.locator("#username:invalid")).isVisible();
            assertThat(page.locator("#password:invalid")).isVisible();
            assertThat(page).hasURL(BASE_URL + "/login");
        }
        default -> throw new IllegalArgumentException(
                "Unknown expected login outcome: " + expectedOutcome);
    }
}
```

The CSV columns map to the method parameters by position: `username`, `password`, `expectedOutcome` and `expectedText`. Because `numLinesToSkip` is `1`, JUnit ignores the header and creates one test invocation for each data row.

The custom test name uses `{0}` and `{2}` to include the username and expected outcome in the report. The username and password are converted from `null` to empty strings before they reach `submitLogin()`. This also protects the test if an unquoted empty CSV field is introduced later, because JUnit represents an unquoted empty value as `null` by default.

The `default` branch prevents an unknown or misspelled outcome from passing silently. For example, `INVALD` instead of `INVALID` will fail with a message identifying the unsupported value.

## Understand the login outcomes

Each outcome verifies different application behavior while reusing the same login workflow.

### Valid credentials

For `VALID`, the test confirms that the page title is `BookStore` and the expected user text from the CSV file is visible:

```java
assertThat(page).hasTitle("BookStore");
assertThat(page.getByText(expectedText)).isVisible();
```

These web-first assertions wait for the expected browser state instead of relying on a fixed delay.

### Invalid credentials

For `INVALID`, the test locates the application's alert by its accessible role and compares its text with the value in the CSV file:

```java
assertThat(page.getByRole(AriaRole.ALERT))
        .hasText(expectedText);
```

Using an accessible role expresses how the message is presented to the user and avoids depending on a styling class.

### Blank credentials

For `BLANK`, the browser should reject the empty required fields before the login request proceeds:

```java
assertThat(page.locator("#username:invalid")).isVisible();
assertThat(page.locator("#password:invalid")).isVisible();
assertThat(page).hasURL(BASE_URL + "/login");
```

The `:invalid` CSS pseudo-class matches a form control whose value does not satisfy its HTML validation constraints. The URL assertion confirms that the unsuccessful submission has not navigated away from the login page. This differs from the `INVALID` scenario, in which non-blank credentials are submitted and the application returns an error message.

## Handle empty and special CSV values

CSV parsing rules matter when the file becomes executable test data:

* An unquoted empty field is treated as `null` by `@CsvFileSource` unless configured otherwise.
* A quoted empty field, `""`, represents an empty string.
* A value containing a comma must be quoted so that it remains one column.
* Column order must continue to match the Java method parameters.
* Leading or trailing whitespace should be intentional because it can change form input and assertions.

For example, an expected message containing a comma should be written as one quoted value:

```text
user@example.com,wrong-password,INVALID,"Login failed, please try again."
```

The null-to-empty conversion is useful because Playwright's `fill()` expects a string. If a suite needs different parsing rules, `@CsvFileSource` also provides options such as `nullValues`, `emptyValue`, `delimiter` and `ignoreLeadingAndTrailingWhitespace`.

## Improve test readability and maintainability

Keep the CSV file focused on data and keep browser interactions in Java. A helper such as `submitLogin()` prevents navigation, field locators and button clicks from being repeated in every scenario. For a larger suite, those interactions can move into a login page object.

Use one clearly defined scenario per CSV row and descriptive parameterized-test names. Keep outcome names consistent and fail explicitly when the test encounters an unsupported value. If the set of outcomes grows, a Java enum can replace raw strings and provide stricter validation.

Test resources should remain small enough to review. Split unrelated behaviors into separate files and tests rather than putting every variation into one large dataset.

## Run the tests and interpret the results

Run the parameterized method with Maven:

```shell
./mvnw "-Dtest=BookstoreLoginTest#shouldValidateLoginUsingCsv" test
```

JUnit runs the method once for every CSV data row. The report will contain invocations similar to:

```text
Login scenario: username=siva@gmail.com, outcome=VALID
Login scenario: username=siva1@gmail.com, outcome=INVALID
Login scenario: username=, outcome=BLANK
```

Each invocation passes or fails independently. Adding another row to `login-data.csv` automatically creates another invocation without requiring another Java test method.

## Limitations and alternatives

CSV values do not have strong types, and CSV becomes difficult to use when a scenario contains nested data, collections or extensive setup. A switch with many outcome branches can eventually make one parameterized test harder to understand than several focused tests.

Use JUnit's `@MethodSource` when scenarios need calculated values, typed records or custom objects. Test-data builders are useful when defaults and optional fields must be composed, while JSON can better represent nested structures. The best source is the simplest one that keeps both the data and test behavior clear.

## Conclusion

CSV-driven testing separates login scenarios from the Playwright workflow that exercises them. JUnit reads each row and creates an independent test invocation, while Playwright submits the credentials and verifies the expected browser state.

With one parameterized method, we covered successful login, an application-level authentication error and browser-side validation for blank fields. Additional cases can now be introduced as new CSV rows without duplicating the login actions. Start with a small, focused dataset and move to typed sources when the scenarios become too complex for a flat file.
