# PowerShell_OpenSSH Community Call-20250320

Transcript
June 19, 2025


Jason Helmick   2:56

OK, well, let's get started. Hello, good morning, good afternoon and good evening. Welcome to the June 2025 PowerShell Community Call, where everyone is welcome to join and discuss the PowerShell project and its development. We have our agenda posted in the meeting chat and a link to the meeting discussion. Please feel.
Free to post questions in the discussion issue or right here in chat. And well, please follow our code of conduct. We we really care about that for announcements. Happy summer. Welcome to the summer, everybody. We've made it. I know some of you are warmer than others, so enjoy.
With that, let's go ahead and dive in with our agenda for today. And I I think First up I I think I saw you around Stephen. Stephen, do you do you want to talk about a I Shell this morning?

Steven Bucher   5:39
Yeah, thanks Jason. Just wanted to talk a little bit about the latest AI show release. Let me just share my screen for the blog. I was preparing a demo this morning and my computer and my API keys decided they all needed to read fit themselves. So.
Gonna have to just show off some of the stuff I've posted in the blog. We just released AI Shell Preview 4. Actually, technically Preview 5 came out last week, but that was just a small security fix. So Preview 4 has a lot of the new features of.
Available to it. So some of the key things that we have improved is Mac OS support. We made a Mac OS support with Iterm 2 much much more reliable and workable. So you can see here I'm actually in Iterm 2 in this GIF here and we can start AI Shell and a side pane opens up.
With Iterm 2 and you can get the same experience that you'd get in Windows Terminal, which is pretty sweet. So you'll see me just typing a command to my Open AI agent and then I can get a response back and it can post it and do all the same stuff with it and you can see this is.
A disk, a Mac kind of command. I can post it here, run it and go on my merry way. So made a lot of really essential quality improvements there, so you have a lot more parity. Another cool thing that we supported is support for Microsoft Entra ID. We heard really good feedback around putting.
Keys in these configs doesn't make sense and so we supported enter ID for Azure Open AI. So if you use an Azure Open AI instance and have a deployment, you no longer need to put your API key in your configuration to get it connected to AI Shell. Here's kind of a.
A description of of the the configuration you you can do for it and then we support all the you know the credential hierarchy with the entry ID supports. Usually the best way to do it is via Azure CLI or Azure PowerShell.
Or other or any of these other credentials. But yeah, so that now works pretty seamlessly as well. Some other things that we've included is additional commands to invoke AI Shell to kind of keep you in the flow of your current terminal here. And So what you can do now is kind of operate and use AI Shell.
Directly from your working terminal, I can invoke AI Shell, ask a question and see that question put on the side. So my main buffer isn't really cluttered with descriptions and generated code and stuff. So I can kind of refer to my working show this way and then I can even post it directly so I can say hey.
Copy the code, it's gonna copy it and I can paste it in my window if I'd like or I can run invoke AI shell code or post code and it will the next item I have there so you can reduce the number of copy and paste you do there so.
I would love some feedback on this. If this is kind of a right flow you feel better fits your workflow within the CLI, give it a shot and please feel free to drop some suggestions in our GitHub page. The last thing is we added a new experimental agent called.
This is only available on Copilot plus PCs and it is allowing you to have an offline experience with AI Shell. This is very experimental and so you can only build it if you build it from the repo source, but essentially you get the same experience as you get with AI Shell, but you can do it in an offline scenario.
And so the Phi Silica agent connects you to the Phi Silica model that comes with the Copilot plus PC so you can get.
AI assistance without having to be online, which is a fun scenario there. And yeah, but very, very early on. It's gonna be buggy. It's gonna be tricky for its first iteration, but wanted to call that out. Number of other minor improvements, but I'll post this blog in the chat.
And it will show you how to install AI Shell and links to our repo and so on and so forth. So yeah, that's a bit about AI Shell. I think I kind of covered everything with the latest release and so I posted the blog there. I will, yeah, the demo gods were not in my favor. I apologize for that. It's been a hectic morning, but.
That's why I recorded gifs for the blog, so um.
I will be, yeah. And I'll address some of the questions in the chat. If there's any other questions, feel free to throw them in the chat and I'll let you know. I'll pass it back to Jason to keep the agenda going.

