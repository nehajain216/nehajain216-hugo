---
title: "Getting Started With Playwright Using Java, Maven and JUnit"
date: 2026-08-05
description: "Learn how to set up Playwright with Java, Maven and JUnit, then write a first browser navigation test."
categories: ["QA"]
tags: ["QA", "Testing", "Playwright", "Java", "Maven", "JUnit", "Automation"]
image: "/images/start-playwright.png"
readTime: "4 min read"
---

Playwright is a modern end-to-end testing framework that helps automate browser interactions across Chromium, Firefox and WebKit. It is commonly used for UI automation, regression testing and smoke testing web applications.

In this post, we will create a basic Playwright setup using Java, Maven and JUnit. The first test will open a Chromium browser and navigate to `https://nehajain.netlify.app/`.

## Prerequisites

Before starting, make sure you have:

* Java Development Kit (JDK) installed
* Maven installed
* IntelliJ IDEA, Eclipse or any Java IDE
* Basic knowledge of JUnit tests

You can verify Java and Maven from the command line:

```bash
java -version
mvn -version
```

## Step 1: Create a Maven project

Create a new Maven project in your IDE or use Maven from the command line:

```bash
mvn archetype:generate \
    -DgroupId=dev.nehajain \
    -DartifactId=playwright-java-demo \
    -DarchetypeArtifactId=maven-archetype-quickstart \
    -DinteractiveMode=false
```

After the project is created, open the `pom.xml` file and add Playwright and JUnit dependencies.

## Step 2: Add Playwright and JUnit dependencies

Update your `pom.xml` file as shown below:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>dev.nehajain</groupId>
    <artifactId>playwright-java-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>25</maven.compiler.source>
        <maven.compiler.target>25</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <playwright.version>1.61.0</playwright.version>
        <junit.jupiter.version>5.14.4</junit.jupiter.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.microsoft.playwright</groupId>
            <artifactId>playwright</artifactId>
            <version>${playwright.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.jupiter.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.6</version>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.6.3</version>
            </plugin>
        </plugins>
    </build>
</project>
```

This configuration adds Playwright for browser automation and JUnit Jupiter for writing and running tests.

## Step 3: Install Playwright browsers

Playwright needs browser binaries before tests can run. Use the Playwright CLI through Maven:

```bash
mvn exec:java -e -Dexec.mainClass=com.microsoft.playwright.CLI -Dexec.args="install"
```

This installs the default browsers used by Playwright.

## Step 4: Write the first Playwright JUnit test

Create a test file at:

```text
src/test/java/dev/nehajain/demo/PlaywrightNavigationTest.java
```

Add the following code:

```java
package dev.nehajain.demo;

import com.microsoft.playwright.Browser;
import com.microsoft.playwright.BrowserContext;
import com.microsoft.playwright.BrowserType;
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Playwright;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class PlaywrightNavigationTest {
    private static Playwright playwright;
    private static Browser browser;

    @BeforeAll
    static void setUp() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
        );
    }

    @AfterAll
    static void tearDown() {
        browser.close();
        playwright.close();
    }

    @Test
    void shouldOpenBrowserAndNavigateToWebsite() {
        BrowserContext context = browser.newContext();
        Page page = context.newPage();

        page.navigate("https://nehajain.netlify.app/");

        assertEquals("https://nehajain.netlify.app/", page.url());
        context.close();
    }
}
```

In this example:

* `Playwright.create()` starts Playwright.
* `playwright.chromium().launch()` opens a Chromium browser.
* `browser.newContext()` creates a clean browser session.
* `page.navigate()` opens the target website.
* `assertEquals()` verifies that the browser reached the expected URL.

The browser is launched in headed mode using `setHeadless(false)`, so you can see the browser window while the test runs. For CI pipelines, remove that option or set it to `true`.

## Step 5: Run the test

Run the test from your IDE or use Maven:

```bash
mvn test
```

You should see Chromium open, navigate to `https://nehajain.netlify.app/`, and then close after the test completes.

## Step 6: Create an abstract base test

The first test works, but as the test suite grows, repeating Playwright setup and cleanup in every test class becomes hard to maintain. Each test class would need to create Playwright, launch a browser, create a page and close everything correctly.

A better approach is to move the common browser setup into an abstract base test class. Then each test class can extend it and focus only on the actual test steps.

Create a base test file at:

```text
src/test/java/dev/nehajain/demo/BasePlaywrightTest.java
```

Add the common setup and cleanup code:

```java
package dev.nehajain.demo;

import com.microsoft.playwright.Browser;
import com.microsoft.playwright.BrowserContext;
import com.microsoft.playwright.BrowserType;
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Playwright;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;

abstract class BasePlaywrightTest {
    // Shared between all tests in this class.
    static Playwright playwright;
    static Browser browser;

    // New instance for each test method.
    BrowserContext context;
    Page page;

    @BeforeAll
    static void launchBrowser() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false).setSlowMo(50)
        );
    }
    @AfterAll
    static void closeBrowser() {
        playwright.close();
    }

    @BeforeEach
    void createContextAndPage() {
        context = browser.newContext();
        page = context.newPage();
    }

    @AfterEach
    void closeContext() {
        context.close();
    }
}
```

Now the navigation test becomes much smaller:

```java
package dev.nehajain.demo;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class PlaywrightNavigationTest extends BasePlaywrightTest {

    @Test
    void shouldOpenBrowserAndNavigateToWebsite() {
        page.navigate("https://nehajain.netlify.app/");

        assertEquals("https://nehajain.netlify.app/", page.url());
    }
}
```

This gives each test a fresh browser context and page, which helps keep tests independent. 
It also keeps setup logic in one place, so changing browser options, switching headless mode can be done from the base class instead of updating every test class.

## Next steps

Once the basic navigation test is working, you can expand it by:

* Locating elements with Playwright locators
* Clicking buttons and links
* Filling forms
* Adding assertions for page text
* Running tests in headless mode in CI
* Capturing screenshots or traces for debugging

Playwright works well for fast browser automation, and using it with Java, Maven and JUnit makes it easy to include UI tests in an existing Java testing workflow.
