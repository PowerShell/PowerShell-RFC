# PowerShell_OpenSSH Community Call-20250320

Transcript
May 15, 2025

Transcript
May 15, 2025, 4:27PM

Jason Helmick started transcription

Sydney Smith (She/Her)   2:48
Hello everyone., Happy Thursday. Hope everyone is having a great day and thank you for being at our community call. As always,. Please just remember to follow our code of conduct and treat each other with kindness. Another reminder is that this call is being.
And recorded., If you do not wish to be in the recording, please just go ahead and leave the call now. The recording will be uploaded to our YouTube channel, usually within a week or so. And I just want to like, I guess particularly shout out Jason for doing so much work to make sure we get those recordings.
Up. So Jason rocks. With that, our primary agenda for today is going to be going over the team's 2025 plans. I know we're a bit of the way into 2025 at this point, but each year Steve posts A blog.
Overviewing, the team's investment areas, and these aren't so much like a promise of things we're going to accomplish, but instead areas of focus for the team. So we're gonna go through the agenda and engineers who are on the project, primarily a few PMS will.
Speak as well. We'll just speak at a high level to the investment area and then we'll have a docs update from Sean and then we have a couple of awesome community demos that I'm pretty excited about. So to kick things off, our first focus area is PowerShell Core and specifically PowerShell 7.
6 will be the release that will come out later this year. We've already released a few previews of 7.6, so please do go check those out and test those. As always, the earlier in the release cycle that we find out about issues, the more likely we're going to be able to get a fix in before we reach GA.
As always with our investment, security is a primary focus for us., So we're always partnering with other teams and looking into security features to make sure we're delivering as secure of a product as possible.
And with that, I'm going to pass things over to Dongbo, who's going to speak a little bit to how we're looking at community PRS right now and then talk about a couple of our RFCS that he has up.

Dongbo Wang   5:17
Thanks, Cindy. I'll share my screen.
So we created the PowerShell Team Reviews investigation project to make it transparent for our community members to see what the team is actually working on. Regarding the regarding to the issue, investigations and PR reviews.
Which will also make it easier to track the progress of the of the reviews and investigations. I think it's it's public, it's it's in the partial project and the first one review and investigations, so you can take a look.
And uh, like send us feedback. Uh.
So that's that's that's for the partial reviews and like, since I'm speaking now, so maybe I should continue with with a few items that I'm I'm not gonna cover. Which are.
I like we're gonna make some, like, make some effort to improve the native command integration with PowerShell. There are two targets here. The first one will be update the passing variable for when gets and it's not just when get it's for like when gets chocolate.
Or maybe other package managers as well. So basically the problem is that today if you use Winget to install a package, the passing variable from the current partial session will now be updated, which is by design, but.
But it costs problem because when you install the package, you cannot use it directly. You may not be able to use it directly within the same session, because maybe there's a new pass adding to the passive variable, but the current partial process doesn't know. So this feature is basically.
Try to make it special when you are invoking a package manager, like when ghetto chocolate tea. So that we detect if there's any changes in the user passing vulnerable or the OR the OR the machine scope passing vulnerable and if there is.
We like bring it back to the process level passing variable so that running a command after installation will work. That's the first one., The second one is trying to make it easier for a native command installer to like install.
The tab completers or feedback providers or predictors along with the native commands. So today we when a tool tries to do that, they will have to like change the user's profile to include something so that when users start a part of session, the integration is available there.
Which which is like error prone. So we try to come out with a design to make it auto to to to to like make power to to make PowerShell able to auto discover and auto load the completers, feedback providers or predictors that come with the native command.
Installation. So we have published 2 offices here.
Let's see. Yeah,. So the first one is a pass update for two installation in a in a partial RC repository. The first one is pass update for two installation., The second one is the the draft spec.
For always discovering feedback provider and type completers. So please take a look and leave your feedback over there. It's just like currently there are the that like this one this past update.
We we have reviewed the proposal with the team, so it's more mature compared to the other one. The other one is mainly a draft spec for now,. So we're definitely gonna need more feedback from the community to see how we're gonna improve this proposal.
Uh, yeah, that's, that's, that's, uh,. That's about it.

