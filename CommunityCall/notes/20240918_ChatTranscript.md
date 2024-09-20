# PowerShell_OpenSSH Community Call-20240919

September 19, 2024
31m 8s
 
Steven Bucher   3:07
Alright, Jason posted the the agenda in the chat and.
A reminder of our where you can find all of our past community calls, our code of conduct, a contributing guides and agendas, and.
A place where you can put the questions and stuff, so feel free to put the questions there.
We'll start off with a plat EPS update from Jason.
Jason, do you wanna take it away?
 
Jason Helmick   3:37
Sure. Thank you.
Let's see here.
Oh yeah, so apply DPS update now I've I actually want to talk about this for a second and I have a little little demo to to show you, but let me give you the TLDR real quick. We're going to ship platy PS by the end of the year.
But hang on, I got more details.
So first of all, those of you that already know Platy PS probably.
I hope you find this to be welcomed and exciting news. Those of you that don't know what planet PS is let me kind of give you a couple of the.
Problem statements that.
Plan EPS is looking to help out with first of all, one of the biggest problems.
And this is what the the the foremost I think problem for planet PS is is that if you're a module author and you want to add help to your commandlets and stuff that you've been making, that's great.
You can use comment based help but if you want to have updatable help like the cool kids at Microsoft kind of thing, the PowerShell updatable help system.
Specifically, wants you to write that help into maml now.
I don't know if you've seen mammal or not, but let's just say that it's very hard to author in and you don't want to do that.
So mammal is one challenge that you have to overcome. Another challenge is and I I just kind of throw this out.
If you've noticed those beautiful PowerShell documentations on the learn platform that that Shawn and Mikey and Mike worked so hard on.
With all of the PowerShell help in order to render all of that, what has to happen is all of the help has to actually get generated not into mammal, but into YAML.
Now for the publishing system to be able to publish it.
Now that's that's easy.
Author, but still not exactly quite a whole lot of fun.
So with the solution here and what platy PS tries to do is is to try to make your life a little bit easier with this.
And that's you can author in Markdown.
That's easy to write in and easy to deal with, and then what plan EPS is going to do is transform that schema based markdown.
To well, in our case mammal and YAML.
But I'll I'll get to that.
It's it's even cooler, but and that doesn't really sound like a hard problem.
To solve. But when you start to add scale to it and we and this is the kind of the cool part. When you add scale to it, it becomes a little bit more challenging.
See. It turns out that we use Platy PS ourselves.
We use it on the learn docs pipeline. In other words, not just for all of the PowerShell content, but all the teams that produce PowerShell content planning PSS.
Job is to make those conversions so that we get that beautifully rendered YAML and we get that mammal into cabs so that you can have that as downloadable.
Data will help.
So every night this process has to go on on a larger scale.
So there are features that you want on a planet PS.
There is things that we want out of planet PS.
Let me just kind of give you what we did. First of all, if you go out to, I'm going to drop a couple things into chat. If you go out to the PowerShell gallery and you look up Platy PS, you're going to see that the current version for.
That is play DPS version 0.14.2.
Let's now refer to this as classic version of planet PS. That's gonna continue to work.
It's gonna continue to be out there. Life is good.
Everything's fine, but here's what we're gonna do.
We're gonna release.
Well, let me just drop it into chat here so you can see it.
We're gonna release Microsoft dot PowerShell dot platy PS version 1.0 and some of you might be going well. Wait a minute. Why didn't you just take the classic planet PS and just release that as one?
Well, let me put it this way.
Powershell.net PS version one this is going to be a breaking change, so hang on. What are you going to get for this?
Well, what we did as we sat down and we re looked at this problem and we did a redesign from the ground up on planet PS 'cause.
Like I said, not only do you need to use it, but we need to use it too.
And we focused on three well particular points that I want to kind of talk about or show you real quick.
Those were performance, scalability and usability.
Now when it comes to performance and scalability, that was a big focus. And let me just say it's faster than grease lightning.
But here's what I want to try to do.
I want to try to show you a demo so it just so happens that Shawn Wheeler and I were doing some validation work this morning on Platy PS and we just happen to record this. So there are two things that I want you to see out of this.
That we're going to do, we're going to grab hold.
We're actually going to import all of the PowerShell modules so that we can create.
All of the markdown using Platy PS.
Those modules now at that point with that markdown, you could edit it and all that kind of stuff, but then in the next example, we're going to take all of that generated content and pretend that we're going to send it to our publishing system and convert it all.
To YAML and what we're going to do is we're going to measure command on all that. So you can see what the time is.
Let me go ahead and share my screen and see if I can run this without screwing this up too much.
Here we go here.
So let me just go ahead and start this.
And what you can see here is this is where we're grabbing a list of all of the PowerShell core modules so that we can quickly grab hold of them.
And if you take a look, we're doing a measure command.
We're gonna take that module list.
We're gonna make sure that we're selecting the unique ones, cause Sean and I both have multiple versions of the same thing. And then we're gonna generate the new markdown command help for all of that into a folder.
So as soon as this guy, oh, there it goes. It's starting to run.
And it's gonna run through. It's gonna do that.
And what I want you to notice is.
How long it takes for it to go from zero to hero and generating this markdown content took about 7 seconds.
And it's pretty cool, isn't it?
I mean that's I think that's cool.
And here's where we're going to take that generated content like we have to do at night and make sure that it gets.
Turned into XAML or in your case, if you're a module author, converting it to mammal and as you can see it takes well about a second at scale to do it so.
Kinda Cole. Kinda sorta cool.
What do you think?
Is it kinda cool?
I don't know.
I think that's kind of cool.
Anyways, let me also talk about the usability side.
So not only do we work on performance and the scalability of it, one of the most important things was to focus on was usability. And if you've worked with Platy PS in the past, you may have kind of noticed the commandlets were a little bit weird you had.
Full of cmdlets and each cmdlet did a lot of stuff and sometimes it made things confusing and reduced the flexibility of what you could do.
So part of the redesign and the reengineering is all the commandlets got reworked before.
They were rather monolithic.
Now they're very monadry and what I mean by that is each commandlet does one thing.
It does it really well. And what this does is really opens up the flexibility for your scenarios in the way that you can use this. In fact, you might find that while we've set up converters to mammal and YAML.
You're getting this is now.
All on the pipeline and we now generate a complete command help object that moves through the pipeline.
Hey, you can do your own thing with that.
You can make your own converters. You might find other scenarios to use this with so part of making the commandlets very monetary was to open up that flexibility in those scenarios for you.
Now let me get to the part to when does it ship?
Well, I said at the beginning that it'll ship by the end of the year. But let me kind of be honest with you.
Umm.
The my engineer on this during the redesign has been Jim Truher and he's retiring next month. So guess when we're going to ship? This also gives me a great opportunity to. Thanks Jim Truher for the amazing work on Platy PS. But also just his entire career working on.
PowerShell. So I want to thank Jim for all of that and for the great work on Platy PS.
For some more information on this, now look, we're going to have when we release.
Been chomping at the bit to release a whole bunch of documents on this, plus scenarios all that will have this great blog post announcement, but in the near term and I know Sydney's going to talk about this in a little bit if you happen to have a.
Chance stop IPS Conf EU the mini conf that's coming up. Like I said, Sydney's going to talk more about it.
Jim's going to be doing a discussion about the design and the design changes.
You know the internals in detail, so that'll be a great place to show up and to learn more about.
Some of the changes that we made to platy PS.
So look forward to showing it off for you more later after we get it out and we released some documentation.
So thank you very much. Thanks Steven.
That's all I got.
 
