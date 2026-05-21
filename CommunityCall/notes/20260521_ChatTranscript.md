PowerShellOpenSSH Community Call-20260521_092633-Meeting Recording
May 21, 2026, 4:26PM

Jason Helmick started transcription


Jason Helmick   5:39
Perfect. Perfect. Well, I'll get started here and then Artem, I'll bring you up. Give me one second here to fix my screen.
All righty, this sounds like a good time to get started. Hello, good morning, good afternoon, and good evening. Welcome to the May 2026 PowerShell Community Call, where everyone is welcome to join and discuss the PowerShell project and its development. We have our agenda posted, and what an agenda it is in the meeting chat and a link to the meeting discussion.
Please feel free to post questions in the discussion list or right here in the meeting chat, which is probably the easiest to do. And please follow our code of conduct as well. First off, for this May, I want to apologize for the late cancellation of the last community call. That one's on me.
I had to step away unexpectedly. But now we're back in action with a fully loaded agenda today. So here we go. Oh, and one other thing. Happy 20 years to PowerShell. Yep, every call, folks, I'm going to say it. 20 years and still the best tool in the room. So with that, let's get started with
our very active agenda. So first things up, some of you may have noticed some announcement last week from the Windows Server team at their summit. Well, we are very fortunate today to have Artem from the Windows Server team to stop by and to talk a little bit about it. So Artem, thank you so much for joining us today.
If you'd like to introduce yourself and have a little chat, take it away.

Artem Pronichkin   7:22
Thanks, thanks, Jason. Thanks, folks. Very happy to see you all here. Good morning and good afternoon, good evening, depending where you are. So, I'm Artem from Windows Server team, and I'm here to briefly talk to you about Windows Server.
Yes, Windows Server. So earlier this month, as Jason said, my team hosted an online event called Windows Server Summit. And in case you missed it, I'd like to share one particularly exciting announcement that we made there. So we've heard many times from you folks that if you love PowerShell and you use Windows Server,
the two could be a little bit more better together, a little bit more integrated, a little more effortless. So as the Windows Server team, we are working very closely with the PowerShell team right now.
to make PowerShell 7 installed by default on the next version of Windows Server. So when Windows Server vNext, as we call it, the next version ships, it will include PowerShell 7 as a pre-installed app. So it will be available to you immediately once you install the operating system or upgrade to it.
And the user experience will be very similar to how you have PowerShell 5.1 installed by default today. And it will also be similar to other management apps that we ship in box today, such as Windows Terminal and then get if you are familiar with those of Windows Server, current versions of Windows Server.
And full transparency, this is still a work in progress, and we are very well aware that there are several incompatibilities potentially that exist that may prevent you from using PowerShell 7 on Windows Server in all the same scenarios that you might be using PowerShell 501 today. And we're working very actively right now with the respective teams throughout Microsoft.
to fix or remediate those issues in time. We know we'll have to make it work.
And at some point in the near future, I don't know when exactly, but hopefully sooner or later, we'll share a next Windows Server Insider builds that will contain a preview of this new functionality. So if you are familiar with Windows Server Insider program, this is the previous channel where we share what's next for Windows Server, the previous builds of Windows Server the next. And
It won't be the final state initially, some incompatibilities might not be remediated initially, but you will get a sense of what it looks like and you will see us making progress and remediating those issues over the course of next year or so.
And right now, guess what? We need your help shaping this future. And we very much welcome feedback from the community, especially such a great community as you folks. So that's why I'm here to talk to you about this specifically. So please share your thoughts with us. What do you think about this idea to include PowerShell 7?
inbox with the next version of Windows Server. If you know that there are some limitations that exist today, what prevents you from using it efficiently, like as it is today? And what else, I guess, what else you need from us as the operating system team that I represent here to make you productive?
and successful with PowerShell. So we have an online forum for Windows Server insiders. I will post a link to the chat window, but it's basically aka.ms slash WS for Windows Server dash forum. Again, I will put it in the chat window. So if you folks can.
go there and share your feedback on this announcement that we made this month. That would be great and we will be very appreciative of your help and feedback.
Any thoughts immediately?

Jason Helmick   11:20
I see some congratulations going on in chat and lots of little questions.

Steve Lee (POWERSHELL)   11:24
Artem, look for any comments and questions in the chat.

Artem Pronichkin   11:28
Yes, I will. I will try to address them in the chat.
Or maybe if we have time, let me see what's there.

