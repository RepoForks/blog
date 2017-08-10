---
title: Yoyo Wallet - Shipping fast and frequently with Bitrise
date: 2017-07-17 13:55 UTC
tags: guest-post
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/Yoyo_Cafe_Nero.png
---

_Guest blog by Tariq Patel, head of engineering at Yoyo Wallet. The original post appeared on [Yoyo Engineering](https://engineering.yoyowallet.com/shipping-fast-and-frequently-with-bitrise-ci-1d9f310dc5cc)_.

>[Yoyo Wallet](http://yoyowallet.com/) is the fastest growing and largest multi-retailer mobile wallet in Europe. Now in its 4th year, Yoyo Wallet powers retailer experiences across the UK on the high street, in universities and in corporate locations. It was recently chosen by Marketing Week as one of the Top 100 Disruptor Brands and by KPMG as one of 50 FinTechs to watch globally.


##Context

In order for you to understand some of the complexity behind our deployment process, I will briefly explain how our mobile clients are architected.

![yoyo a single platform and code base powering multiple apps](yoyo1.png)

We are able to build and deliver multiple apps, including Yoyo Wallet and Caffè Nero, from a single codebase. Our mobile architecture allows us to build functionality and fix bugs in one place, while also giving us the flexibility to tailor the user experience for each retailer app.

To achieve this, our mobile app is split into two high-level components.

##Yoyo SDK

The Yoyo SDK is a proxy between the client and our platform. It consumes our platform API while maintaining an offline cache.

##Yoyo Client

The Yoyo mobile client codebase is further split into two components.

###App Core

Our “App Core” contains all of the business logic within the mobile client. This includes interactions with our SDK, state management, input validation components, third-party service integrations etc.

By decoupling this from the UI, we are able to better test all of the logic independently more easily and quickly.

###Retailer-centric User Experience

![yoyo consumer apps](yoyo2.png)
***Upcoming Yoyo v6 alongside Caffè Nero***

Even though each app shares the same app core, their user experiences may vary vastly, as can be seen above.

This final layer allows us to enable bespoke functionality within individual apps, say the **NUS Extra** integration in Yoyo, or the **Store Locator** in Caffè Nero.

##How do we remain confident in our releases?

With all of the above complexity in mind, we need to ensure that even the smallest change across any of our architectural components does not negatively affect any of our apps.

At the very least, this means added emphasis on automated testing and some form of continuous integration.

However, as noted [here](https://engineering.yoyowallet.com/yoyo-wallet-powers-the-new-caff%C3%A8-nero-app-c1a4fe469c73#726b), we like to be a lot more thorough. Peace of mind requires:

1. Continuous testing (automated and manual via Applause)
2. Dogfooding, in the form of bleeding edge internal builds (via Fabric)
3. External beta testing (via Google Play and TestFlight)
4. Regular early releases to the wider public
5. Feature flags to decouple feature releases from app store releases (via Taplytics)
6. Emphasis on actionable metrics (via Mixpanel)

Most of these requirements can be entirely automated.

##Enter, Bitrise CI

So what is Bitrise? Essentially a mobile continuous integration and delivery platform as a service. What really stands out, is their ability to help us automate not only the testing, but also the deployment of our mobile apps in a few short steps.

I won’t go into comparing multiple CI services, but it’s worth noting we used Travis CI up until the end of January 2016, at which point we migrated over to Bitrise. Since then, we’ve run over 20,000 builds through the platform.

![Bitrise deployment steps](yoyo3.png)
***Some of the deployment steps Bitrise offers***

Of the requirements outline above, 1–4 are fundamentally testing and deployment problems. While we have other services taking care of the actual solution, if we want to ship quickly and frequently, we need to automate the process itself; this is where Bitrise comes in.

I’ll now run through how some of our Bitrise workflows solve our automation problems.

###Simple Continuous Integration

Our _build_ workflow runs on every commit pushed to GitHub. These builds take roughly 8–12 minutes for Android and 20–24 minutes for iOS.

![Example build iOS workflow summary](yoyo4.png)
***Example build iOS workflow summary***

![How this looks on GItHub](yoyo5.png)
***How this looks on GItHub***

This is what you might expect from any other CI service but what I personally love about Bitrise is its modular nature. Its workflow editor allows you to easily add or remove steps — most of which have already been built by Bitrise themselves or the community, so it becomes a simple matter of plug-and-play. To take things further, each workflow you create can also be plugged into larger workflows as a prerequisite function. An example of this can be seen below.

As a result, setting up new workflows takes very little time and engineers are able to focus on building rather than the operational needs.

###Internal Bleeding Edge Alpha

We run a bleeding edge alpha programme internally within Yoyo. What does this mean? We want all Yoyo staff, barring sales (as they’ll be demoing frequently to clients), to have the latest and greatest version of each of our products. Everyone is widely encouraged to use Yoyo Beta on a daily basis for breakfast, lunch and coffee so that we can get extensive, early user feedback on new functionality and fixes.

Every Git commit pushed to master triggers our deploy-to-fabric workflow. Since master itself is protected, this happens on any merge into the branch. These builds take roughly 13 minutes for Android and 23 minutes for iOS.

![Multiple modular workflows combined to produce our deploy-to-fabric workflow](yoyo6.png)
***Multiple modular workflows combined to produce our deploy-to-fabric workflow***

###External Beta and Public Release

Beyond our internal testing, we want to test our products with a wider audience. For this, we have enabled TestFlight for our sales team, and an open external Google Play Beta programme — at Yoyo, staff have historically been iOS-heavy so we are comfortable with internal numbers, while we need the wider reach Google can provide for heavier Android testing.

![Scheduled daily Android deployment builds](yoyo7.png)
***Scheduled daily Android deployment builds***

As seen above, Bitrise allows us to schedule builds. We utilise this functionality in order to run and update our external testing programmes on a daily basis. Our _deploy-to-store_ workflow is scheduled to run every day at 3pm. These builds take roughly 12 minutes for Android and 25 minutes for iOS.

![Deploy direct to iTunes](yoyo8.png)
***Deploy direct to iTunes***

To keep with our goal of achieving frequent, regular releases to the public, we promote each of our iTunes builds every Tuesday, and our Google Play builds twice a week, on Tuesdays and Thursdays.

But with all of these checks in place, there is still the possibility of something going wrong once you hit production, particularly with the level of fragmentation in the Android market. This is where continuous manual testing comes into play.

We work with [Applause](https://www.applause.com/), a crowd-sourced QA firm, to help us catch anything our automated processes may miss. We do _not_ allow Applause to become gatekeepers, and to that end, we send them builds the same time we promote our beta builds to production, ie Tuesdays and Thursdays. This is also automated. Our _deploy-to-applause_ workflow is scheduled alongside our other _deploy_ workflows and take roughly 11 minutes for Android and 18 minutes for iOS.

##Summary

We have a very complex build and deployment workflow at Yoyo. Even before adding the majority of this complexity, releases were a major manual event on every occasion we decided to ship — usually every 2–6 weeks.

![Going from manual deployment to automated with Bitrise has helped us tighten our feedback loop](yoyo9.png)
***Going from manual deployment to automated with Bitrise has helped us tighten our feedback loop***

Bitrise has helped us deliver multiple times a week with certainty and without cause for concern. With our progression to 6 high-value releases every week, we were able to double our number of releases last year and are looking to triple that figure this year.

Our build and deployment process now requires very little thought when setting up initially, when maintaining or modifying workflows, and also at the point of every release.
