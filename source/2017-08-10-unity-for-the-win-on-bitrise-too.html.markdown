---
title: Unity for the win, on Bitrise too!
date: 2017-08-10 12:03 UTC
tags: unity, how-to
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/unity-blog.jpg
---

Did you know that you can automate your Unity builds on Bitrise?


>The leading global game industry software [Unity](http://unity3d.com) plays an important part in a booming global games market. More games are made with Unity than with any other game technology. More players play games made with Unity, and more developers rely on their tools and services to drive their business... Global gamers downloaded Unity-made games to nearly 2 billion unique mobile devices, in Q1 2016 alone.
34% of the top 1000 free mobile games are made with Unity.
Unity customers include Coca-Cola, Disney, Electronic Arts, LEGO, Microsoft, NASA... From large and small studios to independent professionals, more and more developers are moving to Unity.
[Source](https://unity3d.com/public-relations)


#How to add your Unity project to Bitrise?
Unity is not supported 1-on-1 on Bitrise, but there is a way to setup your projects and then to test, build and release your games as fast as any other apps of a different project type. As this is a cross-platform project, in the end you'll get an IPA and an APK too, hooray! It is really easy to release them, innit? Let us give you a hand to get there. 🖐

When you set up your project, don't use the scanner but choose the other/custom option...

![Setup](images/unity-skip-scanner.png)

...and choose a Xamarin stack.

![Choose a Xamarin stack](images/unity-stack-selection.png)


##Download and install
This is a script that downloads the version of Unity you choose, so you can choose whichever matches your project. Install the  editor and then you'll need its Android and iOS platforms too.

Let’s see an example: if you want to use Unity 5.5.0, this is how you download the editor and the two platform support packages (Android & iOS) :

<pre><code>#download unity pkg
curl -o ./unity.pkg http://download.unity3d.com/download_unity/38b4efef76f0/MacEditorInstaller/Unity-5.5.0f3.pkg

#download android support platform
curl -o ./android.pkg http://download.unity3d.com/download_unity/38b4efef76f0/MacEditorTargetInstaller/UnitySetup-Android-Support-for-Editor-5.5.0f3.pkg

#download iOSle support
curl -o ./ios.pkg http://download.unity3d.com/download_unity/38b4efef76f0/MacEditorTargetInstaller/UnitySetup-iOS-Support-for-Editor-5.5.0f3.pkg
</code></pre>

This is how you install them:
<pre><code>#install unity pkg
sudo -S installer -package ./unity.pkg -target / -verbose

#install unity android support pkg
sudo -S installer -package ./android.pkg -target / -verbose

#install unity iOS support pkg
sudo -S installer -package ./ios.pkg -target / -verbose
</code></pre>

After installation you will find Unity in the following location:

<pre><code>
/Applications/Unity/Unity.app/Contents/MacOS/Unity
</code></pre>

For running Unity we need `Cacerts.pem` to be placed at the right location. Unity will generate this file for us, so let's run it for the first time.

<pre><code>/Applications/Unity/Unity.app/Contents/MacOS/Unity -logfile &
sleep 15
sudo killall Unity
</code></pre>


##License activation
You'll need to have a valid license key to build Unity projects. Sadly, only paid plans are working through the command line - and therefore in Bitrise.

> Running the Personal (free) version needs activation through the UI and using your personal serial number.

To activate your license key use the following code, which will activate the license and link the VM with your account:

<pre><code>/Applications/Unity/Unity.app/Contents/MacOS/Unity -quit -batchmode -serial $UNITY_SERIAL -username $UNITY_EMAIL -password $UNITY_PW -logfile
</code></pre>

## Building your project
Now let's jump to a more interesting part, building your app!

By inserting this script to your workflow you can start building your Unity apps. In the end it will generate an APK for your Android app and an IPA for your iOS app.

You'll need this for Android:
<pre><code>/Applications/Unity/Unity.app/Contents/MacOS/Unity -nographics -quit -batchmode -logFile -projectPath "$BITRISE_SOURCE_DIR" -executeMethod BitriseUnity.Build -androidSdkPath "$ANDROID_HOME" -buildOutput "$BITRISE_DEPLOY_DIR/mygame.apk" -buildPlatform android
</code></pre>

... and this for iOS:
<pre><code>/Applications/Unity/Unity.app/Contents/MacOS/Unity -nographics -quit -batchmode -logFile -projectPath "$BITRISE_SOURCE_DIR" -executeMethod BitriseUnity.Build -buildOutput "$BITRISE_SOURCE_DIR/xcodebuild" -buildPlatform ios
</code></pre>

An Xcode project will be generated, and that will have to be built in the usual way to obtain the IPA.

##Deactivation
We've already connected your license with the actual virtual machine, but we have to make sure it's deactivated at the end, so you will be able to re-use it. If you fail to deactivate the license you can still contact Unity helpdesk and get it sorted out, but it is way simpler to do it here 😊

To make sure it's deactivated add the following script step to your workflow:

<pre><code>/Applications/Unity/Unity.app/Contents/MacOS/Unity -quit -batchmode -logFile -returnlicense
</code></pre>

Set the step to _Run even if previous Step failed_ otherwise you risk that it won't get deactivated if your build fails!

![Run if previous Step failed](images/run-if-prev-step-failed.png)

This is what a finished build looks like on Bitrise. Pretty cool, eh?

![A finished Unity build](images/unity-build.png)

As you can see there is an iOS.ipa and an Android.apk generated in one build! Next time you'll use this, it'll be as quick as all your other apps! :)

__Do you have any more questions? [There is a more detailed description of Unity projects available](https://discuss.bitrise.io/t/how-to-build-a-unity-project/1317).__

Happy building, happy gaming! 🚀
