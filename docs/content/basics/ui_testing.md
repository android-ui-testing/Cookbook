# UI testing

UI tests is a part of instrumentation tests.
That's why everything from <link> Instrumented tests topic everything is applicable to UI testing.

## Main tool - Espresso

Nowadays there is no doubt - for native UI testing on Android we should use Espresso for testing your application.
<br/> This is a main tool that allow us to make every test possible to access your codebase. That means tests with Espresso allow us to write white-box tests.
<br/>But you still able to write black-box tests or even gray-box with Espresso.

Why Espresso is so cool? It is synchronized with the main thread of your app and performs action only when main thread is in idle state.
<br/>It makes Espresso extremely reliable and stable (than any other existing tool) for Ui testing on Android.

Keep in mind that everything that isn't related to your app (for example, permission dialogs, volume +/- buttons, home button and etc) - can't be accessed directly within Espresso.
<br/>But don't worry - you can use `UiAutomator` for that purpose, and it can be called directly from test, written in Espresso.
<br/>`UiAutomator` is a slower tool and can support tests only in black-box style, but it can test everything outside your application on your phone.
<br/>Many cross-platforms tools for testing using `UiAutomator`.

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
![alt text](../images/ui_testing/ui_testing_button.png "First state")![alt text](../images/ui_testing/ui_testing_button_clicked.png "Second state")
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

!!! Disclaimer

        Important addition - when you are trying to find some `View` that, for example, has same `id` like another View on that screen (that situation usually happens when you are trying to test screens with `RecyclerView`) you can add more `Matchers`
        Simply place all `Matchers` that apply to single `View` on your screen inside of `allOf()` method
        It will look like that
        
        <br/>@Test
        <br/>fun someTest() {
        <br/>&nbsp;&nbsp;&nbsp;onView(allOf(withId(R.id.button), withText("Click me")))
        <br/>}
        

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
![alt text](../images/ui_testing/ui_testing_results.png "First state")

**Final code of our test**
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

### What else?

Espresso is quite powerful tool with a lot of abstractions inside it. 
<br/>But there is a quite famous "cheat-sheet" for Espresso, I hope it will be helpful for everybody.

![alt text](../images/ui_testing/ui_testing_cheat_sheet.jpeg "Cheat sheet")

From that cheat-sheet you can understand which methods you can use for finding the `Views`, which methods for interactions with Views and which methods for checking the data.

### Something like conclusion

To be honest, there is a lot of edge-cases once you are trying to write your own tests. That's one of the reasons why that CookBook was created, so we are suggesting to read next articles to understand how to overcome many of common problems that you will face in Ui testing.