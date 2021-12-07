# Screenshot testing

## What it is
Screenshot testing (also called Snapshot testing) has been in the Android world since 8th October 2015, when Facebook open sourced the first version of their snapshot testing library.

They are a special type of UI tests that inflate a view, take a screenshot of it, and compare it to an already stored image taken as reference. This reference is considered the source of truth: it depicts how the UI must be displayed, pixel by pixel.
If the generated snapshot file from the test differs from the reference, the test fails, passes otherwise.

## Motivation
Screenshot tests are easy to write and maintain and run fast (≈ 1 sec per test), what makes them affordable to execute upon PRs.
They focus on detecting visual bugs that standard **UiAutomator** and **Espresso** cannot, namely

1. UI defects introduced by updates in libraries like Material design and Constraint Layout
   
2. Spacing, styles and themes (light mode vs. dark mode)
   
3. Layout correctness:
     - Multiline text does not cut off
     - No view overlapping on visibility changes
     - Proper view alignment under different screen densities and/or font sizes
     - Text, icons and UI alignment on RTL & LTR languages, among other language related issues

## How it works
First of all, we need to understand that snapshot testing process differs from standard testing.
Most snapshot testing frameworks provide two main tasks:

1. **Record**: Generates a snapshot file. Executed locally. this file will be reviewed by our peers, and once approved, uploaded to the CI as our source of truth. All further verification fo that test will be made against it.
2. **Verify**: When executed, it generates a snapshot file for each test that will be compared, pixel by pixel, with the one taken as reference when recording. Executed on the CI.

Once everybody in the team has agreed the same device setup for snapshot testing, the process goes as follows:

1. Create a test that snapshots a view. This could range from a view dynamically inflated to the activity that is shown on the emulator/device screen.
2. Make sure adb only lists the snapshot testing device/emulator. That is because instrumentation tests run on all emulators/devices adb detects. This might cause some issues like snapshot file duplication errors, longer running times, and the like.
3. Execute "record" locally to generate the corresponding snapshot files and open the corresponding PR containing them.
4. Some teammates review the snapshots to confirm that they reflect the state of the view we are expecting. Once approved and merged, this snapshot is taken as source of truth for future verifications of the tests.
5. Verify the snapshots periodically (e.g. on every PR, once a day, etc.). Ideally you have a CI with a job configured to "verify" them. This means, it executes the snapshot tests with the most up-to-date code, and takes a screenshot. After that, it compares this screenshot with the last one uploaded to the CI or the one being pushed in the PR. If they both match, the test passes, fails otherwise.


## Challenges
Unfortunately, screenshot tests also come with their own problems. 

