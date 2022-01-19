# Companies experience

Looking to how other companies implemented `UI/Snapshot testing` may help you to choose proper tools for your project
and save your time.

We would be really grateful if you
could [contribute and share your experience](https://android-ui-testing.github.io/Cookbook/home/contribution) directly
to this page, to help other people.

## Revolut

`UI testing`

:   
`Write:` [Kaspresso](https://github.com/KasperskyLab/Kaspresso) <br>
`Who write:` Android Engineers<br>
`Runner:` [Marathon](https://android-ui-testing.github.io/Cookbook/practices/test_runners_review/#21-marathon) locally and on the CI <br>
`Where:` [Headless emulators in Docker (Avito Image)](https://hub.docker.com/r/avitotech/android-emulator-29) <br>
`How often:` Each 4h and before each release <br>
`Network:` Mock, by Custom [OkReplay](https://github.com/airbnb/okreplay) <br>
`Test report:` [Allure](http://allure.qatools.ru/)<br>

: `Other:` We use custom `OkReplay` to achieve requests indexing, and the same request time as it was while recording.
We're going to open-source this solution

`Snapshot testing`

:   `Tools:` [Screenshot tests for Android](https://github.com/facebook/screenshot-tests-for-android) <br>
`How often to run:`  Each commit to the Design System module

: `Other:` We write snapshot tests per each component in the design system in all possible states, we don't write them
for screens which implemented by using that components.



## Kaspersky

`UI testing`


: `Write:` [Kaspresso](https://github.com/KasperskyLab/Kaspresso) <br>
`Who write:` QA and developers<br>
`Runner:` AndroidJUnitRunner locally and on CI we use [Marathon](https://github.com/MarathonLabs/marathon) + custom tooling on top <br>
`Where:` On CI: Emulators (we use custom Docker container) and real devices (custom integration with [STF](https://github.com/openstf/stf)) <br>
`How often:` Each pull request (functional tests), before the release (e2e tests) and nightly (e2e tests) <br>
`Test report:` On CI we use custom internal solution<br>

`Snapshot testing`

:   `Tools:` [Kaspresso](https://github.com/KasperskyLab/Kaspresso) <br>
`How often:` Many times per new feature to check new strings and translations



## Check24

`UI testing`


: `Write:` [Kaspresso](https://github.com/KasperskyLab/Kaspresso) <br>
`Who write:` developers<br>
`Runner:` AndroidJUnitRunner with Android Orchestrator <br>
`Where:` On CI: real devices <br>
`How often:` at noon and at night <br>
`Test report:` Junit4 <br>

`Snapshot testing`

:   `Tools:` [Shot](https://github.com/pedrovgs/Shot) <br>
`Who write:` developers<br>
`Runner:` TestButler + Composer for test sharding (Shot support out of the box) <br>
`Where:` On CI: emulators <br>
`How often:` On every PR <br>
`Test report:` Shot report, which includes image diffs when tests fail <br>

## Headhunter (hh.ru)

`UI testing`

: `Write:` [Kaspresso](https://github.com/KasperskyLab/Kaspresso) wrapped with custom DSL for creating a test data  <br> 
`Who writes:` QA with support of Android Engineers <br>
`Runner:` [Marathon](https://android-ui-testing.github.io/Cookbook/practices/test_runners_review/#21-marathon) on the CI <br>
`Where:` Headless emulators in Docker (Custom Image) at k8s <br>
`How often:` Every night on every Portfolio branch(protected branch for each business feature) and develop; Every PR to develop.<br> 
`Test data:` End2End testing with test stands<br> 
`Test report:` [Allure](http://allure.qatools.ru/)<br>
`Test stability monitoring:` Custom tool for success rate visualization of each test between CI runs; [Grafana](https://grafana.com/) for common graphs.

## Delivery Club

`UI testing`

:   
`Write:` [Kaspresso](https://github.com/KasperskyLab/Kaspresso) <br>
`Who:` QA and developers <br>
`Runner:` [Delivery Club fork of Avito Runner](https://github.com/materkey/avito-android/tree/dc-fresh), [Argo Workflows](https://argoproj.github.io/workflows/) <br>
`Where:` [Fork of Avito Emulator](https://github.com/materkey/avito-android/tree/dc-fresh/ci/docker), [DockerHub](https://hub.docker.com/repository/docker/materkey/android-emulator-29) <br>
`How often:` Each commit for Courier App, Nightly for Consumer App, Before regress testing <br>
`Network:` [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) <br>
`Test report:` [Kaspresso Allure Integration + Avito Runner Integration](https://github.com/materkey/avito-android/blob/ea458699701727fc0bfd2cb2022e25539e0746e1/subprojects/test-runner/client/src/main/kotlin/com/avito/runner/finalizer/action/WriteAllureReportAction.kt) <br>
`Other:` [Run Marathon in cloud](https://github.com/materkey/run-android-ui-tests-in-cloud) <br>

## Auto.ru

`UI testing & Screenshot testing`

:   
`Write:` Espresso, [Screenshot tests from facebook](https://github.com/facebook/screenshot-tests-for-android)<br>
`Who:` Android Developers and QA Engineers <br>
`Runner:` Custom runner based on Android Orchestrator <br>
`Where:` Emulators <br>
`How often:` Each commit, nightly and before release <br>
`Network:` [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) <br>
`Test report:` [Allure](http://allure.qatools.ru/)<br>
`Test monitoring:` Collecting allure info in Postgres db and displaying it in [DataLens](https://cloud.yandex.com/en/services/datalens), finding flaky packages, common errors, alerts, etc. <br>

## CFT

`UI testing`

:   
`Write:`[Kaspresso](https://github.com/KasperskyLab/Kaspresso)<br>
`Who:` Android Developers and QA Engineers <br>
`Runner:` [Marathon](https://android-ui-testing.github.io/Cookbook/practices/test_runners_review/#21-marathon) on the CI <br>
`Where:` Emulators <br>
`How often:` On every PR and one time per day on Main branch <br>
`Network:` Custom mockapi server <br>
`Test report:` [Allure](http://allure.qatools.ru/)<br>
`Test monitoring:`Using Allure reports and Grafana monitoring for stable, resources work` <br>

## Ситимобил

`UI testing`

:
`Write:` [Kaspresso](https://github.com/KasperskyLab/Kaspresso)<br>
`Who writes:` QA Automation and QA Engineers<br>
`Runner:` [Marathon](https://android-ui-testing.github.io/Cookbook/practices/test_runners_review/#21-marathon) on the CI <br>
`Where:` Emulators <br>
`How often:` Each merge request <br>
`Test report:` [Allure](http://allure.qatools.ru/)<br>
`Network:` Custom mock system <br>