Jason Helmick   11:39
That's awesome, Artem.
Artem is going to hang out with us in the chat for a while. And so feel free to ask some questions. There's not a whole lot of information that we're sharing right now, but Artem can do that. Artem, thank you so much. Much appreciated. So folks, getting PowerShell 7 into Windows has been the community's number one ask and our commitment for a long time.

Artem Pronichkin   11:42
Tess.

Jason Helmick   12:04
We hear that. And this team is actively working on it. We will share more information with you when we have some things to share. So with that, thank you so much and congratulations to everyone.
I'll let the chat kind of spin here for a little bit as Artem and folks can kind of address a few things. Steve, did you want to say something? Oh, yes, of course Steve wants to say something because the next...

Steve Lee (POWERSHELL)   12:26
No, I, I got, well, I got the, I have the next agenda item anyways, right?

Jason Helmick   12:29
Yes, you do. Steve, go ahead with whatever you want to say.

Steve Lee (POWERSHELL)   12:30
Alright, but I guess...
No, no, the only thing I'll just mention, and I just put this, I already put this in the chat, you know, my team is prioritizing fixing some known issues with the Apex install. So it is the Apex version of the inbox. And also, you know, one of the things on top of our mind is how do we help users migrate from 51 to 7, right? So nothing to announce at this time, but it is something that we're thinking through how to make that.
I won't say seamless, but at least less painful. And so we won't have anything announced until we have a plan, I guess.

Jason Helmick   13:02
Yeah, we will be making that announcement, as Steve pointed out, especially around migrations and things like that. We have, and I'm just going to share this right now, we have a code word for the migrations called 527. Get it? 527. So yeah, we'll release more information as that becomes available for us to release. And with that, Steve, since you're still on, why don't you talk about this great release we just had of DSC version 3.2.

Artem Pronichkin   13:02
Yes.

Jason Helmick   13:26
Two.

Steve Lee (POWERSHELL)   13:27
Yes, right before I get to that, actually, now that the cat is out of the bag about Windows Server and PowerShell 7, Jason, why don't we create a discussion in a partial repo to have people bring up concerns about the migration?

Jason Helmick   13:39
In fact, I totally agree. Yep, in fact, I totally agree. Am I right? We're going to talk a little bit more about that towards the end of the call, in fact, so.

Steve Lee (POWERSHELL)   13:39
or compatibility and stuff like that. All right.
Okay, so while you do that, so let me just quickly cover the DC32 release. So we did GA, DC32 3 weeks ago, and we actually had a servicing release, I'm gonna scroll up here now, three days ago, based on some feedback from some internal partners. All right.
Most people probably won't hit this one. Basically, if you have a DC resource in a path that has a PowerShell based DC resource in a path that has spaces, there was a problem. So that's been fixed in 321. In 32, and I think Sean had added a link in the chat to the Jason's blog post, so I'm not going to cover all the changes and stuff in detail.
But I did want to maybe take this time to talk a little bit about 3.3 preview that happened 2 weeks ago, and specifically one of the new capabilities, so is.
This thing called a registry list. So again, we have a lot of internal partners starting to use DSC. And one of the things that was apparent is that they're touching the registry in a lot of different places. So rather than spawning the registry resource for every instance, I created this registry list resource. So if I go to here.
So this, this is obviously a test for this resource, but this kind of shows you what a configuration. Again, this is a test case, but your actual users will look different. But basically, instead of having individual key paths and values for each instance of registry, you can have an array of them as part of 1 registry list. So the resource only gets spawned once.
And you can point this to anywhere. So they don't have to be like all under one keypad. You can point all over the rest tree. This is a get example. So if I copy this here.
Go here real quick.
Paste that in there to get.
Yes.
So one of the, so a couple of benefits here. One is you don't have this super large configuration because you have all these things kind of repeated for the resource. But also it's going to be much faster because we're not spawning the registry resource every time. We're just spawning the registry list resource once and getting all the information. So play with that and provide feedback in the DSC repo. And that's all I got for this one.

Jason Helmick   16:06
Awesome. So PS resource get 1.3 preview one. Anum, can you help me with more than just the title for that?