1. **Flakiness**: Same as with UI tests, snapshot test face flakiness problems. On one hand, we face similar problems to UI Testing. Most of the issues described in the [flakiness](../practices/flakiness.md) section also apply to screenshot tests.
Additionally, snapshot test suffer from other issues. Eli Hart from Airbnb has already written [an article](https://medium.com/airbnb-engineering/better-android-testing-at-airbnb-a11f6832773f) about most of them, and I can say I've faced even more. I'll state them briefly:
     - *Rendering issues*: Hardware accelerated drawing model might cause issues like wrongly rendered pixels, especially with Composables. See more [here](https://developer.android.com/guide/topics/graphics/hardware-accel)
     - *Drawable caching*: the Android system maintains a cache of drawables. If the bitmap of such drawable is shared in several locations, the result could vary depending on whether the bitmap has already been cached in a previous test or not. This introduces flakiness in pixel by pixel comparisons. 
     - *Cached state in Views*: some views like ConstraintLayout have their own cache, so that in the next pas they do not have to recalculate anything, what might cause flakiness. 
     - *Dates*: if displaying dates that depend on the current time, the screenshots will change on every rerun and fail while comparing to the one taken as reference.
     - *Image loading from urls*: some libraries like Picasso and Glide download images from urls asynchronously under the hood. These makes tests nondeterministic, because we cannot ensure the image loading state of the ImageView before taking the screenshot:
          1. Random blank image or loading if it was taken before the image was downloaded
          2. Error image placeholders if the download failed, etc.
     - *RecyclerView prefetching*: By default, RecyclerView’s LinearLayoutManager prefetches off screen views outside its viewport while the UI thread is idle between frame.
     - *FillViewPort measuring*: the size of the fillViewPort view might not have been calculated before taking the screenshot. This might result into that View being cut off, so some of its children would not be visible
     - *Animation libraries*: libraries like lottie, that load an animation into an ImageView from a Json file. Disabling animations do not take effect on this. They happen asynchronously so we cannot guarantee at which stage is the animation before taking the screenshot.
     - *Webview mocking*: same problems as asynchronous loading. We cannot ensure the content will be loaded before taking the snapshot 

We'll describe them in detail in their own section in the future.

2. Emulator configuration in all parts involved (*instrumented snapshot tests only*)
     - Emulators freezing on the CI when idle for a long time. The best practice is the same as with UI tests: close them right after running the tests
     - Synchronizing emulators start up before running tests. Instrumented snapshot tests run on every device adb can detect. For that, we need to start all the emulators and wait till they are ready. We can achieve this by using tools like [swarmer](https://github.com/gojuno/swarmer). The new official [Gradle Managed Virtual Devices](https://developer.android.com/studio/preview/features#gmd) looks also promising, although it enables us to run UI tests only for now.
     - Avoid "Insufficient Storage" errors by starting adb emulators with the `--wipe_data` option

One could also use real devices for snapshot testing, but it does not scale. Devices are costly, and you need to provide the same model to every single developer in the team, as well as your CI.

!!! Important note

      Developers that have integrated UI test previously into their build pipelines, know that instrumented UI tests are rather slow.
      One popular alternative is to use Robolectric to run UI test on the JVM. This makes UI tests much faster.

      But this is not the same with snapshot tests. 
      
      1. **Execution time**: Instrumented snapshot tests without interactions can run very fast, in less than 1 second.
      2. **Test Sharding**: JVM snapshot tests can not be sharded. That hinders speed wins by running them in parallel.
      Therefore, the real power of running snapshot tests on the JVM is not having to deal with emulators.

## Frameworks
1. Instrumented
     - [Facebook Screenshot tests](https://github.com/facebook/screenshot-tests-for-android): First screenshot testing framework on Android.
     
     - [Shot](https://github.com/pedrovgs/Shot): Written on top of Facebook by Pedro.
     
     - [Testify](https://github.com/Shopify/android-testify#readme): Shopify framework. Last one open sourced

2. JVM
     - [Paparazzi](https://github.com/cashapp/paparazzi): Square framework
   
Unlike UI tests, it is not possible to write shared tests

## Related links
### Blog posts
[An introduction to snapshot testing on Android in 2021](https://sergiosastre.hashnode.dev/an-introduction-to-snapshot-testing-on-android-in-2021)</br>
[The secrets of effectively snapshot testing on Android](https://sergiosastre.hashnode.dev/the-secrets-of-effectively-snapshot-testing-on-android)
### Code repos
[Road to Effective Snapshot Testing](https://github.com/sergio-sastre/Road-To-Effective-Snapshot-Testing)

## Conclusion
1. Although it has become very popular recently, screenshot testing has been possible on Android since 2015.
2. Differs from UI testing on its purpose: Screenshot testing focuses on the visuals, how the views are displayed under different configurations.
3. Screenshot tests requires at least 2 tasks, record (locally) and verify (on the CI). The process also requires more steps than standard UI testing.
4. Flakiness is also an issue. Screenshot tests share some common troubles with UI tests, but face many more problems additionally.
5. There are numerous open source frameworks for instrumented screenshot testing, and even one for the JVM. However, no shared tests are possible.

 