Sydney Smith (She/Her)   9:20
Awesome., Thank you. So much, Dongbo, and thank you for the thoughtfulness that went into those RFCS. Please do go ahead and take a look. Now is the best time to give feedback on those. And I also just want to thank the Community for the idea about using a GitHub project and kind of pushing us to be better in that area.
Next, Justin is going to talk about a very popular issue that we've been looking into, which is sort of what's called maybe the OneDrive, My Documents issue.
Justin, are you here?.
Justin may not be on the call in which.

Steve Lee (POWERSHELL)   10:12
I'll find Justin.

Sydney Smith (She/Her)   10:13
OK, sounds good. Next up, the next kind of bucket of issues we're going to talk about is PS Resource Management, and Amber is going to cover these topics.

Amber Erickson   10:26
Let's see. OK, yeah. So as Cindy mentioned,. I just wanted to talk about some updates with the gallery and PS resource get. The first is regarding Microsoft Entre ID support for the gallery. Over the past few months, we actually had an intern on the team who worked on integrating Entre ID with the gallery.
Specifically, with the intention of allowing users to publish a package without using API keys. So with the support you can actually use the Publish PS resource commandlet to log in with your Microsoft credentials and publish to the gallery. And our goal with this was to provide a more secure alternative to API keys.
And we're really excited to have this work in progress, but it'll probably be a few more months before it's fully released. Second thing is that we actually GAD PS Resource, get the MAR and ACR support within PS Resource get. So if you want to test this out, you can get it.
From the gallery, our latest version is 111. And last thing is that we've been working on a huge effort to modernize the gallery infrastructure. The big thing here is that we're moving off of cloud services, which is what the galleries run on for the past like roughly 10 years or so.
And moving on to Azure Kubernetes service. So at the moment we don't anticipate any impact to users, but if there's ever any concern from our side, we'll post updates through the banner on the Gallery's website or through the Gallery's GitHub status page.
I don't hit it exiting.

Sydney Smith (She/Her)   12:00
Awesome. Thank you. Yeah,. Thank you. So much, Amber, for all those updates and all the work that's gone into this. It's a lot of work to maintain the gallery and it's very much appreciated. I know by our community. I do want to also just briefly touch on the MAR or Microsoft Artifact Registry.
The only like group of modules that we have in there right now is the AZ module,. So please do test that out and give us feedback to make sure the experience is working for you as we onboard more modules. Again,. This repository is for Microsoft owned modules specifically, so if you have any sort of.
Priority list of Microsoft owned modules that you would like in Mir. That's also helpful feedback for us to encourage teams to publish there.
With that, I will pass things over to Tess to talk about Windows SH updates.

Steve Lee (POWERSHELL)   12:53
Actually, let's go. Justin just hopped on. I saw. Justin, you're there, right?. I just saw you. OK, let's. I don't wanna. Let's finish the PowerShell side and then we'll move on.

Sydney Smith (She/Her)   12:56
Oh.
Yeah.

Justin Chung   13:00
Oh, yeah.

Sydney Smith (She/Her)   13:04
Perfect. Justin, do you want to speak a little bit to the My Documents issue you've been working on?

Justin Chung   13:08
Yes. So I I know it's like a very the community has really been wanting this for a long time and finally getting around to working on it and hashing out the details. I think the.

Amber Erickson   13:23
Mhm.

Justin Chung   13:24
We're thinking about moving PowerShell module path over to the My App directory and not the My Documents folder so that the OneDrive sync doesn't happen.

Israel Milgrom   13:25
Which?.

Justin Chung   13:39
And that will lead to like a performance improvement, like loading PowerShell would be a lot faster and having a lot of PowerShell documents or all the PowerShell contents would be.
In a place that other other programming like infrastructures would have it in the My Apps document. So yeah, we're we're working on it and we're currently working through an RFC to document all the features and like where exactly we would move it.
And how the migration would work. So we're still hashing that out and.
Yeah.