Anam Navied   16:17
Yeah, let me share my screen.
Hi everyone. So PSResourceGet just had a 1.3.0.preview1 release, and this included some features and bug fixes. So one of the features is that now MAR is going to be registered as a default trusted repository. And what that means is that if you have a fresh machine,
and you run PS resource get commands for the first time. MAR or Microsoft Artifact Registry will be registered as a default trusted repository. Like we've kind of done in the past, PS Gallery will also be registered, but it will continue being registered as a untrusted repository and MAR will have a higher priority.
If you already are using PS resource get on your machine, you can kind of get this behavior by running register PS resource repository with the new Microsoft artifact registry switch. And what that's going to do is it's going to add MAR as a trusted repository to your repository configuration.
And one thing that we do want to note here is that, as the user, if you don't want MAR registered, you could obviously run unregister PS resource repository to unregister it, or you could also run set PS resource repository to change the priority to be lower, so MAR isn't checked first, but really this.
feature is just kind of introducing a transition to MAR to be checked first where possible for Microsoft owned modules and kind of provide that seamless transition where we can. Another feature that was added in this release is concurrent installations. So if you're installing a package that has dependencies, that's going to now leverage.

Haviland, Wallace D.   17:51
Yeah.

Anam Navied   18:06
concurrency for improved performance. So would love kind of some feedback on these new features. Oh, and then another feature that we added is DSC V3 resource for PS resource get. And Aditya will talk about that later in the community call today. We also had some community PRs in this release.
So one of them is a bug fix to better detect if the current PowerShell session is Windows PowerShell or PowerShell Core in certain instances, such as when the session is running in VS Code or.NET Interactive or with Ruby or Puppet. So shout out to Borg Quit. I'm hoping I'm pronouncing that right.
for that PR. We also had another fix where pre-release labels are honored in update PS resource scenarios. So let's say locally I have a version 120 preview on my machine and there's a new version 120. That pre-release label is going to be compared.
more accurately and then it'll update the user correctly to the new version. So shout out to Sean R. Williams for that PR. The other thing I quickly wanted to mention is that as we did this 1.3.0.preview1 release, the team also published A blog post for our roadmap.

Jason Helmick   19:18
CAI.

Anam Navied   19:27
and vision for the 1.3 releases. So I'll share a link to this in the chat right after. But over here, we are kind of covering our vision for DSC resource support, Microsoft Artifact Registry, kind of becoming the default repository, as well as what we envision for cross-feed dependency resolution.
And those scenarios, I also have a RC for cross feed dependency resolution, so I'll share that as well after this. We also talk about concurrent installations and performance improvements, standardizing or as support. So right now we're supporting ATZRC, Azure Container Registry, and...
publishing OCI artifacts there, but we want to extend this to GitHub Container Registry, Amazon's Container Registry, and others. So adopting the ORAS SDK library will allow us to do this. We also talk about dropping support for Windows PowerShell in future PS resource get releases, so 1, 3, and beyond.
And this would really just let us adopt the ORS SDK library and standardize that, as well as adopt new features that maybe we've been a bit limited for in the past, and then just better align with cross-platform scenarios. And obviously, 120 and previous PS resource get release versions.
will still be supported for Windows PowerShell. But yeah, give the blog a read. I'll drop a link for that. And as usual, we really appreciate actually all the feedback and insight we've been getting from the community and would love to hear your feedback as you guys test this out. Thank you.

Jason Helmick   21:07
Thanks, Anam. That was awesome. I'm so glad to see what's going on with PS Resource Git. And we're not done with PS Resource Git. Aditya, are you around? Do you want to talk about the resource?

Anam Navied   21:08
I.

Aditya Patwardhan   21:17
I am, so hello everyone.

Jason Helmick   21:18
Oh, perfect.

Aditya Patwardhan   21:21
So, as Anam announced, in the first preview of PS resource at 130, we have included the DSC resource or PS resource get. I'll show you, it has two resources implemented right now. I'll quickly show you what they look like.
So the two resources that we have right now is the repository resource, which is used for configuring the repository, for resting, unresting, setting, trusted, changing priorities and so on. And another one that we have over here is where
you can install a list of modules. So let me come up with a bit. So yeah, so install the list of modules. So this is called a PS resource list. And these repository, these resources are implemented in PowerShell as part of the module itself.
And with DSE latest version, it should be able to look it up and directly use them once you have this module installed and in path, as in a PS module path.
So, the I guess I'm next for topic as well.

Jason Helmick   22:29
And, and, and since you're since you're here, why don't you go ahead and talk about the exciting 77 preview one?

Aditya Patwardhan   22:32
Yes.
So we released 7.7.0-preview.1 about 3 weeks ago now. That's the first preview release for 770 train based on.NET 11. So please give it a try and give us feedback. One of the amazing things that happens in previews is we get a lot of community.
PRs included in the first preview because we have not been shipping previews since we are doing the work on GA and RC. So this is always exciting to see a lot of new community PRs coming in. And we would shortly be getting, I don't have a date yet, but a preview to coming up very soon. And in addition to that, we will also be having some servicing releases for 7475 and 76 coming.
very shortly, either this week or next week.

