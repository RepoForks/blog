---
title: Build different release signed IPAs for your Xamarin.iOS project!
date: 2017-08-11 10:23 UTC
tags: xamarin, code signing, release, how-to
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/release_signed_xamarinios.jpg
---

If you have a Xamarin project, by default you'll have a new project generated with a Debug and a Release configuration. For both configurations Signing Identity is set to: `Developer Automatic` and Provisioning Profile is: `Automatic`. However, you may want to generate multiple Release builds, using different code signing types to end up with an App Store IPA for Store deploy and an Ad Hoc IPA for testing.

This post will show you how to do that. ⚙

When you add a new Xamarin project to Bitrise, at the end of the process you need to choose which solution configuration and platform to build.

![Xamarin Solution settings](images/xamarin-solution-settings.png)

If you select the Release configuration and the iPhone platform, our scanner generates the following Bitrise config for a Xamarin project:

<pre><code>format_version: "4"
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
project_type: xamarin
app:
  envs:
  - BITRISE_PROJECT_PATH: Multiplatform.sln
  - BITRISE_XAMARIN_CONFIGURATION: Release
  - BITRISE_XAMARIN_PLATFORM: iPhone
trigger_map:
- push_branch: '*'
  workflow: primary
- pull_request_source_branch: '*'
  workflow: primary
workflows:
  primary:
    steps:
    - activate-ssh-key:
        run_if: '{{getenv "SSH_RSA_PRIVATE_KEY" | ne ""}}'
    - git-clone: {}
    - script:
        title: Do anything with Script step
    - certificate-and-profile-installer: {}
    - nuget-restore: {}
    - xamarin-archive:
        inputs:
        - xamarin_solution: $BITRISE_PROJECT_PATH
        - xamarin_configuration: $BITRISE_XAMARIN_CONFIGURATION
        - xamarin_platform: $BITRISE_XAMARIN_PLATFORM
    - deploy-to-bitrise-io: {}
</code></pre>

If you want to setup Bitrise for multiple different release code signing (e.g.: AppStore, AdHoc) for your Xamarin iOS project, you have two options:

1. to configure Bitrise with two workflows and setup the workflows to install the desired code sign files per workflow...
2. to branch your code signing types based on your solution configuration.

We'll go through no. 2. below, step by step. 🙂

Let's say we want to create an __AppStore__ and an __AdHoc__ build. You will need to create separate solution configurations for the specific code signing types:

##Update your Xamarin project

First create two new project configurations for you iOS project:

1. Double click on your iOS project in the project navigator.
2. Select General / Configurations and create new configurations: “ReleaseAppStore” and “ReleaseAdHoc” with “iPhone” platform.

Then specify the code signing settings for the newly created project configuration:

1. Double click on your iOS project in the project navigator and select Build / iOS Bundle Signing.
2. Select the "ReleaseAppStore" config.
3. Set Distribution (Automatic) or the specific Distribution Code Sign Identity.
4. Set the Provisioning Profile to App Store (Automatic) or to the specific App Store Profile.

Now do the same steps for the ReleaseAdHoc project config!

![Xamarin](images/xamarin-1.jpg)

Create two new solution configurations:

1. Double click on your solution in project navigator and select Build/Configurations/General tab).
2. Select "ReleaseAppStore" and "ReleaseAdHoc" with "iPhone" platform.
3. Disable Create configurations for all solution items.

Now setup Configuration Mappings

1. Select the ReleaseAppStore solution configuration.
2. Map this for the iOS project's "ReleaseAppStore\|iPhone" config.
3. Check the Build box.

Do the same for the ReleaseAdHoc solution config!

![Xamarin](images/xamarin-2.jpg)

You are done updating your Xamarin project. Now your solution's "ReleaseAppStore\|iPhone" configuration will only map to your iOS project's "ReleaseAppStore\|iPhone" config and the iOS project will be built with your Distribution Certificate and your App Store type Profile.

##Update your Bitrise config

The last step is to update your Bitrise configuration:

1. Upload your Distribution Code Sign Identity and the Provisioning Profiles (App Store and Ad Hoc) to Bitrise.
2. Update your bitrise.yml with the following:


<pre><code>format_version: "4"
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
project_type: xamarin
app:
  envs:
  - BITRISE_PROJECT_PATH: Multiplatform.sln
trigger_map:
- push_branch: '*'
  workflow: primary
- pull_request_source_branch: '*'
  workflow: primary
workflows:
  app_store:
    envs:
      - BITRISE_XAMARIN_CONFIGURATION: ReleaseAppStore
      - BITRISE_XAMARIN_PLATFORM: iPhone
    steps:
    - activate-ssh-key:
        run_if: '{{getenv "SSH_RSA_PRIVATE_KEY" | ne ""}}'
    - git-clone: {}
    - script:
        title: Do anything with Script step
    - certificate-and-profile-installer: {}
    - nuget-restore: {}
    - xamarin-archive:
        inputs:
        - xamarin_solution: $BITRISE_PROJECT_PATH
        - xamarin_configuration: $BITRISE_XAMARIN_CONFIGURATION
        - xamarin_platform: $BITRISE_XAMARIN_PLATFORM
    - deploy-to-bitrise-io: {}
  ad_hoch:
    envs:
      - BITRISE_XAMARIN_CONFIGURATION: ReleaseAdHoc
      - BITRISE_XAMARIN_PLATFORM: iPhone
    steps:
    - activate-ssh-key:
        run_if: '{{getenv "SSH_RSA_PRIVATE_KEY" | ne ""}}'
    - git-clone: {}
    - script:
        title: Do anything with Script step
    - certificate-and-profile-installer: {}
    - nuget-restore: {}
    - xamarin-archive:
        inputs:
        - xamarin_solution: $BITRISE_PROJECT_PATH
        - xamarin_configuration: $BITRISE_XAMARIN_CONFIGURATION
        - xamarin_platform: $BITRISE_XAMARIN_PLATFORM
    - deploy-to-bitrise-io: {}
</code></pre>

You are done! 🙂 With this config running the `app_store` workflow will generate an AppStore signed IPA and the `ad_hoc` will generate an Ad Hoc type IPA. 🎉
