# Emulator vs Real device

This question is a trade off and there is no right and wrong answers. We'll review pros/cons and basic emulator setup on
CI

## Real device

Here is pros/cons

➕ Real environment </br>
➕ Doesn't consume CI resources

➖ Breaks often </br>
➖ Requires special conditions </br>

A real device will help you to catch more bugs from the first perspective, however talking about scalability, if you
have a lot of devices, you need to locate them in a special room with no direct sunlight and with a climate-control.

However, it doesn't save from disk and battery deterioration, because they are always charging and performs I/O
operations. It may be a reason of your tests failing not because of them caught a real bug, but because of an issue with
a device.

## Emulator

Here is pros/cons

➕ Easy configurable </br>
➕ Can work faster than a real device </br>
_Keep in mind that it's achievable only if you applied a special configuration and have powerful build agents_</br>
➕ Тot demanding in maintenance </br>

➖ Not a real environment </br>
➖ Consumes CI resources </br>

The most benefit that we may have is a fresh emulator instance each test bunch. Also, it's possible to create a special
configuration and disable features you don't need to have in tests which affects device stability. However, you need to
have powerful machine (and definitely not one, if you want to run your tests on pull requests)