---
title: Missing dSYMs
date: 2017-09-06 09:03 UTC
tags: how-to
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/
---

Missing dSYMs (Fabric / Crashlytics deployer)

In the last few days we have faced an interesting issue with missing dSYMs which was not an issue in fact. :)

Everything started with a user reporting that in the build log everything seems to be fine but the dSYM file was not uploaded to crashlytics. At first we thought that something went wrong with the step so we asked for the build log, but we didn’t find anything related to it, also we checked the version number and UUID on crashlytics and tried to upload the generated dSYMs manually,
but we didn’t succeed.

![Crashlytics](images/crashlytics_missingdsym.png)

Finally in the documentation of fabric we found some useful articles about missing dSYMs.
One says that “There can be certain situations however, when dSYM uploads fail because of unique project
configurations or if you’re using Bitcode in your app”, so we checked again the build log and we found these two lines:
  - UploadBitcode: yes
  - CompileBitcode: yes

Partial success :smile:

The documentation also says that “For Bitcode enabled builds that have been released to the iTunes store or submitted to TestFlight, Apple generates new dSYMs so we have to download the regenerated dSYMs from Xcode and then upload them to Crashlytics”.

But we wanted all these things to work automatically :disappointed:

Setting “NO” for “Include bitcode” and “Rebuild from bitcode” in Xcode Archive & Export for iOS Step does not fix this, but you can do it with Fastlane, just follow [these instructions]((https://krausefx.com/blog/download-dsym-symbolication-files-from-itunes-connect-for-bitcode-ios-apps).

Or if you feel the challenge, you can contribute to Bitrise by developing an open source step to do all this and you can earn a well-deserved discount for the contribution.




https://docs.fabric.io/apple/crashlytics/missing-dsyms.html#all-about-missing-dsyms
https://krausefx.com/blog/download-dsym-symbolication-files-from-itunes-connect-for-bitcode-ios-apps