Steven Bucher   12:40
Awesome. Thanks Jason.
I think we're in.
The Next up is Sydney for the PS resource get release.
 
Sydney Smith (She/Her)   12:48
I'll take the next couple of items.
They all go together, so I did want to bring up that.
We had a recent release of PS Resource get.
It was a preview release.
I'll put the release here in chat. One call out on this release. It would be great thing for folks to go ahead and test and give us feedback on while it's still in a preview stage is a new commandlet and feature that we introduced in.
This release, which is called compress PS resource.
And this pairs with new functionality in published PS Resource which allows you to compress your module into nupkeg form and then pass that. You can, you know sign then your knob tag and pass the sign nutkeg into publish PS resource and then pass it on to your end.
So that's kind of a new functionality of published PS Resource to be able to take in.
A compressed module rather than a path to a folder.
So that's that's now in preview. So go ahead and check it out.
Can we get expand PS resource?
That's that's what install does.
So install PS resource expands it.
And since install already works with file systems, I don't think that there's a need for expand.
But if you have a compelling reason, feel free to open an issue James and we can talk about it.
Second thing that would also be great to give get feedback on while we're working on it is we're currently.
Thinking about.
A future that's been asked for for a long time, which is how can I, at scale register repositories and also sort of create an allow list or slash deny list.
Right now we're just thinking about it in terms of an allow list for the repositories.
And so we're solving this initially with group policy.
Have an issue open.
Kinda describing the approach we're planning on taking. We're starting development on this very soon.
So if you do have feedback on, it would be great to go ahead and read the issue.
Give us some feedback. And while we're in preview stage as well would be great for folks to test that out and give us feedback as well. If you think that would be.
Useful as well.
And I think those were my two items. So I will pass it on. I think, Michael, you might be up next on the agenda.
 