Jason Helmick   10:36
Hey Stephen, I'm really excited about the Phi silica experimentation. That'll be kind of cool. Really excited to see how that works out.
So Next up, oh, I'm the next one up. So let's let's let's talk about DSC. So I don't know if you've been following us along on this journey or not, but in March of this year we released DSC version three, specifically 3.0.0 and since that release.
We've had a couple of servicing releases, 3.013.02. Along the way with that, we've also been making pushing out previews for 3.1. Well, yesterday Congrats Steve and team we.
Released 3.10 as a general availability for DSC. Now this particular release for DSC was mainly work for internal partners like Winget and some other internal teams. So a huge shout out and thank you.
For all of the work from Winget and and the Winget team.
So a lot of the updates in there were for the internal partners. Some of it was community based as well. Just looking a little bit more forward, I think you know there's a strong possibility that maybe even as early as next week, Steve might release.
The first preview for 3.2, what we're going to be focusing on during the 3.2 run is a couple things. We still have a lot of work we're going to be doing with internal partners. We have the DSC working group and DSC community helping us out greatly and a huge shout out.
To the DSC community right now, there are some folks out there that have been doing a tremendous amount of work with us on version three that has been very important and very impactful. So these folks will be highlighted more in blogs as we move forward and stuff like that. But I wanted to give a huge shout out. The the work you've been doing has been greatly appreciated.
So as we look at the focus for 3.2, we're going to focus in on things like community stuff and some other internal partner work, and we're also going to focus in on biceps, so we.
We announced with 3.0.0 that Bicep is is the DSL that we're looking towards and we've had some great proof of concepts with Bicep so far. Now we're actually doing a lot of the work to make sure that we we kind of close all of the loops so that we can both release.
And talk more about how to use Bicep in building configurations. And yeah, we're looking at at all of the things that you would have expected if you were using PS DSC and the PowerShell DSL, but just elevated up into Bicep and a lot of those same concepts, that kind of thing that work into Bicep.
The beauty behind Bicep is, and I I what what I love most about this is, is that as somebody that is on Prem doing, let's just say server configurations, yes, it is a shift, but it's a small shift to using Bicep. But here's the great thing for me and this is what I hope to show when we get.
Or towards ignite and stuff like that is that with Bicep I can do all of my work on Prem and then I can take that same stuff and just drop it, move it on into Azure and I can add infrastructure as code. So I can add the Azure infrastructure as code and everything works the way you expect. There's only one set of tools.
That you have to know DSC is running underneath the hood that'll take care of enacting the resources, you know, enacting those configurations. So really looking forward to that work and looking forward to talking more about it as we get around to that.
And I think that's all I have on the the DSC front, except I also want to post in. Oh, I forgot to do this part. Let me drop a link to the announcement blog that we just had and let me go on to the next thing. Oh, so Aditya, there's a CDN update for PowerShell Gallery you want to talk?
About.

Aditya Patwardhan   14:53
Muted. Sorry. So I have a quick announcement. We updated the CDN URLs for PowerShell Gallery and one get. So if you have firewalls for firewall rules in your enterprise or in your environment, you would need to open up two extra.
Rules, I'm sending them in chat over here. So we had some hiccups with one get. We missed updating couple of URLs within the sweet tags, so it was some customers did see some issues downloading.
The newer provider, but those have been fixed. If you still see issues, please report it to the PSC Sourcegit repository or the PowerShell value repository. We do monitor both of them and in other news, we are planning to have a PSC Sourcegit preview coming very soon, hopefully next week.
We are waiting on an excellent community PR from ICE. His his get a name is Olaf for getting dependencies from Nuget V3 repositories which was missing for a long time. So really looking forward for that and getting feedback about that.
Thank you.

Jason Helmick   16:02
Oh, outstanding. Thanks, Aditya. And Alex, you're U next to talk about some updates to AZCLI and Azure PowerShell.
Your PowerShell.

Alex Wang   16:13
Yes, let me share my screen. Yes, can you see my screen?
Bring.

Jason Helmick   16:19
Yeah, sure can't.

