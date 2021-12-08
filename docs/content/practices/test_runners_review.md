# Test runners

Test runner is a test component responsible for.
1. Preparing the test runs
2. Providing test results

`AndroidJunitRunner` — The official solution and low-level instrument. It requires a lot of effort from engineers to run
tests on the CI and make them stable.

It's worth mentioning that UI testing tools are getting better year by year. However, some basic functionality still doesn't work out
of the box properly.

### 1. Problems with AndroidJunitRunner:

* Overcomplicated solution with state clearing
  <br>
  _It would be good to do the state clearing of the application by setting a flag.
  <br>
  It exists, however to make it scalable on the CI as well as the opportunity to use filters, you still have to
  install `test-services.apk` and `orchestrator.apk` on each device manually_


* Impossibility to scale
  <br>
  _As soon as you started your tests, it's impossible to let more devices join the test execution on the fly_

* Impossibility to prevent flakiness
  <br>
  _Flakiness is one of the main problems in instrumented testing. Test runners should provide some mechanisms to fight against 
  flakiness, like support to retry failing tests among other strategies_

* Impossibility to validate flakiness
  <br>
  _Flakiness can be validated by running each test multiple times. If the test passes every single time it runs, it's not flaky. It would be
  great to launch each test 100 times by one command_

* Poor test report
  <br>
  _The default test report doesn't show enough valuable information for each test. As an engineer, I want to see a video of the test and its logs. Moreover, I'd like to know whether
  the test has been retried. If yes, I'd also like to see how many retries, and their corresponding videos and logs_

* Impossibility to retry
  <br>
  _It's only possible to do it via a special test rule which does the retries for us. However, it's up to the test runner to retry each
  test. That's because: instrumented process might have crashed and device less reliable than host machine._
  1. *The instrumented process might have crashed*. In this case the test rule may not execute the code where the retry actually happens.
  2. **
  <br>
  _Also, it should be possible to define maximum retry count: Imagine, your application reliably crashes on start, and you have plenty of tests executing that code. We shouldn't
  retry each test in that case: we would overload build agents on the CI with tests that are doomed to fail._


* Impossibility to record a video
  <br>
  _It's possible to achieve and implement manually. However, it would be really great to have such a functionality already built-in_

Almost all of those problems can be solved, but it can take weeks or even months of your time. Beside running tests,
you also need to care about writing tests which is challenging as well.
<br>
Having those problems solved for you lets you focus on other tasks.

### 2. Open source test runners

All of them use `AndroidJunitRunner` under the hood, as it's the only possibility to run instrumented tests.

#### [:green_square: 2.1 Marathon](https://github.com/MarathonLabs/marathon)

Powerful and the most pragmatic test runner. All you need to do it's just to connect devices to `adb`, and Marathon will
do the whole job for you.

➕ CLI, Gradle Plugin <br>
➕ Easy data clearing _(without an Orchestrator)_ <br>
➕ Flexible configuration with filters <br>
➕ Flakiness strategies <br>
➕ Dynamic test batching (test count/test duration) <br>
➕ Smart retries with a quotas <br>
➕ Screenshots & video out of the box <br>
➕ Improved test report with video & logs <br> 
➕ Automatically rebalanced test execution if connecting/disconnecting devices on the fly <br>
➕ Pull files from the device after test run,
e.g. [allure-kotlin](https://github.com/allure-framework/allure-kotlin) <br>
➕ Basic [Allure](https://github.com/allure-framework) support out of the box <br> 
➕ adb client `ddmlib` replacement: [Adam](https://github.com/Malinskiy/adam) <br>
➕ Cross-platform (iOS support) <br>
➕ Fragmented test execution (similar to AOSP's sharding): split large testing suites into multiple CI builds <br>
➕ Parallel execution of parameterised tests <br>
➕ Interactions with adb/emulator from within a test (e.g. fake fingerprint or GPS) <br> 
➕ Code coverage support <br>
➕ Testing multi-module projects in one test run <br> 
➕ Flakiness fixing mode to verify test passing probability improvements <br>

➖ Doesn't auto-scale devices <br>
_(Marathon will utilise more devices in runtime if some other system connects more to the adb, but marathon itself will
not spawn more emulators for you)_<br>
➖ HTML report doesn't contain test retry information (but the Allure report does) <br>
➖ For complex test executions that solve test flakiness requires to install TSDB (InfluxDB or Graphite) <br>

[Documentation](https://marathonlabs.github.io/marathon/)

#### [:green_square: 2.2 Avito Test Runner](https://github.com/avito-tech/avito-android/tree/develop/subprojects/test-runner)

Powerful test runner. Works directly with `Kubernetes`

➕ Easy data clearing _(without an Orchestrator)_ <br>
➕ Auto-scaling on fly _(There is a coroutine in the background which tries to connect more devices)_
➕ Retries 

➖ Complicated adoption <br>

This test runner has been used by Avito for 4+ years and runs thousands of tests every day. It's not as powerful
as Marathon, however it doesn't have an analogue in terms of auto-scaling out of the box.<br>
If you want to run your UI tests on pull requests in a large teams, this test runner is one of the best option.

Engineers from Avito are ready to help with adoption. You can reach out to [Dmitriy Voronin](https://github.com/dsvoronin)

[Documentation](https://avito-tech.github.io/avito-android/test_runner/TestRunner/)

#### [:green_square: 2.3 Fork](https://github.com/shazam/fork)

//To be done

#### [:green_square: 2.4 Flank](https://github.com/Flank/flank)

//To be done

#### [:green_square: 2.5 Flade](https://github.com/runningcode/fladle)

//To be done

#### [:red_square: 2.6 Spoon](https://github.com/square/spoon)

Deprecated and not maintained anymore. Do not use it

#### [:red_square: 2.7 Composer](https://github.com/gojuno/composer)

Deprecated and not maintained anymore. Do not use it