---
title: Droids on Roids - How to Connect Physical Devices to Bitrise.io
date: 2017-08-07 13:14 UTC
tags: guest-post, android, how-to
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/droids-on-roids-overlay.jpg
---

_Guest blog by Karol Wrótniak, mobile developer at Droids On Roids. The original post appeared on [Droids On Roids blog](https://www.thedroidsonroids.com/blog/how-to-connect-physical-devices-to-bitrise-io)_. 

>[Droids On Roids](https://www.thedroidsonroids.com/) is a full-service iOS, Android and backend development company from Poland offering sophisticated mobile solutions from design to store release and building high-performing mobile apps for start-ups and great brands. With 50+ mobile applications in their portfolio, Droids on Roids' solutions serve millions of users across platforms and channels.

Bitrise.io is a cloud CI/CD service. It can build, test and deploy your apps. In this article we will focus on testing, namely on Android apps [UI](https://developer.android.com/training/testing/ui-testing/index.html) and [instrumented unit tests](https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests.html).

#Why physical devices? Isn’t an emulator enough?

Android SDK provides an [emulator](https://developer.android.com/studio/run/emulator.html) which can also be started in headless mode in a CI environment. However, this is a theory as, in practice, there is a major issue which makes emulating very inconvenient.

On a machine without [KVM](https://www.linux-kvm.org/page/Main_Page) (Linux) or [HAXM](https://software.intel.com/en-us/android/articles/intel-hardware-accelerated-execution-manager) (macOS, Windows), the Android emulator is very slow. How slow? You can check out the details [in this thread](https://discuss.bitrise.io/t/actual-android-emulator-issues/2106) on the bitrise discussion forum. That’s because virtual machines, provided by [Google Compute Engine](https://cloud.google.com/compute/) and used by bitrise.io, do not support [hardware acceleration](https://developer.android.com/studio/run/emulator-acceleration.html). This limitation is applicable [to more than just bitrise](http://devcenter.bitrise.io/faq/android-x86-emulator/).

Another advantage of real devices is that they support a bunch of features not available on the emulator. For example WiFi, Bluetooth, NFC or USB. See [all the limitations](https://developer.android.com/studio/run/emulator.html#limitations) in emulator documentation. Finally, real devices contain device or vendor-specific stuff, such as preinstalled system apps and specific behaviors, which do not need to occur on emulators.

#Clouds over the farm

There are several options for connecting devices to bitrise.io. One of the easiest and effortless ones is to use a cloud-hosted device farm. Such farms are offered by Google ([Firebase Test Lab](https://firebase.google.com/pricing/)), Amazon ([AWS Device Farm](https://aws.amazon.com/device-farm/pricing/)) or Bitbar ([public cloud](https://bitbar.com/testing/pricing/public-cloud/)). However, it may not necessarily be the cheapest solution, especially if you already have physical devices. Let’s check how they can be used on bitrise.io!

#Local device on a remote host

[Android Debug Bridge](https://developer.android.com/studio/command-line/adb.html) (ADB) allows you to run the client and server on different machines. The server will be the one running on the machine which your devices are connected to, while the client needs to be started on a CI machine. If you use ADB with `-H <server hostname or IP>` on the client side, it will just use devices connected to a server instead of its local ones.

As easy as pie! Well, not really. There are several inconveniences that need to be considered. Firstly, this method is not supported by the Android Gradle Plugin. This forces you to use the ADB commandline manually with the additional parameter. Moreover, device serial numbers need to be known in advance, somehow. If you have many devices with different characteristics – such as manufacturer, OS version, screen size, whether it is a smartphone, phablet, tablet or something else etc – you also need to know which one will have which serial. Finally, these devices won’t be cleaned up/restored to their original state before usage.

#STF to the rescue!

Thankfully, there is a tool called [Open STF](https://openstf.io/) which can solve the aforementioned issues. STF is an acronym for Smartphone Test Farm. Its core feature is interactive remote control of devices from the web browser. However, it also has a REST API and supports remote debugging.

#Putting all that stuff together

Several things will be needed to connect physical devices to bitrise.io. The first and most obvious part is a device. Most modern (supporting API level 14+) models should work. However, there are known issues with certain models (e.g. [ZTE Max Pro 981](https://github.com/openstf/stf/issues/614)). Some others, like various Xiaomi smartphones, require [additional steps](https://github.com/openstf/stf/issues/603) to be performed before connection. USB debugging also has to be enabled in the device development settings.

The most important part is a properly configured STF instance. For testing and evaluation purposes with up to several devices you can deploy STF on a single machine, even on your laptop. Just install [required dependencies](https://github.com/openstf/stf#requirements) and [run](https://github.com/openstf/stf#running) a local instance. However, if you want to use dozens or hundreds of devices, you will need to split a farm between many machines. See [deployment documentation](https://github.com/openstf/stf/blob/master/doc/DEPLOYMENT.md) and [setup](https://github.com/openstf/setup-examples) examples for more information. Make sure that the `--public-ip` parameter is specified, as STF won’t be accessible from different hosts otherwise.

If you can control your device(s) on a farm using a web browser, then it’s time to setup access. Go to the settings panel and generate an access token, as you can see in the picture below. On the right, you can also add ADB keys, so generate one pair using `adb keygen` if you don’t have it yet.

![Settings / Keys](images/droids1.png)

>TIP: ADB keys can be added automatically. Just use the device from STF web UI and start a remote debug session. The ADB key adding prompt will appear after invoking `adb connect`.

There has to be a reliable internet connection between bitrise.io and your farm. You can just expose a farm on a public IP: however, it is not much secure option, especially when default, mock authorization is used. Some kind of VPN may be used for secure connections. There is a ready-to-use [step](https://github.com/DroidsOnRoids/bitrise-step-vpnc-connect) for Cisco VPN. For other VPN types, you need to perform a connection setup yourself, probably using script step; see the [tutorial](http://devcenter.bitrise.io/tutorials/vpn-configuration/) on bitrise devcenter for more info.

Finally, you can add a [Open STF Connect](https://github.com/DroidsOnRoids/bitrise-step-openstf-connect) step to your workflow. The recommended step order is as follows:

1. Compile Android tests, usually using Gradle Runner step with `assembleAndroidTesttask`
2. Connect to a VPN (if needed)
3. Connect to the STF
4. Perform the tests, usually using Gradle Unit test step with `connectedChecktask`
5. Disconnect from the STF

With this setup, devices are occupied only for the minimum required period of time, not for the entire build time. It is important that the disconnection steps have the `Run if previous Step failed` option enabled, so the devices will be always returned to the pool, even if the tests fail. Otherwise, they will be occupied until they exceed their inactivity timeout periods (15 minutes by default).

#Bitrise workflow example

Here is the complete example of the workflow using STF in workflow editor:

![The workflow using STF](images/droids2.png)

And as a `bitrise.yml`:

<script src="https://gist.github.com/koral--/440f9cd04fb759d92b661add47cc57c4.js"></script>

Note that the Open STF Connect step exposes 2 additional parameters: `device_filter` and `device_number_limit`, allowing you to use only a subset of connected devices.

#Wrap up

Connecting bitrise.io with Open STF is pretty much easy. On Bitrise, you just need to add ready-to-use steps and configure them. Setting up a device farm requires some amount of work but, properly configured, it doesn’t need much maintenance (devices are restored to the previous state after usage). Note that the same devices on STF can also be used by QA or developers. Unless you need physical interaction (e.g. detected by accelerometer or gyroscope) they can be connected to the farm all the time.

---