Michael Greene (POWERSHELL)   15:20
Yeah. Thanks, Sidney.
Mine is actually really short.
I just want to remind everyone that PowerShell 7.5 as work in progress is coming near GA, so we expect later this calendar year that PowerShell 7.5 will reach GA and will be the next shipping version of PowerShell. But perhaps more importantly, a reminder that.
PowerShell 7.2 will reach its end of life on November 8th of this year.
So as already transitions.
Towards 7.3 or 7.4 if you're on the long term stable branch that 7.2 will reach end of life and we welcome the new release of 7.5 and I think that's it for me.
I will actually hand it back to Sydney to discuss Minikon.
 
Steve Lee (POWERSHELL)   16:10
Has his hand up.
 
Michael Greene (POWERSHELL)   16:10
Oh yeah.
Hey, Ryan.
 
Steven Bucher   16:11
Yeah.
 
Steve Lee (POWERSHELL)   16:15
And go ahead.
 
Steven Bucher   16:18
You're a mute if you're talking rain.
 
Steve Lee (POWERSHELL)   16:18
If.
 
Ryan Yates   16:21
Joys of microphone.
I'm hearing there's a an open issue for upgrades from 7.2 to 7.4 at the moment in the repository.
Is that something that wants 7.2 becomes end of life that we're going to have an auto upgrade functionality 27.4.
 
Steven Bucher   16:42
That's a great question because I actually was gonna add to that was for Microsoft update.
 
Ryan Yates   16:42
So.
 
Steven Bucher   16:47
We are gonna upgrade those on 7.2 to the 7.4 LTS latest LTS.
But we won't be upgrading them to seven, three or 75.
We'll just go from LTS to LTS.
That's in our plan.
 
Ryan Yates   17:02
And I'll continue adding that into the issue.
 
Steven Bucher   17:06
Could you?
Could you link the issue?
I I I don't. I've seen.
 
Ryan Yates   17:08
Yeah, I I'm not it.
 
Steven Bucher   17:10
Seen this one.
So I'm I'd be happy to to add details there.
 
