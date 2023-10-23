[Thursday 9:24 AM] Jason Helmick

Hello and welcome to the Spetember 2023 PowerShell Community Call -  where everyone is welcome!  We will get started shortly.

This meeting is recorded. Recordings will be upload about a week after the call.  You can find our past Community Call’s on our YouTube channel: https://www.youtube.com/channel/UCMhQH-yJlr4_XHkwNunfMog

Please remember our Code of Conduct: https://opensource.microsoft.com/codeofconduct/

Please review our contributing guide: https://github.com/PowerShell/PowerShell/blob/master/.github/CONTRIBUTING.md


Jason Helmick 2:25 Well, it looks to me like we have quorum. So let's go ahead and get started.
Hello and welcome everyone to the October. Yeah, it's October 2023 PowerShell community call. Thank
you all for attending. We really look forward to having this community call and we, as I keep saying
early on, we have an action packed agenda today. And so I put that agenda along with the link to the
agenda, and you can also drop in questions there or you can drop in questions into the chat. So
we'll go ahead and get started with that agenda today. I'm Jason. I'm going to be your host today.
Good to see you all. Here we go. So in today's agenda, one of the first things that happened, I'm
I'm so excited to finally be able to say, hey, Sydney, can you talk about PS resource get the GA?

Steven Bucher   3:15
Thank you for me.
OK, OK.
I think we were just figuring out our our meetings, Jason or Steven and are in the same room.
So hopefully you can hear us.

Steven Bucher   3:33
I'll put this camera on.
This is so confusing.
OK, well, anyways, we jade yay.
Thank you so much for all the support.

Steven Bucher 3:42 
As always, we've talked about this for so many calls, but we finally reached this
milestone, which is so huge for us.

Steven Bucher   3:50
So thank you again for all the support.

Steven Bucher   3:53
Really, really huge shout out.
Yeah, GA is general availability.
That's the 1.0 stable release, so really huge release for the team.

Steven Bucher 4:04 
Thank you so much to Amber and Aditya, kind of the primary engineers on it. And
then everyone else who supported this, whether through issues, PR's, or in any other way, it's taken
a lot to get here.

Steven Bucher   4:19
But we're at 1.0 and we're not stopping there.
If you're having curious what's next, we are planning a patch release and so you can see that project in our GitHub repo.
Please continue to try out the release with all of your scenarios.
Keep opening bugs for us.
We're continuing to triage them and we have kind of our next patch and feature releases.
Starting to manifest and get planned.
That's kind of all I had.
Oh, we'll preview 7 include PS Resource Get view one.
I'm not sure if this is later on in the agenda, so no spoilers, but the.

Steven Bucher   4:59
Uh, PowerShell 74.
Next preview, which is likely to be an RC, will include power, PS resource Gap, one dot umm we are hoping to get in a patch release of PS Resource get just a few little bugs before the PowerShell 74 GA so that's kind of our plan right now.

Steven Bucher   5:22
But yeah, in the RC for PowerShell 7 four you will see PS resource get one.

Jason Helmick 5:30 That's awesome. Sydney, congratulations to you and the entire team of Anam, Amber
and Aditya and everyone else that put in so much effort and time into this. It's been a long
project.

Jason Helmick   5:43
Please let us know what we can fix so and with that just as celebratory.

Steven Bucher   5:47
Yeah.
Thank you.

Jason Helmick   5:50
Let's go ahead and go to Andy and talk about that stable release of VS code.

Andy Jordan (THEY/THEM)   5:56
Good morning everyone.
It's been a hot minute since we did this.
I normally I would demo VS code, but this was honestly a big stabilization release.
I'm gonna go ahead and share a link to the release notes, but yeah, this is the first stable release we've had a VS code since our last release in June, so it's been a little bit a lot of that was team members were busy between vacation and be as comp vu and travel, but also, I mean this went through, I counted them up six different prereleases.

Andy Jordan (THEY/THEM)   6:24
So we've been releasing and thank you to everyone who you know subscribes to that pre release channel in the marketplace and test this all out because guess this was truly a big stabilization release.
The majority of these patch notes that I linked to are bug fixes like your CWD.
Setting now supports tilde.
If you wanna say the Home directory was tilde, that works.
It doesn't have to be an absolute directory anymore.
Same for the additional PowerShell EXE.
Setting a whole bunch of, just like quality of life improvements, we've also got the latest PS read line is in there now.
So we bundle version 234 of psreadline, the PS Editor API saw some bug fixes be introduced if you use that dollar PS editor variable, you can actually like call dot close file on it so that the API was there was a bug or two that we fixed.

Andy Jordan (THEY/THEM)   7:10
And then we also noticed that like 90% of several things have been wired up and not turned on.

Andy Jordan (THEY/THEM) 7:15 So we turned them on and tested them. Went oh, we should have had those
there. So close file save file. You can use those in the API. Now we had some great community
contributions. There's now like a start location setting for that extension terminal. Thanks Sir
community for that. And then a whole paragraph describing a bunch of bug fixes like around setting
error, action preference to stop.

Andy Jordan (THEY/THEM)   7:36
We found yet another bug where that could cause something, and we've fixed it.
We've found a race condition when you would change which version of PowerShell you were using.
The status bar item would an update till after it changed, which wasn't great because it should update as soon as you've asked to change.

Andy Jordan (THEY/THEM)   7:52
So we fixed that race condition for the final time.
The DSC breakpoints now actually work.
Those may have been broken for a long time for the majority of users because it was kind of hard coded to look for the DSC module in a particular C program files X86 directory that does not work, it just looks for them by by module name now, which then helped us find another bug in testing with the community.
Again, through these prereleases where there was some stickiness with like a loading thing that would happen once the feature was lit up.
So the feature works now and the bug around the feature was fixed and then like one more thing that happened in here just because it was a big release.
Our underlying library that Omnitrope library that we use for the LSP server.
It's the C language server protocol.
Like based abstraction, it went through a big update we took.
In that update, we found a bug in there update with some deserialization.
We had to work with them to get it fixed.
Roll in another update and that actually is great because that brings us up to the latest LSP spec, so we can now support 3.17 of LSP, which means that our communication between the extension you see in VS code and the server that powers everything underneath is like up to date and can support all the latest features.
Now we still have to implement latest features, but the big blockers out of the way.
So yeah, a whole lot of improvements.
We really, really appreciate people using those prereleases.
I don't know how many times I can say that in fact, like when we went to go release this to stable a month or two ago, we went to check our telemetry and was like how many people are using pre-release because that's what we want to know.

