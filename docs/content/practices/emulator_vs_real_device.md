# Emulator vs Real device
Instrumented tests can run either on emulators or real devices. Which one should we used?
This question is a trade off and there is no right or wrong answer. We'll review the pros and cons of both approaches.

## Real device

Here the pros/cons

➕ Real environment </br>
➕ Doesn't consume significant RAM, CPU and memory of the CI.

➖ Breaks often: Battery swells, USB port failures, OS software failing... all this happens because real devices are not designed to be intensively used continuously. </br>
➖ Requires to place them in a room with special environment conditions </br>

Although it seems that a real device is a better alternative because it helps you catch bugs on a full-fledged Android environment, it comes with its own issues. However, talking about scalability, if you
have a lot of devices, you need to locate them in a special room with no direct sunlight and with a climate-control.

But that doesn't prevent them from disk and battery deterioration, because they are always charging and performing I/O
operations. Therefore, if your tests fail, could be because of an issue with
a device and not because of a real bug in the app under test.

## Emulator

Here the pros/cons

➕ Easy configurable </br>
➕ Can work faster than a real device</br>
_Keep in mind that it's achievable only if you applied a special configuration and have powerful build agents_</br>
➕ Not demanding in hardware maintenance e.g. battery, disk, USB ports, display...) </br>

➖ Not the real environment on which the app will end up running</br>
➖ Consumes significant resources of the CI like RAM, CPU and memory</br>
➖ Emulators might freeze if stay idle for a long time and need to be restarted.</br>

The main benefit that we may have is a fresh emulator instance on each test run. Also, it's possible to create a special
configuration and disable features you don't need to have in tests, like sensor or audio input/output, which affect device stability. Nevertheless, you need
a powerful machine (and definitely not one, if you want to run your instrumented tests on very pull requests)