Sydney Smith (She/Her)   17:16
That that's great.
I don't know why I'm responding, but I think I'm the next item.
So I was just kind of ready to go.
Anyways, so I think the next item was Mini Con, which has been referred to mini. Con is a conference that's put on by PS Conf.
Eu in the fall and it's virtual and free to attend.
But you do need to register, so I'm going to post a link in the chat.
Registration is at mini or is at that link there. PS comp dot EU.
And the reason why you need to register is so that you can get a link to the platform since it's all online.
I will be presenting there talking about how we're building trust within repositories in PowerShell.
I think Michael's gonna be doing a session on DSC V3. Jim's gonna be doing a session on Platy PS and then of course there'll be many community members giving sessions as well.
So it's a great way to connect as a community coming up in about a month here on October 15th.
That also reminds me that there was a couple of questions in GitHub on the discussion that I was going to answer about.
PS Resource get, so I'm going to, I'll just go ahead and do that now.
So if you hadn't seen, there were two questions in the discussion about PS resource get.
The first one is related to moving the install path out of OneDrive and when that might happen.
Given that it's a highly requested feature, what I can say about this is that we have triaged this issue as a high priority for us in the 76 timeframe.
So we've been having active discussions within the team as to how we can solve this problem. As recently as today.
What I will do my best to commit to is getting a proposal up on GitHub in terms of our design, probably in the month of October, but really the time frame we're looking at to get that change in would be in the 76 timeframe, certainly not going.
To happen in 75.
But we're aiming to make that change in the 76 timeframe, so we understand it's highly requested and working on it, but would love community feedback once we get the proposal up.
So hopefully I can come back in the next community call and be able to point to an issue there.
I know there's a number of issues already open as well.
Second question was the status on PowerShell Gallery moving modules into Mar. If that's still happening and if there's a preview available.
So I I can say about this is in the latest previews of PS resource get support for ACR is already available.
So what ACR is Azure Container Registry is the sort of private instance, so you can stand up your own container registry and publish your modules there.
That sort of thing in terms of having Microsoft owned modules into Mar. This work is still in progress.
You'll see if you look closely at the change log that there was some changes.
That we made in the last preview to help support Mar, so look forward to this coming very soon in the next few months.
But yeah, there's still changes that need to be made and it will be a long haul effort to get all the modules kind of moved over to Mar.
So expect that to not happen overnight, but we are working closely with partners to make that happen and work is in progress.
That's all I have to say.
 
Steven Bucher   20:51
I think next is Steve for a quick DSC V3 demo.
 
Steve Lee (POWERSHELL)   20:58
Yes, hold on. Just replying to chat stuff.
 
Steven Bucher   20:59
Are you ready?
 