Andy Jordan (THEY/THEM)   9:28
And we may have found some stuff broken into inventory due to some upstream changes with like another team at Microsoft.

Andy Jordan (THEY/THEM)   9:35
Our telemetry just was getting completely dropped.
We had a fix that we now have our telemetry and we're able to see how many of you people are using our pre release channel and we're able to feel comfortable moving from pre release to stable.
So please keep that up and trust us when we say we check those stats.
We don't do a stable release unless we know that pre release has been out there in the wild.
Getting tested so yeah.
Thanks so much.
They use the stable release.
But if you really need that, but please keep that prerelease channel installed, you can always downgrade with the switch of a button.
Alright, I think that is it.

Andy Jordan (THEY/THEM)   10:12
Thanks so much.

Jason Helmick   10:13
Awesome, Andy.
Yeah, folks, keep the feedback coming.
You can tell that Andy's working really hard to deliver that stuff.
So welcome and congratulations, Andy on on on VS code Ohm Ohm man.
One of my favorite topics, these are all my favorite topics.
It's really killing me today, Steven.
You wanna talk about PS read line, the general availability of PS read line.

Steven Bucher   10:36
Yeah, absolutely, Jason.

Steven Bucher   10:38
So I just wanted to announce a couple weeks ago we released the GA version of PS3 Line 2.3 Dot four.
Like Andy mentioned, it's now included in the extension as well as on the gallery.
We're hoping to include it with seven, four release and not too much changes since the latest previews.

Steven Bucher   10:57
So majority of the improvements in the PS read line stable, latest stable release is around predictors list view predictors now can scroll.
You can have more than 10 results as well as it will now fit in your window, so it cool thing that will happen now is in cloud shell you can actually use listview predictors without it silently failing.

Steven Bucher   11:16
So JSON is very, very happy about that.
He's been waiting for that for a long time, so we're pretty excited to get that out.
We also had a bunch of numerous bug fixes and stuff.
Umm, there you and I will link A blog post that we were working on or blog post that, umm, highlight some of the other key changes.
Uh, lots of stability fixes you.
You might wonder also just we had a small regression in 2.3 dot three for older versions of Windows, but it was a very small set, but we wanted to make sure we get that fixed.
That's why we have two dot 3.4 as our GA version.

Steven Bucher   11:56
Just to clarify, any confusion around that, but yeah, similar to what Andy said, please give this a try.
Download it and have it used for your daily driver.
Submit bugs.
Give us feedback and like I mentioned, we're hoping to roll this into 7, four and so everyone will get that with the seven, four release later this year.

Steven Bucher   12:20
So I think I cover everything.
Big shout out to Dongbo Wang, who put in a lot of work for this release.
I think he's on the call, so I just wanted to give a shout out to Dongbo who who really put in a lot of effort with PS read line this year.
So I think that's all I have.

Jason Helmick   12:36
Well, a personal thank you from the entire cloud shell team and all of our customers.
Thanks for fixing that.
That's awesome.
Yeah.
And a big shout out to Dong, but we appreciate all of that work and now pen to the moment, I think that we all kind of wait for is the the the discussion around well, the reason that we're here which is PowerShell.
So let's go ahead and hand it over to Steve Lee and talk about 7 four of the RC.

Steve Lee (POWERSHELL HE/HIM)   13:00
All right.
And just to be clear, like this is the PowerShell SSH community call.
So yes, but the big news is we are heading towards the end of the calendar year and we are on track for our year release of an update to PowerShell 7.

Jason Helmick   13:04
Yeah.

Steve Lee (POWERSHELL HE/HIM)   13:13
So 74 should hit RC one later this month and then assuming that net themselves ship on schedule, then we'll follow similarly, barring any unforeseen circumstances and have a GA ideally in November.

Steve Lee (POWERSHELL HE/HIM)   13:29
And then we'll start.
So you 75 reviews.
Hopefully in December it kind of depends on who's available.

Steve Lee (POWERSHELL HE/HIM)   13:35
Uh, during that month and what kind and what kind of changes we've had.
So one clarity is the master branch in the partial repo right now is 75.
We are only cherry picking changes in A7.
4 Actually, since the last preview anyways, as we're trying to not introduce any new regressions, so look forward to that the the new piece read line and piece resource get are gonna be in the seven four which is our next LTS uh LTS mining support for three years.
So there will be 1 overlap year between 7 two which is our last LTS was 7/4.
That's pretty much it.
I guess I'm trying to think was anything else worth mentioning for 7/4?

Jason Helmick   14:12
It it ships with .net 8, right?

Steve Lee (POWERSHELL HE/HIM)   14:15
Ohh yeah.

Jason Helmick   14:15
It's gonna ship with, yeah.

Steve Lee (POWERSHELL HE/HIM)   14:15
So, well, yeah, with every release we moved to the latest version of .net.
So with seven four we are moving to.
We have already moved to net eight and I believe with our our COB on net eight RC two.
So again, that should be hopefully coming soon this month.

Jason Helmick   14:30
That's awesome.

Steve Lee (POWERSHELL HE/HIM)   14:31
So yeah, and and if you guys find any last minute issues in seven four, please open those up.

Jason Helmick   14:31
That's awesome.

Steve Lee (POWERSHELL HE/HIM)   14:37
I do spend Monday communities trying to find those things and what we triage it and you know there are gonna be some cases where we're going is too risky to fix.
But there are the cases where we can still have opportunity to fix those before we actually have the final version.
For some for yeah.

Jason Helmick   14:53
That's awesome.

