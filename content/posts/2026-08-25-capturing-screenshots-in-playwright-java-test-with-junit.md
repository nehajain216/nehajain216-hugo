---
title: "Capturing Screenshots in Playwright"
date: 2026-08-25
description: "Learn how to capture explicit screenshots and automatic failure screenshots in Playwright tests using Java and JUnit."
categories: ["QA"]
tags: ["QA", "Testing", "Playwright", "Java", "Maven", "JUnit", "Automation"]
image: "/images/playwright-screenshots.png"
readTime: "4 min read"
---

## Introduction

Screenshots are useful in UI testing because they provide a visual record of the application at a specific point during a test. When an assertion fails, a screenshot can reveal problems that are difficult to understand from an error message alone, such as an unexpected dialog, a missing element, an incorrect page state or a layout issue. Screenshots can also document important steps in a user journey and make test reports easier to investigate.

In Playwright, screenshots can be captured in two common ways:

* **Explicit screenshots** are taken at a chosen point in the test by calling Playwright's screenshot API. They are helpful when you want to record a particular page state, capture a specific element or add visual evidence to a test report.
* **Automatic failure screenshots** are captured only when a test fails. With JUnit, this behavior can be implemented in a test lifecycle callback or extension so that every failed test produces diagnostic evidence without adding screenshot code to each test method.

This article explains how to use both approaches in Playwright tests written with Java and JUnit, including how to name and store the generated image files.

## Prerequisites

Before starting, make sure that:

* The [Bookstore GitHub repository](https://github.com/nehajain216/bookstore) is cloned locally.
* The [Bookstore application](https://github.com/nehajain216/bookstore) is running at `http://localhost:8080`.
* A test user exists in the application.
* Playwright, JUnit and the Playwright browsers are installed.
* The `BasePlaywrightTest` from the [previous post](https://nehajain.netlify.app/posts/2026-08-05-getting-started-with-playwright-java-maven-junit/) is available in the test project.

## Playwright screenshot basics

Playwright's `page.screenshot()` method captures the current browser page. The following example saves a screenshot of the visible viewport:

```java
Path screenshotPath = Path.of("target", "screenshots", "bookstore-home.png");

page.screenshot(new Page.ScreenshotOptions()
        .setPath(screenshotPath));
```

The screenshot format is inferred from the file extension. In this example, Playwright creates a PNG image because the file name ends with `.png`. The method also returns the screenshot as a byte array when the image needs to be attached directly to a test report or processed in memory.

### Configure screenshots with `Page.ScreenshotOptions`

`Page.ScreenshotOptions` controls how and where the screenshot is captured. Some commonly used options are:

* `setPath()` specifies the output file path.
* `setFullPage(true)` captures the entire scrollable page instead of only the visible viewport.
* `setType()` explicitly selects PNG or JPEG output.
* `setQuality()` sets the image quality for JPEG screenshots.
* `setOmitBackground(true)` hides the default white background and allows transparency in PNG screenshots.

For example, capture the entire Bookstore page with:

```java
page.screenshot(new Page.ScreenshotOptions()
        .setPath(Path.of("target", "screenshots", "bookstore-full-page.png"))
        .setFullPage(true));
```

### Choose a screenshot output directory

Choose a dedicated output directory for screenshots. The following example uses `target/screenshots`:

```java
Path screenshotDirectory = Path.of("target", "screenshots");
Files.createDirectories(screenshotDirectory);
page.screenshot(new Page.ScreenshotOptions()
        .setPath(screenshotDirectory.resolve("bookstore-home.png")));
```

`Files.createDirectories()` is safe to call when the directory already exists.

## Capture an individual element

Call `screenshot()` on a `Locator` to capture a specific element, such as the **Sign in** button:

```java
@Test
void shouldCaptureSignInButtonScreenshot() throws IOException {
    page.navigate("http://localhost:8080/login");

    Files.createDirectories(Paths.get("screenshots"));

    Locator signInButton = page.getByRole(
            AriaRole.BUTTON,
            new Page.GetByRoleOptions()
                    .setName("Sign in")
                    .setExact(true)
    );

    signInButton.screenshot(
            new Locator.ScreenshotOptions()
                    .setPath(Paths.get("screenshots/sign-in-button.png"))
    );
}
```

Playwright scrolls the element into view and captures only its bounding box. This is useful for focused evidence of forms, buttons, messages and other UI components.

## Capture screenshots automatically when tests fail

JUnit 5 extensions can capture failures without adding a `try-catch` block to every test. We will use `AfterTestExecutionCallback` because it runs after the test method but before `@AfterEach` cleanup, while the Playwright page is still open.

### Register the failure callback in the base test class

Add an `AfterTestExecutionCallback` extension to `BasePlaywrightTest` and register it with `@RegisterExtension`:

```java
public abstract class BasePlaywrightTest {
    protected Page page;

    @RegisterExtension
    final AfterTestExecutionCallback screenshotOnFailure = context -> {
        if (context.getExecutionException().isEmpty()) {
            return;
        }

        Path directory = Path.of("test-results", "screenshots");
        Files.createDirectories(directory);

        String testName = context.getDisplayName()
                .replaceAll("[^a-zA-Z0-9.-]", "_");

        page.screenshot(new Page.ScreenshotOptions()
                .setPath(directory.resolve(testName + "-failed.png"))
                .setFullPage(true));
    };

    // Existing Playwright setup and cleanup methods
}
```

`@RegisterExtension` registers the callback for every test class that extends `BasePlaywrightTest`. The callback uses `context.getExecutionException()` to determine whether the test failed. When no exception is present, it returns immediately and does not create a screenshot.

For a failed test, the callback creates the `test-results/screenshots` directory and captures the active `page`. `context.getDisplayName()` supplies the test name, and `replaceAll()` replaces characters that may be unsafe in a filename with underscores. The `-failed.png` suffix makes the purpose of the image clear, while `setFullPage(true)` includes the entire scrollable page.

Because `AfterTestExecutionCallback` runs immediately after the test method and before `@AfterEach`, the `Page` is still available. The existing Playwright setup and cleanup methods in `BasePlaywrightTest` do not otherwise need to change.

### Run the invalid-login test

Run the same invalid-login example. No annotation is needed on this class because it inherits the registered extension from `BasePlaywrightTest`:

```java
class BookstoreFailureScreenshotTest extends BasePlaywrightTest {

    @Test
    void shouldCaptureScreenshotWhenLoginAssertionFails() {
        submitLogin("Neha@gmail.com", "neha@");

        Locator errorMessage = page.getByRole(AriaRole.ALERT);
        assertThat(errorMessage).hasText("Wrong error message.");
    }
}
```

The application displays its real validation message, but the test expects `Wrong error message.`. The assertion therefore fails, and JUnit invokes `screenshotOnFailure` before browser cleanup. A full-page image derived from the test's display name and ending in `-failed.png` is written to `test-results/screenshots`. Passing tests do not produce screenshots.

The automatically captured screenshot provides a visual reference for the application state at the moment the assertion failed:

![Bookstore login page showing the invalid email or password alert captured after a test failure](/images/bookstore-login-failure-screenshot.png)

## Conclusion

This article demonstrated how to capture viewports, full pages and individual elements with Playwright, as well as how to automate failure screenshots with a JUnit 5 extension.

Automatic failure screenshots turn unexpected test failures into actionable visual evidence, helping teams identify root causes faster and debug with greater confidence.
