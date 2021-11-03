# Test runners

Test runner is responsible for tests run and providing test result for us.

`AndroidJunitRunner` — Official solution and low-level instrument. It requires a lot of effort from engineers to run
tests on CI and make them stable.

It's worth to mention — tools are getting better year by year. However, some basic functionality still doesn't work from
the box properly.

### 1. Problems with AndroidJunitRunner:

* Overcomplicated solution with clearing
  <br>
  _It would be good to have only one flag which does application clearing for us.
  <br>
  It exists, however to have scalability on CI and opportunity to use filters, you still have to
  install `test-services.apk` and `orchestrator.apk` to each device manually_


* Impossibility to scale
  <br>
  _As soon as you started your tests, it's impossible to connect more devices to tests run on fly_

* Impossibility to prevent flakiness
  <br>
  _Flakiness is one of the main problems in instrumented testing. Test runner should play role as a latest flakiness
  protection level, like support retries from the box or other strategies_

* Impossibility to validate flakiness
  <br>
  _Flakiness can be validated by running each test multiple times, and if test pass N/N, it's not flaky. It would be
  great to launch each test 100 times by one command_

* Poor test report
  <br>
  _Default test report doesn't show any useful information. As an engineer, I want to see a video of the test, logs and
  to make sure that test hasn't been retried. Otherwise, I'd like to see retries and each retry video and logs._

* Impossibility to retry
  <br>
  _It's possible to do only via special test rule which does retry for us. However, it's up to test runner to retry each
  test, as instrumented process can be crashed and device less reliable than host machine._
  <br>
  _Also, it should be possible to define maximum retry count: Imagine, your application crashed on start. We shouldn't
  retry each test in that case because there is no sense to overload build agents on CI._


* Impossibility to record a video
  <br>
  _It's possible to achieve and implement manually, however It would be really great to have such functionality from the
  box_

Almost all of that problems possible to solve, but it can take weeks or even months of your time. Beside running tests,
you also need to care about writing tests which is challenging as well.
<br>
It would be great to have that problems solved from the box

### 2. Open source test runners

All of them used `AndroidJunitRunner` under the hood, as it's the only possibility tun run instrumented tests.

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
➕ Automatically rebalanced test execution if connecting/disconnecting devices on fly <br>
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
➖ For complex test executions that solve test flakiness requires an installation of TSDB (InfluxDB or Graphite) <br>

[Documentation](https://marathonlabs.github.io/marathon/)

#### [:green_square: 2.2 Avito Test Runner](https://github.com/avito-tech/avito-android/tree/develop/subprojects/test-runner)

Powerful test runner. Works directly with `Kubernetes`

➕ Easy data clearing _(without an Orchestrator)_ <br>
➕ Auto-scaling on fly _(There is a coroutine in the background which tries to connect more devices)_
➕ Retries

➖ Complicated adoption <br>

This test runner has been using by Avito company for 4+ years and runs thousands tests every day. It's not as powerful
as Marathon, however it doesn't have an analogue in terms of auto scaling from the box.<br>
If you want to run your UI tests on pull requests in a large teams, this test runner is one of the best option.

Engineers from Avito are ready to help with adoption. You can contact to [Dmitriy Voronin](https://github.com/dsvoronin)

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