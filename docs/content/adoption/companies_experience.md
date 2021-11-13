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
`Who:` Android Engineers<br>
`Runner:` [Marathon](https://android-ui-testing.github.io/Cookbook/practices/test_runners_review/#21-marathon) <br>
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