James Brundage   14:53
Thank you for the clear expectation setting there, especially that is fantastic.
And so looking forward to new PowerShell Bits, it's Christmas. Almost.

Steve Lee (POWERSHELL HE/HIM)   15:02
Yeah.

Jason Helmick   15:03
It's it's it's Christmas.
That's that seems to be the tradition now, right?

Jason Helmick   15:07
It's the Christmas present to the community and as Steve mentioned, it's not just the PowerShell call, it's also the open SSH call and it just so happens that Tess is here to talk about open SSH High Tess.

Tess Gauthier   15:21
Thanks Jason.
Hi folks.
Just a quick update on open SSH.
We released the 94 GitHub release last week.
This was mainly focused on seeking the changes from upstream, but there are two things I just wanted to call out.
Specifically, the first change is within that stage agent and it's a change in the default behavior for adding PKCS 11 libraries remotely.

Tess Gauthier   15:46
This change was made upstream in response to a CVE that at a very high level, just demonstrated an exploit of sequentially loading specific Linux libraries.
Obviously there's some platform differences in the windows.
SSH agent already spawns a temporary process when loading the PKCS 11 libraries, but for a defense in depth measure we also incorporated the change in default behavior, so this only affects scenarios where the agent has been forwarded and a PKCS 11 library is attempting to be loaded remotely.
There's a command line argument to override this default behavior and continue to permit this if you wish.
That argument just needs to be passed in upon starting that stage agent, and for more info on that you can check out the release notes and then the other change I wanted to mention was with the fsserver.
So when starting the server, it will now check the permissions of the program data SSH folder just to verify that only users in the administrator group and system have owner or right access and any other user is only permitted read access on that folder.

Tess Gauthier   16:53
A similar check already exists in the install script, but that's only for the first time that the server is started.

Tess Gauthier   16:59
So now we're just checking anytime that stage server is started, just in case after a restart or an update that service has been stopped.
If invalid permissions on the stage folder are detected, the service will fail to start and it's kind of a generic system error.
So just keep that in mind if you are upgrading to 9 four and you encounter any issues with that Sage server, there's already an issue that's been opened and closed on the GitHub that you can take a look at for that.
And as always, thank you for your feedback.
And please just open an issue if you encounter any problems with the release. Thanks.

Jason Helmick   17:35
That's awesome.
Thanks, Tess.
Thank you so much.
And with the after the open SSH, now we're going to transition to my friend Damien.
Damien, are you online?
And we're going to talk about easy CLI and Azure PowerShell updates.

Damien Caro   17:52
Umm, yes, that's yes, I'm online.

Jason Helmick   17:56
The great.

Damien Caro   17:57
I. Hi everyone.
I wanted to share a quick updates on the work we're doing around the Azure CLI and Azure PowerShell command line tools.
We've been working on the on some features in the in the last few months to improve how we address some some scenarios with our client tools and what we've been looking at is under the cover.
So from the command line tool point of view, nothing really changed, but under the cover we are bringing better consistency between our client tools and portal.
So what it means for you as users of those client tools is that consistency with portal.
So when you deploy your resource, you will have the end state identical regardless of which environment or which client you're using.
It brings the item buttons here.
One of the biggest challenge we had is around you run the command again.
You want to have exactly the same state, and it's not necessarily the case everywhere, so we are bringing that idempotency here and that allows us to have.
What if, like the ability to know what kind of resources would be changed when you run the command against an existing resources for example?
So this is something that we've been working on.
It took a lot of effort to get there, but we are we have a preview, we have a public preview that we've made available not so long ago, but it happened.
I heard recently that it wasn't very well known, so I wanted to take the opportunity of this call to share a bit.
Broadly, the blog posts that we have that explains those features that we have implemented, as well as where to go, how to get the bits, how to try them out, and the most important thing for me and the team is how we get feedback, what is important for us is that you try out, but also that you let us know if you fail, if you find any problems with it, if you find any behavior that is not what you would expect, any issue that you face all up.
So we have Forman posting in the chat.
I go to the form after you've tried out.
Let us know what you think and our goal is to release this at some point in the client tools.
We don't have a strict deadline, but I would expect that probably in the next calendar year at some point in order to make sure that we have a very robust mechanism.
We've been trying with keyword because it's fairly simple to service resource provider and and we control that behavior from end to end.
But we're going to look at expanding that to other resources on the wrong run.
Uh, that's what I wanted to share with you again.
Read the blog post and let us know what you think about it.
This is the most important thing for us.
Thank you very much.

Jason Helmick   20:50
Thank you, Damien.
That's great.
That is really great.
Hey, Michael Greene.
Do you want to?
Let's go off agenda.
And do you want to say a few words about DSC?

Michael Greene   20:59
Yes, sure.
I was just putting in that chat, but we we didn't have it on the agenda for today, but I just wanted to let everybody know DSC is still making really good progress over in the GitHub repo.
It's just github.com PowerShell DSC.
Very regular check-ins.
You're welcome to go comment on issues and let us know there's a lot of very early for version three decisions being made on a ton of new features.
So as much feedback as you'd like to give us, even if you don't want to build it yourself and start tinkering, if you just want to look at the issues and kind of give us your uh, you know, this is how it would work in my environment that is greatly appreciated.
So thank you for any time you can put towards that.

Jason Helmick   21:40
That's awesome.
And yeah, I was just talking with Michael about a couple of things yesterday and I had to say, if you've been waiting to to, you know, let's let them get through some development before I start plunking into it.

Jason Helmick   21:51
Now's a great time if you want to be on the early portion of the DSC stuff and giving us feedback.
Feedback.
This is the perfect time, so if this is your space, now's the time to join us on it.
It so thanks, Michael.
Really appreciate it.
And with that, let's go over to my friend Sean, and let's talk about some docs updates.
How you doing, Sean?

