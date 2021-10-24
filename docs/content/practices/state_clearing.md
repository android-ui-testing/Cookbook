# Clearing state

This question appears as soon as you need to run more than 1 ui test.

## Problem:

We run `Test1`, it executes some http calls, saves some data to files and database.
<br/>When `Test1` finished, `Test2` will be launched.
<br/>However, `Test1` left some data on the device which can be a reason of `Test2` failing.

Solution — clear data before each test

## 1. Within process clearing

In this case we don't kill our application process.

##### 1.1 Component from a real code base <br/>

``` kotlin
@Before 
fun setUp() {
   DI.provideLogoutCleanerInteractor().clear() 
}
```

The same component which does clearing data _(For instance, while logout)_. It's should honesty clear everything in
application: Databases, Files, Preferences and Runtime cache and should be called before each test.
!!! danger

    This solution is a bottleneck and it's better to avoid it at all. If LogoutCleaner broken, all of the tests will be failed

<br/>

##### 1.2 Clear internal storage  <br/>

All cache in android application stored in internal storage: `/data/data/packagename/`
<br/>This storage — our application sandbox and can be achieved without any permission.

Common idea: avoid usage of components from real code base and use some test rules, which do the job for us.

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

    This solution won't work on 100% for you:

    1. You may have runtime cache, which is also can affect your tests
    2. Test or application process can be crashed, which prevents launching of the next tests

##### 1.3 Conclusion<br/>

➕ Fast implementation<br/>
➕ Fast execution in the same process<br/>
<br/>
➖ Don't have any guarantee that your app will be cleared properly<br/>
➖ Application or Test process killing will break tests execution <br/>
➖ Can be a bottleneck<br/>

Use these solutions only as a temp workaround, because it won't work on perspective in a huge projects

## 2. Clear package data

Idea — simulate the same behavior when user press `clear data` in application settings.
<br/>Application process will be cleared in that state, our application will be started in a cold start.

##### 2.1 Orchestrator

Basically is it possible to have cleared state, if you are executing your tests like this:

```bash
adb shell am instrument -c TestClass#method1 -w com.package.name/junitRunnerClass
adb pm clear
adb shell am instrument -c TestClass#method2 -w com.package.name/junitRunnerClass
adb pm clear
```

Each test should be executed in isolated instrumented process and junit reports should be merged into a big one when all
tests finished.

That's the common idea of `Orchestrator`.
<br/>
It's just an `apk` which consist of
only [several classes](https://github.com/android/android-test/tree/master/runner/android_test_orchestrator/java/androidx/test/orchestrator)
and which does test running and clearing for us as it's described above.

Besides `application.apk` and `instrumented.apk`, orchestrator should be installed on the device.

However, it's not the end.
<br/>
Orchestrator should somehow execute adb commands. Under the hood, it
uses [special services](https://github.com/android/android-test/tree/master/services)
It's just shell client which is also represented as an `apk` and should be installed to the device.

Full picture:

//TODO

[An official documentation and guide how to start with Orchestrator](https://developer.android.com/training/testing/junit-runner#using-android-test-orchestrator)

!!! warning

    Despite the fact that it does the job, this solution looks overcomplicated:

    1. We need to install +2 different apk to each emulator
    2. We delegate this job to the device instead of host machine. 
    <br/>Devices are less reliable than host pc

##### 2.2 Other solutions

This is also possible to implement by usage
of [3rd party test runners](https://android-ui-testing.github.io/Cookbook/practices/test_runners_review/), like
Marathon, Avito-Runner or Flank. Marathon and Avito-Runner clear package data without orchestrator. They delegated this
logic to host machine.

##### 1.3 Conclusion<br/>

➕ Does the job for us in 100% <br/>
<br/>
➖ Slow execution _(can take 10+ seconds and depends on apk size)_ <br/>
➖ Orchestrator — over-complicated <br/>

//TODO Test report sample

!!! success
     

    Clear package data is more reliable, consider to use it instead of within process clearing.
    
    Marathon and Avito-Runner provide the easiest way to clear application data.

    1. You can set it just by one flag
    2. They don't use orchestrator under the hood



    
    



