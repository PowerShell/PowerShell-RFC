PowerShellOpenSSH Community Call-20260115_093006-Meeting Recording
January 15, 2026, 5:30PM

Jason Helmick   2:00
Oh, awesome. With that, let's go ahead and do I have the recording going? I do have the recording doing. OK, good. Let's go ahead and get started this morning. Hello. Good morning, good afternoon and good evening. Welcome to the January 2026 PowerShell community call.
Where everyone is welcome to join and discuss the PowerShell project and its development. We have our agenda posted in the meeting chat, so check the link, check the correct link that Greg posted, not the one that I had my little chat. Please feel free to post questions in the discussion.
Right here in the meeting chat and please follow our code of conduct. We have a relatively short agenda today. It's the beginning of the year and we have a release to get out. So but before we get started I.
Hey folks, I just wanted to take a moment to call something out that's kind of fun and for many of us very meaningful. This year marks 20 years of PowerShell since its release, which honestly feels just wild and crazy to say out loud.
A lot has changed over that time. The platform, the tooling, the problems that we're all solving. But what's been remarkably consistent is the community around it. The fact that people are still showing up, they're sharing, they're teaching, and they're pushing things forward 2 decades later says a lot. I know that I've met a lot of.
New friends and and I continue to learn how PowerShell makes a messy world a better place. So I just wanted to to thank you all for being a part of the story and hay 20 years of PowerShell.
Oh, before somebody asked me the the official birthday date, if I'm not mistaken, is November of 2026 is the the official 20 years. I think it's actually the 14th. Now my brain always screws that up with the 18th.
But I think it's the actual 14th was the release to web. At any rate, let's go ahead and get started into our agenda. Like I said, we we're working on a release that we're going to get out. This is our long term service release. And Aditya, can you maybe give us some information on what to expect over the next maybe month or two?

Aditya Patwardhan   4:19
Hello everybody, Happy New Year and I have a very quick announcement. We did release a preview six of PowerShell 7.6 in December on December 11th. If you have not had a chance on trying it out, please do.
We are working on an RC release which will be out in next couple of weeks and if you have any feedback which you think should go in 7.6, we would love to hear it as soon as possible if you have any issues that you want to post that you think should be included in the next release.
Please do put this on chat, put them on chat so we can have a look at them. We once the RC is out, RC is supposed to be a go live release, so it is it is supposed to be used in production. It is a officially supported release.
So we would love to get your feedback on how the migration process goes and if we have any problems with that. Once the RC is out, we would be starting to work on the next release which would either be an RC or a GA depending on the feedback that we receive and then shortly after we would have a GA release.
Getting somewhere in next month, but we will work depending on the feedback that we get. So that's that's the announcement I had. So if you have not tried things out, please do and this is the time to give any feedback that you think should make 276.

Jason Helmick   5:45
No, that's awesome. Thank you, Aditya. And just just to remind everybody, especially if you're newer to the community, 76 is our LTS or long term service release with three years of support, one of our big, big, big releases and as Aditya mentioned after that after we get through this month's RC and the tentative.
Based upon feedback at next month, we'll be moving into 77, our stable release, which will be supported for one year and those will start with previews and if the next community call, assuming that feedback is good and we can.
Plan for that maybe end of September GA release. Sounds to me like the next community call would be great for us to talk about some of the what's new in in PowerShell 7.6 and talk about that release along with some of the things that we're thinking about in 7.7. So thanks Aditya, appreciate it.
Also coming up here shortly is, I don't know, the previews and release. Anam, can you talk about tell us what's going on with PS Resource get?

Anam Navied   6:48
Yeah, let me share my screen so we can look at it together. Hi everyone. Since our last community call, PS Resource Get has had a few preview releases. So in December we released version 120 Preview 5 and this had a bunch of bug fixes to highlight a few.
We have a bug fix for when you have a required resource hash table and a trust repository parameter is specified. We make sure now that that value is respected. We also had some improvements for Container Registry repositories.
So we improve performance by caching the token across operations as well as some permission related fixes for container registries. So if you have a container registry with anonymous poll enabled, we're making sure we get the appropriate token for that.
Which is separate from when you're doing a publish and that has different permissions. We also added a commandlet reset PS resource repository and that's to help recover from having a corrupted repository store and there are a couple of other bug fixes as well. And then yesterday.
We did a RC release for PS resource get and this includes a bug fix for switch parameters like what if now we'll have their value respected instead of just simply going off of the presence of the switch parameter.
And since this is an RC release and we're looking forward to Ging soon, it would be great if folks get an opportunity to try it out and give us feedback and if there's anything that's urgent or a bug fix you would really like to see.
Now is a great time to give that feedback and other larger features will get in an upcoming 1-3 previews. Yeah, thank you. I will drop the release or I'll drop the link for the releases in the chat. Thank you.