Sean Wheeler   22:11
Good.
So as always, we have the monthly update.
What's new in docs in September, we did a lot of updates for all of the GA releases of the different things.
So crescendo and PS read line and.
The uh PS resource get umm some new feedback provider documentation?
Umm.
And I just want to also reiterate the all the work that's going on in the DSC space, you can get to the DSC docs from the top NAV here.
Umm uh.
Mikey has been a documentation fiend.
There's, I think, 93 total new articles between the version 3 documentation he has here and the DSC samples documentation that he's published.
Umm.
And you can see the source code for all those samples here in the DSC samples repo.
Umm I've got links for all of this in the chat.
Umm, I wanted to also point out a new Azure PowerShell Docs style guide that's been published publicly.
Mike Robbins recently added this.
This is builds on top of the PowerShell Docs style guide that we have over in in the PowerShell dock space and calls out the differences here mostly around.
When to use the interactive?
Uh, try it now.
Buttons and things like that.
Umm.
So if you're contributing to docs, be sure you take a look at our style guides and Speaking of which, contributing to docs, it is October still October, October Fest is going on.

Sean Wheeler   24:17
We participate in hackfest as well as the PowerShell code repo, so if you want to make contributions and they qualify will mark those so you can get credit for Hectograph Fest.

Sean Wheeler   24:36
And I I think that was my list.

Jason Helmick   24:41
Ohh really well and and and I just have to say everyone Sean is so appreciative of everyone that makes contributions.

Sean Wheeler   24:42
Thanks.

Jason Helmick   24:48
I mean, he calls me on the phone is like, look at this.
Look at.
This is great.
So this is a great time right now to be doing some contributions if that is your thing.
It's very helpful to all of us on the team to help us check our stuff.

James Brundage   25:01
Sean, is there a particular area you'd like extra contributions?

Jason Helmick   25:02
So thank you. Uh.

James Brundage   25:06
There's still half of October or October Fest left roughly.

Sean Wheeler   25:12
Umm, so we have let me share my screen again.

James Brundage   25:16
That actually let's not derail.

Jason Helmick   25:16
Sure.

James Brundage   25:18
You can you can take this offline with me.

Sean Wheeler   25:18
Well, just real quickly out in our contributors guide space, we have this contributing quality improvements and there's cleanup work.
We would certainly appreciate, UM.

James Brundage   25:34
OK, cool.

Sean Wheeler   25:34
So it, yeah, we we have some some guidance here about what we're looking for also in the issues, umm, there's at least one up for grabs here.

James Brundage   25:35
I'll take that and run with it.

Sean Wheeler   25:52
And I should probably mark a couple of more, but umm.

James Brundage   25:58
That one looks right up an alley of mine.

Sean Wheeler   25:59
Yeah.

James Brundage   26:00
I would love to give you some $0.02 on it, but I also already feel bad for wasting the Community time by asking so.
Well, let's keep going. Sorry.

Jason Helmick   26:11
It's all good.
It helps everybody.
So thanks, Sean.
That's actually really great.

Jason Helmick   26:16
Umm, so let's see if we can find Steve Lee again, and let's have him talk about some GitHub issue updates.

Steve Lee (POWERSHELL HE/HIM)   26:25
OK, so this is specifically for the partial repo, and there's a discussion that we've been having in the partial committee over the last couple weeks and the big.
So First off, we really wanna thank all the people have opened up issues.
OK.
But because of that, we have around 3500 issues in the partial repo and it's become basically unmanageable and it's very hard to tell what is actually important and what should we focus our efforts around, right.
And just by the way, I think the working groups have been doing a tremendous job pairing those down.
But even then, like 3500, and I'm not even sure what our incoming rate is, but it's been very difficult to kind of, umm, get it down to something that makes sense.
So what we decided is we are going to have a bit more aggressive resolution and closing of the issues so that we can hopefully just see what's remaining is the ones that need attention.
So what we're gonna do, and I believe we decided we're gonna enact this on November 1st, right?
So we're gonna.
We're gonna do it right away, but November 1st, we're gonna have a basically a bot where any issue that has had no activity for six months or more, we'll get a resolution tag of no activity.

Steve Lee (POWERSHELL HE/HIM)   27:39
And then along with our normal bot closing.
So basically any issue to this this won't affect pull requests.

Steve Lee (POWERSHELL HE/HIM)   27:46
So any issue that has a resolution tag and we have a bunch of like you know by design won't fix.
Answer whatever the case may be, then basically, after seven days a different bot will automatically close that issue.
All right, So what this means is that some really old stuff, if it's still has activity, then nothing will change for any old again, six months or more issue that has had no activity on it, it will get a label attached resolution, no activity.
And then after seven more days, if there's still no activity, then it will get closed.
Now, if the original person or anyone who is following that issue decides, hey, this is actually something that is important than what we would recommend is probably opening up a new issue, making sure that you try it with the latest version of partial 7 to make sure that it's still an issue.

Steve Lee (POWERSHELL HE/HIM)   28:32
And we really need to also understand like is it blocking?
Is it an inconvenience?
Things like that sort to help us prioritize how we focus our efforts and how the working groups can focus their efforts.
Umm so that is coming again November 1st and I believe our first estimate looking at the issues and the dates for them, we expect I think it was almost 3000 or so is gonna get resolved.

Jason Helmick   28:58
Yes, Sir. Yeah.

Steve Lee (POWERSHELL HE/HIM)   29:00
So what?
We'll be left with around 5 to 600, so that sounds like a big number, but to be honest, even 5 to 600 active issues is a large number and I expect that some people may decide to reply so that will affect the activity and those will stay open for now and people may open new ones, right?

Steve Lee (POWERSHELL HE/HIM)   29:19
So it's gonna be an ongoing process.
So just keep that in mind.
That's coming.

Jason Helmick   29:25
It's awesome.
Thanks, Steve.
Yeah.
We appreciate your help during this as we try to call out these issues that are not no longer important because of their age, but keep the ones that really are so we can do a better focus on them.
We really appreciate the Community's help in doing that.
So sure.

