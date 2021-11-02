# Emulator vs Real device

This question is a trade off and there is no right and wrong answers. We'll review pros/cons and basic emulator setup on
CI

## Real device

Here is pros/cons

➕ Real environment </br>
➕ Doesn't consume CI resources

➖ Breaks often </br>
➖ Works slow </br>
➖ Requires special conditions </br>

A real device will help you to catch more bugs from the first perspective, however talking about scalability, if you
have a lot of devices, you need to locate them in a special room with no direct sunlight and with a climate-control.

However, it doesn't save from disk and battery deterioration, because they are always charging and performs I/O
operations. It may be a reason of your tests failing not because of them caught a real bug, but because of an issue with
a device.

## Emulator

Here is pros/cons

➕ Easy configurable </br>
➕ Works faster </br>
➕ Тot demanding in maintenance </br>

➖ Not a real environment </br>
➖ Consumes CI resources </br>

The most benefit that we may have fresh emulator instance each test bunch. Also, it's possible to create a special
configuration and disable features you don't need to have in tests which affects device stability. However, you need to
have powerful machine (and definitely not one, if you want to run your tests on pull requests).

## How to create an emulator?

Firstly, you need to define an `ini` configuration:

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

Pay your attention that we
disabled `play store, sensors, accelerometer, humidity, pressure, magnetic_field_uncalibrated, gyroscope_uncalibrated`
because we don't really need them for our tests run. It also may improve our tests performance, because there are no
background operations related to that tasks.

After that, you can run your emulator by `avd manager`, which is a part of android sdk manager.

You may see an example below:

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

  # Apply emulator's config
  cp -p YOUR_INI_FILE.ini $HOME/.android/avd/${EMULATOR_NAME}.avd/config.ini

  # Run new emulator
  pushd ${ANDROID_HOME}/emulator/

  nohup ${ANDROID_HOME}/emulator/emulator @${EMULATOR_NAME} >/dev/null 2>&1 &
}

define_android_sdk_environment_if_needed
define_path_environment_if_needed
create_and_patch_emulator
```

Pay your attention that you also need to wait until your emulator is fully booted.

## How to run an emulator in a Docker?

Running an emulator in a docker a way easier than manually. If you don't have an experience with docker, you can check
[this guide](https://www.youtube.com/watch?v=zJ6WbK9zFpI) to check the basics.

There are some popular already built docker images for you:

* [Official Google emulator](https://github.com/google/android-emulator-container-scripts)
* [Agoda emulator](https://github.com/agoda-com/docker-emulator-android)
* [Avito emulator](https://hub.docker.com/r/avitotech/android-emulator-29)

Talking about [Avito emulator](https://github.com/google/android-emulator-container-scripts), it also patches your
emulator with adb commands to prevent tests flakiness and to speed them up

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

## Host emulators on different machines

//TODO