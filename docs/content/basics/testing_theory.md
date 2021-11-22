# Testing theory


This article will cover the most primitive things that will allow you to understand better the terms and general concepts regarding testing.
<br/>It is also worth noting that there will be an accent on UI tests, because we are in CookBook for UI tests.


## Where it all starts
<i>Why do I need testing?</i> Users of your application/system/whatever should receive a quality product. 
<br/>Thus, testing is about ensuring the quality of the product. Testing also includes creating a test plan, creating/conducting tests themselves and analyzing results of testing.

There are different means to classify tests. We will focus on the following

1. *Granularity*
2. *Implementation details known*
3. *Running environment*
4. *Purpose*

## Classifying tests by granularity
To begin with, let's see the testing pyramid.

![Testing pyramid](../images/pyramidE2E.png "Testing pyramid")
> Testing Pyramid (source Martin Fowler, triangle authorship Kent C. Dotts)

1. **End-to-End tests (E2E)**: these tests verify that all components in every single layer of the application work together as expected. They allow to test User stories - how our users are going to use the application. This makes them incredibly important both for business and for an ordinary developer or tester.
   However, this is the most unstable type of test, and the cost of creating and maintaining them is high. That's because there are a lot of factors we cannot control (e.g. network failures, clean user state before and after the test, etc.).
2. **Integration tests**: they focus on validating the interaction of 2 (or more) entities at once at the same time, but not the full system as with E2E tests. Most components involved are not mocks or stubs, but doubles or fakes.
3. **Unit tests**: they only cover the smallest testable unit, usually a function, or view.

The width of each block is actually the ratio of the number of different types of tests to each other. 
For example, a lot of Unit tests is usually considered correct in the pyramid, but much less E2E tests. 
<br/><br/><b>This is due to two key parameters - stability and the cost of supporting each type of testing.</b>
<br/>Unit tests have the highest stability, they are the fastest and they have the lowest cost of support. However, Unit tests do not intend to verify User stories e.g. the login flow of your application (which for example contains 5 screens).

## Classifying tests by the implementation details known
Testing can be classified regarding their access to the implementation details as follows:

1. **White-box testing**: is a type of testing in which we have full access to the implementation and can interact with it. We know which output data will be with given input data.
2. **Black-box testing**: is a type of testing in which we don't have access to the implementation and cannot interact with it, however, we know which output data should be with given input data.
3. **Gray-box testing**: is a type of testing when you have partial access to the implementation (for example, not to all entities that being tested). At the same time, we know what output data will be with given input data.

Most of the instrumentation test frameworks like Espresso and UiAutomator use Gray-box testing: They use view ids or text to interact with the Views.
The Robot and Page Object pattern try to help with this by adding another abstraction layer that segregates the *WHAT* from the *HOW*. You can find more on that in the [Page Object](../practices/page_object.md) section
</br>
</br>
</br>
Apart from these 2 clusters, there are two other interesting means to classify tests, as pictured below

![Group 5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1637010289284/lS74gW9vvt.png)

!!! Disclaimer

      Important to note that these classifications are not mutually exclusive. One can write, for instance

      1. Ui tests that run on the JVM and only test one view (i.e. unit test)
      2. Non-Ui tests that run on a device (i.e. instrumented) and test several components (i.e. integration test)

## Classifying tests by the environment where they can run on
Depending on the environment the tests can run on, we get the following distribution

1. **JVM (Java Virtual Machine) tests**: can run without the need of an emulator or physical device. These tests do not contain Android-specific code, although they might mock it under the hood.  

2. **Instrumentation tests**: those that require an emulator or physical device. These are tests that
   contain Android-specific code, like Views. Such code requires a DVM (Dalvik Virtual Machine) or Android Runtime (ART) since 5.0 to be executed, and therefore it cannot run on the JVM but on a device.

3. **Shared tests**: These are tests that are written once, but can run either as JVM or as Instrumentation tests. The principle is simple: *Write once, run everywhere*.
   These tests contain Android-specific code. Such code would need a device to run on. In order to enable that, when running on the JVM, that code is replaced by mocks under the hood.
   
## Classifying tests by their purpose
Depending on the goal of our tests, they are split into the following categories

1. **UI tests**: focus on testing the WHAT (interaction/navigation); *WHAT is displayed* when interacting with the screen elements.
2. **Screenshot/Snapshot tests**:  focus on testing the HOW (visuals): *HOW a view is displayed* under a given state or configuration.
3. **Non-UI tests**: focus on testing non-ui related code like *BUSINESS LOGIC* or *DATABASE MIGRATIONS* among others

## Manual testing
Although we've mainly focused on automated tests, it is also worth noting the importance of **manual testing** in the quality of an app.
<br/>In manual testing se do not write automated tests, but rather steps we need to reproduce the User story or bug.
This is especially needed for User stories that are very hard or impossible to completely automatize, or just not worth the effort. In this group are also very edge cases that would require extremely complex and large tests, if even possible. That is where manual testing comes in.
<br/>Most E2E tests are performed manually. Automated E2E tests are the hardest to get stable, due to various components being involved, some of them out of our control (e.g. stable internet connectivity).
<br/>Hence manual testing has an important role and cannot be completely replaced by conventional types of testing.
<br/>But don't forget that this is essentially a question of the acceptable quality of a particular product.

## Conclusion
* We understood that we can classify tests by different criteria, and every test belongs to one group in every classification
* E2E tests are the most unstable, the longest and the most expensive in terms of support but they allow you to test entire User stories and can do it even on every merge.
* We got the basic concepts of testing, learned about the difference between Black/White/Gray-box testing.
* We are aware that, although Snapshot tests and UI tests verify the Ui, they serve different purposes.
* We got that we can write Shared tests that can run on the JVM or on the device.
* We understood that automated tests cannot completely replace manual testing.