Steve Lee (POWERSHELL HE/HIM)   29:40
Well, let me add one more thing before we move off to stop it.
I've almost forgot.
So one other thing that really helps us and by the way, some of this will get documented within the repo.
I know a lot of people don't read the repo documentation.
Reactions to issues also help the committee and the working groups kind of decide what is actually important.
So one of the problems has been like people are using thumbs up to mean different things, right?
Like I agree or it has happened to me or whatever.
Sure.
What we'll try to define some more clear rules on which emojis mean what, but if there are issues that are impactful, then adding a reaction does help us know, like hey, even though this is an older thing, maybe we should still pay attention to it, right?
So.
So that's a very minimal thing to do is just click on that little reaction button and that gives us a small piece of data. Thanks.

James Brundage   30:35
I will try to be more reactive.
I would also lightly suggest that you differentiate blockers into hard blockers and soft blockers hard blockers or I can't do it anyway without POWERSHELL's help.
And soft blockers are well, I could do it with PowerShell help better, but I could figure out this work around and as much as I love PowerShell growing and improving, I'm able to accept a lot of soft blockers and I think a lot of other people should also be able to accept a lot of soft blockers and getting that differentiation for yourself as you create it might help clarify every issue authors real level of blocking so that we're not all like yes, I'm totally blocked.
You need to fix this tomorrow because we don't really.
Alright, that's two cents. Done.

Steve Lee (POWERSHELL HE/HIM)   31:20
Yep, I think that's a good call out.
Like we're work arounds are various shades of Gray.
Like there's also discovery problem, but those are taken into account as well.
If you're truly blocked and can't, we've moved forward with your business.

Steve Lee (POWERSHELL HE/HIM)   31:32
So, alright, thanks Jason.

Jason Helmick   31:35
Thanks Steve.
Now folks, I I don't know.
Have you?
Have you guys heard about this little tiny conference that's gonna pop up here around N15?
I think it's called ignite.
Yeah, that's it.
So Microsoft's getting ready to do their big Ignite show, and this year is probably the biggest one for Microsoft in a very long time.

Jason Helmick   31:54
So well, how?
What are we gonna do?
And so fortunately we've got Sydney and Steven to tell us what's gonna happen at Ignite.

Steven Bucher   32:03
Well, thanks, Jason.
Well, we're getting very excited for Ignite.
The countdown is definitely on.
We're less than a month away and we're very excited that we have two kind of opportunities to present as like official type sessions at Ignite.

Steven Bucher   32:20
So we'll be doing a demo on secret management and then we have a bunch of the team is gonna be there to do a discussion.
You want to kind of speak to that. Yeah.
So we'll be doing a discussion around kind of everything.

Steven Bucher   32:33
PowerShell 7 we'll be talking about kind of open source community stuff as well as enhancements that we're doing in PowerShell 7 as well as all of the modules, kind of an overarching discussion about everything that we're working on.
The we don't have a date yet of when those sessions will be.

Steven Bucher   32:51
We know it's gonna be during ignite.
We just know, don't know the exact date or time.
However, for those appearing in person, we would love to see you there.

Steven Bucher   32:59
Otherwise, these sessions will both be recorded and available on the free digital registration that you can do adding notes.
So yeah, we're are very excited to be at Ignite and we'll be expecting a lot of content kind of coming out around there.
Yeah.
And certainly if you are in Seattle during Ignite, please find us.
Please reach out to us.
We are gonna be there all week and are there to hang out with you.
Talk to you.
Hear about what you're doing with PowerShell and all sorts of issues that you're running into, I'm sure, and hopefully have some fun social time as well, so we look forward to connecting with you at Ignite, whether that's you know, virtually or in person.
Yeah, I think that's about it.
Yep, I think it covers it.
So thanks.

Jason Helmick   33:42
That is awesome.
I think I think even Sydney's gonna let me show up too.
This is gonna be a blast.

Steven Bucher   33:46
Yeah, we have passes for our whole team.

Jason Helmick   33:47
So yay.
So hey folks, if you already know that you're coming to ignite, give us a thumbs up.
So we just have some kind of idea that we have some of our our community call folks that are gonna be shown up, but we're going to be there.
So it's going to be a lot of fun if you're going to be making it there.

Jason Helmick   34:03
So we look forward to seeing everybody.
Oh hey.
Yeah, I got somebody raising their hand.
They're that's awesome.
So, umm, next on the list?
Ohh, this is actually easy.
Thanks guys for hanging around because the next one comes to you too.
So what are we gonna do about this November community called?
There's this thing like Thanksgiving in the way and stuff like that.

Steven Bucher   34:25
Yeah.
So well, actually Speaking of Ignite, so a well, the third Thursday of November is a is around the American holiday for Thanksgiving.
So a lot of us on the team will be out during that Thursday.
We tried doing it the week before, however that's also ignite, so we're looking at the last Thursday of November, which I believe is November 30th, uh, that we will have the November PowerShell Open SSH Community call then UMM and I think do you want yeah, so so we're planning on doing a little bit of a special call for the November call.

Jason Helmick   34:57
Is it the last team call for us? Ohh.

Steven Bucher   35:03
The plan at this point is to do sort of like a working group focused call and have like various Members from the working groups kind of present on what they're working.
Group has been up to and that sort of thing.
So you won't be hearing as much from the team and it won't be as much of like a traditional just like update on the things we've released, but more from the community and more from working group members.

Steven Bucher   35:23
Kind of recapping the year and that sort of thing.
So that's sort of the general plan for the November call.
And then I think last month we talked about using the December call as sort of giving it to the Community and having the Community kind of do whatever y'all would like to do with that call.

Jason Helmick   35:40
Ohh, that's awesome.
That's awesome.
So how do we know what the community wants to do?
For that call is there.

Steven Bucher   35:47
Just so we have a the I believe Ryan Yates open up an issue in our discussions tab for our community call.

Jason Helmick   35:47
Is there a place that way they can contact us or some?

Jason Helmick   35:53
Ah.

Steven Bucher   35:57
We also have, of course the discussions tab for our November call, which also gives the updated date there.
But I believe will be kind of sorting out and figuring out what the community wants to do in the December call in that location there.
So if anyone has any particular topics or things they feel strongly about to bring up or talk about or show off, definitely add it there. Ohm.