Alex Wang   16:21
Yeah, yeah, OK, I will start for the CRI and the partial feature update. The one currently is the default authentication mechanism on the windows. So you can see like this demo screenshot. So we we providing A enhancement authentication capabilities for the CRI.
And PowerShell here and also if users want to switch back to the original flow and also can switch back and so we can see currently.
We can see the adoption rate of the 184 percentage for the PowerShell and 93 percentage for the HCLI. So our goal is to try to an adoption rate of 90th percentage of this both tour.
Yeah, and this is the first update and the second one for the Azure PowerShell only. Currently we have the easy get access token which is a start from the EZ 40 dot 0.0 introduce.
Change because we updated this parameter, the output parameter token from string to security stream and this is I think this is a significantly impact for this this command because this command usage is a larger.
And we make the security string as a default. So this is A and this change in world in the easy account which is a very popular module in the.
In the Azure PowerShell command. So this is a example in here get access token. So in this token we will output a security string and if you want to convert this string you can use this command script to convert yeah.
This is for the Azure PowerShell feature update. Another one is for the Azure CRI and Azure PowerShell both. We already announce we support the long term solution, long term support release LTS version.
In the November, in the November 2024 Azure PowerShell, we already announced the RTS and the STS versions and we provide a feasibility management tools for in the 2025 in May we announce CRI.
Support the STS and the RTS both. So we allow users to choose the version that fit fit their project needs, whether it's prefer stability or RTS version or wants to stay up to date.
With the latest feature in SDS release so users can continue choose this both version and will come to give us feedback. If you have any questions or issues can give us the the GitHub easier feedback, yeah.
So this is the CRI lifecycle and the PowerShell lifecycle doc.
Another update is both the Azure compilers response quality and performance for the Azure CRI and the PowerShell. So in the past six months we have optimized the following scenarios. So we introduced the Azure concept documentation to.
Direct to provide more accurate and comprehensive answers and also improve the accuracy and the relevance of knowledge retrieve query and chunking strategies. And we also support more accuracy reject of out of scope questions.
So this is what we optimize. So also we encourage the users can use our Azure compilers to try the Azure CRI and the PowerShell script. If any questions and we are you can feedback to us.
Yeah. And next we have some plan for the authentication broker. We will be rolling out the support WSLOS. So this is one planning in the six in the next three months or six months. Another one is for the.
Multiple factor authentication MFA. We will rolling out in the CRI and the Pro Shell in early September. So we will encourage you. You can upgrade to the MFA promptly. Yeah, yeah, that's all from my side.
Thank you.

Jason Helmick   21:18
That's great. Thanks Alex. Really appreciate it. Before we turn to Sean for a Docs update, I want to add an agenda item. I just got a side chat here and I I thought, well, why not just just talk about it. So the the question was on Platy PS. Some of you have been following us along with Platy PS and I just.
Just wanted to give you an update of where we're at. We are this close to actually are seeing and then Gaing Platy PS getting it out. We've been working in with a internal content partner who has been feverishly working away on rebuilding brand new pipelines to use the new Platy PS and.
Test all of the content and I also happen to know that when when Sean comes on, you could even ask him about it. But Sean has been working through some content issues as well to make it all happen and the testing is going really well. So we're hoping that we can close on this RC here any day now and then start moving it towards.
GA in a pretty rapid fashion with that partner and then we look forward to, well, looking forward with community and additional partner requests for it. So thanks for your interest in Platy PS.
And with that, let me go ahead and Sean, I know you're around here somewhere. You want to give us a Docs update and tell us how hard you've been working on Platy PS.

Sean Wheeler   22:35
Uh.
Yeah, so not a lot of new stuff so far in June for a couple of reasons. The first couple of weeks I've been out of office moving my son across country.
And getting them married off, but um.
Also been doing a lot of content remediation to support the new Cloud EPS. It's it's it's been a Herculean task and I've barely scratched the surface.

Jason Helmick   22:59
Congrats.

Sean Wheeler   23:14
But yes, we're getting very close. It looks really nice. You're gonna see major changes to the layout of the docs.
And I know I'm looking forward to this being released. So anyway, with that, let me share my screen and we can talk about what's new in Docs. I'll drop a bunch of links here.
Let me get to the right chat.
There we go.
So one new article released in May about performance when doing parallel execution and how to optimize for parallel execution.
And then a bunch of release updates. Also retired, you know, 5300 articles in the Windows Server PowerShell content for all the old, now unsupported versions.

Zachary L Siebers   24:18
E.