Steve Lee (POWERSHELL)   21:01
All right, let me share my screen real quick.
Let's see which one this one.
All right. First thing is we did ship out Preview 10.
I wanna say earlier this week.
So even though there's a small window, it doesn't matter what is actually in this window.
I just wanna highlight that this is the previews are published to the Microsoft Store, making it easy to install and get auto update.
So do play with that and give us that feedback.
And before you look on the rest of the screen.
We are targeting trying to get like a release candidate out next month and then depending on customer and partner feedback, maybe Aga in November time frame, but that might slip depending on the feedback that we get from partners and customers.
OK.
So what I wanna show you here is actually some new stuff that is is actually post preview 10.
So it's not in the current release, but one of part of it is merged.
So if you build the main branch, you'll get it. The other one is actually still work in progress.
But I wanted to show a change that I made in the WMI adapter because I think W my so for those who don't know like I used to work on deadmau like a long time ago.
All right.
One of one of the first deliverables I worked on was a Northern T4 out of band package for WMI.
But anyways, that that really dates me even further than anyone else. But what I'm showing here is like Debbie might have a lot of classes that can be used.
So what you can use here and and this is YAML, right?
But you can also do in Jason.
Is a example.
Configuration so I'm starting to build these things and the idea is that this was shipped with DSC itself, so you can actually use it as is after you install.
So you have something useful that you can kind of use and also.
Build on top of but anyways.
This is a example inventory, so this is right now. So the demo adapter only supports get, so you can't do sets or method invocation yet. But forget you can still do like inventory right?
So dynamic has a lot of stuff, so here if I skip down to the bottom here, you can look at I'm going to get went through computer system.
I'm gonna get the name of it, the domain. The machine is joined into.
This is AVM, by the way, that that you're seeing here.
Physical memory is some other information. I'm also gonna query went through the operating system system enclosure, BIOS and network adapter and the one thing that you may there's two things you may notice.
One is there's a list of properties with no value, so these properties are basically informing the adapter you only want to get this information. These properties from these.
These instances, this last one does have one value 2 in here it is because.
All these other ones above are what's in Dev, my term's called singletons.
There's only single instance that doesn't even have a key.
Network adapter obviously can have multiple network adapters, network connection status 2I happen to know. In fact, I should probably add a common in here.
Connected, yeah.
So so if I were to run this right?
So let's see.
Not the baseline 1.
I'll do that second.
They get P.
C.
Inventory, right?
Then this is gonna run and it's gonna find the demo adapter.
It's gonna run through all these classes and you can see it's relatively quick. And by the way, we've been working on a bunch of changes for caching, so things with DSC V3, even using PowerShell resources over and over should be much faster than if using the old.
The older DSC, but any case you can.
Kind of. See some information about the VM I'm running here.
Some serial number stuff.
Again, none of these are really that interesting because it's it's all hyper V anyways.
But you kind of see that network connection status too.
We turn the one Nic that I have on here, which is again it's a virtual network card, but you can kind of use this. The other thing I'll mention real quick on top here.
This is leveraging also the assertion resource, which I've done talks about in other venues and also at PS Confu.
I think basically this like if you run the same configuration on Mac OS or Linux then it'll fail early on because this assertion will fail. It won't, it won't.
Windows family.
So the other one I wanna show real quick is and this is a work in progress 'cause there's a this is not small.
So Michael had actually pointed me to some.
Policies and configurations used by Azure machine config.
It's in a GitHub repo.
So it's open source and it has a large set of checks in terms of, hey, this is what we think the baseline security profile for a Windows system is.
There's also one for Linux, and I'll try to do that later over time, but this is not a small work item.
These are like MOF files, so I've been.
I've been manually so far just kind of porting over some interesting stuff.
So for example here I checked that.
Let me see.
I checked out the RDP port is 3389.
I checked that SMB V1 is disabled. The Windows search service apparently is best practices disable it and then there's also ensuring that Windows Defender also scans removable drives.
And it's a little bit confusing 'cause the way you do is you have a thing to disable it. So just make sure that the disable is disabled means meaning that it's enabled.
So if I were to run this one, let's do the search instead.
What did I call this thing? Yeah, OK.
Again, I'll do a get.
There's a problem with test right now that I found as I was doing this development, which I will try to fix later, but you can also see you know I have the assertion.
Make sure there's only running on Windows because it won't is using the registry for a lot of this and won't work on non windows.
It's saying that I'm I have to do elevated because some of the registry checks requires me to be admin.
And basically kinda see this information, right?
So of course Git is not as gonna be as interesting as test to validate that your system is meeting the baseline security.
But again, I'm hoping that I will get a initial version checked in the repo, and then hopefully the community can help me kind of build it out over time to get a more complete baseline configuration that you can use to audit your systems using the CV3.
That's pretty much it.
Yeah, rout policy.
Oh, by the way, one other thing that we are it won't happen for three of them.
Maybe for three one timeframe is having an adapter for group policy.
So you can have a single view for everything.
All right. Thanks.
 
Steven Bucher   27:29
I think the next step is Alex.
Alex, do you wanna give a big give a brief introduction and then share some of your feedback on Wham.
 
Alex Wang   27:39
Yeah, yeah, let me share my screen.
Just give me a one second.
I see my screen.
 
Sydney Smith (She/Her)   28:00
Looks good.
 