Jason Helmick   8:37
Thanks, Anam. I really appreciate it.
Outstanding. Outstanding. Well, with that, let's turn to DSC. And Steve, do you want to talk about what's going on?

Steve Lee (POWERSHELL)   8:47
Actually, can you skip the mickey first? I need to get some water 'cause I'm I'm let me cough in on the microphone. I'll be back.

Jason Helmick   8:51
Oh yeah, sure.
No problem, no problem. Mikey, are you around? You want to give us a doc's update?

Mikey Lombardi   8:59
Sure, one SEC. I'll drop some links in here. So the.
Biggest thing is that Sean took the time to work on some new content for troubleshooting startup issues when when your PowerShell fails at startup. It's a pretty extensive doc that clarifies a bunch of things that go wrong.
Sort of removes the scary magic from why is this broken and then also made just some routine changes to content too that we want to call out here. So we updated the docs for the preview 5 release of PS resource get.
In particular using Azure Artifacts Prudential Provider and also made sure that the invoke web request docs reflect a breaking change introduced from a CBE fix.
And that's all I have currently.

Jason Helmick   9:57
Oh, cool.
That's awesome. Thanks, Mikey. Appreciate it. Before Steve gets back, I just wanted to say real quick, Alexander. Yeah, thanks for reminding about that 10 years of PowerShell Conference EU, which is amazing. And by the way, Alexander, thank you for the 20 plus years of your contribution to this community.
I remember you back in the early days. You were a big help and you still are. So thank you very much. Steve, you want to talk about DSC?

Steve Lee (POWERSHELL)   10:27
Yeah, give me a second. No. Uh, all right. Um, so.

Jason Helmick   10:33
Take your time.

Steve Lee (POWERSHELL)   10:35
I was trying to look up what's new, so I'm gonna actually just give a quick demo.
Let me share my screen here. Excuse me, I was like eating some breakfast and I got something in my throat now, so pardon me if I do end up coughing this one. All right, so I'm I am getting ready to have a 3.2.
Preview 11 coming out later today and this is one of the new features that I do want people to kind of exercise and give feedback on. And even as I was thinking about it, there's actually at least two other things I need to fix with it, but I'm just so.
All right, one of the big changes coming in the Preview 11 is I added a Windows Update resource, which is the resource list. Excuse me.
By the way, you're not gonna on release, you're not gonna see a bunch of these test ones. They're just because I'm building in the repo.
And there's a lot of dupes because I have a bunch of DSCS installed, but under here let me see this. This is gonna scroll.
There's a new one called Windows Microsoft Windows Update List, so let me just run.
So it supports get, set and export. So I think I have it in my. Let's see, let's try and export here. So if you don't provide any input, it's just going to enumerate all of the Windows updates that are available for your system. So this is just calling the Windows Update Agent com APIs.
And which is not I found to be particularly fast, but.
It occurred to me that one of the changes I need to do is allow resources to emit progress to make this a little bit more enjoyable experience. But anyways, you can kind of see here's a whole bunch of updates. Some are installed. I don't know this this one, for example, isn't installed yet.
But for example, if I wanted to export all the ones that are not installed, then I mean you can have a configuration file for this, but I'm just going to do it directly to the resource. So the way this resource works is that this is actually a array of updates.
So go to the very top here.
See how many updates there are.
There we go. So there's a there's a single property called updates, which is an array of basically update infos.
So if I get to the bottom here.
That's actually if I type, is it gonna just? Alright, so you just do updates.
So basically this means that you can provide what what or more update infos as a filter and if you provide more than one then it's a logical or right? Meaning that for example here so there's an array of objects. So for example I can do title which supports wildcards. So I can for example search for all the ones that.
Or for PowerShell.
And I could also let me see if I do this. I don't want title. I want um.
Let's say is installed.
Uh, let's say false. Alright.
So let me see if I got all that right now.
Uh, I'm eyeballing it now, alright?
And I'm just to make it more complete here, I'm gonna add one more thing here. So this one I'll just say it's installed.
I already know I have all these installed in it, so I'll just say true. So basically this should return all PowerShell updates that were installed and any other updates that were not installed if I got the Jason right here.
Didn't take too long.
But while that's going here, I see these are the properties that get returned to get the title, whether it's installed, the description, the ID. So this is where the unique identifier. Alright, so here we go.
So you're gonna have to trust that you're gonna get all the PowerShell ones and all of these down here should be is installed as fault, which is not too many, just a few. And then you can kinda see there's also if a update is a MSRC.
Tagged one, if you will, or identify one. You can also query, for example, whether they're critical, important, or I forget what the other terms where they use, but we can query against all those things as well. So let's kind of maybe pick on this one here. That means we can also do get. Get does require an exact match, but you can exact match.
off of either the title or the ID. Well, I already have this one in my history. I'll just use this one.
Right, so you can see how you can check whether systems have particular updates installed. You can check if systems have any updates that are not installed yet. You can check if systems have updates that are critical, they're not installed, stuff like that for auditing. You can also use set to install updates, but you can also notice that.
There's a bunch of these. In fact, what I've noticed is most of these are is uninstallable as fault. So if we're to play around with it, just give them one that there's a bunch that you can't uninstall.
There's also a difference between software updates and driver updates. Oh, I know what the problem is. I had renamed this to update list, which is why it's trying to query PowerShell now.
Actually, I also notice one other thing. This isn't gonna work. This is what happens when I make modifications to the design and don't keep my history up. So that should be updates and then this is an array.
I think that's right.
So while this is running this preview version of this resource will be available in three to preview 11 and I asked people to kind of play with it. Let me know if the schema makes sense to you.
If there's other things that you expect from a Windows Update resource, just open it in the DC DC repo. Let me see. OK, well, I'm not gonna. I'll pay attention to what's in the chat later and I'll respond if needed, but that's kind of where that is. There's a whole bunch of other changes that have been accumulated since.
The last release which was I think was in mid-december, so it's been about a month and I want to just call out. I know Hice has been doing a lot of PRS adding value to DSC. I think some additional functions and stuff like that. I also want to call out Thomas and GAIL, other committee members at Hice, Thomas and GAIL are all members of our DSC.
Working group. So I do appreciate them contributing that way as well and I think things are progressing well. I don't, we don't have a target date per SE yet for doing this GAS. I'm expecting it sometime in the first quarter of this year, this calendar year.
But again, we're not trying to be day driven. There's a few additional features that need to be implemented for Win get and then after that we'll probably target like an RC and then GA and then move towards 3-3 as soon as possible.
That's pretty much it.
Any questions? Put in the chat.