James Brundage   36:21
I might I.
It might be festively perfect, but I do actually already have smart light automation, including around twinkly lights.
So if we don't have anything else by default it gives me an excuse to actually set up Christmas lights and do PowerShell stuff with them this year.

Steven Bucher   36:35
Have be fine for sure.
So yeah, so look forward to two fun calls the last two months of the year.
Umm, definitely.
I'm just seeing some chat messages too.
If you are curious, Ignite is in the new summit building and umm, definitely we will be doing some sort of social event during the night for PowerShell people.
We're just waiting on the final Ignite schedule to come out to kind of announce when that might happen.
Yeah.
And if you're an MVP, there's a number of MVP events going on outside of the normal Ignite session that you should keep an eye out.

Jason Helmick   37:02
That's.
Awesome.
Well, thank you guys.
Thank you very much.
And with that, we're now going to go to our community announcements demo and all that kind of stuff with the time that we have remaining.
And the first step that I have and actually this is the only thing that I have, but we we'll figure it out.
We're kind of scrappy as we move along here.
James.
James Petty, are you on?

James Petty   37:30
Yeah, I'm here.

Jason Helmick   37:32
Oh, great.
Let's talk about summit, man.

James Petty   37:34
Yeah.
So the PowerShell and Dev OPS Global Summit, it's gonna be in Bellevue.
We had to move it up a few weeks.
I don't know miss's on the call.
Call was something about a high school graduation, and I was.
I was threatened very heavily, so I had to move it.
So April 8th.

Januszko, Missy   37:48
It is not my fault.

James Petty   37:50
So April 8th, 8th to the 11th, we're moving back to the Maiden Bauer.
But right now, the CFP or the call for papers is open and while we are tracking to have, you know, a full agenda and misses missing the Committee Content Committee, there, jobs are already going to be tough, but we want to make sure we get as many kind of papers and stuff as possible so that we can make the best just so we can have the best conference and these and the people here on the call are the ones that we want to be able to join us.

James Petty   38:19
You all have something valuable to share, so we wanna hear.
We wanna hear your story.
So if you have any questions about, hey, will this make a good will this make a good presentation or is it something that you wanna hear about, feel free to shoot me a message or some of the other content Committee members will be happy to kind of work through those for you.
The deadline is November 15th, so you got just under a month to get those pencils sharpened.
And also we're bringing back something we haven't done this I think since 2016, but we're bringing, we're doing half day conference and hands on labs on Monday.
So we're kind of changing the schedule up a little bit.
So instead of traditional 45 minute, 90 minute, I think these are three or three and 1/2 hour and 1/2 day deep dives.
So we already got a lot of those, but we definitely need a few more of those as well.

Jason Helmick   39:07
Well, that's awesome.
That is awesome.
Very exciting.
And also, I'd like to point out a message that Sydney had just put in.
Also, hope to see folks at the PS Comp EU mini Con next week.
Sydney, do you have any more information about that?

Steven Bucher   39:21
Umm, I'm sure if you go to I don't have the link handy.

Jason Helmick   39:22
Sorry.
OK.

Steven Bucher   39:26
These are my hands, but I do know that it is on the 24th, which is next Tuesday.
Ohh, there we go.
We've got a link free to attend.

Jason Helmick   39:35
Justin Grote.
Thank you, Sir.

Steven Bucher   39:37
It's all virtual.
Think we've got some great sessions lined up?
I think it's about 1/2 day conference taking place in the evening time and Europe.
So for any West Coast, umm, US folks, it's gonna be in the morning on Tuesday looking forward to connecting with folks.
It's on the gather platform, which is kind of like fun.
You get to like, walk around and kind of see people gives conference vibes.

Jason Helmick   40:00
Yeah.

Steven Bucher   40:01
So yeah, it should be a good experience.
And yes, also don't wanna not plug.
Also very excited for PS Summit in April as well, so.

Jason Helmick   40:12
It's awesome.
Thanks Sydney.
Already I'm folks real quick.
I just wanna check with the PowerShell team if there's anybody here on the call of the PowerShell team that has anything else that they'd like to let the community know or say anything about before and folks, I've been watching the chats I I haven't seen necessarily any questions that we haven't answered yet, so feel free at this time to chat in some questions.

Jason Helmick   40:39
Also, we have a few extra minutes.
If there are, as if there's one or two people that I'd like to do, a really quick demo, we might be able to squeeze those in.
If you'd like to type in what you'd like to do, since we have a couple of minutes but just checking with the PowerShell team, is there anybody anything else?
Any other announcements?
And I'm not getting an immediate responsible.
I'm gonna take this opportunity to make one for the cloud shell team.
Uh.
Hopefully, maybe next month.
I'll even get a chance to demonstrate this where y'all, but we are looking forward to coming out with two new things over the next month.
One of them is ephemeral sessions.
Ephemeral sessions don't require that you have a storage account.
It would ephemeral sessions basically means is that you don't have persistent storage in cloud shell and if you don't need persistent storage, this is a much lighter weight way of doing it.
Umm, we've made a lot of VNET improvements along the way, but the other thing that we have coming out is a new UX.
Basically a new toolbar to get us ready for some new additional features that we'll be making announcements about down the road. So.
I'll be happy to show you those new features as soon as they come out, and that's the only thing that I have.
So with that, let's see here, let me check for questions and I I'm not looking at the issue or the discussion page itself.

Jason Helmick   42:04
So if somebody wants to check that could help me out here with questions right now.
I'm not seeing any love the ephemeral session, I think you.
What is PSE?
What am I missing here?

James Brundage   42:19
A PowerShell editor Services I think.

Steve Lee (POWERSHELL HE/HIM)   42:48
Alright, I see a question from Chris.
The very common question basically I think the question is basically when is Windows gonna get partial 7 inbox?
So I there's no update on this at this point.
This is the same answer I've had in the past.
Basically, net meaning net not .NET Framework doesn't have to support lifecycle of that is required to ship in Windows itself.
Windows requires at least a 5 + 5, and if it's an Azure, they add more years for support.
So basically, that's the blocking factor for us to ship partial 7 and Windows.

