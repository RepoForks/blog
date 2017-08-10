---
title: Let's automate together! Bitrise + IFTTT = ⚙
date: 2017-07-20 10:00 UTC
tags: how-to
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/ifttt.png
---


Bitrise finds challenges in automation. Do you?

We're almost sure you've used [IFTTT](https://ifttt.com/) (turning one's phone ringer volume up FTW),
but have you ever tried to use it with Bitrise? Let us give you some pointers to
start on! We'll show you two step by step guides how to use IFTTT with Bitrise:

1. if your Bitrise build fails then IFTTT will send you an SMS
2. if you press the Big Deploy Button on your phone then IFTTT will deploy your last build to App Store

##1, Know instantly if a build fails
Let's make IFTTT send you an SMS if your build fails!

![Send me an SMS!](images/ifttt-build-failed.png)

__On [IFTTT](https://ifttt.com/)__

1. After login, either click My Applets or your profile and click New Applet
2. Click __+this__ and search for Webhooks

    ![Select Webhooks](images/ifttt-step-1.png)

3. Select `Receive a web request`
4. Set event name (e.g. `build_failed`) and `Create trigger`
5. Click __+that__ and select SMS

    ![Select SMS](images/ifttt-step-2.png)

6. Select `SMS`
7. Set a message, for example: `Sorry, your build has failed. Why not try again?`
8. You can add your build number too by inserting `Value1` under `Add ingredient`: `Sorry, your #{{Value1}} build has failed. Why not try again?`

    ![Add a message](images/ifttt-step-3.png)

9. `Create action`, review and `Finish`

![Finish](images/ifttt-step-4.png)

__On Bitrise:__

1. Go to your selected app's Workflow Editor and after the last step, click `+` and
 add a `Script`
2. Turn on `Run if previous step failed`, to make sure this runs in all cases and replace the script in the textbox with
the following:

    ```
    curl -X POST -H "Content-Type: application/json" -d '{"value1":$BITRISE_BUILD_NUMBER}' https://maker.ifttt.com/trigger/build_failed/with/key/$IFTTT_TOKEN
    ```

    ![Add a Script to your Workflow Editor](images/ifttt-bitrise-script.png)


3. You can find the token at the [Webhook's maker page](https://ifttt.com/maker_webhooks) under `Documentation`.

    ![Find the token!](images/ifttt-webhooks-documentation.png)

4. Add this token as an Environment Variable to Bitrise in your Workflow Editor with the name `IFTTT_TOKEN`
5. Save and you are done!

You'll get an SMS when your build fails. We hope you won't get (m)any! 👻



##Big Deploy Button
How about a Big Deploy Button? You could deploy your last release straight to App Store.
Do it yourself!

__On Bitrise:__

1. Log in and select your app. Click `Start/Schedule a Build` in the top right corner.
2. Choose `Advanced`, select your preferred `brach` and the `deployment workflow`.
3. Find the `cURL` command at the bottom of the popup and save it somewhere. You will need this later.

![Get the cURL command](images/ifttt-deploy-button-1.png)

__On IFTTT:__

1. After login, either click My Applets or your profile and click New Applet
2. Click __+this__ and search for Button widget. Select it and then `Button press`
3. Click __+that__ and search for Webhooks. Select it and then click `Make a web request`  
4. Copy and paste your cURL from (see 3. above)
5. Set `Method: Post` and `Content Type: application/json`
6. Copy the build parameters following `data` from your app's cURL command (see 3. above)
and insert it to `Body`. Change "triggered by" parameter to "ifttt"
7. Create action, name your applet (e.g. Deploy with Bitrise or Big Deploy Button) and Finish

![IFTTT setup](images/ifttt-deploy-button-2.png)

You are almost done :)

__On your phone:__

0. Install the IFTTT application and log in ([Google Play](https://play.google.com/store/apps/details?id=com.ifttt.ifttt), [App Store](https://itunes.apple.com/app/apple-store/id660944635?mt=8))
1. Add widget to your home screen: IFTTT small
2. Choose your applet (e.g. Deploy with Bitrise or Big Deploy Button)
3. Press it and 🎉

![Big Deploy Button](images/ifttt-big-deploy-button.png)


##Think on!

Similarly, you could track all your builds in a Google Spreadsheet and have all
the details there: project and build number, build time, success, status, etc. Or...

Do you have another IFTTT applet for Bitrise? Share it with us!
