It might be confusing to understand when to write Ui test rather than Screenshot tests and vice versa. They do not replace each other. Their focus is different as previously mentioned.
So let's imagine the following screen, which is a RecyclerView

![language learning app](../images/snapshotVsUiTests.gif "Snapshot testing example")

## What to Ui test
A *Ui test would verify*, e.g. that after deleting a row in the recycler View, that row is not displayed anymore. It would test *WHAT is displayed after interacting with the view*
</br></br>
Therefore, write a Ui test if:

1. You need to interact with one or more views
2. You need to assert a certain behaviour after such interactions
    1. Navigation to another screen
    2. Visibility of some Ui elements
    
You do not mind how pixel perfect every single ui element looks on the screen. You just care about the result of your interactions: *WHAT is displayed* instead of *HOW it is displayed*

## What to Screenshot test
On the other hand, a *snapshot test would verify HOW that row is displayed* under numerous states and configurations: e.g. dark/light mode, LTR/RTL languages, different font sizes, wide/narrow screens...
Therefore, write a Snapshot test if:

1. you've made a visual change in a Ui element
2. you want to verify HOW that change is displayed under different configurations

In this case you are saving time to yourself and everybody involved in the QA process: **nobody needs to play around with numerous settings/states** to ensure everything looks pixel perfect. That process is cumbersome and you've automated it.
</br></br>
![snapshot testing example](../images/snapshotTesting.png "Snapshot testing example")
> Up: Row when system font size set to huge </br>
Down: Row in dark mode

## Use the right tool for the job
If you are new to Screenshot testing, don't fall into the trap of thinking that it can replace Ui testing. 

You might be thinking that you could write a full screen snapshot test after deleting the row to verify it is not displayed anymore.
If you had the test already written, you'd just need to change your view visibility assert with a snapshot assert.
Keep in mind that this approach does not solve some common problems with Ui tests:

1. **Flakiness**: Screenshot tests also come with flakiness, and even its own issues e.g. mind dates if displaying any (more about this later). As with Ui tests, those problems can be mitigated though.

2. **Speed**:
     - *Fake Snapshot tests*: Writing Screenshot tests that interact with views the same way as Ui tests, *do not make them any faster*. For that you need to write Screenshot tests that just inflate a view under a given state and snapshot it. This is what I call a *fake Screenshot test: a Ui test disguised with a snapshot assert*.
     - *Less-scalable test sharding*: If you are thinking about using a cloud device service like Firebase test lab with test sharding to speed up the execution, it is not that simple. Snapshot file comparisons are done pixel by pixel.
       This means, all tests must run on the same device models across all parts involved (devs and CI) to ensure that the resolution, screen size and api create screenshots with identical pixels. This restricts a lot the speed wins of test sharding usually gained with such services.
       
!!! Disclaimer
     *Test sharding* allows to evenly split up the test suite into all connected devices. This enables to parallelize your tests. So if you have 300 tests and 30 devices, 10 tests run on every device in parallel, resulting in considerably lower test execution times.
       

Moreover, you will face the following issues:

1. **If still no Snapshot tests in place & planning to run Snapshot tests on emulators**, dealing with them does not make things easier 
    1. ensuring every part involved (i.e. developers and CI) has the same emulator config: snapshot assertions happen pixel by pixel.
    2. rendering issues due to hardware acceleration (also on physical devices)
    3. insufficient storage errors due to the space taken by the generated files, etc.
    
    Actually, if you were using physical devices on the CI and move to emulators due to Snapshot testing, you'll additionally face the following troubles:
   
       1. freezing emulators on the CI
       2. synchronizing emulators start up before running the tests

2. **Tests become brittle**: they fail badly if altered in a seemingly minor way.

Let me explain the last point with an example.
Imagine you change how the row is displayed. If you verify the snapshot test, it will fail. You need to record a new snapshot including those changes on the row. The issue here is that the focus of your test was to verify that the deleted row is not displayed anymore. What does it have to do with changing the appearance of the row? You guessed it. Nothing. But the test fails because of that.

You do not want your test to fail for the wrong reason. You want them to be meaningful. You want them to have a purpose

I'd like to mention that you do not even need to change the appearance of the row intentionally. It's enough that your screen displays dates depending on the current time:
the date changes on every run, making your "fake screenshot test" fail. Again, failing for the wrong reason.

Therefore, *every subtle change on the screen will require to record a new snapshot, although that change had nothing to do with the initial intention of the test*.

On the other hand, a Ui test would have not failed since we would be asserting whether the deleted row was displayed or not. No visuals involved.

## Conclusion
1. **Use both Ui test and Snapshot tests**, they complement each other. They aim to assert different things.
2. **Avoid fake snapshot tests**, they usually add up troubles compared to ui tests rather than mitigating their issues.
3. **Use the right tool for the job**: using Screenshot tests for testing interactions leads to **brittle tests**.

