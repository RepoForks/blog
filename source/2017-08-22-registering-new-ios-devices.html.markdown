---
title: Registering iOS devices
date: 2017-08-22 13:12 UTC
tags: how-to, ios
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/register_device.jpg
---

Bitrise has an integrated App Deployment system you can use for app and other build artifact file distribution. 👻

On the workflow level the distribution process is done by the combination of the `Xcode Archive` step (which archives your iOS project and exports the generated archive into an IPA file) and the `Deploy to Bitrise.io` step (which deploys your IPA file).

This guide will cover how to deploy your iOS app to a new device using bitrise. 🤓

#A public install page is generated

When you add your iOS project to Bitrise, our scanner will generate two workflows:

1. the `primary` workflow (if your project contains a testable target): this workflow is responsible for testing your project
2. the `deploy` workflow, which automates your release process using Bitrise's App Deployment system. The deploy workflow will contain both the `Xcode Archive` and the `Deploy to Bitrise.io` steps.

Our deploy step will generate a public install page (by default) for your iOS artifacts. If you open this install page on your iOS device you can install the generated IPA file on the device, but only if your device is registered in the provisioning profile (used by the xcode ipa export process) and you have added your device to the Test Devices list on your profile's page on bitrise.io. 📱

#So if you need to add a new device... ➕

To deploy to a new device for the first time you will need to

##1. update your provisioning profile

- [Register your devices using your Developer Account:](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html), see `Registering Devices Using Your Developer Account`.
- [Add the registered device to your provisioning profile](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html), see `Creating Provisioning Profiles Using Your Developer Account`.
- Download your updated provisioning profile and replace your existing one on Bitrise, in your app's workflow editor tab.

![Provisioning profile](images/provisioning-profile.png)

When you run a new build, the generated IPA file is provisioned to be installed on your new device.

##2. add your device to Bitrise
To be able to install the new IPA from the Bitrise generated public install page add your new device to Bitrise as well:

- Open up your profile's `Test Devices` [page on Bitrise](https://www.bitrise.io/me/profile#/test_devices) __using your iOS device__.
- Click `Register this device`.
- The website will ask for permission to open up your device's Settings to install a web profile. This profile allows the bitrise.io website to install an IPA file on your device. Once you have installed the profile, you will be redirected back to bitrise.io to finish registration.
- Set a name to your new device and click `REGISTER DEVICE`.

Tada! From now on you can visit your iOS app's public install page and install your IPA file on your new device.

Happy coding! 👋
