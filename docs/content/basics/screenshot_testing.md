# Screenshot testing

## What it is
Screenshot testing (also called Snapshot testing) has been in the Android world since 8th October 2015, when Facebook open sourced the first version of their snapshot testing library.

They are a special type of UI tests that inflate a view, take a screenshot of it, and compare it to an already stored image taken as reference. This reference is considered the source of truth: it depicts how the Ui must be displayed, pixel by pixel.
If the generated snapshot file from the test differs from the reference, the test fails, passes otherwise.

## Motivation
Screenshot tests are easy to write and maintain and run fast (≈ 1 sec per test), what makes them affordable to execute upon PRs.
They focus on detecting visual bugs that standard UiAutomator and Espresso cannot, namely
1. Ui defects introduced by updates in libraries like Material design and Constraint Layout
2. Spacing, styles and themes (light mode vs. dark mode)
3. Layout correctness:
  - Multiline text does not cut off
  - No view overlapping on visibility changes
  - Proper view alignment under different screen densities and/or font sizes
  - Text, icons and Ui alignment on RTL & LTR languages, among other language related issues

## How it works
First of all, we need to understand that snapshot testing process differs from standard testing.
Most snapshot testing frameworks provide two main tasks:

1. **Record**: Generates a snapshot file. this file will be reviewed by our peers, and once approved, uploaded to the CI as our source of truth. All further verification fo that test will be made against it.
2. **Verify**: When executed, it generates a snapshot file for each test that will be compared, pixel by pixel, with the one taken as reference when recording.

--- more to write here

## Challenges
--- Draft 
1. Flakiness (name them)
2. Emulator configuration in all parts of involved (elaborate)

## Frameworks
--- Draft
1. Instrumented
   - Facebook
   - Shot
   - Testify

2. JVM
   - Paparazzi

## Conclusion
--- Draft
 



