# UI testing

UI tests is a part of instrumentation tests.
That's why everything from <link> Instrumented tests topic everything is applicable to UI testing.

## Main tool - Espresso

Nowadays there is no doubt - for native UI testing on Android we should use Espresso for testing your application.
<br/> This is a main tool that allow us to make every test possible to access your codebase. That means tests with Espresso allow us to write white-box tests.
<br/>But you still able to write black-box tests or even gray-box with Espresso.

Why Espresso is so cool? It is synchronized with the main thread of your app and performs action only when main thread is in idle state.
<br/>It makes Espresso extremely reliable and stable (than any other existing tool) for Ui testing on Android.

But if you need to test several apps in your test - you should use UiAtomator.

### Where to start

Firstly, you should add dependency for Espresso library to your Android app module

```kotlin
dependencies {
    androidTestImplementation('androidx.test.espresso:espresso-core:3.4.0')
}
```
After that, you can create your first test
<br/>But before that, let's get familiar with Espresso.
<br/>Basically to start writing Ui tests we should:
* Find the `View` (using `ViewMatchers`)
* Interact with that `View` (using `ViewInteraction`)
* Check state of `View` (using `ViewAssertion`)

Let's start with the basic test.
<br/>We have activity with one button (and no text shown), once we press it - text "Button pressed" shown.
<image of app with ids>
1. We should specify which activity we should run.
```kotlin
import androidx.test.ext.junit.rules.ActivityScenarioRule
import org.junit.Rule
...
     @get:Rule
    var activityRule: ActivityScenarioRule<MainActivity>
            = ActivityScenarioRule(MainActivity::class.java)
```
2. Create test and find our button (with `onView()` method and `withId` assertion by id)
```kotlin
    @Test
    fun pressButtonAndCheckText() {
        onView(withId(R.id.button))
    }
```
3. Then we should perform click on it (with `click()` method from `ViewInteraction`)
```kotlin
    @Test
    fun pressButtonAndCheckText() {
        onView(withId(R.id.button))
            .perform(click())
    }
```
4. And finally, let's check that our text is shown (find the `TextView` and assert that it is displayed)
```kotlin
    @Test
    fun pressButtonAndCheckText() {
        ...
        onView(withId(R.id.textview))
            .check(matches(isDisplayed()))
    }
```
Once we run it (run it as usual test, but you should connect real phone or run emulator), we will see standard test result
<image with result>

Final code of our test
```kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {

    @get:Rule
    var activityRule: ActivityScenarioRule<MainActivity>
            = ActivityScenarioRule(MainActivity::class.java)

    @Test
    fun pressButtonAndCheckText() {
        onView(withId(R.id.button))
            .perform(click())

        onView(withId(R.id.textview))
            .check(matches(isDisplayed()))
    }
}
```
### Finding views

### Interact with views

### Your first test

### Handling edge cases

### Other tools