Steve Lee (POWERSHELL HE/HIM)   43:18
But you know, we've made a lot of effort to make it easier to install partial 7 on Windows, right?
You got winget, you got the Microsoft Store?
Umm, we also have the new install PowerShell command.
And beyond, we have like Microsoft update.
So we're like almost in Windows, but not exactly so, and that's unfortunately how it's gonna be for some time.

Jason Helmick   43:44
Thank you, Steve.
So what other questions does everybody have? What?

James Brundage   43:52
Well, I got some high level community goings on to kind of report, although I wish I had already created the invite.

Jason Helmick   43:57
OK, go ahead, James.

James Brundage   44:00
So we just finished up month two of the Pacific PowerShell user group.
We had a Anthony Howell come and talk powershell.net that was fun.
That was great.
Ohm.
Next month, we're gonna be having Bruce Payette talk Braid and PowerShell, which should be really fun.
So that's gonna be October 8th.
I'm sorry, November 8th, not in the past.
Sorry, November 8th, 6:00 PM.
You can go to meetup.com Pacific PowerShell user group and sign up there.
We'll be getting the link up shortly.
We'll also be uploading this month to YouTube, so yay.
I also I will continue to do the nudge, nudge, Wink, Wink, January, February still unbooked PowerShell team members.
Got cool things you wanna talk about with the community?
We will want speakers, so nudge, nudge, wink, wink.
Say no more.
Umm, that's that's the big one on that front.
I can always jump into a fun demo or two.
I would love to talk through some of the stuff that Steven both the Steve, Steve Lee and Steve Bucher and I have been talking about in terms of shell improvement.

James Brundage   45:13
There's also some really cool PR's coming down the Pike that we could discuss.

James Brundage   45:18
This is the last 15 minutes though, and it's meant for all of us, so if I'm the only one running my lips, it's a big problem.

James Brundage   45:25
So shutting up now.

Steve Lee (POWERSHELL HE/HIM)   45:25
No, no, there's there's someone D kattan.
I don't know the first name says they have a demo if you wanna.

Jason Helmick   45:32
Oh, yeah, so let's go ahead.

Darren Kattan   45:33
Yeah, I'm in.

Jason Helmick   45:34
And Darren, let's do it.

Darren Kattan   45:34
All right.
I'm Darren. Hi.
Alright, so I'll share my screen and show you guys like kind of what I've been working on here.
Ohm, where is it?
Ohh, it doesn't let me.
I think you have to allow me to share my screen maybe.
Where we at here?

Jason Helmick   45:54
Ohh.
But.

Darren Kattan   46:05
I'm really curious what you guys are going to do for Intellisense.

Steve Lee (POWERSHELL HE/HIM)   46:06
Alright you you should have presenter right now.

Darren Kattan   46:09
Yeah, I see it.
OK, cool. Perfect.
OK, so I've been spending a lot of time with dynamic parameters recently and I did look it kind of neat.
I built a little web form around so I was kind of inspired when I discovered the notion of show command like I didn't know this existed.
Did you guys know that this existed?

James Brundage   46:36
So I did.

Darren Kattan   46:36
Where like a a formal pop up with all the parameter sets.

Jim Truher   46:39
It doesn't exist on.

James Brundage   46:39
But I always wish it was better and less like tied to the ISE, like I really always wanted to show command in browser.
Built a few of them over the years myself, but yeah, I I knew it existed.
I just wish it was not so locked to the ISE and nudge, nudge, Wink, Wink, PowerShell team.

Darren Kattan   46:54
It's not.

James Brundage   46:57
Give show commands and love.

Darren Kattan   46:58
It's it's.
It's not what I see.
Look, I'm in PowerShell 7 right here.
Just a regular terminal and it spawns it so like that was cool, but this one really powerful enough for what I wanted it to do.

Darren Kattan   47:09
So what we did is we made our own of our own version of this in the web.
And so with this lets me do in my little script editor is go in and actually get real time data back.

Darren Kattan   47:24
So OK, poor example, let me comment out this.

Darren Kattan   47:28
Now I'm running a constraint language mode and so I can't instantiate a lot of types that aren't considered core types.

Darren Kattan   47:35
So on the left you would have like the parameters that exist.
So this is our my little debugger pane, all right?

Darren Kattan   47:40
And if I were to drop, like a real parameter in here, I could do my param and that would show up as like a text box by default.
If I were to refresh a little parameter guy, I can make it mandatory using all your standard attributes, right?
So if I were to do, let me get this in the right spot parameter like this and then mandatory.
You know it'll do, by the way, did you all know and editor services by default?
You can't like control space in.
Get the attribute like properties.
I had to add the tab completion plus plus to get that anyway that should be built in.
Regardless, I'm gonna make this mandatory.
And I'm gonna go here.
That's gonna make it like, whatever.
OK, I could also built in a lot of like you know other helper things like drop down attributes.
So I could say like make this drop down based on the output of a script block or whatever, but for the sake of argument and what I'm trying to get done here, I made these a wrapper around the creation of the runtime parameter dictionary and then I made a wrapper commandlet around the new runtime defined parameter and then ultimately if I were to refresh this again, this goes in just does like get command that it binds in whatever values I have here and it'll let me you know select things.
Now this is actually point from API which I thought was kind of neat right?
So now you could have a form that comes from an API.
That's kind of neat, but what if I wanted to be able to select something from there?
OK, so let's say this isn't that edge versions, but let's say this is what would you call that like stable Canary?
Let's call it branch, right?
So I'm gonna rename that to branch and then I wanna see what the available versions are for that given branch, right?
So then I could take this and I could say, alright, well then the releases are gonna be basically the branch that was selected.
So I could say all right edge versions where.
Uh, let's see.
Let's go.
Look, I'm putting this the variable here, so I could say, OK, they have product Canary.
I guess I could say where product equals branch right?
And then that's gonna be the releases.
OK, so here instead of I'm going to provide the the valid options over here, so I could simply say.
Uh.
Valid values now becomes the what I call it releases right?
Anyway, so what's the problem?

