---
title: Slack away!
date: 2017-08-18 08:20 UTC
tags: how-to, slack
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/slack_post.jpg
---

Slack - as you may have guessed - is one of the most used integrations on Bitrise 💪. There are a lot of ways to get tons of value out of it, so we will go through a couple of examples to give you an idea what you can achieve with it to speed up and facilitate your app development.

The main purpose is to get fast responses so your team will know if a build failed, if your code coverage is dropped or if an application was successfully deployed. The best thing is that you can simply connect different integration outputs as well, so you will have heaps of information in your hands!

## Get those build notifications 💥

First, let's change your app's workflow, so that it will update your team about the build statuses. 😎

1. On your Bitrise Dashboard, choose your app and open your app
2. In the workflow editor click + below the step finishing the build and find the Slack step. (This should be one of the last steps).

    ![Slack step addition](images/slack-1.png){:height="60%" width="60%"}

3. Now we have find the right Webhook URL, so your integration can reach your team and post awesome notifications. To do so, go to https://__YOURTEAMNAME__.slack.com/services and select `Incoming Webhooks`.
4. Select the channel you are ready to add the integration to and customize the username and icon that will be used for the user when the notification hits your channel.
5. If you are ready, save the updated settings and copy the newly generated Webhook URL
The hook should look like this:

    <pre><code>https://hooks.slack.com/services/your/really/secure_token</code></pre>

6. And add it to the Slack step on Bitrise!
![Slack step!](images/slack-5.png)

You can also update the icons and setup different notifications that will fire if your build is successful or if it fails. But before doing so, let's try out the current setup! After a successful build you should see something like this:

![YAY!](images/slack-2.png){:height="80%" width="80%"}

## GIFs for the rescue

Let's add a `Script step` and `GIFs with Giphy` to your workflow before your Slack step. In the script step we will generate the correct keyword for Giphy and you'll always end up with an trendy and relevant image. Why see the same image over and over again?

So let's change that part in your YML file in the following way:

<pre><code>    - script@1.1.4:
        inputs:
        - content: |-
            #!/bin/bash
            set -e
            set -x

            if [ $BITRISE_BUILD_STATUS -eq 1 ]
              then envman add --key GIPHY_KEYWORDS --value "fail,disappointed,try again"
              else envman add --key GIPHY_KEYWORDS --value "success,congrats,victory"
            fi
    - giphy@0.1.1:
        inputs:
        - gif_words: "$GIPHY_KEYWORDS"
    - slack@2.6.1:
        inputs:
        - webhook_url: "$SLACK_WEBHOOK_URL"
        - channel: "$SLACK_CHANNEL"
        - message: "Yay! Build was successful! \U0001F4AA"
        - message_on_error: "Ah! Something went wrong \U0001F631"
        - image_url: "$GIF_URL"
</code></pre>

This is today's best:
![How 'bout a gif?](images/slack-3.png)

## Deploy with QR codes

Instead of (or next to some) GIFs you can send additional information to your QA team. For example, if you move the Slack step (or add another) after the deployment step you can send out a QR code, so your testers can easily install the latest update to their phone 📲

To do so, let's change your workflow the following way:

<pre><code>    - create-install-page-qr-code@0.1.2: {}
    - slack@2.6.1:
        inputs:
        - webhook_url: "$SLACK_WEBHOOK_URL"
        - channel: "$SLACK_CHANNEL"
        - message: "Yay! Build was successful! \U0001F4AA"
        - message_on_error: "Ah! Something went wrong \U0001F631"
        - image_url: "$BITRISE_PUBLIC_INSTALL_PAGE_QR_CODE_IMAGE_URL"
</code></pre>

And woala! Your testers will ❤ you for that!

![QR code](images/slack-4.png)

## There's more!

We hope that we could give you a couple of ideas that you can use to grow your notification armada. If you have some great use cases definitely let us know!

Next time we will check how to setup a Slack bot to start your builds for you!
Stay tuned! 🙌
