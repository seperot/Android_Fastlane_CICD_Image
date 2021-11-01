# Android_Fastlane_CICD_Image
The Docker image contains the Android SDK and most common packages necessary for building Android apps in a CI tool like GitLab CI (Android SDK, git, fastlane). Make sure your CI environment's caching works as expected, this greatly improves the build time, especially if you use multiple build jobs.

This has been updated for Android 12 using Open JDK 11

A `.gitlab-ci.yml` with caching of your project's dependencies would look like this:

```
image: ijhdev/gitlab-ci-fastlane-android

variables:
  ANDROID_COMPILE_SDK: "31"
  ANDROID_BUILD_TOOLS: "30.0.3"
  ANDROID_SDK_TOOLS:   "7583922"
  LC_ALL: "en_US.UTF-8"
  LANG: "en_US.UTF-8"
  GIT_STRATEGY: clone

before_script:
  - export GRADLE_USER_HOME=$(pwd)/.gradle
  - chmod +x ./gradlew

cache:
  key: ${CI_PROJECT_ID}
  paths:
    - .gradle/

stages:
  - unit_test
  - debug_build


unit_test:
  tags:
    - your_build_runner
  dependencies: []
  stage: unit_test
  artifacts:
    paths:
      - fastlane/screenshots
      - fastlane/logs
    expire_in: 1 hour
  script:
    - fastlane tests

debug_build:
  tags:
    - your_build_runner
  dependencies: []
  stage: debug_build
  artifacts:
    paths:
      - app/build/outputs/
    expire_in: 1 week
  script:
    - bash ./version_updater.sh
    - fastlane yourDebug
```

The version updater takes your Google Play store listing and checks it against your Gradle version. If your Gradle version number is higher than the app store it will stick with it, if it’s equal or lower that the store version then it will take that, +1 to the end, then change the Gradle version for that one.

All you need to do is take the version_updater.sh file from here and put it in the root folder of your app and edit this line to have your bundle ID

 ```
curl -O playstorepage https://play.google.com/store/apps/details\?id\=<YourBundleID>
```
