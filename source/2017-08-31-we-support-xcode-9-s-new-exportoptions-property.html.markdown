---
title: We support Xcode 9's new exportOptions property!
date: 2017-08-31 09:05 UTC
tags: xcode, how-to
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/xcode9_blog_pt2.jpg
---

We have released the new Xcode Archive step version ([2.2.0](https://github.com/bitrise-io/steps-xcode-archive/releases/tag/2.2.0)), which is able to auto detect the new exportOptions property, so the [workaround](http://blog.bitrise.io/2017/08/15/new-export-options-plist-in-Xcode-9.html) we provided two weeks ago is no longer needed.	👻
However, you still have to set some things up.

#The problem...

In Xcode 9 a new required exportOptions property was introduced: `provisioningProfiles`, and it describes which provisioning profile should be used to sign which target in your project.
Even if Xcode 9 is in beta, we created a stack with it to let our users test their projects with Xcode 9.
But even if you could use Xcode 9 on Bitrise, our Xcode Archive step was not able to automatically detect the provisioning profile - bundle id mapping (that is the new exportOptions property), so it caused several builds to fail.

#...is now sorted out 👨‍💻

The step archives your project as before, there is no change in this part. When the step has generated the .xcarchive file from your project it starts to collect the exportOptions (if xcode version is > 6).

When exporting an IPA file from the archive, you need to define how the step should do the export. There are a couple of options, let's see the relevant examples:

If `custom_export_options_plist_content` input is set, the step will use the exportOptions you provided.
End of story :)

If you don't set this and the `export_method` is set to `auto-detect`, then the step uses the provisioning profile embedded  into the .xcarchive file to determine the export method (as previously) and from now on every target in your project will be signed with this profile. (Second) End of story.

If you specified any other export method for the `export_method` input, the step will list every installed provisioning profile which is provisioned to sign your app, then the step filters this list for the profiles which match the specified export method.

- If the step has found only 1 matching profile (per target), this profile will be used to sign your target. (3.a end of story.)  
- If multiple profiles match a target and the specified export method, the step cannot decide which one to use, so it will fail. In this case you should specify the `custom_export_options_plist_content` as described in [this blog post](http://blog.bitrise.io/2017/08/15/new-export-options-plist-in-Xcode-9.html). (3.b end of story)

Happy coding! 🚀
