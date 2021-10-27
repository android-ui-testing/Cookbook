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

//To be done

#### [:green_square: 2.2 Avito Test Runner](https://github.com/avito-tech/avito-android)

//To be done

#### [:green_square: 2.3 Fork](https://github.com/shazam/fork)

//To be done

#### [:green_square: 2.4 Flank](https://github.com/Flank/flank)

//To be done

#### [:red_square: 2.5 Spoon](https://github.com/square/spoon)

Has been deprecated and not maintained anymore

#### [:red_square: 2.6 Composer](https://github.com/gojuno/composer)

Has been deprecated and not maintained anymore