Alex Wang   28:02
Oh, yeah. OK. Hello everyone.
My topic is some feedback and updates on the warm features.
Yeah, we can go to the warm feature.
So warm feature refers to the ability for the user to interactive log into the to Azure through Azure CLI and PowerShell.
Actually, we conduct a long preview of fees for the preview testing before.
May 2024, but we haven't receive a lot of feedback on the issues.
In the 24 building ones which is in May, we will start the the warm functionality by default for the users who upgrade to the the latest version. So we can see the version of the the Azure CLI and the partial below.
After the new version is released, some users will gradually upgrade to the latest version and we have started to receive some issues.
With warm not working properly for the no issue list below.
We quickly provide a mitigation solution which mainly affect the interactive login flow and other automated authentication flow to not affect it.
Yeah, like manage identity also like a source principle, something like that.
Not affected.
We can see what happened in here.
On the May 21st.
The each client tool team we released the the new version for CRI and post and one feature is enabled by the default.
So after release, we just received the issue on the social media and GitHub repos and the issue include like partial logging using the device codes and user name and password.
The logging are affected and also we receive. The issue is incomplete 1 log in Page and the failure to the pop up the window.
After successful logging and no command line can be run.
So all all of the issues we received at same time.
And we we immediately we can go to next page.
We immediately to investigate how many users were affected.
So first I just share some data in here. We can see from the OS level.
The most the window users use the Azure portal.
So if the worm encounter issues impact on the CRI is minimal, but for the impact CRI and will be more wider. So secondly, for the Azure portal, it only affects the in the interactive login part.
So which actually accounts for the you can see in the in the red block.
2.9 percentage of the data.
In the process three months.
So it's basically number of the. This is a based on the number of request.
So the scope of the issue impact is a further narrowed.
And another data is we checked and curated failure rates of the interactive login in the week before and after. So you can see this this read.
This read is increase increase from the 1.13 percentage to the 3.35 percentage, so it means.
From the May to August at same time in the May to August, the number of the disable warm user.
You can see the the, the, the right the chart, the the number of disable warm users has been growing change. So in the end we found the user who use interactive login in the last.
In the latest version and.
Almost 100 percentage disabled. The one feature in the HP Pro shell.
So the scope of the influence CRI is very small.
Just the one percentage.
Yeah, we can see that data.
And also I can share some customer feedback in here.
So we quickly receive a large number of negative comments on the linked LinkedIn or Twitter.
Technical blogs and user satisfaction survey and the from the GitHub issues.
Yeah. And you can see the from the.
It can be seen that the detractor reads off the Azure parcel.
Towards logging, authentication has been increased rapidly.
Yeah, you can see this is the the.
The Orange Line for the Azure portal is increased.
OK and.
So for the no issues list and progress, below is a summarize of our known issues.
Which is a result of provided with a workaround in the July release.
So there are mainly 2P zero issues and 3P1 issues.
So all of all of the issues already have the workaround solution.
All has been already fixed, so you can see we can we. I just put the the status in here.
OK, we can go to the next one for the lesson learning from the current issues with the. We also have some lesson learning that requires for example like build automated automated test case and test case baseline to improve the the test coverage. And also we will consider a.
Shift left the pipeline.
Builder monitor mechanism and record telemetry error code, but this is a require collaboration with the MSO team.
Yeah. And for the next plan?
For the P0 issues CRI already.
Has been solved all the issues and the CRI already.
For all one features will be fully released for HP Pro Shell, we will fully testing before the prepare to enable warm in the Ignite. There are still some P1 issues that have a limited impact and we will continue to work with the M cell team to Sol.
The issue.
After waiting for all issues to be resolved, we may consider post public technical blog for the issue. OK.
That's all for the warm update, yeah.
OK.
 
Steven Bucher   35:48
So, thanks so much, Alex.
I think there's a question in the chat, but it looks like I think Jeremy answered it.
 
Alex Wang   35:55
Yeah.
 
Steven Bucher   35:56
And this is only for Azure, right?
 
