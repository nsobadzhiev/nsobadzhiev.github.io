---
id: 384
title: Separating UIStoryboard
date: 2015-06-16T11:09:35+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=384
permalink: /ios/separating-uistoryboard/
categories:
  - iOS
tags:
  - Interface Builder
  - ios
  - separate UIStoryboard
  - storyboard
  - UIStoryboard
---
Lets get serious about storyboards. In today’s iOS (and Mac) development scene they are an integral part of every new app. Of course, there’s a lot of debates regarding UIStoryboard’s shortcomings and whether or not it makes your programming life easier. But even if you happen to think that they are a buggy mess and even if you’re able to convince your team against using them, you must admit that they will pop up from time to time. So you better be prepared to work with them.

## UIStoryboard in source control

Ever since UIStoryboard came out in iOS 5, it’s been notorious for having a lot of issues. Now, a lot of time has passed since iOS 5 and we have to give Apple credit for putting a tremendous effort into fixing many of them.

But one rather big one remains. That’s using a UIStoryboard in a team. I know, I know... It’s true that there has been loads of improvements in that regard in recent updates, but I personally feel like this is still a serious issue. So serious that developers are still being put off from using UIStoryboards by it. And I cannot blame them.

## The problem with UIStoryboard and source control

For those who are unfamiliar with what I’m talking about in this article, here’s an explanation. If you’re working on a storyboard by yourself, everything is fine. But once you get several people working on the same project, things quickly turn bad. Since many applications just have one massive storyboard file, it often happens that several people are working on it at the same time.

So when your work for the day is done, and you want to commit the changes to your remote repository for all other team members to enjoy, disaster strikes. A colleague of yours has changed and committed the storyboard file in the meantime. And now your source control system (be it SVN, GIT, Mercurial or something else) is reporting conflicts.

Now, think about what a UIStoryboard is. In your project tree it is this massive XML file using a scheme that you can hardly fully understand. Moreover, most times Xcode tends to modify sections of this XML that you never made changes to. Now, what to do? You don’t know which changes are part of the functionality you want to commit, which ones are your fellow colleague’s work and which are just Xcode being silly. You can try to merge the file manually, and often this works fine. But other times it can corrupt the file or lose changes someone made to the storyboard.

And to be honest, even if your source control system manages to automatically merge the file, there’s no guarantee that everything’s fine.

So, even for small teams, maintaining version control on a single UIStoryboard turns into a major hassle. It takes more time to keep the file in sync between the local repositories, than to actually introduce new features to it.

## Keep conflicts to a minimum by separating UIStoryboard into several files

Most people consider that a storyboard is a single place for your user interface definition. And they’re kind of right. But it doesn’t have to be that way. Nothing can stop you from having several UIStoryboards. What I’d recommend is having a separate UIStoryboard for all your major app components and features. For example, a Sign-in storyboard, a settings storyboard or a chat storyboard. Whatever makes sense for your apps. Just look for a few view controllers that tend to stay close together and implement a common feature. And put them in a separate file! That simple.

## UIStoryboard source control solved

How does that help your team? Well, since your user interface is segmented into several different UIStoryboards, it makes it harder for conflicts to arise. And since one storyboard file contains UI for only one feature, it is unlikely that two developers would be working on the same storyboard at the same time. It will probably still happen, but hopefully much less frequently.

## A Problem with separating UIStoryboard

Moving stuff around in the storyboard file is not as easy as it should be. In fact, everyone how has ever tried to move a view around in Interface Builder, knows the struggle. As soon as you move a view to another place in the view hierarchy, it tends to lose its constraints. And constraints can be a lot of work to recreate.

**Note:** *Things get a little better if you use the “Embed in View” functionality in Xcode (Editor -> Embed In -> View). But you’ll probably still have to manually add many of the constraints that you had.*

The same happens with view controllers. You CAN copy/paste view controllers across storyboards, but as soon as you do that, their views lose constraints. I’m not sure why it would be that difficult to “transplant” them to the new, place, but that’s how things are at the moment.

### A solution?

I’ve read about people having relative success in transplanting constraints by opening the storyboard/XIB as XML and adding all constraints as child elements to the new container. When you do that, you need to be careful to change the IDs referencing  relations between elements.

However, that’s hardly an easy and simple solution. In fact, I imagine it’s rather cumbersome and error-prone.

So, **what I’d recommend is making a copy of the storyboard file. Just duplicate the file and add it to your Xcode project. After that, open the copy and delete all scenes apart from the ones that are part of the module you wanted to separate**. Do that for all sections you wanted to separate your UIStoryboard into and voila! You’ve got yourself a separated UIStoryboard. 

It’s a pity that we need to resort into such... unorthodox methods in order to make UIStoryboards manageable, but hey... it least it works.