James Brundage   50:19
Uh.

Darren Kattan   50:20
I have here.
Alright.
And I put in a PR for this, but I cannot get community buying, which is why I'm bringing this up.
Is this is all great and it works in my editor because of the way that I'm abusing the parameter binder.

Darren Kattan   50:31
But if I were to actually make this into a function and try to call it, it wouldn't work because the way you have to return that parameter dictionary means that the value of branch is never bound.

Darren Kattan   50:44
So this condition of like oh I need these versions that are based on this branch you selected in the previous dropdown.
I'll never get it, so if anyone's trying to do things with dynamic parameters where the value of 1 parameter controls the available values on the next parameter, I really need you guys to show your support for this.

Darren Kattan   51:06
For this PR you just look for real time parameter binding.
It's in there.

James Brundage   51:10
Yeah, I this is really cool work.
Adding a little bit of extra bonus points here.
I think that beyond just the real time dynamic parameter binding, we should really as a language be better with mutually exclusive parameters and conditional parameters like right now you're either really complicated dynamic parameters or bust if you wanted to have a parameter that only sometimes shows up when other things are provided or when the engine is in a certain state.

James Brundage   51:44
I think that if we figure out a good way to crack the conditional parameters, not we'll open up a lot more capability for PowerShell overall one and two.
Umm.
Real time binding would be really cool.
To deal with dynamic parameter is a lot and every time that I end up dealing with dynamic parameters I end up rewriting the same like 5 lines of call stack peeking to go look through my AST because it's the only way I can actually get a sense of what things will be bound to before they bind and being able to do that.
Sort of conditional emitting of of values would be really, really handy for the scenarios where there is a lot of ambiguity.
Umm so that that would be really, really awesome.
I'm I think I've already given a couple of cents into the PR itself, but I I think that it would be a really big win to have both this and to invest in how the hell we do.
Dynamic parameters are not dynamic parameters.
Conditional parameters better.
So that's that's one big one.

Darren Kattan   52:53
Yeah.
Anyway, that's what I got.
That's my demo.

Jason Helmick   52:56
Hey, Darren, that's great.
And I I would I I just wanted to alert you to James Petty leaving you, dropping you.
A comment in the chat.
I think he's trying to get you to present that at PS Summit, which would be awesome.

Darren Kattan   53:04
OK, let me see.
Oh, wow.
OK, sure.

Jason Helmick   53:07
Yeah, that'd be kind of cool.

James Brundage   53:09
Uh.
What about parameter sets 2.0 GB Lewis at external?

Don Hunt   53:12
Umm.

James Brundage   53:16
What?
What?
What do you mean parameter sets 2.0?
I am failure to parse here, although I don't entirely disagree.
Uh, OK.
Moving a bit on for that.
If you guys want other light demos, I keep making this nice little PowerShell module called Posh that keeps making the shell experience better, and I did add a few bells and whistles this month that I could show off if nobody else has anything, but I continue to say community first.

Jason Helmick   53:41
Would.

Jason Helmick   53:47
Tell you what, James, if you can do it in, if you can do it in 4 minutes, it's yours.
But at 4 minutes I'm gonna have to call you Sir.

James Brundage   53:52
I can do it in 4 minutes.

James Brundage   54:07
OK.
Maybe next then 30 seconds, OK.

James Brundage   54:14
Uh, back over to the meeting.
Start share.
It looks like I do not have rights yet.

Jason Helmick   54:22
Ohh, I'm trying to give you permissions.

James Brundage   54:26
So OK, I can I can wait.
It's a relatively quick one.
Although I will keep going through the basics if anybody hasn't already seen it.

Jason Helmick   54:38
And I don't seem to be able to find where I can give you the permissions.
If somebody can help me out.

James Brundage   54:43
I think that might be a Steve Lee problem.
So that's OK.
I'll be here next month too.

Steve Lee (POWERSHELL HE/HIM)   54:49
No, I I should not be the only one who can give permission.
You just have to hit the the three dots, find the person.

James Brundage   54:54
I hope not.

Steve Lee (POWERSHELL HE/HIM)   54:58
Let's see.
James Brundage shift to three dots.

Jason Helmick   54:59
Is it?
Is it just allow camera?

Steve Lee (POWERSHELL HE/HIM)   55:01
No, make a presenter.

Jason Helmick   55:03
Oh, I don't have that.

James Brundage   55:04
Yeah.

Steve Lee (POWERSHELL HE/HIM)   55:05
You don't have.
Well, I think Sydney and Steven also Jason, we can think this offline to get you permission to do that.

Jason Helmick   55:06
Yeah, we'll figure it out.

Darren Kattan   55:09
Oh, there it is.

Jason Helmick   55:11
OK.

James Brundage   55:13
That's OK.
I also can wait till next month.
We said only three minutes and we've burned two of them on setup, so back to you, Jason.

Jason Helmick   55:18
Well, we're yeah, we burn two of them.
So if you don't mind James, wait until next month, we'll just do it next month.

James Brundage   55:22
I I don't mind.

James Brundage   55:24
Posh keeps getting cooler every new release.
So you know it'll be cooler next month too.

Jason Helmick   55:30
That's awesome.
Then let's do that, my friend.

James Brundage   55:32
Alright.

Don Hunt   55:34
So.

Jason Helmick   55:34
Well, with that, is there any last minute questions before we go ahead and wrap up this call?

JH (Guest) left the meeting

Jason Helmick   55:44
Oh, I don't see anything.
So let me just say everyone, thank you so much for all your support for the to PowerShell.
We hope we're we're being as open and and providing you with the information to help you make decisions about where you're going with PowerShell.
We love talking to everybody, so we look forward to seeing you in the future at the the the November community call, which remember I think Steven said is November 30th.
So that is a change from our normal lineup, but we look forward to seeing you then until then, hope to see it Ignite.
Have a great month, everybody. Bye.
