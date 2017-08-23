---
title: Install missing Android SDK components step for React Native
date: 2017-08-23 10:55 UTC
tags: how-to, android, react-native
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/missing_android_react.jpg
---


We released a new major version for `Install missing Android SDK components` step. The previous version had strong limitations and these limitations caused lots of issues, especially for React Native projects. Now they are gone! 🙌

##Earlier versions' process 🚶
Before version 2.0.0 the step:

- First searched for the root build.gradle file.
- In the same directory as the root build.gradle, it searched for a settings.gradle file to collect the active modules in the project.
- Based on the collected module names it created the list of the module's build.gradle file paths.
- Then it parsed the build.gradle files to detemine the required `compileSdkVersion` and `buildToolsVersion` and installed them.

##The earlier versions' limitations 😖
The following strong limitations were caused by the fact that the step was using file parsing:

- it was able to find the modules build.gradle file, only if it existed on the default path (REPO_ROOT/MODULE_NAME/build.gradle)
- it was able to determine `compileSdkVersion` and `buildToolsVersion` only if they were directly specified (not with variables)
- it was not able to follow a module's dependent modules to determine every required `compileSdkVersion` and `buildToolsVersion`

##And to get rid of these limitations... 	✂
In version 2.0.0 we dropped the file parsing solution and switched to using gradle commands to determine or install the required SDK components:

- First, the step determines your project's Android Gradle Plugin version.
- Then the step runs the `gradle dependencies` command.
- If the Plugin version is 2.2.0 or later, the plugin will download and install the missing components during the Gradle command.
- Otherwise the command will fail, and the step will then parse the command's output to determine which components are missing and installs them.

This way we let gradle to do the weight lifting, 🏋 and it is doing it quite well: the result is a more reliable version of the step.

Happy building! 🤖
