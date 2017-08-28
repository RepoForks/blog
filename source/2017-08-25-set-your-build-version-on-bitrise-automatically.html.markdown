---
title: Set your build version on Bitrise automatically
date: 2017-08-25 11:01 UTC
tags: how-to, android, ios
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/versioning_blog.jpg
---


You have already setup your app on Bitrise, and now you are ready to start to deploy the app to your users or QA. There might be several versions going out and it might cause you some headache if you cannot identify these versions easily. Why have conflicting merges and misunderstandings and long conversations on who did or didn't fix what or when... You can easily refer to the exact build on Bitrise if you get an error report from a tester.
Differentiate the versions to know who uses which and to make sure everybody uses the up-to-date app!
Use (semi)automated version control on Bitrise too! 🤓


>In software versioning a typical version identifier consists of 3 numbers, divided by full stops, e.g.: 3.4.2 (major.minor.build)
Numbers are generally assigned in increasing order and correspond to new developments in the software.


Versions should be tracked in git repository. You can set version numbers in Info.plist, and you can update them manually there too. It is advised to do set the major and minor numbers by hand to be able to manage versions as you can decide which is a bigger or a smaller update. As for the build numbers, in a CI/CD environment it is useful to update the build while the build is happening.

On Bitrise there are steps to do just that: `Set Xcode Project Build Number` for iOS and `Set Android Manifest Version Code and Name` for Android. These steps insert the build number into the info.plist. If you did track versions otherwise earlier, you can offset these numbers to your existing numbering, so you don't have to start it all over again.

##Android 🤖
This is how it looks in reality for Android:

<pre><code>---
format_version: '4'
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
project_type: android
trigger_map:
- push_branch: "*"
  workflow: primary
- pull_request_source_branch: "*"
  workflow: primary
app:
  envs:
  - opts:
      is_expand: false
    GRADLE_BUILD_FILE_PATH: build.gradle
  - opts:
      is_expand: false
    GRADLEW_PATH: "./gradlew"
workflows:
  deploy:
    steps:
    - activate-ssh-key@3.1.1:
        run_if: '{{getenv "SSH_RSA_PRIVATE_KEY" | ne ""}}'
    - git-clone@3.5.1: {}
    - cache-pull@1.0.0: {}
    - script@1.1.4:
        title: Do anything with Script step
    - install-missing-android-tools@2.0.1: {}
    - set-android-manifest-versions@1.0.5:
        inputs:
        - manifest_file: "$BITRISE_SOURCE_DIR/app/AndroidManifest.xml"
        - version_code_offset: '0'
        - version_name: 1.0.0
    - gradle-runner@1.7.6:
        inputs:
        - gradle_file: "$GRADLE_BUILD_FILE_PATH"
        - gradle_task: assembleRelease
        - gradlew_path: "$GRADLEW_PATH"
    - deploy-to-bitrise-io@1.3.6: {}
    - cache-push@1.1.4: {}
</code></pre>

##iOS 📱
... and for iOS:

<pre><code>---
format_version: '3'
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
project_type: ios
trigger_map:
- push_branch: "*"
  workflow: primary
- pull_request_source_branch: "*"
  workflow: primary
workflows:
  primary:
    steps:
    - activate-ssh-key@3.1.1:
        run_if: '{{getenv "SSH_RSA_PRIVATE_KEY" | ne ""}}'
    - git-clone@3.5.1: {}
    - certificate-and-profile-installer@1.8.8: {}
    - set-xcode-build-number@1.0.5:
        inputs:
        - plist_path: "$BITRISE_SOURCE_DIR/greg/Info.plist"
        - build_version_offset: '0'
        - build_short_version_string: '1.0'
    - xcode-archive@2.1.0:
        inputs:
        - export_method: development
        - configuration: Debug
    - export-xcarchive@0.9.2:
        inputs:
        - export_method: app-store
        - team_id: TeamID
app:
  envs:
  - opts:
      is_expand: false
    BITRISE_PROJECT_PATH: greg.xcodeproj
  - opts:
      is_expand: false
    BITRISE_SCHEME: greg
</code></pre>

Happy version counting! 🔢