Jason Helmick   17:55
Oh, that's perfect. You answered all the questions.

Steve Lee (POWERSHELL)   17:58
Oh yeah.
Makes my job easy.

Jason Helmick   18:00
Well, you answered my questions. We'll see what pops up. I don't think anything's popped up yet for that. So that's that's great. Oh, wow. That brings us towards the end of our agenda. Like I said, brief as we're working on a release, I want to point out that.
There's a lot of wonderful conferences going on this year, and of course the usual suspects should be pointed out during this 20th year of PowerShell. So PowerShell Summit North America up in April and then PowerShell Summit EU is coming up shortly after that in June, early June.
Make sure that if you have interest that you you look at that stuff up and go and more importantly for this community go and contribute. Go ahead and you know show up and do a session and and and talk about how you use PowerShell. Really looking forward to a a a year with the community during this time so.
With that, and since we're at the end of our agenda, oh, and please, next month, if you want to do a demo, we're starting demos next month. So just as a reminder, when I post the agendas in GitHub in the discussions group, feel free to tag on when you want to do a demo. My thoughts are just like last year, we'd like to have a.
Demo at least one every session, about 5 minutes in length, if you can time it that way. So with that, having said that, look forward to seeing you all next month and our progress towards releasing 7.6. So have a wonderful month. Oh oh sure.

Steve Lee (POWERSHELL)   19:29
I should before we go, I I see that Heist has a call out to Tess. I don't know if Tess you want to just briefly talk about SSHD config resource.

Tess Gauthier   19:41
Uh, sure I can. I can talk about what progress has been made.

Steve Lee (POWERSHELL)   19:43
It's about where, where it's at and maybe what's coming or whatever.

Tess Gauthier   19:47
Sure, yeah. So I have been working on a SSHD Config DSC resource. It's not quite complete yet, but right now it's supporting a basic get and set for what I call like the regular keywords in SSHD Config and the.
Steps will be to add support for some of the keywords that can be repeated or have multiple arguments and also handle match blocks. So hopefully once those three things are complete, I can do a demo in one of the community calls.
6.

Jason Helmick   20:25
Oh, that's awesome.
Ryan, did you have a question? You can either type it in or.
Say hello.
I see your hand up. Yep, sure can.

