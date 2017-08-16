---
title: One iOS app, multiple configurations
date: 2017-08-16 9:23 UTC
tags: how-to, iOS
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/og_default.png
---

Exporting your IPA with multiple configurations is a great tool in the developer's hand:

You can have

- only a single IPA exported
- two IPAs: one for AdHoc and one for AppStore
- several IPAs only slightly different from each other

Cool, huh? 😎


##Get a single IPA (for example: ad-hoc)
If you need a single IPA to be exported, that's easy as π! 😁

You should already have your project in a git repo that is added to Bitrise.io as an app and the required code signing provision profiles and certificate are uploaded. (Read more about codesigning [here](http://devcenter.bitrise.io/ios/code-signing/#collect-the-required-files-with-codesigndoc))
Use the xcode-archive step: it will create the .xcarchive __and__ it will be exported as a codesigned IPA as well in a single step.

<pre><code>    - xcode-archive@2.1.0:
        inputs:
        - export_method: ad-hoc
</code></pre>

##Export two IPAs (with multiple methods)
This is for you if you don't want to run builds multiple times and you know exactly what kinds of IPAs are needed, for example: one exported for ad-hoc and one for app-store. You can send out the ad-hoc version for testing and while it is being tested you have some time to prepare the app-store version in the App Store and as soon as the testing team sends the green light, you only need to click a button. - This is soo smooth 🙂

This is also pretty useful when you have a test and a production version of the app, because if you have the production one preloaded in the AppStore then you are only one click away from release if the test version has passed the tests.

Use the xcode-archive step: it will create the .xcarchive __and__ it will be exported as a codesigned IPA as well. (For example: for ad-hoc)
Then use the export-xcarchive step which will use the .xcarchive generated previously, so without a re-build it will export the IPA with the second chosen method. (For example: for app-store)

>This scenario works only if you do not have any code (or other) changes in the project itself, only the signature is different.

<pre><code>    - xcode-archive@2.1.0:
        inputs:
        - export_method: ad-hoc
    - export-xcarchive@0.9.2:
        inputs:
        - export_method: app-store
        - team_id: $TEAM_ID
</code></pre>


##Get multiple variations of the same "base-code"
This is very useful if (let's say) you have an app with different icons for couple of countries (EN, DE, IT, RU, etc...) It is pretty easy to preload the affected Info.plist files (one for each of the actual icons), this you have to do only once and the rest can be automated easily.

Use the xcode-archive step in this case. It will create the .xcarchive __and__ it will be exported as a codesigned IPA as well. (For example: for ad-hoc)

In this case you will need to use the xcode-archive step for each IPA as the project is changed and you will need a new .xcarchive to export your IPA again and again. In this example we have 3 ready made **Info.plist** files, and all we have to do is to replace them between xcode-archive steps so we will have 3 IPAs with different names and icons. 🙂

<pre><code>    - xcode-archive@2.1.0:
        inputs:
        - export_method: ad-hoc
    - script@1.1.4:
        inputs:
        - content: |-
            #!/bin/bash
            set -ex

            #change configuration
            mv $BITRISE_SOURCE_DIR/myapp/Info.plist $BITRISE_SOURCE_DIR/myapp/Info_1.plist
            mv $BITRISE_SOURCE_DIR/myapp/Info_2.plist $BITRISE_SOURCE_DIR/myapp/Info.plist
    - xcode-archive@2.1.0:
        inputs:
        - export_method: development
        - configuration: Debug
    - script@1.1.4:
        inputs:
        - content: |-
            #!/bin/bash
            set -ex

            #change configuration
            mv $BITRISE_SOURCE_DIR/myapp/Info.plist $BITRISE_SOURCE_DIR/myapp/Info_2.plist
            mv $BITRISE_SOURCE_DIR/myapp/Info_3.plist $BITRISE_SOURCE_DIR/myapp/Info.plist
    - xcode-archive@2.1.0:
        inputs:
        - export_method: development
        - configuration: Debug
</code></pre>

Have you tried to use multiple configurations? Tell us how and what do you use them for?

Happy building! 🏢