Sydney Smith (She/Her)   14:21
Great., I think that the RFC is still in a bit of a draft state, but feel free to go give it a read and start giving us feedback on the proposed changes. And thanks Justin,. As you can see in chat, very popular issue.
With that, I will go ahead and pass things over to Tess to talk about Windows SSH.

Tess Gauthier   14:43
Thanks, Sydney. Yeah, to expand on the planned investments for Open SSH this year, we'll continue pulling in changes from upstream. If you're following along at home, upstream release 10.0 earlier this year. So we will be taking those changes in and 1st releasing a preview release.
Release on GitHub and eventually shipping the latest stable release with Windows. Another effort for Open SSH and DSC is developing an SSHD config resource to simplify managing and configuring the.
SSHD config file. We've talked about that in previous investment blogs before, but hopefully this year we will definitely maybe get around to it and there will also be a demo later on from Amir on some of the SSH posture control work that will hopefully utilize the SSHD config resource.
Actually.

Sydney Smith (She/Her)   15:46
Awesome. Yeah,. Thank you. I think Next up is Jason to talk about DSCV 3.

Tess Gauthier   15:46
Thanks, Sydney.

Jason Helmick   15:55
Thanks, Sydney. Hey, folks,. It's really good to see all of you. So let me provide some information, both referring to the blog announcement and some additional comments. So we started this journey about two years ago. This has been and will continue to be a very active project. Let me give you an example.
I I need to give you some links here while I'm doing this. So VSC version 3.0.0 GA Way back in March and let me give you some links to those announcements.
Well, there's. Well, that didn't work out well. That didn't copy and paste very good at all. Let me try one more time, see if I can get the rest of them.
Sorry for the delay.
Well, let's go with that and see what happens. So since the time of this of these announcements, we've done some servicing releases on 30., We have 301 and 302 have come out and you can get those from GitHub or from the Microsoft Store. But wait, there's more.
So along the way, since March, if you check out our GitHub, you'll also see that since March we've started to release DSC version 3.1.0 previews. We are actually up to Preview 5 already,. So this is primarily work that we've been doing for internal partners.
Towards build, but you can tell this is a very active project and you can also get those from GitHub along the way. Also, as we've been marching through DSC version 3.1, I just wanted to make sure if you didn't catch this out of the blog announcements.
That you got the links here. We're also you can see what features that we're working on at the moment, which ones have been approved and which ones are under consideration. So let me give you some links to that that you can follow us along., We certainly could use your feedback.
As we go through 3.1, as we continue to work on it, I don't have any information today on when that will release, but we are actively working on it right now. Now as Tess had mentioned, a couple of other things that you'll notice from the announcement one.
They're working on a resource for SSHSSHAD. I'm having a hard time this morning. They're working on a resource for DSEV, 3 for SSH.
SSHD config, I finally got it out. You heard about that. Also,. I want you to notice from the blog post that we're actually going to make PowerShell a resource for DSC V3 so that you can configure all your settings in PowerShell in a declarative manner and in addition.
Quickly add some additional comments before I shut up and go away. Additionally, other teams. You will notice also are working on resources as they adopt DSC V3. You may have seen some of that traction with.
Winget and Dev Box and you will probably be seeing some others, so watch for those announcements as they gradually start to come. Out. Also, additionally, the DSC working group has been reactivated to discuss and resolve issues. Thank you to those working group members. I really look forward to working with you folks as we progress.
Through this and your feedback also in additionally, while I don't have anything particular to announce today, you have probably noticed in our announcement blogs on DSC V3 that we are working on Bicep as a DSL for DSC V3. I just wanted to say that this work is progressing.
Rather nicely. Thank you, Andy, for all of the hard work, and we'll be making additional public announcements in the future as the timing makes sense. With that, I I certainly invite Steve to add in anything else that he'd like to, but Sydney.
Thank you very much.

