---
layout: post
title: "The Essential Role of Test-Driven Development in Modern Software Engineering"
date: 2024-06-24 16:00:00 +0000
categories: Testing
tags: testing tdd
---

I've been a software engineer now for 13 years now delivering many applications to production, used by millions of
people. For 12 of those years, I've coded utilizing Test-Driven Development (TDD). It's been instrumental in allowing me
to deliver fast and with confidence. In this post, I'll be covering high-level steps on what it is, why do it, and how
to do it, with tips that I've learned through experience.

## What is it?

TDD is a testing methodology where you:

1. **Write a single unit test:** Begin by creating a unit test that targets a specific functionality or behaviour you
   want to implement. This test should be clear and concise, defining what the code should do before any actual
   implementation begins.

2. **Run the test and see it fail for the correct reason:** Execute the test to ensure it fails. This step is crucial as
   it validates that the test is correctly written and that the functionality doesn't exist yet. The failure confirms
   that the test will detect when the functionality is added correctly.

3. **Write the smallest amount of code possible to make the test pass:** Implement just enough code to pass the test.
   This doesn't mean using single-letter variable names or writing cryptic code, but rather avoiding any extra features
   or logic not required by the test. This helps keep the codebase lean and focused.

4. **Run the test and see it pass:** Re-run the test to verify that the newly written code fulfils the requirements. A
   passing test confirms that the code works as intended for the specific case the test covers.

5. **Refactor:** Once the test passes, review the code and improve its structure and readability without altering its
   behaviour. Refactoring might not be necessary after every test, but it should be done regularly to maintain code
   quality and manageability.

6. **Repeat:** Continue this cycle for each new piece of functionality. The iterative process helps build up the
   application incrementally while maintaining a robust set of tests that ensure ongoing reliability.

It is **not** a separate task done after implementation; rather, it is an integral part of the development process.

## Why do it?

There are many benefits to TDD, both in the short term and in the long term:

- **Forces you to think about the API:** TDD encourages developers to design the interface and behaviour of their code
  before implementation. This upfront consideration often leads to more thoughtful and well-designed APIs.

- **Reduces the amount of unnecessary code:** By focusing only on code that makes tests pass, TDD helps avoid writing
  superfluous features or logic, resulting in a more efficient and maintainable codebase.

- **Guarantees high test code coverage:** TDD ensures that every piece of functionality is accompanied by a test,
  leading to comprehensive test coverage and reducing the likelihood of bugs.

- **Keeps the main/primate branch always shippable:** Continuous integration of tested code ensures that the main branch
  remains in a deployable state, minimizing the risk of introducing breaking changes.

- **Provides quick feedback:** Immediate test results provide rapid feedback on code changes, allowing developers to
  detect and fix issues early in the development process.

- **Gives confidence when refactoring:** A robust suite of tests acts as a safety net when refactoring, ensuring that
  changes do not unintentionally break existing functionality.

## Won't my PR's be huge?

![vertical-slices](/images/tdd-vertical-slices.png)

Without planning, yes, they can be. To avoid this, aim for "vertical slices" in your pull requests.

- **Vertical slices:** These slices should contain all elements related to a specific feature, including a **small**
  part of the implementation, unit tests, functional tests, and documentation. This approach ensures that each PR is
  focused, manageable, and easier to review.

- **Horizontal slices:** In contrast, horizontal slices involve separating the **whole** implementation from the tests
  and documentation, often leading to larger and more complex PRs. Avoiding this approach helps keep PRs concise and
  relevant to a particular feature or functionality.

## Tips

- **Write one test at a time:** Focus on one test at a time to ensure that you are only writing the tests you actually
  need. If you like to plan ahead, consider creating empty test methods or listing tests in code comments.

- **Only assert one thing per test:** Limiting each test to a single assertion makes it easier to diagnose failures and
  simplifies test naming and organization.

- **Make sure the test is failing for the correct reason before implementation:** Verify that the initial test failure
  is due to the missing functionality and not a test issue. This ensures that the test is valid and will correctly
  indicate when the functionality is implemented.

- **Group test code in given, when, then sections:** Organize your test code into distinct sections: "given" for
  setup, "when" for the action, and "then" for the assertion. This structure keeps tests clear and reduces the need for
  complex logic within tests.

- **Don’t extract into too many helper functions:** While helper functions can reduce code duplication, overusing them
  can make tests hard to follow and modify. Aim for a balance that maintains readability and simplicity.

- **Think about test naming:** Well-named tests act as living documentation, clearly describing the functionality they
  cover. Use descriptive names that convey the purpose and expected outcome of each test.

- **Don’t test private methods/functions:** Testing private methods can couple your tests too closely with the
  implementation, making refactoring difficult. Focus on testing public APIs to ensure that tests remain resilient to
  internal changes.

- **Use mocking only when necessary:** Mocking can simplify tests by isolating dependencies, but overuse can lead to
  brittle tests. Mock only when dealing with downstream calls or when the unit is too complex to test directly, reducing
  coupling and making refactoring easier.

- **If you’re unclear on implementation, run a spike or a POC beforehand:** Conducting a proof of concept (POC) or spike
  can clarify the implementation approach. However, do not use the spike/POC code in production. Rebuild it from scratch
  using the knowledge gained as a reference.

- **Keep things consistent:** Consistency in test and code structure helps maintain readability and ease of maintenance.
  Establish and follow coding standards and practices across your team.

By adhering to these principles and practices, you can leverage TDD to produce high-quality, maintainable code, enhance
your development workflow, and ultimately deliver more reliable software.

## Conclusion

Test-Driven Development is a powerful methodology that enhances code quality, maintainability, and developer
productivity. By integrating testing into the development process, TDD ensures that your code is robust and reliable
from the start. I encourage you to try TDD in your next project and experience its benefits first hand. Share your
experiences or ask questions in the comments below!