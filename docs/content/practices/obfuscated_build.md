# Testing obfuscated build

How to test the closest possible to production build.

## Problem

We work on debug builds and most often see debug builds and our UI tests run on debug builds. But user has release build. It is the same build apart from optimisations we do to reduce binary size and protect our apps identity. Any problems related to these optimisations are very rare but still would be good to catch them even before they hit beta.

## Solution

Run UI tests on obfuscated build. For that we need to use [keeper](https://slackhq.github.io/keeper/) plugin. The reason is that Android Gradle Plugin doesn't include usages from `androidTest` sources and will throw out all code referenced by UI tests. So they won't work.

We can use Keeper in two ways:

### Run UI tests on release build 

Can be done by adding following to `build.gradle`:

```
android {
    testBuildType = "release"
}
```
or rather:
```
android {
    if (hasProperty("testingMinimizedBuild")) {
        testBuildType = "release"
    }
}
```

But this has downside: sometimes you separate code via build type folders. E.g. place dummy implementation in `/debug/` and real implementation into `/release/` source set to make sure debug code never gets into production builds. Or the same principle applied to dependencies like:

```
debugImplementation 'com.facebook.flipper:flipper:0.154.0'
debugImplementation 'com.facebook.soloader:soloader:0.10.1'
releaseImplementation 'com.facebook.flipper:flipper-noop:0.154.0'
```

If you have such configurations, it's not a way to go.

### Run obfuscation on debug build.

Instead of running tests on release build type, we run them as usual on debug, but apply obfuscation to debug build via:

```
buildTypes {
    debug {
        ...
        if (hasProperty("testingMinimizedBuild")) {
            isMinifyEnabled = hasProperty("testingMinimizedBuild")
            isShrinkResources = hasProperty("testingMinimizedBuild")
            proguardFiles 'proguard-rules.pro'
        }
    }
}
```

### Main trick

Once we have build and minimize for tests we need to keep all needed classes. To do so we apply [keeper](https://slackhq.github.io/keeper/) plugin:

```
if (hasProperty("testingMinimizedBuild")) {
  apply plugin: "com.slack.keeper"
}
```

As you noted we do everything if some property(eg `hasProperty("testingMinimizedBuild")`). This way we can run UI tests normally and to run tests on obfuscated build.

To pass param to the build:

```
./gradlew assembleDebugAndroidTest -PtestingMinimizedBuild
```

### Other tricks

#### R8 repo

Keeper adds R8 repo on project level so if your project uses 

```
repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)
```

it will fail the build. What you need to do is to tell keeper not to add any repos and do it yourself:

```
keeper {
  automaticR8RepoManagement = false
}
...
repositories {
    ...
    maven { setUrl("http://storage.googleapis.com/r8-releases/raw") }
}
```

#### Memory and time

Obviously build will take longer time depending on project size. But you also need to increase heap memory for JVM, otherwise you'll get lots OOMs. 

You can either do it in gradle.properties file:
```
org.gradle.jvmargs=-Xmx16G -XX:+UseParallelGC -Dfile.encoding=UTF-8
```

Or to give more memory only for those runs(your final command may look like):

```
./gradlew assembleDebugAndroidTest -PtestingMinimizedBuild "-Dorg.gradle.jvmargs=-Xmx16G -XX:+UseParallelGC" -Dfile.encoding=UTF-8
```

*Note*: double quotes for `"-Dorg.gradle.jvmargs=-Xmx16G -XX:+UseParallelGC"` otherwise gradle may be unhappy with incorrect `org.gradle.jvmargs`.

#### Additional Proguard rules

Depending on your UI tests you may want to disable obfuscation of certain classes in addition to you main Proguard rules so that your test code can find needed stuff.

`proguard-debug-r8.pro`

```
# Make UI tests able to find needed stuff.
-keep class org.yaml.** { *; }
-keep class okreplay.** { *; }
-keepattributes InnerClasses
-keep class **.R
-keep class **.R$* {
    <fields>;
}
```

And in:

```
buildTypes {
    release {
        minifyEnabled true
        proguardFiles 'proguard-rules.pro'
    }
    release {
        if (hasProperty("testingMinimizedBuild")) {
            minifyEnabled true
            proguardFiles 'proguard-rules.pro', 'proguard-debug-r8.pro' // here we extend proguard with our test specific rules file
        }
    }
}
```

#### AGP version

If your Android Gradle Plugin version is less than `7.1.0` than you need not the latest version of keeper. You need `0.11.2`.

This is because of new gradle API through which you apply the plugin.

Also on different versions of AGP work different R8. If something doesn't work(you see some `PrintUses` stack trace) you may want to try new R8 `TraceReferences` API(worked for us on AGP 7.1.+):

```
keeper {
  traceReferences()
}
```

Otherwise you may want to try different version of R8. Look for tags [here](https://r8.googlesource.com/r8/). More [here](https://slackhq.github.io/keeper/configuration/#custom-r8-behavior).


### Further reading:

1. [Keeper advanced configuration](https://slackhq.github.io/keeper/configuration/) and reading source code.
2. [Testing minimized build at Avito](https://avito-tech.github.io/avito-android/blog/2020/03/testing-a-minimized-build/)




    
    