Steve Lee (POWERSHELL)   20:05
I'll just have one thing real quick, like DCV 31 is, as Jason mentioned, being developed very quickly., So don't be surprised to see a 3.1 release. And then I'm not saying when, but relatively soon, let's say, and then we'll immediately go to three two. So right now, DCV 3 is under rapid development.
Development based on our partner feedback, also community feedback. So we're trying to get there into where we need to be as soon as we can., So don't be surprised that we'll have more frequent releases of DCB3 than partial. Partial is kind of on a one year cycle because we're aligned with.net anyway.

Sydney Smith (She/Her)   20:44
Great., Thank you both for that. Next, if Steven, if you're still here to talk about AI Shell.

Steven Bucher   20:53
Yeah, I'm here. Yeah,. I can talk a little bit about some of the investments we're we're doing for the next year on AI Shell. Predominantly we'll be taking kind of the feedback we've been working on, we've been getting based on AI Shell to improve the tool overall. We're trying to make it a little bit more delightful in the CLI for you better.

Sydney Smith (She/Her)   20:54
OK.

Steven Bucher   21:13
Kind of bug support and enhanced. We've done a lot of work already for better Mac OS support, better Azure PowerShell support and expansion of new agents. We'll be working also on better integration with PowerShell itself, and the hot new thing is MCP, the model context.
Protocol. We hope to surface AI Shell as a MCP client, so you can enhance its capabilities pretty rapidly with the vast variety of MCP servers that are out there and usable.
A lot of that works in in progress there, but we should be having a release of AI Shell out soon. A new release of AI Shell. Of course, you can follow along on the GitHub page, submit issues, give it a try, test it out. We're doing a lot of.
But.
Lot of work there and would love any feedback. So yeah, I think that's all we got. If anyone's gonna be at Build, I'll be sharing a little bit about it at at Build as well next week.
And yeah, I think that's that's it, so.

Sydney Smith (She/Her)   22:25
Awesome., Thank you. So much, Steven. And that's a good reminder too. If anyone is going to be at Build, please do let us know so that we can make sure to connect with you. With that,. I'm going to pass things over to Andy to talk about all the miscellaneous other tool updates.

Speaker 1   22:44
I'm not sure if this is news or if we shared it previously, but like the big miscellaneous tool update, I wanted to share is the the major update we did to PowerShell Script Analyzer. I'm just going to go ahead and find which window I want to share here out of 12 Windows.
This one.
Yes, PS Script Analyzer release. OK, great. Yeah, so this came out at this point a couple months ago. This was late March, but you know, it had been a long time since we got an actual full release of PS Script Analyzer out, in fact, I think before.
1.24 our 1.23 was last October, so I was really happy to work, especially with our great community member. I know has Bergmeister here. Christoph really helped me a lot go through. I mean a dozen in each of these categories. We did drop a PowerShell.
From three, we do now only support for this version of PowerShell Script Analyzer, PowerShell, Windows PowerShell 5.1 or PowerShell 7., But otherwise we went through and we got a whole lot. As I said, a dozen enhancements got in, a dozen bug fixes, a dozen process fixes, which is really all about making sure we can release this securely.
And we had a whole lot of new contributors., So I just, I love this community success. Here. Thank you to everyone who made contributions to PowerShell Script Analyzer. This kind of falls under my purview because one of the main ways that people use PSSA is via the VS Code extension.
And so a lot of issues often come into the extension and we have to transfer them over to PSSA. And trust us,. We know it's still our same issue because we're still the same people going looking and fixing those. But Speaking of, we have a preview out. I pushed that out the same day. If you've been using the pre released version of VS Code PowerShell, then you've been testing.
Out.
PSSAV 1.2.4 for us,. So thank you for that. I'm actively working on rolling this to stable. That said, Justin, another community member, probably on the call, Justin Groce and I are working on a major overhaul of our actual testing infrastructure in VS Code. It's just been a little too long. It's still using and now.
Update version of ESLint, so this is getting worked on as we speak. Once we have sorted out the move to VS Code's latest testing packages and get all that sorted, we'll do one more pre release of the VS Code extension and then roll that all out to stable so you will have.
PSSAV 1.24, the latest beta of PS Read line and you know, a great new extension release, hopefully with very minimal bugs because we haven't done a whole lot of other changes to it., So yeah, thank you. So much. I don't think I have too much else here to share.