Jason Helmick   23:23
Outstanding. Thank you, Aditya. That's awesome. Folks, yesterday, Sydney posted a blog on the PS Gallery roadmap, and I got a chance to read through it, but I am so grateful that Amber is going to be here to kind of talk about what was in that blog. So Amber, you want to tell us what's going on with the roadmap?

Amber Erickson   23:45
Oh.
Hello, yes. Okay, so just going to talk a bit about a little of the future of the gallery and what we're working on right now. So we've actively been working on migrating the PowerShell gallery from cloud services to Azure Kubernetes service. And this move is driven like first and foremost by necessity. Cloud services are being deprecated.

Jason Helmick   23:48
Hello.

Amber Erickson   24:09
within the next year or so, but it also provides us an opportunity to modernize our architecture. By moving to AKS, we are shifting to a more flexible container-based platform that is better suited to handle the scale we operate at today. The gallery currently sees roughly 625 million requests per day and about 40 million downloads.
And the existing architecture wasn't really meant to handle that level of load. It also wasn't really meant to support a microservice architecture, so it's really difficult to modernize isolated components of the gallery. The result of that is that we have a system that's been quite challenging to evolve. So this migration work, even though it is non-trivial,
will put us in a better position to address some of the more customer-facing issues that continue to arise, like issues with reliability. And it also allows us to kind of incrementally reduce the technical debt that slows down some of the other work that we want to do for the gallery. So one of the more...
pressing user facing issues. For roughly the last nine months or so, we've been seeing intermittent issues with the gallery where search either returns no results or only a small subset of what users expect. And this issue stems from our Lucene based indexing system, which is the backbone of our search functionality.
And at a high level, the system itself is still functional, but under heavy load, it really struggles to keep up. We do suspect that what's leading or what's causing this is delays in indexing, which in turn makes searching more inconsistent. And based on our investigations, we believe that this is largely driven by the scaling limitations of cloud services.
and an architecture that just becomes more fragile under significant load. However, we are optimistic that these issues will improve as we complete the migration to AKS, which will allow the service to just be more scalable and have a more resilient foundation. In the meantime, we are working on trying to reduce the frequency at which these issues occur.
We do have a workaround, so if you run into these search issues, we recommend using PS Resource Get to search for and download packages. That said, some scenarios are still impacted, particularly commandlets that rely on tags or commands or wildcard names.

Haviland, Wallace D.   26:37
Yes.

Amber Erickson   26:40
So those still may return incomplete or empty results. We do have more information on this on the gallery status page, and we will update there as we make progress. And I will drop that in the chat in just a minute, but just one more quick item I wanted to talk about, which is the work to integrate Entre with the gallery.

Haviland, Wallace D.   26:50
Yeah.

Amber Erickson   27:03
So Shamu's actually been working on this, and unfortunately, after having made some significant progress, had to rewrite some things to accommodate for a new breaking version of the MSAL packages. That said, this work is now getting close to completion, and it's a huge meaningful improvement for us security-wise.
It moves us away from long-lived API keys to an identity-based authentication that uses short-lived tokens. And this just better aligns us with secretless security best practices, and in general, reduces the risk of credential leaks or misuse. So we're really excited to have that out and hopefully out in production soon.
And that's about it for the gallery. If there's any questions or concerns, feel free to drop it in the chat. And I will also link to the status page with the issue that I previously mentioned.

Jason Helmick   27:59
Outstanding. Thanks, Amber. That's wonderful. So next up is, so we got a lot of discussion about this and we got some folks actually in this meeting discussion asking about Rel 10 and Debian 13, that kind of stuff. And we have updates for that. So Anam, welcome back. You want to update us on Rel 10 and all that?

Amber Erickson   28:01
Mhm.

Anam Navied   28:21
Yeah, I just want to say hello again everybody. So I wanted to give some updates for PMC support for different distributions. So Route 10 is now available for PowerShell 77 Preview 1. It will also be available for the upcoming 76 release.
So expect to see that soon. And then Debian 13 is also available for the PowerShell 7.7 Preview 1 release. And expect to see 7.6 soon in the upcoming release. I know we've also talked about Debian ARM support. So for Debian 12, Debian 13 ARM.
So that will be coming in future 7.6 releases, not in this next one, but in future ones really soon. And then we're also doing work to have Ubuntu 2604 support for PowerShell releases coming soon as well. So I hope to have a better update on that in the next community call.
But coming very soon for sure.

