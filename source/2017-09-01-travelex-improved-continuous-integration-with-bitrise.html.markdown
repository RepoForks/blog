---
title: Travelex - Improved Continuous Integration With Bitrise
date: 2017-09-01 13:37 UTC
tags: guest-post
authors: Anna Bátki|anna.batki@bitrise.io
overlay: /images/travelex_blogpost.jpg
---

_Guest blog by Rémy Chanteney, lead of engineering at Travelex. The original post appeared on [Travelex Tech Blog](https://blog.travelex.io/bitrise-travelex-digital-f3388019bae)_.

>From a software engineering (mobile oriented) background, __Rémy Chanteney__ recently transitioned to lead of engineering at Travelex. Passionate about everything related to mobile, cloud infrastructure and disruptive technologies.
With products like [Wire](https://wire.travelex.co.uk/) or [Travelex Money](https://www.travelex.co.uk/services/travelex-money-app) and other B2B projects, __Travelex__ digital is striving to innovate in the international money transfer ecosystem.


You might know [Travelex](https://www.travelex.com/) as currency exchange company and will probably have seen one of our bureau de change outlets — we are present in a vast majority of airports around the world.

Our [digital team](https://blog.travelex.io/) strive to tackle technological challenges across a variety of products. As with any businesses, the technology choices we make eventually become outdated and need re-evaluation.

#Retrospective

Trying to fill the features gap to keep up with the competition leads to engineering challenges and technical debt that accumulates over months as you stretch an app to fit new features.

With that in mind, it was time for our team to take a step back, pinpoint what was slowing us down and __spend some time investigating the options to improve our workflow__. A retrospective needs to be seen as an investment, just like upgrading an engine before the next race.

Having experienced frequent build failures, we acknowledged some issues with our integration and delivery process and decided to challenge the tools used by our teams.

Firstly, we had to ask ourselves what was wrong with the way we were currently doing integration and delivery and secondly, how we could improve it.

##Jenkins

Like lots of companies, we were still using the somewhat __outdated__ _Jenkins_ (raise your hand if you think that Jenkins is a CI from a past era).

Yes, it’s highly configurable and powerful, but definitely over-complicated for what we want to achieve with the [Travelex apps](https://play.google.com/store/apps/dev?id=8499965144598470440). Not to mention mentioning the horrible UI (we’ve come a long way since 2011), it’s far from straightforward.

On top of that, we were hosting our CI server internally for enhanced security reasons (OK, remotely accessible, but still…).

To summarise, the flip side is that we were losing a lot of time maintaining or fixing it (and I mean, easily several hours per week) due to our relatively complex process involving __Nexus and usage of VPN__.

##Nexus Repository

To put this in context, we have __5 different flavors__ for the [Supercard](https://play.google.com/store/apps/details?id=com.travelex.supercard) app and 4 for [Travelex Money](https://play.google.com/store/apps/details?id=com.travelex.money). What does that mean?

In the case of Supercard, for every build triggered by _Jenkins_, 5 different APK’s were created.

We were using [Nexus Repository](https://www.sonatype.com/nexus-repository-sonatype) to store these 5 artifacts, so they can be picked up by any QA’s or developers later on. Once Jenkins finished building the different variants, a [Gradle](https://gradle.org/) __publish__ task was executed to deploy these APK’s on _Nexus_.

##Deployment Through VPN

To make things more complicated, the only way to deploy to our _Nexus_ repository was through a VPN (again, security reasons).

To summarise, imagine deploying roughly 5 x 20Mb from a local hosted _Jenkins_ server to a __dicey__ _Nexus_ repository, though a VPN, for every build. It seemed pretty clear at this point that the process could be __simplified__.

Adding to that the fact that we had to occasionally update the dependencies and build tools, the process was leading to a lot of failure.

The complexity of the overall process definitely contributed to the failures (as you can see below):

![Travelex local network](images/travelex1.jpeg)
_Process before Bitrise_

1. A developer creates a pull request on GitHub
2. Another developer merges the pull request — a build is triggered on Jenkins
3. The different artefacts are deployed on Nexus
4. The QA’s could access the artifacts from Nexus
5. Remote QA’s had to access to Nexus through a VPN

#Requirements

Once we knew what the problem was, we needed to figure how to simplify everything and define the requirements, __without compromising the security aspect__.

##Sine Qua Non
There are obviously mandatory specifications, regardless of the final decision:

- Very little to no maintenance
- Straightforward and easy to set up
- Pattern matching triggers (push, PR, …)
- _Slack_ integration
- Scheduled builds for nightlies
- _GitHub_ [pull request status check](http://blog.bitrise.io/2015/04/23/pull-request-support-for-github-repositories.html?utm_source=bitriseblog&utm_medium=web&utm_campaign=17w35travelex) support to run the tests

##Local vs Cloud
Nowadays, looking for a new CI solution without considering cloud-based offers is impossible. Therefore, the __subscription fee__ and the __level of control__ on the build environment are new concerns added to the list above.

While the cost of the subscription fee isn’t that much of a problem here, we wanted our CI to be __customisable__ and __flexible__.
I won’t tease you any longer, we decided to go ahead with Bitrise.

#Why Bitrise?
When comparing different alternatives, we were impressed by the multiple features Bitrise offers, in addition to its user-friendly interface.

The __highly customisable__ build workflows and the freedom of running limitless custom scripts (bash, ruby, golang, you name it) made this a no-brainer.

##Support

They are using the now notorious [Intercom](https://www.intercom.com/) live chat solution. It’s __smooth, frictionless__ and __feels very personal__ (at least for the few times we needed it).

Any question? Just tap the floating action button in the bottom right corner of your screen from anywhere in your dashboard.

![Support](images/travelex2.jpeg)

##Feature Requests

The quality of support is easy to appreciate and feature requests are another pleasant surprise you will discover further down the road. The Bitrise team let you create suggestions or comment and vote for already existing ones.

Now, will your ideas be added to their roadmap? That’s a different story, but you can speak your mind, and that’s important.

##Documentation

Their documentation is literally __off the charts__ — clear, complete and easy to navigate. They recently introduced a brand new [DevCenter](http://devcenter.bitrise.io/?utm_source=bitriseblog&utm_medium=web&utm_campaign=17w35travelex) and guess what? It’s [open source](https://github.com/bitrise-io/devcenter). The least we can say is that they have a strong sense of community.

##Role Granularity

Owners, Admins, Developers, QA/Testers — the [different roles](http://devcenter.bitrise.io/team-management/?utm_source=bitriseblog&utm_medium=web&utm_campaign=17w35travelex) provided by Bitrise are a big plus, allowing you to limit rights for users who don’t need full access. Unfortunately, there is no way to create custom roles yet, which means that if you don’t find what you need in one of the roles above, you will most likely end up giving Admin rights to everyone, which isn’t ideal.

##Flexible Configuration

The project workflow configuration is __user-friendly__, graphically represented as a timeline. You can add, delete or even move steps in a simple drag and drop interface.

Bear in mind that once you save your workflow, a __[YAML](http://www.yaml.org/start.html)__ file is generated. Of course, if you feel like it, you can directly edit your workflow from this file.

This also means that if you have _project A_ and you want to start _project B_ with a very similar workflow, you just have to copy/paste the content of the \*.yml file.

#Now

![Process](images/travelex3.jpeg)
_Process with Bitrise_

As you can see on the diagram above, the process is much simpler now — the magic now happens on _GitHub_ and _Bitrise_, period.

That’s what we call trimming the fat, isn’t it?

##Store Publishing

While Bitrise provides a [specific workflow step](https://github.com/bitrise-steplib/steps-google-play-deploy) to sign and release the app on different stores, we chose to not change this step of our process yet. We have some concerns about giving access to the certificates/keystores to third-parties, therefore, we are still signing and delivering our apps manually (for now).

We are currently doing due diligence and evaluating the security risks and consequences to do this final step through _Bitrise_ as well.

#Room For Improvement

For now, Bitrise is not perfect — even though they provide all the key features and everything runs smoothly, some nice-to-haves are definitely missing.

##Two-Factor Authentication
_(Editor's note: __[this feature has since been implemented](https://discuss.bitrise.io/t/2fa-two-factor-auth-two-step-auth-is-finally-available/1032?utm_source=bitriseblog&utm_medium=web&utm_campaign=17w35travelex).)___

Now that we ditched some security layers (locally hosted Jenkins, mandatory VPN for Nexus), Bitrise clearly became the Achilles heel. This may or may not be an issue, depending on how your team is using it.

The way Bitrise works is the same as _GitHub_ — you create a __personal account__ and the __owner__ of the company account will attach the __organization__ to your personal account, giving you some rights.

While being very convenient, the flip side is that the vulnerability of your projects depends on how conscientious your team members are. As we all know there are 1001 ways for a password to be compromised, the two-step authentication à la _GitHub_ would significantly improve the security of the entire organization projects.

At _Travelex_, every developer has to enable the two-factor authentication of their _GitHub_ account — it’s optional on _GitHub_, but __mandatory within the company__. For consistency, we would like to apply the same rule for __Bitrise__.

Hopefully, this option will land [sooner or later](https://discuss.bitrise.io/t/two-factor-authentication/469?utm_source=bitriseblog&utm_medium=web&utm_campaign=17w35travelex) on Bitrise, meanwhile the best we can do is be very rigorous with the rights that we grant to different users — a simple developer or QA shouldn’t need admin rights.

##Lack of Mobile App

It’s 2017, mobiles are running the world. Their web version is fairly responsive, but the experience isn’t great. It’s probably not a priority but I personally would like to be able to start a build or read the logs __on the go__.

Anyhow, it’s still possible to trigger builds directly from Slack using the [Outgoing Webhooks API](https://api.slack.com/outgoing-webhooks).


___


If you needed more evidence that idle time is crucial for a team, you got it.

Our engineers are happy with the new process, QA’s and developers can easily get access to the artifacts __from anywhere, without setting up a VPN on their mobile devices__. In case of failure, you don’t need to be a _Jenkins_ Guru to read the log and figure out what the problem is.

The cherry on the cake— engineers can now focus on their work and not on fixing something that is supposed to work continuously.

However, from a security point of view, we had to make concessions but we definitely gained a lot in __efficiency__ and __understanding__ — a classic tradeoff.
