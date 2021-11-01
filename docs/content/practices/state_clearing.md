# State clearing

This question appears as soon as you need to run more than 1 ui test.

## Problem

We run `Test1`, it performs some http requests, saves some data to files and databases.
<br/>When `Test1` is finished, `Test2` will be launched.
<br/>However, `Test1` left some data on the device which can be a reason of `Test2` failing.

Solution — clear the data before each test

## 1. Clearing within a process

In this case, we don't kill our application process, and we have 2 options here:

##### Use component from a real code base <br/>

``` kotlin
@Before 
fun setUp() {
   DI.provideLogoutCleanerInteractor().clear() 
}
```

The same component which clears data _(For instance, while logout)_. It should honestly clear everything in your
application:
Databases, Files, Preferences and Runtime cache, and should be executed before each test.
!!! danger

    This solution is a bottleneck and it's better to avoid it at all. If LogoutCleaner is broken, all of the tests will be failed.

<br/>

##### Clear internal storage  <br/>

All cache in an android application is stored in the internal storage: `/data/data/packagename/`
<br/>This storage is our application sandbox and can be accessed without any permission.

Basic idea is to avoid using components from a real code base. Instead of them, use some tests rules which do the job
for us.

```kotlin

@get:Rule 
val clearPreferenceRule = ClearDatabaseRule()

@get:Rule 
val clearFilesRule = ClearFilesRule()

@get:Rule 
val clearFilesRule = ClearPreferencesRule()

```

They have already been implemented in [Barista](https://github.com/AdevintaSpain/Barista/) library, you can find
them [here](https://github.com/AdevintaSpain/Barista/tree/master/library/src/main/java/com/adevinta/android/barista/rule/cleardata)

!!! warning

    This solution won't in 100% of cases:

    1. You may have runtime cache, which can also affect your tests
    2. Test or application process may crash and prevent the launch of next tests

##### Conclusion<br/>

These are pros/cons for both solutions which don't kill the process:

➕ Fast implementation<br/>
➕ Fast execution in the same process<br/>
<br/>
➖ Don't give you any guarantee that your app will be cleared properly<br/>
➖ Application or Test process killing will break tests execution <br/>
➖ Can be a bottleneck<br/>

Use these solutions only as a temp workaround, because it won't work on perspective in huge projects

## 2. Clearing package data

Our aim is to simulate the same behavior as when user presses the `clear data` button in application settings.
<br/>Application process will be cleared in that case, our application will be started in a cold start.

##### Orchestrator

Basically, you can achieve an isolated state, if you execute your tests like this:

```bash
adb shell am instrument -c TestClass#method1 -w com.package.name/junitRunnerClass
adb pm clear
adb shell am instrument -c TestClass#method2 -w com.package.name/junitRunnerClass
adb pm clear
```

Each test should be executed in an isolated instrumented process and junit reports should be merged into a big one
report when all tests are finished.

That's the common idea of `Orchestrator`.
<br/>
It's just an `apk` which consist of
only [several classes](https://github.com/android/android-test/tree/master/runner/android_test_orchestrator/java/androidx/test/orchestrator)
and runs tests and clears data, as described above.

You should install an `orchestrator` along with `application.apk` and `instrumented.apk` on the device.

However, it's not the end.
<br/>
Orchestrator should somehow execute adb commands. Under the hood, it
uses [special services.](https://github.com/android/android-test/tree/master/services)
It's just a shell client and should be installed to the device.

![alt text](../images/orchestrator.png "orchestrator and test-services")

[An official documentation and guide how to start with Orchestrator](https://developer.android.com/training/testing/junit-runner#using-android-test-orchestrator)

!!! warning

    Despite the fact that it does the job, this solution looks overcomplicated:

    1. We need to install +2 different apk to each emulator
    2. We delegate this job to the device instead of host machine. 
    <br/>Devices are less reliable than host pc

##### Other solutions

It's also possible to clear package data by
using [3rd party test runners](https://android-ui-testing.github.io/Cookbook/practices/test_runners_review/), like
Marathon, Avito-Runner or Flank. Marathon and Avito-Runner clear package data without an orchestrator. They delegate
this logic to a host machine

##### Conclusion<br/>

These are pros/cons for an `orchestrator` and 3rd party test runners solution:

➕ Does the job for us in 100% <br/>
<br/>
➖ Slow execution _(can take 10+ seconds and depends on apk size)_ <br/>
➖ Orchestrator — over-complicated <br/>

Each `adb pm clear` takes some time and depends on apk size. Below you may see some gaps between the tests which
represent such a delay

![alt text](../images/package_clear.png "ADB package clearing takes some time")

!!! success

    Only package clear can guarantee that your data will be celared properly.
    Marathon and Avito-Runner provide the easiest way to clear application data.

    1. You can set them just by one flag in configuration
    2. They don't use orchestrator under the hood 




    
    