Sydney Smith (She/Her)   25:34
Awesome. Thank you, Andy, and thank you to all the wonderful community members who contributed to this. Next up,. I have Sean for a docs update.

Sean Wheeler   25:46
All right, let me share my screen. I'm going to drop some links in the chat here. Not a whole lot of docs updates in the last month. April was busy with the PowerShell Summit activities and so on.
There were a few minor releases of PowerShell and updated the release notes for those as Andy was talking about with the release of Script Analyzer 1.24.
We've also updated the docs to match. There's been some changes to some rules, a couple new rules, and a couple of changes to existing rules, so you can find all those here. And then the other big news I wanted to share was finally got around to.
Retiring some content in the Windows PowerShell module space., So if you're looking at that documentation, you'll see that it no longer includes Server 2012 and 2012 R2.
Those have been archived, moved to our previous version, site, so you can still access that information, but that allows us to get over 5300 files out of the.
The current versions repo which helps with our maintenance. This isn't content that I own or actively manage, but I help out the Windows team quite a bit with.
So I was happy to to help them get this retired. That also included retiring the the content for MDOP., So Server 2012, 2012 R2 and MDOP are out of support, but you can still get.
The documentation if you need it and that's my docs update.

Sydney Smith (She/Her)   27:55
Awesome., Thank you. So much, Sean. That concludes the first sort of section of the call. And now we'll move on to our wonderful community demos. First up,. I believe we have Israel.

Israel Milgrom   28:13
Oh, yes. Hi.

Sydney Smith (She/Her)   28:15
Hi, let me give you presenter rights.

Israel Milgrom   28:17
OK.

Sydney Smith (She/Her)   28:27
OK, I think you should have rights now.

Israel Milgrom   28:29
OK, thank you. Hi everyone. My name is Israel and I work for Jane St. And for the past year also we've been working here on a project to migrate our code base to Bowser 7.
We have a pretty large code base, about 500,000 lines of code, so it was essential that we can do this efficiently. And so we came up with a workflow that kind of relies heavily on what was just mentioned, PS Script Analyzer.
Which wonderfully integrates with VS Code and also allows you to write custom rules. So it was really helpful in doing this. The main reason that I'm showing this is kind of like to continue the conversation on the topic of migrating to PowerShell 7.
There is a special section for that on the GitHub PowerShell discussion board. So yeah, please check it out and contribute. So on. All right,. So I'm going to show my screen now.
OK, so this is pretty much this. On the left side here you can see a list of rules that these are custom script analyzables that we wrote.
And in the main panel you can see this is just a bunch of code that we wrote that will kind of like break PowerShell 7 for various different reason, deprecated type and deprecated methods and various different things. I'm just going to turn on the script analyzer rule.
So we can see what gets flagged up.
And you can see the type that I mentioned. You can see the out file command which changed the encoding between PowerShell 5 and seven. You can see the method that was deprecated. Get WMI object was deprecated. Also there's some peculiarities about how a select object work in PowerShell 7. We flag that too.
And so on. And so on. Passing HTML is also not available in PowerShell 7, so this is another thing we flag now the same thing that you get this in VS Code. You can also get when you want it in the terminal.
If you just run against the command the the the script file, you can see all of these different issues that came up with it. And this is very useful because you can take it into your CICD pipeline and essentially maybe have like some kind of a comment or a marker on the file that says oh this file is ready for PowerShell.
7 Don't let anything that is may conflict with that go through and then for every file like that, you just on this command and if anything comes out, you block it from releasing to PowerShell to block it from releasing.
So it's very useful. Like I said, the only one other aspect of it is that we kind of incorporate into this, especially if you want to do it with the CICD pipeline, you need to have a way to retain some of these despite having an having an issue with PowerShell 7.
So I I just picked up random modules from the partial gallery,. The speculation control module module. There isn't too much interesting here. The 2 instances. I think that it uses the get WMI object, which on its own would be a problem, but it's on inside a version check wouldn't run if it's more.
PowerShell version three or or higher, but we want to keep it even for PowerShell compatibility. So but at the same time we want to also incorporate into our workflow of preparing the files to be ready to partial 7 and so on. So we need a suppression here.
And the suppression, we can take the form with this kind of decorator, which you can, it needs to be before the command binding or the parameter attribute. But this basically will suppress all of the L from this.
Command from therefore deprecated commands. If you wanna be more granular you can add another parameter to you that basically says here that basically says the specific thing that you want to.
To highlight, to suppress, and if you look at the problems tab, you can see that we have only one issue here. If I remove it, we'll have two and that's kind of that's kind of the whole thing. So yeah, you can, you can have an iterative approach here, you know you fix a couple of issues, you suppress a couple of of.
Of errors, you rescan your files and and so on and and over time you can make a a, a real dent in the in the work towards preparing it to PowerShell 7.
Yeah, so this is this is pretty much it. I just wanted to highlight again the the discussion channel on on GitHub., I will drop a link later and also we published these rules that I that I showed on GitHub on.
I'll drop a link for that as well. Thank you.