Jason Helmick   29:23
Oh.

Haviland, Wallace D.   29:24
HEHIM.

Jason Helmick   29:24
Oh, but that's awesome. So we got Rel 10 out, Debian 13 out. We're working on the ARM stuff for Debian, and we're working on Ubuntu 2604 for a future announcement. So we're moving right along, folks, and getting those out there. So let's, okay.

Anam Navied   29:37
Yeah.

Jason Helmick   29:43
So let's talk about the MSI deprecation. I want to start by thanking everyone who engaged with the blog post and in the PowerShell Discussions. You were, shall we say, thorough. We heard you. So for those who missed the blog post, essentially what the blog post is saying is starting with PowerShell 7.7.
We're moving away from MSI. PowerShell 7.6 LTS still has MSI, so nothing has changed for the current deployments today. And in all seriousness, the feedback that we've gotten is really valuable. The scenarios you've raised around MSI X gaps are real, and we know it. And the PowerShell team is actively working with several other teams to address those.
Now, we're going to get more transparent about this going forward. As a matter of fact, Steve started to mention about this. So you're going to start seeing those gaps publicly tracked and marked in GitHub so that you can follow along with that work. We know that it's important to you to make sure that those scenarios are being responded to.
I expect that to happen in the near future when I'll start creating that list. So I'll let you know when that happens, and we'll make sure that we have announcements out about it. There will be continued active discussions both around MSIX in the scenarios and around, well, now you also now know one of the other reasons is around inboxing into Windows.
And so we will be happy to receive your scenarios and what you think are issues. And I want to point something out. We already had a lot of research on MSIX and we were well aware a lot of these scenarios and a lot of the work had already begun. However, you coming in and pointing in
pointing these scenarios out to us, re-emphasize them, and we have learned things from you. So if you have additional feedback around MSIX or around Windows, keep it coming in the discussion thread. I'll create one specifically for Windows Server. You can create one yourself too, but I'll create more of a formal one for Windows Server.
We already have one for MSIX. Please let us know how we're doing, and we'll give you a way to see how that work's being accomplished in the near future.
So thank you. That was a lot of fun. The next topic, boy, this has been a long road getting here. Is Andy still on the call? I didn't look. I know he had to drop.

Steve Lee (POWERSHELL)   32:13
No, Andy Andy had to drop off.

Jason Helmick   32:15
Okay, that's fine. Folks, I'm going to bring Andy back at the next call just so that he gets a chance to talk a little bit about this. But a lot of folks that work with Mac today, the Mac OS, have been asking for, are we going to deal with the Mac notarization requirements? And the answer has been, we've been working on it, we've been investigating. It's been very challenging in our pipelines.
And it's a combination of, you know, what's required for Microsoft security, what's required for Apple security. And sometimes those things don't blend really well. Well, it got figured out. And so yes, Mac notarization is starting now. And matter of fact, you may have noticed the blog post that went out this morning about it. This is going to be Mac notarization will be for 74
75, 76, and 77. And so go ahead and use the brew formula. We haven't committed to new brew packaging yet. We're just now getting these releases out and we want to make sure that everything's working okay. But we'll have a chat with them and we'll let you know in the future.
specifically more about Brew. But I want to congratulate Andy, the maintainers, and the rest of the PowerShell team on getting the Mac notarization done. So a woohoo for all the Mac users out there.
And I'm just pausing for a moment to let that one sink in before Sean.
You want to give us a docs update? And I know, I know for a fact that you have one particular doc update from, I think he's a pretty relatively new contributor that is just outstanding.

Sean Wheeler   33:57
Yeah, so let me, I put all the links in the chat right after the agenda so you can find all of that. Let me share my screen.
Just one more thing before we get there, wrapping up what you said about the Mac notarization. The packages for the next releases will be notarized. So you'll be able to download the PKG file and just double click it to install.

Jason Helmick   34:19
Yeah.