Ryan Yates   20:37
Yeah. Can you hear me?
So could we get next month's meeting notes discussion a bit earlier than three days beforehand and could we do all of the year ones like I asked last year so that we can start to prioritise like maybe adding a demo?
DMO in you know, so sort of what would be nice to throw in say June time, July time, September time, see what I get so we can start building up the sort of content of what these community calls are going to be.
About because obviously there's a a bigger amount of work that's coming out of us as a community with the DSC community kind of growing again and the PowerShell community growing again and one of the big ones that.
I did want to bring up was about the.
OneDrive changes that Justin's been doing, which I did briefly have a chat with Steve the other day. This is something that we I know you need some more data to be able to.
Look at prioritising and changing any plans that may happen with that. But the community I know are looking forward to being able to say we're not storing our PowerShell modules.
In our OneDrive locations and I know a lot of people that are saying we want this yesterday and really, really want it yesterday because of it's all to do with data.
The LP is, I think, is the right terminology, the free acronym, if I'm remembering correctly.
Does that make sense?

Jason Helmick   22:49
Oh yeah, that makes sense. Thank you so much for the feedback, Ryan. As far as I'll do my best to get out the announcement for the meeting in the discussion group a little bit sooner than that and I'm more than happy to work with you on and in that discussion we can work with on the demos. I don't know if anybody, if, if anybody wants to like Steve, if you want to make a comment about our.
Our conversations around PS content path at this point, but I I can say that it's yeah.

Steve Lee (POWERSHELL)   23:12
Well, Justin, Justin's on the call.
Or at least he was, unless he knew I was gonna call him out.

Justin Chung   23:16
Yeah, I.
Yeah, yeah. Thanks, Ryan. Yeah, I think I really feel like it shouldn't have taken long, but the decisions we are making are big and I really want it to be a seamless experience and I don't wanna break anyone.

Steve Lee (POWERSHELL)   23:21
Mhm.

Justin Chung   23:35
Because this would be a breaking change and just introduce it in the best way possible to to encourage users. So I think we are at very close to a solution. I've updated the PR. It's not complete yet, so maybe I should make it into a draft again.
But I think that first initial PR that introduces the infrastructure to be able to customize the PS content path is very close to complete and I think subsequent PR's from that.
Are going to be the breaking changes and will kind of force users users to migrate to the local app data path or wherever path issues.

Ryan Yates   24:18
Yeah, so on that I think you you get a lot of good feedback if you were to engage with the the SharePoint side of the M365 community. I can probably give you some people that definitely have shouted about it.
In the past, if you go and look through some of the issues and comments, there's a good couple of people that I know that have got a lot of consulting experience over the years and have got a lot of big, big customers that are really, really wanting this.
As soon as soon as possible.

Justin Chung   25:03
That sounds good. Yeah. Could you, uh, um, let me know, like, who these people are so I can reach out to them? That sounds great.

Ryan Yates   25:10
Yeah, I said to Steve when I spoke to him a couple of weeks, a couple of days back that I was going to get that all sorted out. So that's on on me thing to get this done.
Today to get you in contact with them, if that makes sense.

Steve Lee (POWERSHELL)   25:29
So Ryan, you know how to reach me, so just send me whatever you need and I'll include Justin and Jason as appropriate.

Ryan Yates   25:35
I'm I'm. I've got the e-mail started already. Yeah.

Steve Lee (POWERSHELL)   25:37
OK, OK. If you already got it, then it's fine. You can you can do it directly.
I just wanna mention, you know, for this feature area, like we we recognize the importance of it, but it's been extra challenging and it's not like a technical challenge. It's really again what Justin was calling out. It's like it's an area that we wanna make sure we don't break anybody. But then if we don't make it easily discoverable, then people may not know that they're already incurring this.

Jason Helmick   25:43
It's great wrong.

Steve Lee (POWERSHELL)   26:01
Uh, potential pain point in terms of perf because their stuff got moved to OneDrive for example. So we're trying to figure out how to best alleviate some of that.

Ryan Yates   26:08
Yeah.

Steve Lee (POWERSHELL)   26:11
All right.

Jason Helmick   26:12
Thanks, Ryan.
Thanks, Justin and Steve. Well, thank you folks. And once again, I hope you have a wonderful month of January and we'll see you back in February. We'll talk more about 7-6 and 7-7 and all the other things going on. So have a great start to your year. Happy New Year. Cheers.

Ryan Yates   26:32
Be near everyone.

Jason Helmick stopped transcription