Sydney Smith (She/Her)   33:56
Thank you. So much., I'm sure that this discussion and these roles will benefit so many people. It's such a common problem of how to migrate your scripts from 5:00 to 7:00. So I really love to see the community support and folks helping each other. It's really great.
Alright, we have one more community demo today coming from I believe it was Amir on SSH.

Amir Bredy   34:21
Yeah, that's coming for me. Thank you so much, Sydney, for the hand off. I want to First off thank Test and Steven and Michael and the whole PowerShell team for helping me enable this feature. So I really want to thank them there.
To start, I will kind of just show a couple of slides just to kind of give everybody A-frame of reference in regards to SSHD based management., So bear with me as I try to present to teams.
All right, so I'm sure you guys can see this now. So today I'll just be covering the SSHD security based policies for Windows Server, just to kind of give you guys a note of.
How this is built. It's powered by our OS config, PowerShell module. It's essentially A modular security configuration stack that works on Windows, and there's a Linux flavor too that supports multi off device management across Azure, Azure Portal.
CLI and other local management tools. The great part about it is that it's super modular. It has drift control, which will essentially bring settings back after a change happens. It's portable and extensible, not a huge overhead on any system. It's not.
Permanently tied to any manageability of management authority, and it has a declarative style, so things are done a lot easier. But going forward from that I'll just show you guys SSH posture control for Windows Server.
Which it also has a Linux flavor as well.
So what is SSH posture control?. It allows you to enable and configure SSH security posture on Windows Server 2025 currently and my dev team who's actually on the call right now, who will be showing a quick demo after I show the Azure Policy experience.
Directly through PowerShell is actively working on a Server 2022 and 2019 experience. It integrates with Azure Governance Services. Like I said, Azure Policy and Machine Config so you can ensure compliance, reduce your attack surface and ensure consistent.
HD rules across your fleet of devices. It kind of comes in two flavors or two modes, one being the audit only policy and the other one being an audit and configure policy. So essentially the workflow if you are an IT admin at an organization, would be I'm going to set the audit policy.
First, check which SSHD based settings are on my whole fleet of devices, such as the port, such as what Mac algorithms you can connect with, such as what ciphers you can use, and then you can go through with the remediation or auditing configure policy to actually set those rules.
So here's a little bit of the rules that are pulled from the Linux version. The Windows version has a slightly different set of rules, but I will drop the links in the chat after this presentation so you guys can actually spend some time looking through it. And in our demo that I'm about to start now, you'll be able to see some.
The rules that are in there, so I will stop sharing this presentation and start sharing my screen.
All right. So for now, we're just going to be covering the very quickly the Azure Policy experience and then I will let folks see the PowerShell experience, which is better catered to this community in a sense.
All right, so just spun up some VMS a couple of minutes ago so you guys can actually see it in action, right?. The thing that I did today was I put in the.
The audit policy for folks to actually see let me there you go. I would go to configuration management here within one of my Windows VMS and this would work for any Arc connected machines as well.
And then here I'd be able to see the Secure Shell Preview, which is essentially SSH partial control. After it audited this machine, it showed that there were 9 rules that were compliant and 9 rules that weren't compliant. The 9 rules that weren't compliant ended up being SSH host key authorized key.
Files, the warning banners, the ciphers, Mac algorithms, log and grace time, things that really can affect the security of any of the machines or the VMS that you're working with. So being able to set these rules gives you a stronger security posture across the.
Of the board. And in the instructions that I will be posting in the chat, that will actually show you how you can actually put in these rules., But I can actually run through a quick demo.
Of being able to actually add this. So first you go to Azure Policy, go to authoring, go to definitions, add a policy definition.
Apply the definition to your subscription. I will name it configure SSH.
I would use an existing category such as guest configuration come going forward. This will be a built in policy within Azure so we you won't will not have to actually import the Jason file for all of this I would.
I would add the policy rules which will be in the instructions that I will post later. Click on save.
And assign the policy to whatever subscription resource group that I'd want to, which for me would be my SSH Windows Server demo.
I would add the ability to actually work with our connected machines, create a managed identity for remediation and then from there I'd be able to review and create and then after this it would essentially go through all of the machines that I had created and essentially.
Actually change all of the rules to be compliant with the parameters that were shown here.
So moving on to the PowerShell part, I'm going to hand it off to my wonder, my wonderful developer Hongshu to show you what the experience looks like in PowerShell. Thank you.
Oh, I think you're muted.

