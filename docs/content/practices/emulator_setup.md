# Emulator setup

Basically we have 2 choices:

* Manage devices automatically via `avd`
* Manage docker containers with emulators with `docker`

Using a docker image is the easiest way, however it's important to understand how docker creates emulators for you.

## Creating an emulator

Before starting to read this, make sure you've read
[the official documentation](https://developer.android.com/studio/run/emulator-commandline)

Firstly, you need to create an `ini` configuration file for the emulator:

```ini
PlayStore.enabled=false
abi.type=x86_64
avd.ini.encoding=UTF-8
hw.cpu.arch=x86_64
hw.cpu.ncore=2
hw.ramSize=2048
hw.lcd.density=120
hw.lcd.width=320
hw.lcd.height=480
hw.audioInput=no
hw.audioOutput=no
hw.accelerometer=no
hw.gyroscope=no
hw.dPad=no
hw.mainKeys=yes
hw.keyboard=no
hw.sensors.proximity=no
hw.sensors.magnetic_field=no
hw.sensors.orientation=no
hw.sensors.temperature=no
hw.sensors.light=no
hw.sensors.pressure=no
hw.sensors.humidity=no
hw.sensors.magnetic_field_uncalibrated=no
hw.sensors.gyroscope_uncalibrated=no
image.sysdir.1=system-images/android-29/google_apis/x86_64/
tag.display=Google APIs
tag.id=google_apis
skin.dynamic=yes
skin.name=320x480
disk.dataPartition.size=8G
```

Pay attention to what we have disabled:

* Accelerometer
* Audio input/output
* Play Store
* Sensors:Accelerometer, Humidity, Pressure, Light
* Gyroscope

We don't really need them for our test runs. It also may improve our tests performance, because there are no background
operations related to those tasks.

After that, you can run your emulator via `avd manager`, which is part of the android sdk manager. After creating the emulator, you need to switch the default generated `ini` file to the custom one we defined previously. 
You can achieve that with a script like this one:

```bash
function define_android_sdk_environment_if_needed() {
  android_env=$(printenv ANDROID_HOME)
  if [ -z "$android_env" ]; then
    if [[ -d "$HOME/Library/Android/sdk" ]]; then
      export ANDROID_HOME="$HOME/Library/Android/sdk"
    else
      printf "Can't locate your android sdk. Please set ANDROID_HOME manually"
      exit
    fi
  fi
}

function define_path_environment_if_needed() {
  if ! command -v adb &>/dev/null; then
    export PATH=~/Library/Android/sdk/tools:$PATH
    export PATH=~/Library/Android/sdk/platform-tools:$PATH
  fi
}

function create_and_patch_emulator() {
  EMULATOR_API_VERSION=29
  EMULATOR_NAME="ui_tests_emulator_api_${EMULATOR_API_VERSION}"

  # Install sdk
  $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager "system-images;android-${EMULATOR_API_VERSION};google_apis;x86_64"

  # Create new emulator
  echo "no" | $ANDROID_HOME/cmdline-tools/latest/bin/avdmanager create avd --force \
    --name "${EMULATOR_NAME}" \
    --package "system-images;android-${EMULATOR_API_VERSION};google_apis;x86_64" \
    --abi google_apis/x86_64

  # Change emulator's config after emulator's creation
  cp -p YOUR_INI_FILE.ini $HOME/.android/avd/${EMULATOR_NAME}.avd/config.ini

  # Run new emulator
  pushd ${ANDROID_HOME}/emulator/

  nohup ${ANDROID_HOME}/emulator/emulator @${EMULATOR_NAME} >/dev/null 2>&1 &
}

define_android_sdk_environment_if_needed
define_path_environment_if_needed
create_and_patch_emulator
```

Keep in mind that the emulator must fully boot before running any test. Otherwise the tests will fail because there is still no device ready
on which they can run.

### Summary
1. create an `ini` configuration file for the emulator
2. run your emulator via `avd manager` and 
3. switch the `ini` file generated in 2. with the one we create in 1.

## How to run an emulator in a Docker?

Running an emulator in a docker is way easier than manually, because it encapsulates all this logic. If you don't have
experience with docker, check
[this guide](https://www.youtube.com/watch?v=zJ6WbK9zFpI) to get familiarized with the basics.

There are some popular docker images already built for you:

* [Official Google emulator](https://github.com/google/android-emulator-container-scripts)
* [Agoda emulator](https://github.com/agoda-com/docker-emulator-android)
* [Avito emulator](https://hub.docker.com/r/avitotech/android-emulator-29)

Talking about the [Avito emulator](https://github.com/google/android-emulator-container-scripts), it also patches your
emulator with adb commands to prevent tests flakiness and to speed them up.

##### Run Avito emulator

```bash
#run emulator 1 in a headless mode
docker run -d -p 5555:5555 -p 5554:5554 -p 8554:8554 --privileged avitotech/android-emulator-29:915c1f20be
adb connect localhost:5555

#run emulator 2 in a headless mode
docker run -d -p 5557:5555 -p 5556:5554 -p 8555:8554 --privileged avitotech/android-emulator-29:915c1f20be
adb connect localhost:5557

#run emulator 3 in a headless mode
docker run -d -p 5559:5555 -p 5558:5554 -p 8556:8554 --privileged avitotech/android-emulator-29:915c1f20be
adb connect localhost:5559

#...etc
```

##### Stop all emulators

```shell
docker kill $(docker ps -q)
docker rm $(docker ps -a -q)
```

## Conclusion

* Use docker emulators </br>
  _You'll also have the opportunity to run them with `Kubernetes`, to make it scalable in the future_

* Start fresh emulators on each test batch and kill them after all of your tests finished</br>
  _Emulators tend to freeze and may not work properly after idling for some time_

* Use the same emulator locally as on your CI </br>
  _All devices are different. It can save you a lot of time with debugging and understanding why your test works locally
  and fails on CI. It won't be possible to run Docker emulators on macOS or Windows, because
  of [haxm#51](https://github.com/intel/haxm/issues/51#issuecomment-389731675). Use AVD to launch them on such
  machines (script above may help you)_

!!! warning

    To run an emulator on CI with a docker, make sure that nested virtualisation is supported and KVM is installed.
    You can check more details [here](https://developer.android.com/studio/run/emulator-acceleration#vm-linux)