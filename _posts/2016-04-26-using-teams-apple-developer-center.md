---
id: 409
title: Using teams in Apple Developer Center
date: 2016-04-26T09:51:45+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=409
permalink: /uncategorized/using-teams-apple-developer-center/
categories:
  - Tutorials
  - Uncategorized
tags:
  - code-sign
  - ios
  - provisioning
---
## How to manage your iOS development certificates and provisioning (the less painful way)

&nbsp;

---
### How to read this article

The purpose of this article is to bring awareness to the issue of provisioning profile management in iOS and how it can be mitigated using teams in Apple Developer Center.

__If you're reading this because you want to fix a provisioning related issue__, and you don't feel like reading anything more, feel free to skip the next few sections and head over to "Summary" directly.

__If you're reading this because you are curious about the topic__, feel free to continue reading in sequence :).

---

&nbsp;

### The rant

Normally articles on Dev Monologue are quite cheerful. But this time their usual positive nature, will be substituted by a cold and dead serious theme. Provisioning profiles and certificates...


> Oh my god... the horror... THE HORROR!!!!<br/>
> -- anonymous Dev Monologue reader

The security aspect of the iOS developer program has always been a frustrating experience, no doubt about that. Most people cannot seem to get their head around code signing. And what makes it even worse is that in every major Xcode release so far, many things get changed and at the end you never know what's happening.

I'm going to take the liberty of even comparing it to git (and probably earn many people's despise). Don't get me wrong - I love git as much as the next guy. However, my observation is that most developers know a workflow of 6 or 7 commands in git that they use and everything beyond that is black magic. Go outside your comfort zone, and what follows is an endless stream of stack overflow threads and ultimately the shameful `git clone` all over again.

But now imagine that you write `git clone` only to find out that it doesn't work anymore. Instead, there's a "Fix ANY git problem EVAR" wizard somewhere in Xcode that works as good as Windows' network troubleshooting one.

I'm actually old enough to have developed for iOS at the times when Xcode had little to do with provisioning setup. You had to do all that manually. Generate a key pair, upload to the dev center, re-generate provisioning, add your devices manually... It was tedious, it was boring, it was confusing. However, once you actually understood what was going on, it got considerably easier to fix code signing related issues.

In contrast, nowadays many things are automated within Xcode. They don't work... they are not well explained... but they are automated :O.
So problems either get fixed super easily.. or really, really hard. And I'm not really sure about the first because it has never happened to me (ok, ok, adding a device to the portal is much more convenient).

### The example

My frustration with code signing reached its peak recently, when one of our provisioning expired. "It's fine", I thought, I'll just renew it. And for the first time in my life I used the "Fix issue" alert in Xcode for its intended purpose. I revoked the old stuff and created a new one. To my surprise everything seemed to be working, which is already weird. A little investigation encovered that it only worked on my machine, everyone else couldn't sign even after downloading the new provisioning.

![Trying to use Xcode's Fix issue wizard]({{ site.url }}/images/images/2016/04/XcodeFixIssue.gif)

Long story short, I discovered that revoking certificates now also generates a new private/public key pair in your keychain, And ofc... that means the old one can no longer be used and unless you transfer it to all other machines, your teammates can take one rather long coffee break.

My initial thought was that this is horrific! It's so error prone. And unless you take sick pleasure in reading release notes from Apple, you wouldn't even know what's happening. And knowing (unfortunately) many developers, suddenly private keys, using a passphrase like "1234" start flying around in Skype, Slack, email and.. I don't know... maybe even Facebook.

After my anger settled down, I decided that there must be a better way. I instinctively thought about teams. I knew the dev center supported multiple users, but to be honest, I had never been in a team that used them. So what are they all about?

### Teams in Apple Developer Center

The intention is that every person in your team participates in the dev account with his own AppleID. There are three tiers of permissions - team agent, admin and a member. Intuitively, the agent can renew memberships and work with payments and legal documents, a member can work with development certificates and an admin can work with both development and distribution. You can learn more about [team roles in the developer reference][team roles in iOS developer reference].

But the point is different. In that scheme, you generate provisioning on a team basis. This provisioning can be used with several certificates - one for each member of your team.

This is very powerful because now each developer is responsible for his credentials and certificates. Also you don't have to share private keys with others. Apart from an excellent security practice, this makes it very convenient to work with freelancers (because ofc they shouldn't have access to company private keys).

And that being said, those new changes in Xcode start making sense. Yes, if you go to Xcode -&gt; Preferences -&gt; Accounts -&gt; View Details, with only one mouse click, you can revoke a certificate and regenerate a private/public key pair, but you're only doing it for your personal account. Everyone else can continue working undisturbed.

It's so simple... and obvious... and yet I'm scared to know how many companies don't leverage teams in Apple Developer Center. Suddenly provisioning setup is a breeze (relatively... and considering the alternative). For detailed steps how to achieve that, please refer to [Maintaining Your Signing Identities and Certificates][managing certificates reference].

### Summary

Initially, it might be tempting to use only one iOS developer account across your whole team for provisioning. However, with changes to Xcode and the member center, it is becoming increasingly difficult to do so without encountering code signing issues from time to time. Namely, nowadays it's way too easy to revoke a certificate from within Xcode, leaving the old private/public key pair invalid. This in turn prevents other team members from being able to run applications on your test devices.

Introducing teams in Apple Developer Center can help prevent that by having each programmer sign apps with a different certificate, all connected to the same provisioning. Even if someone revokes a certificate, he only does so for his own account - everyone else's workflow remains undisturbed. This brings higher level of security and better organization within the team. Also, since this is the intended way, you get easier and better support from Xcode. You can turn a mammoth task such as provisioning, into a streamlined process.

To get started with teams in Apple Developer Center, go ahead and read the following [developer reference document][managing certificates reference]. For more information on permission tiers, refer to [team roles in the developer reference][team roles in iOS developer reference].

[team roles in iOS developer reference]: https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/ManagingYourTeam/ManagingYourTeam.html
[managing certificates reference]: https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingCertificates/MaintainingCertificates.html