Sean Wheeler   34:31
So there's still work to be done around BREW and Discussions to be had around BREW going forward, but at least the pain of that will be going away with the package itself. Okay, so
We got a couple of months to catch up on here. Of course, you heard about all of the new releases. Let me get back here. So what's doing docs? Yes, Sufficient Dikon is the GitHub handle. I apologize. I'm...
blanking on his name at the moment. But he's made some notable contributions over the last couple of months, and I've got a big PR from him that I still need to review. But one of the biggest ones was this about error handling, finally created
compiled all the various notes from different places about error handling in PowerShell, and really help clarify the terminology around error handling with non-terminating versus statement terminating versus script terminating errors. There's very different behaviors for each one of them. And
Hopefully this article helps people with that. I know it was difficult. It was one of those things that I struggled with trying to coalesce. And he spent a lot of time and a lot of tokens.

Jason Helmick   36:02
Yeah.
And I am.
And I just want to point out that it's not that it was just Sean that has spent a lot of time trying to figure this out. We have a great contributor that many of you know, M. Clement, who gave us a tremendous amount of wonderful information about this. And it has taken us forever to figure out how best to boil this down and to get it succinct and to get it

Sean Wheeler   36:14
Creating this.

Jason Helmick   36:33
clear and sufficient Daikon really, really brought that together for us. So this was a big, big thing on an important topic.

Sean Wheeler   36:42
Yes. Going back, what else? Of course, we had the 7.7 release. Some things I want to call out there, as Aditya said, the initial preview releases, you'll see there were tons of community contributions because we finally get to move forward with an LTS release.
We can't make those kinds of breaking changes. Now in a preview build of the next version, we can start doing that. And one of the things I want to call out is there's been a ton of work here fixing the problem with switch parameters on many of the
commandlets where they weren't being evaluated properly if you passed the value using the colon like this. So
Almost all of that's been cleaned up. There might be a couple of outliers.
still, but I know that that's been a big long-standing one. Several other commandlets have been updated. This is a neat new feature, the PS application output encoding preference variable. Now you can control
how PowerShell treats the encoding of output from native commands. So read more about that. The docs have been updated around that as well.
What else do we have here? Let me go back to...
And then last month.
Last month was the change to how we install for Mac OS and the Brew Community.
removed the task package we had and replaced it with a formula. The formula builds the install from source code, so it gets compiled on your machine. So this is not something we can support.
but it's out there as a community-based solution. But now, thankfully, we have signed PKG files, and your install experience will be much simpler going forward. And then there was a big release with Script Analyzer version 1.25.
had several new rules and a few updated rules, so you can read all about those here.
And also, just circling back about yesterday, I published updated documentation for the PS Resource Git preview. And this is the command that Anam was talking about.
This is how you register the Microsoft Artifact Registry as a repository on your system. So it won't be there by default, but...
Much like if you needed to re-register the PS Gallery, it was dash PS Gallery. There's now dash Microsoft Artifact Registry.
When it does that, it's actually installed with a lower priority number, which is a higher priority. It'll be searched first, and it's automatically trusted, whereas PS Gallery is...
Has a higher priority number.
which means it'll be searched after and it is untrusted by default.
Of course, you can change all of those settings using the set PS resource registry. Also, just wanted to point out that...
The reset command, PS resource, reset PS resource repository has been updated. This wipes out any.
PS resource repositories that you have registered and writes the defaults.
And so now with 310 preview one, it's going to write both the Microsoft Artifact Registry and PS Gallery with the following settings. So.
That's my docs update.

Jason Helmick   41:28
Oh, that's awesome. Thanks, Sean. Really appreciate that. Folks, we have a demo about the new version of Plaster 2.0. We postponed it because of the, obviously, the packed agenda that we had when we postponed it to next month. So you'll see that next month, but also I want to open up next month. Oh, this is going to be so much fun for me.

Sean Wheeler   41:30
Mhm.

Jason Helmick   41:49
Open up next month to a second demo so that we're not too behind. If somebody else would also like to do a demo next month, here's how I figure that out. I will look at the June call that's in our discussions and I will pick the first one that's in there. So that'll be the demo for next month. So everybody race out there if you want to make a demo.
And then we'll add other people to the list and we'll just go from there. So we'll bring back demos next month. Also, we have, I just wanted to say we had a great experience at PowerShell Summit North America last month. I hope to maybe bring Andrew from the podcast.
in for maybe next month to give us some updates, but also right around the corner, any day now, matter of fact, folks are getting ready to leave from here for PowerShell Conference EU. And so we hope that they have a wonderful conference and we're looking forward to all the exciting news that they have. And we'll see how that lands and maybe the following month we can have
also an update on that. So once again, thank you all so much. This has been recorded and we'll get the recording out in a few days. Thank you and have a wonderful end of your month of May and we'll see you back in June.
Cheers, everyone.

Jason Helmick stopped transcription