Sydney Smith (She/Her)   42:25
Yeah, I can't hear you either.

Hongshu Wang   42:42
Can you hear me now?.

Amir Bredy   42:43
Yes.

Sydney Smith (She/Her)   42:44
Yep, sounds good.

Hongshu Wang   42:44
Oh, OK. OK.
So I I will show the PowerShell experience for Windows SSH configuration now and I will share the screen. So basically we need to install the DOS config PowerShell module on the device and I can show.
The experience after installing that module, Amir will pass the link for downloading and installing the module in the chat. So first we can quickly see if we have the the SSH enabled on the device.
Um, we can try to essentially do that.
So this will be a normal experience for the local. So when SSH into that IP address and after we have our module installed, we can starting to check the status of its initial state.
And check the compliance.
So this will show the initial default state after enabling SSH and start SSHD server. This will show what is compliant, what is not. We can also get a full.
List of values and compliance status. Just calling the get OS config desired configuration. So this will show all the values, current values and the compliance reason of it. So to configure it.
We can choose to set the default value for the for all the settings, so and also showing the verbose message of how we configure the SSHD file. So after this configuring all the values to it.
To its default value, we can get the compliance again.
And this will show what is compliant, what is not and if if we go to the.
Client again and we SSH do that. It was it will have the fact of the better file that we said showing the.
New banner file was that.
Uh, we can also um.
Configure a single setting., So if we do set OS config desired configuration.
We also provide um tab um auto completion for all the settings and uh.
If we choose one.
And this will configure only one settings with the custom values. So after that we can get the compliance again we can just do get OS config.
And this will show the current value with the compliance status. And if we go to the client machine again and trying to SSH into that machine, it will show the new banner file that we configure.
So this is the main flow for the PowerShell experience for Windows SH configuration.
That's all.

Amir Bredy   46:48
Thank you so much Hong Shu for giving the PowerShell version of the demo. I put the link in the chat for you guys to access if you'd like to try it out. It would also have any of the contact information from our team as well if you want to reach out to ask any questions, give any feedback and.
Wanted to thank the the PowerShell and Open SSH community again for having us. So thank you so much.

Sydney Smith (She/Her)   47:16
Thank you. So much. And I just wanted to correct one thing I said. Earlier. Amir and friends are from a partner team at Microsoft,. If that wasn't clear, not directly a community. So thank you so much for being here though. And for the great demo. And thank you everyone for participating today and being a part of our call. We look forward to potentially.
Be seeing a few of you at Build next week and then at our June call next month. So thanks again for being here.
Yeah, if that wasn't clear, that was the end of the call. Thank you, everyone.

Jason Helmick stopped transcription