Alex Wang   35:59
No.
 
Steven Bucher   36:00
OK.
Some things I think that's we're at the end of our agenda. If anyone has any last minute questions, I am pulling up the discussion page again, just in case I missed any other questions. I know Jordan was gonna demo, but he had to postpone.
If anyone has any other questions, feel free to.
Add it in the chat or oh I see a hand up.
Jeremy, do you wanna add anything?
 
Jeremy Li   36:29
Sure. Actually I didn't put my item into agenda because it's very quick, short and I'm from Azure CLI.
Hopefully it's not sounds like outlier for partial community, but I do want call out the feature that we release a couple months ago.
Actually it's called the tab completion. So you can actually use the address CLI and PowerShell that you can enable this one.
So which means you can just simply.
Press the button.
Uh, uh. The press. The tap key. So it will give you the indication of uh, you know the command parameters. Uh, to help to, you know, to sip the users time and effort to avoid remembering, remembering specific argument and looking up into the documentation.
I, if I recall correctly, it's available since HOCOI 2.49.
I can share the documentation here and.
One second.
Yeah, I'm not sure how many folks here.
Maybe touching on the Azure CLI in PowerShell or maybe heard this feature already, but here is a quick hot 2 for switching and on to try out.
That's that's. That's it.
That's for my sake.
 
Steven Bucher   38:00
So thanks Jeremy.
James, did you have a question on this?
 
James Brundage   38:05
Yeah, I mean question slash offer.
Tab completion would be great.
It would be even cooler if it was also possible to send output from the Azure CLI directly into PowerShell as objects.
This is something I've done with a few applications as long as you support generic Jason Export, it's a pretty easy set of tricks to write in PowerShell. I'd be happy to take it offline if you'd like.
But it would be very, very nice if a Microsoft project did that out-of-the-box.
It would be a huge example of what PowerShell can truly be capable of, and we get to have our CLI cake and our object pipeline too.
 
Jeremy Li   38:51
Definitely. I appreciate it.
That advice I think I can open the connect connection with the dev team on this one to wrap it up into the PowerShell object.
 
James Brundage   39:02
Yeah, I'll try to ping you and set up some like FAQs on the how.
It's something I've done a few Times Now.
 
Jeremy Li   39:08
Yep, Yep. Happy to talk. Feel free.
 
James Brundage   39:12
The only other thing for the general community I've got if you want to get a look at some of the stuff in the lab is I've been playing a lot with commandlet and adapters in the past few weeks and re familiarizing myself with an ancient part of POW.
That, it seems like most of time is forgot.
Wouldn't be the best demo probably at this community call, but if you are all hoping for some form of demo, it would be something.
 
Steven Bucher   39:48
Does anyone have any questions?
I guess first before we jump into the demos, we may wanna keep this one a little shorter just because I know it's a lot it late for a lot of the folks on the West Coast. So but we'd be happy to have that demo for for next month.
If you wanna prepare it a little bit more.
 
James Brundage   40:02
Yeah, I'd rather the time to get it better for next month anyway, so.
 
Steven Bucher   40:05
Yeah.
No problem.
We'll save that for next month then.
 
James Brundage   40:10
All right.
 
Steven Bucher   40:11
Cool.
Well, I think I vamped enough so.
Well, thank you, Glenn, for the, for the Nice comment.
Yeah, hopefully this this time.
We'll we'll probably keep this cadence or we'll we'll talk with our team to keep the cadence of once every six months.
So we can accommodate those in the Asian and Australian time zones. But yeah, I think we can probably wrap things up here.
Thank you everyone for coming to the PowerShell community. Call this.
And if we'll see you guys next month, next month, we'll be at our normal time, which is 9:30 AM, Pacific Standard Time and then going forward, we'll we'll have that until six months from now, which will be March of 2025.
That I guess will end up here.
Thank you everyone for coming.
 
James Brundage   41:08
Thanks.
 
Jason Helmick stopped transcription
