---
title: New export options Plist in Xcode 9
date: 2017-08-15 12:54 UTC
tags: xcode, how-to
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/xcode9_blog.jpg
---

Even if Xcode 9 is still in beta, Bitrise already has an Xcode 9 stack, which is used by many of you in experiencing mood. 👨‍🔬
However, several builds fail due to the new export option. We are working on an update for the `Xcode Archive` step to automatically detect this new option. Until we release the fix, let's see the problem and the solution.

#Before Xcode 9
Xcode 7 introduced the `exportOptionsPlist` flag for `xcodebuild` command line tool to specify the IPA export parameters.
(To see the available IPA export options call: `xcodebuild -h` and search for `Available keys for -exportOptionsPlist:` in the command's output.)

Before Xcode 9, the minimal `exportOptionsPlist` specified the `exportMethod`:

<pre><code>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt;
&lt;plist version="1.0"&gt;
	&lt;dict&gt;
		&lt;key&gt;method&lt;/key&gt;
		&lt;string&gt;development&lt;/string&gt;
	&lt;/dict&gt;
&lt;/plist&gt;
</code></pre>

These __export options are still available__ and you still have to select these options when you export your IPA in your local Xcode.

1. When you use the IDE to export your IPA for the first time you need to `Select a method for distribution`, this choice defines the `method` key in the `exportOptionsPlist`. (All the other options can be mapped to the UI options similarly.)
2. The `Xcode Archive for iOS` step is able to determine the required export options based on the provisioning profile embedded into the .xcarchive file, if you set `Select method for export` input to `auto-detect`.

#But with Xcode 9... ⚠
Using Xcode 9's xcodebuild tool you need to be more specific than in previous Xcode versions: if you run the step on Xcode 9.0.x stack using `auto-detect` export method you will receive the following error message:

<pre><code>"Error Domain=IDEProvisioningErrorDomain Code=9 \"\"ios-simple-objc.app\" requires a provisioning profile.\" UserInfo={NSLocalizedDescription=\"ios-simple-objc.app\" requires a provisioning profile., NSLocalizedRecoverySuggestion=Add a profile to the \"provisioningProfiles\" dictionary in your Export Options property list.}"
</code></pre>

It is visible from the log that you need to add the `provisioningProfiles` key to the exportOptionsPlist file, and this value should describe which bundle id should be signed with which provisioning profile.

#The solution for the time being
So you have to let your local Xcode 9 generate the exportOptions file for you, and once your Xcode generated this file, you can configure the Xcode Archive step with these options:

1. Open your project in your local Xcode 9
2. Archive the project
3. Once the archive is done, export the generated .xcarchive file into an IPA file. Xcode will copy the used exportOptionsPlist file next to the generated IPA file
4. Open this plist file in your favourite text editor and copy its content
5. Paste its content to the Xcode Archive step's `Custom export options plist content` input

Tada! 🎉  
You can now export your iOS project on the Xcode 9.0.x bitrise stack the same way you would do on your local machine.

Happy coding! 👻