Sean Wheeler   24:29
Of Windows, the new article here about parallel execution talks about the trade-offs and how to measure performance and some examples here.
About which things are performed better than others. As always, your mileage may vary depending on your environment and the the tasks that you're trying to parallelize, but.
Fun article here that shows you how to to measure that and figure out what works best for your solution and a shout out to Santiago Squarezon and Mr. Michael Clement.
Clement for their Stack Overflow posts that outlined all of this. I used a lot of their information with their permission.
This has already been talked about a bit, but I just want to point it out. We've updated the documentation for PS Resource Kit talking about support for the Microsoft Artifact Registry so that you can now register.
The MAR as a repository for PS resource get and it's a read-only repository for you, but as we publish more items there, you'll be able to pull them from there. Now a trusted source for.
Um.
For products published by Microsoft, I know Azure PowerShell is out there now. Um, I'm not sure what else.
Stephen already talked about updates to the AI Shell. I've updated the documentation here. Also wanted to point out that as part of the updates came in preview 2.
They he added support for all of these Open AI compatible models and.
Preview 4 supports the following Open AI models and also documentation here about the Entre ID authentication. So this has all been updated.
Um.
You see 30 and we need to. I need to update this to say 3/1 now. So three one has gone Georgia. Big shout out to Mikey for all the hard work he's done on.
Not only designing, helping design the product, but documenting and we're only scratching the surface here. One of the things I want to point out, I really love the work he's done here in the release notes. They're not.
It's not just your change log that you're that that you see in GitHub, it's actually useful information and if you want to dig into the details you can click the expand the related work items and see the issues and PRS for each of these.
New features that have been introduced.
Oh, and yes, that was for the life cycle that was we were talking about previously. The other thing that I wanted to point out.
You may notice that the layout on the doc site has changed a little bit. I know that there some of this is being done as a a user test, so you may not see these changes yet.
I'm not sure if they've gotten.
Fully deployed, but the the big change here is on the right NAV. Now the you'll have the article TOC in the right NAV like we used to have and this is floating so as you Scroll down it stays.
There at the top of the screen an you can Click to go back and the.
Additional resources are now added at the bottom, so I think it's a better user experience for everyone and that is my Docs update.

Jason Helmick   29:05
Well, that's a fantastic update and thank you Sean for also shouting out. I meant to earlier as well shout out to Mikey for not just the great doc work, but the great design work as well. And I noticed Mikey put in any promises to keep the.
More up to date. Yeah, that's great. OK, great, folks. We've pretty much come to the end of the show. So if you have any questions or anything that you might want to put into the chat real quick that we can answer a couple of while you do that, a couple of upcoming things just to make sure that you're aware of. I think what is it next?
Next week is PS Conf EU. Wow, how fast time has has flown and looking forward to have to everybody having a great show at PS Conf EU. A huge shout out to GAIL and all the folks that'll be there.
And yeah, I'm really looking forward to that. Also, every time I think about PS Conf EU, that reminds me to, well, now start thinking about PS Conf North America for next year. I know that's kind of far away, but there's some other things in the middle that are coming up. Things like there's.
Tech Mentor that's coming up towards the end of the year here in Redmond, and I know that several of us are going to be attending and speaking at that. Also, of course, Microsoft Ignite is coming up where, boy, I sure hope we have some great announcements for that.
So there's all kinds of interesting things come U I'm not quite caught U on what the user groups are doing. I should be, but I'm not. I'm sure the user groups have some things going on so.
Oh, question in there from Heis. Will the Platty PS repository accept pull requests by the community? The answer is post GA. We'll have that conversation with you. So yeah, once we get the partner work done and we've stabilized that.
So post GA is when we'll really start being very interested in community issues and community PR. So we'll we'll we'll talk more about that. Go ahead, Sean.

Sean Wheeler   31:08
It.
Yeah, and and we have a lot of work to do in the repository. We're gonna reorganize the branches so that the master branch will be the new release and there'll be.
A separate branch for the old version, and so stuff's going to move around. Once we have all that stabilized, then you know we're going to clean out the old issues 'cause we're not going to touch the 0.14 branch anymore.
And everything will be going forward on the the new version.

Jason Helmick   31:47
Yes. So thanks a lot for your interest, Heis. Appreciate it. And the book and all the other stuff. So, OK, folks, we are at time. Oh, we managed to get through another one again together. Isn't that wonderful? All right. So I hope you all have a wonderful beginning to your summer and we'll see you all.
All next month.

Jason Helmick stopped transcription
