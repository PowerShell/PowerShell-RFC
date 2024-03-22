# PowerShell_OpenSSH Community 20240321_185644-Meeting Recording

March 21, 2024,

SSydney Smith (SHE/HER) 2:38 Totally a fair ask of there is more than one time zone, so. Umm, great
everyone. Well, we are almost at starting time, but I'll get folks just a couple of minutes to join.
So thanks for being here. If you see the in the chat, you'll see that Jason went ahead and posted
are little welcome message. Just a reminder that the call is being recorded and will be posted to
the community. Call YouTube channel, so if you aren't comfortable with that, feel free to drop from
the call. Also remember that we do take our code of conduct seriously and feel free to always add
agenda items and questions to our discussion on the (PowerShell) Discussions page, which is also
linked here. Per usual, we will begin with our slotted agenda and then I believe we have a Community
demo and then we'll open up to questions and kind of go from there. So I'll give folks just like one
or two more minutes to join and then we will go ahead and fully get started. Alrighty, well we have
a bit of an audience. So I say we go ahead and get started since we're being recorded anyways. Hi,
I'm Sydney. I'm one of the PM's on the PowerShell team and thank you for being at our community.
Call this March, as I just said. They uh call will be recorded and it will be posted to our YouTube
channel. I see a question in here asking. A ohh it would just asking will it be made available? Yes,
it absolutely will be. If you weren't comfortable with being recorded, please drop from the call.
Typically we try and get the call up within two weeks, but it's usually a little bit quicker than
that. Please remember our code of conduct and we'll get started with our agenda. So first up, we
have Danny. I'm talking about SSH inbox. I think you're muted.

Danny Maertens 5:22 I am muted. I click the camera button, not the mute button. Not thought.
Alright, I folks, quick update on the OpenSSH side of things. So as some of you might know, Win32
OpenSSH which is our port of the OpenSSH project from Openbsd which ships in Windows as a feature on
demand. So an optional feature which must be installed starting in Windows Server 2025 that will now
be installed by default. If you want to try that out before Windows Server 2025 releases fully in
this fall, if you part of the Windows Insider Program, Insider builds are already available with
OpenSSH installed by default. Again, by default we SHA is off not on listening and when it is turned
on, it by will default to private network only connections. So we are definitely still security
minded putting additional things into the box. So give it a try. I'd love to hear your feedback and
that's it. An open safe side.

Sydney Smith (SHE/HER)   6:29
Awesome, very cool news from there.
You'll get to hear from me for the next few updates and so first up I'll give a little update on where we're at with PS Resource get.
So first I'll say that we did release another path update to PS Resource get.
So this is 103.
This release just included a few small updates.
Umm these were bug fixes that we found while doing feature work that we wanted to make sure sort of got backported so to speak and included in the patch release.
So if you haven't, you can go ahead and safely update to 103.
That is now out.
Should be just minor bug fixes.
Sort of.
The more exciting update I would say is that we are nearing completion of our first preview for the next iteration of PS Resource get SO110 Preview one should be coming out soon and what this preview does is it adds support for container registries as a Gallery.
So in this first iteration we will be supporting ACR as a private gallery, so I can give just a quick demo as to what that will look like.
And as I mentioned, you can see this all on GitHub now, if you will curious to go look at the source and see what the PR is kind of look like, but the fully available package should be coming out in the next couple of weeks.
So I will share my screen.
We go here.
Alright, so if I do I get PS resource repository.
Uh, pipe to full list.
You can see the repositories that I have registered and you'll see that I have this repository registered called ACR.
You'll see the Uri that I have here and a few other pieces of information.
I'll show you how I got those information.
Pieces of information how I registered the repository and sort of how it works, but it should be a pretty simple demo, so if I go to the Azure portal here you'll see that I created umm this container registry.
It's called MVP 2024 cause originally created this specifically as like a short lived demo for MVP Summit, but ended up scrapping that demo.
But regardless, we will reuse it before I delete this all, so to get the Uri for registering my repository, I'm gonna use this login server here.
So if we copy that, you will see go back here, you'll see that the Uri it just stopped cause the HTTPS name for PS resource repositories or PS repositories.
These are just friendly names, so can be whatever you want it to be trusted.
I Mark to this as true because this is my private repository, so I'm going to know everything I put in there and I'm going to trust it by default, totally up to you.
I just have the default priority here.
Umm.
And then for credential info, you'll notice that this is blank.
So this is kind of an exciting feature in my mind, but for ACR or for Azure Container Registry, you will not need to use the sort of like secret management secret store approach to credential persistence.
We've actually directly integrated with the Azure Identity SDK.
So at your first call to the repository, it will go through the various types of authentication.
So we'll go through the authentication flow set up by Azure.
So usually what this looks like is it will prompt you in the browser to log in.
There are other ways to log in, say like if you're in a CI CD system or whatever it is, but kind of the default option I would say is the browser login and it will save that for you so you don't need to provide it there.
And then API version is something that you sometimes need to worry about sometimes don't.
I didn't set this, it detected it based on my Uri.
It detected it was a container registry, but if you ever knew you were using a container registry and it wasn't set, you could always set that.
So that's a little info about the repository if you wanna see like the command, let's see.
Yes, resource.
Try and spell it right.
Uh.
Name ACR.
OK, cool.
Umm, so you can see this is all I provided when I initially I don't need the force or I I would need the force now because I'd be reregistering it but anyway so this would be all the information I would need to provide because the credentials would be.
For would be prompted and yeah, OK, so once they have that registered.
At publish time I could show you the command I used to publish.
OK, So what I would publish, I would just provide a path to my PS1 or a path to my folder and then provide my repository again.
I would not need to provide an API key or my credentials because those are so just gonna show that and then I've already had this published.
I'm not gonna run this again because it will fail because I haven't updated the version.
That's my package, but if you aren't familiar with container registries and you're wondering like, hey, how in the UI would I see my package?
So what?
It's gonna be called as the repository.
Go to.
Click here and then you can see my test package.
Here the set 100 is the version that I have.
So now if we run find PS resource repository ACR name Mike has package.
Have all the suspense of a live demo.
There you go.
It returns, so hopefully that's a pretty straightforward demo.
I could show you install as well, but it would behave exactly like you're familiar with.
Install umm.
So yeah, pretty straightforward, but that's basically how ACR works as a private repository.
I'm gonna see if there are any.
Are the Azure identity SDK & ALC.
I immediately see potential AZ conflict here.
Yes, we did take very great care umm to avoid assembly load conflicts with AZ and worked closely with AZ Team who's also on the call here to avoid any potential load conflict.
So yeah.
Umm, I'm just going up.
Uh, James, I'm not sure if I understand your question.
Are you asking is this about SSH or is it out?
PS Resource guide.

James Brundage   14:11
It's about how PS resource get would work at in Docker, particular or ACR.

Sydney Smith (SHE/HER)   14:15
Mm-hmm.

James Brundage   14:20
I'm a little concerned because I'm starting to create Docker images that are essentially wrappers and service points around modules, so I'm already kind of potentially hitting a name conflict if they can't coexist, but if they can coexist and this can have an entry point, then this is a great answer here because what I am actually doing is creating a container image, and I think you're doing here is creating a container image.

Sydney Smith (SHE/HER)   14:35
Uh-huh.

James Brundage   14:47
So as long as I can just throw a proper Docker image in there, this is great.

Sydney Smith (SHE/HER)   14:47
Where Kate can.
Yeah.
Yeah.
So what we're containing we're creating as OCI artifacts.
So those should be able to be side by side I would think, but I'm happy to follow up with you about that.

James Brundage   15:04
Yeah, let's take this one.

Sydney Smith (SHE/HER)   15:06
OK, cool.
Sounds good.
Great.
So that is the PS resource get update and then the next update is around PSGallery.
So a few a few things to know about PSGallery.
I will actually just share my screen for this one as well.
OK, great.
So I just wanted to point out a few UI changes that we have made.
So the first thing you'll notice is that, like the statistics has gone here.
I don't know.
It's just saying ohm so I read it.
Statistics was broken and we decided to make the call to remove it and focus on individual Package statistics instead.
So we have begun the process of rewriting the individual statistics pipeline as part of a long term effort to kind of like rewrite the entire Gallery.
We're starting with the statistics individual statistics pipeline.
We kind of have to do the Gallery piece by piece to make sure we are not sacrificing things like availability or have causing downtime, but we are in the process.
Uh doing small rewrites of the Gallery, starting with the individual statistics pipeline.
We are not planning to rewrite aggregate statistics.
It's a really, uh intensive job and we weren't seeing a lot of value for it.
However, more than happy to take feedback if folks feel that aggregated statistics umm would be really useful.
Umm.
So that is one thing, a second change we've made is I'm just gonna pick them very first Package.
We have swapped out owners and then if you go to Package util and authors.
So we've switched these two in the UI so that the first thing you see is a Package owner and a package author.
So I just wanted to explain to folks sort of what's the difference between these two things is so that if you're on the gallery and you're looking at packages, you kind of have a little bit of literacy around what you're seeing.
So in this case, authors is pulled from the package metadata.
It's the pulled from the PSD one anything can be put here.
There's really no verification.
I mean, it has to be a a string, but aside from that, you know there's no verification as to what somebody puts here.
You could put any name here, right?
So I want folks to to be aware of that, that this should be untrusted by default.
This is whoever the person is is claiming to be as the author, right in terms of owner, the owner is the persons username in the Gallery, right?
So this is their like Microsoft account username.
So there is some level of trust there and then that really is an official account.
But at the same time.
You could hypothetically write create an account under any alias, right?
And as the Gallery Admins, we don't have a process in place to verify that.
Uhyouknowmichael@live.com is really my uh, so I want folks to be aware of that.
But one thing that can be handy is you can click on you know this and you can see all the other packages that are owned by this package owner and kind of just view some of the other stuff that they've published and see their whole portfolio and that sort of thing.
So that can be helpful in distinguishing like, hey, is this Package really, you know, a part of the whole Azure like AZ module that's the whole that sort of thing but can't necessarily be used to confirm identity.
So wanted to call out those few things about the Gallery and then we're seeing if there's.
Umm any questions about that?
I did notice that in the issue on GitHub or the discussion for this community call, there was a question about the gallery all up, so I thought this would be a good time to just answer that question as well.
So let me just pull up that to make sure I'm answering the right question.
OK, so the question asks, are there any future plans for (PowerShell) Gallery other than keeping it as is and two options are proposed, one is migrate to new V3 or move to something else like Azure Container Registry and 2nd is to use some Nuget V3 Shim like the one Justin Grody made with Pwsh Dot Gallery.
OK.
Umm so my short answer is yes and no.
So with everything in (PowerShell, I would say that there's uh, we support a users at the bleeding edge and legacy users and there's so many folks out there on down level versions of (PowerShell get that only Support Nugget V2 endpoints that I don't see us ever removing the Nuget V2 endpoint from the PowerShell gallery.
So I, you know, famous last words, but at least in the short to medium term, we have absolutely no plans to get rid of the Gallery as is and the sense that like you can expect it to be there, you can expect it to be available and you can expect to that Nuget V2 endpoint to be as is with that being said, we are making a lot of investments into the Gallery right now and you can see that in some of the things I just talked about also there's a number of other work items that I.
Haven't mentioned we're investing pretty heavily in areas around Security, trust and availability for the PowerShell gallery itself.
Those are are like top priorities, I'd say like those are.
Maybe higher priorities to us at the moment than necessarily adding a V3 and .2 the Gallery, but if you have a particular need for V3 endpoint added to the Gallery, once again happy to hear about that.
Umm, we are adding an additional endpoint related to ACR.
We have plans for that in the pipeline that you'll see very shortly and that's kind of in response to meeting a more trusted package source.
So we see the PowerShell gallery as a really great place for Community resources, and so you'll continue to see, I think Gallery as is (Plus improvements plus rewrites plus more availability, security and trust.
But ultimately the PowerShell Gallery as is as a community resource, and then we also separately have plans to continue to invest in, Umm Microsoft Artifact registry as a trusted repository for Microsoft owned packages.
And so this kind of alleviates some of the pressure that we have as the gallery and as Package owners have as Microsoft to be verified publishers.
So we'll continue to have that discussion talk more about that as we get there in terms of having that available in previews, start that kind of development work.
The first step towards that is having ACR available as a private repository.
Umm.
So happy to answer any more questions about that sort of as a as a plan.
But in short, that's kind of what we're looking at.
So yes and no.
Umm, also sounds like Justin has something in the works which is awesome.
Love to see the community like work around too.
I think that that's totally encouraged and awesome as well.
So love to see that I have one more item and then I will promise I'll pass it off and you get to hear someone else speak.
The third item that I have is VS code update.
So earlier in March, we did have a preview update to the VS code extension.
A few things I would wanna highlight from that release would be we had two updates to modules that are heavily used in the extension.
And so two of those would be PPS read line and PS Script analyzer on those both saw updates which will be talked about later on and we're included in this release.
We also removed the command Explorer as a visible by default, so if you saw that go away, know that the feature and you really loved it.
Know that the feature is still available.
It's still on by default and Inc) mode, and it is.
You can still turn it on, but we've removed it as default in the UI, so feel free to check out the change log for any other updates and look forward to a stable release coming in the next few weeks. Umm.
Thomas is asking another question back to the Gallery around.
Is there gonna be Support for reserved namespace prefixes like nuga.org to alleviate trust issues?
Umm yes.
Uh, so again, look forward to a blog post on this, like hopefully in the next month we're working on some back end support for this right now.
So there's been some we're working on the engineering side of this to support this in the process right now.
Umm, what I will say about my feeling about namespaces, I think that they are great and what they tell you is that the Package owner is who they say they are in the sense that like the package owner, owner has registered with us and they published umm.
And that's how you get the check mark, right?
So that's what the check mark means.
That's what namespaces mean is that you know somebody has reserved a namespace and that actual publisher is publishing under that namespace.
So in that regard, you can trust the package because you can trust the publisher, which is great where I think that namespaces can fall short.
Umm is that?
I think it can give a false sense of trust in the sense that the packages haven't actually actually undergone any additional verification in terms of like Security pipelines or scanning or things like that.
So that's the second step that we're also working on is improving those kinds of scanning as well.
Umm, keep the questions coming in the chat, but I'll stop talking for now and I will pass things over to Michael with the DSC V3 update.

Michael Greene (POWERSHELL)   26:53
Awesome.
Great job, Sydney, and thank you for for so many details on so many projects.
That was awesome and a couple more questions still coming up in chat.
I'm gonna start by putting a link in chat to the GitHub location for the DSC project.
So we were originally planning on being code complete on DSC V3 by end of March, which would be the end of next week.
I think we're gonna be a couple of weeks offset.
I I would look more to the and the the middle of April at this point, but we are doing really well active PR's, many active issues.
We're trying to be as transparent as we can through the issues list to keep the Community informed of what we're working on and like the the the biggest things that we're trying to solve.
It would say if you're, if you're looking at DSC V3 for the first time, first of all, I would probably wait until we get to code complete to start testing, unless you're interested in like testing along the way as we make progress and providing feedback on changes.
If you're just wanted to do validation and kind of see how it's different, I would hold off a couple more weeks until we hit code complete.
But the way you can think about it is with DSC V3, there's basically an abstraction between DSC and the resources and so the the goal is for complete compatibility with existing resources and open the door to.
Basically resource is written in any language, including native commands and so you might ask like ohh well how how, why?
Why do you think that you can have complete compatibility on a new platform with existing resources, which would be a great question because we can abstract where and how we're executing from.
From what we're executing then I have a PR open right now for the adapter for (PowerShell 7, so that like if if you're authoring a new resource and you wanna do it in either in script or in classes.
Hopefully classes in (PowerShell 7 and you want to take advantage of the new features that should just work.
The next one that we'll be working on is an adapter actually for Windows PowerShell 5.1.
So, like literally, if you have a resource that was written for (PowerShell 5 and you wanna use it with DPB 3, then you would just.
If you do have any problems with seven, you could just choose to use the adapter for 5.1 and run it like where it was written for like I I guess run it in the language where it was originally developed is a better way to say that, so that's how we're hoping to reach.
Very good compatibility time will tell, but that is our goal to have very good compatibility with existing resources.
But then also because of that abstraction, you know we've we've always had this community feedback around the script resource.
Great tool for learning difficult methodology for testing and so we taken that to hearts and the one that will be working on after Windows PowerShell will be an adapter specifically targeted at 1 liners and PS1 script.
So the idea there is, if you're just learning and you wanna just take very simple examples, put them in a, you know, get dot PS1 and test dot PS1.
You should be able to do that as a learning tool and and by componentized down to those PS1 files.
You should still be able to write pester tests and evaluate parameters and do cool stuff there, so we think we can even go deeper than we have before on on previously.
Basically we would copy and paste that one liner into a configuration MOF or to a configuration PS1, which was very difficult to write tests around.
So we we think we're gonna hit like the best of both worlds in terms of having a platform that it's easy to get started, good compatibility, great path moving forward.
And then the really cool thing is through that abstraction.
If somebody wants to write their resources in bash or in Python or in go, or even just build it into their application so that you can literally just have configuration options in your application, you should be able to just have a JSON file that maps to that executable and and just use it as a stand.
So we're really optimistic about V3, tons and tons of new features, concepts of doing assertions before you can but before needing or I I guess to run checks and evaluate your environment before getting into a resource being able to pass data between resources.
Having JSON functions like so many cool features, you can see all about those in the issues list and follow our progress.
But again, I would say mid April at this point is a likely time where we can hit code complete.
Code complete just means, essentially that we we will have met the requirements for our partners in machine configuration and winget to start doing their validation.
But once we consider it to be code complete, anything that we do from that point forward would that that makes a major change would be considered a breaking change.
So we might, you know, take a request from machine config, but we have to be cognizant of the fact that if we make changes, we might break winget.
So we're working very closely with partners and optimistic about where we're headed.
So I put the link to the repo in the chat again.
You're welcome to go out.
It's actually a very easy platform to build and play with today, but I was seeing a few more weeks.
We'll be ready for some fun validation and maybe we'll do a public bug bash at that point.
So that's it.
Thank you very much.
I will hand it back.
Let me go back and see who I'm handing it to.
Unless somebody can tell me right.

Sydney Smith (SHE/HER)   32:56
I think it's Jim with the PSA release.

Michael Greene (POWERSHELL)   32:58
Great.
Awesome.
Thank you very much.

Sydney Smith (SHE/HER)   33:03
Thanks.

Jim Truher   33:09
I thought it was going to be a Jason.
But hello, can you hear me?

Sydney Smith (SHE/HER)   33:19
Yep, we got you.

Sean Wheeler   33:19
Yep.

Jim Truher   33:20
Ohh, good.
Are we just did a release of script Analyzer 1.22 it was about so a couple of weeks ago now starting to see some uptake on it.
We've given it off to the editor services folks.
Make sure that that we don't we don't get in the way of any of their scenarios.
We haven't done anything to break them, so give it a.
Give it a take.
Take a look and let us know.
I should shut up.

Sydney Smith (SHE/HER)   33:48
Awesome.

Jim Truher   33:49
I should shout out to Chris Bergmeister, who was instrumental in getting the in in finishing the release.

Sydney Smith (SHE/HER)   33:51
Mm-hmm.

Jim Truher   33:55
So thanks Chris.
So in a whole bunch of friends.

Sydney Smith (SHE/HER)   34:02
Often.
Thanks Jen.
Yeah.
And thanks, Chris.
Definitely.
I'll also just mentioned that this release was really community driven.
We took in a lot of community PR, so thanks to anyone who opened issues or opened PR's and kind of drove this release for us.
Uh, Next up, we have Steven with a PS read line release.

Steven Bucher   34:29
Everyone just wanted to share.
We did a beta 0 release for PS3 line 2.4 dot O.
There is not a whole lot of new features per se.
There's a lot of just bug fixes and kind of polishing up some experiences.
I will share the change log and keys.
You're curious, but yeah, that's that's kind of it.
Nothing super super wild here, but we're we're still looking at other features and improvements, so if you were interested in in sharing ways that you think PS3 line can improve, please of course submit an issue and we'll take a look at it.
So that's kind of all I got to share on the psreadline front.

Sydney Smith (SHE/HER)   35:16
Thank you, Steven.
And then we've got Sean with our docks update.

Sean Wheeler   35:20
Yeah.
So the.
Not a lot of new dogs in the last couple of months, but umm, a lot of incremental releases in the in the (PowerShell space over the last couple of months.
So putting the links in the chat there to updated documentation, of course we've updated the documentation for (PowerShell 75 previewing theses.
There's been a ton of work going on in desired state configuration as Michael said, and.
I'll I'll toss this over to Mikey in a minute to to kind of hit the highlights there, there's.
Uh, a lot of documentation work.
He's done there and then, of course, as Jim just mentioned, the release of script analyzer, there was a couple of new rules.
Umm.
That were added and so the that documentation has been added and updated for the new release.
Mikey, you wanna give us a quick rundown on the the DSC docs?
Mikey, are you there?
He was earlier.
Looks like he maybe had to drop.
Umm.

Sydney Smith (SHE/HER)   36:44
Yeah.

Sean Wheeler   36:44
I all I can say is, well, let me let me just quickly share my screen.
If you go out to the change log, umm documentation, you'll see all the changes that came in.
What was added?
Uh, lots of good information here.
And then the associated documentation for all of this was updated as well.
So a lot of good work.
In the dark space here from Mikey and.
That's my docks update.

Sydney Smith (SHE/HER)   37:26
Awesome.
Thank you so much, Sean.
Well, with that, that concludes the core of our agenda.
Before we jump into questions, Jordan, did you wanna do your demo?

Jordan   37:38
Yeah, sure.

Sydney Smith (SHE/HER)   37:40
Awesome. Thanks.

Jordan   37:41
Thank. Ah.
Hey everyone I'm my name is Jordan.
You may have seen me around the place, but I'm here to demo.
Just sort of a quick project that I've been working on which is remote forge.
Uh, now is it possible to share my screen?
It just says only meeting organizers because it it.

Sydney Smith (SHE/HER)   37:59
Ohh yeah, let me give you a presenter rights once second.

Jordan   38:04
Thank you.

Sydney Smith (SHE/HER)   38:05
OK I found you.

Jordan   38:10
There we are.

Sydney Smith (SHE/HER)   38:12
But.

Jordan   38:14
OK.
Let me know if that's coming through.

Sydney Smith (SHE/HER)   38:21
Yep, looks good.

Jordan   38:22
Yeah.
Awesome.
OK, so essentially remote Forge is a a project that I've been working on.
It's sort of a a provider for remote plugins that you can use with PowerShell remoting, so built-in you've got a whole bunch of ones like win, Aram, SSH, Docker, Hyper-V and sort of process remoting ones in 7.3.
The Remoting subsystem was sort of opened up for third party APIs at the time.
It's sort of just like a exposing the existing API that's out there and I don't think it's sort of went further enough to really make it useful for people to either both write their own remoting plugins or even consume them themselves.
Similar issues that I've sort of come across is you need to reinvent the whole wheel like you have to implement your own commandlets to create the actual session.
You can't really easily use it with invoke command, especially when you're going with parallel because you have to create the sessions manually and then sort of combine that with invoke command and even just sort of generating or creating your actual plugins and themselves.
The API is a bit difficult to actually use, especially if you're not sort of used to how it actually works and dealing with things like error errors and sort of problematic scenarios can be quite difficult to actually implement.
So that's why I've been working on this thing called remote porch, and essentially it provides both a.
What I would call maybe an easy API for people to use.
So essentially they just need to provide a method to create the connection, a method to read the data, and a method to write the data, and that's about it as well as that, it also provides a few different commandlets which is designed to be extensible with third party connections, which in this case I'm calling forges.
So instead of remote command we have invoke remote.
Sorry, the one that I won't get help.
So in this case here, like there's a few different options, you've just got a connection info which takes an array of either strings or peer sessions or connection info objects.
So you can run that in parallel.
You can either specify the script as a file like you can do with invoke command or as a string with the script block some extra features that I've added on there.
So instead of argument list only taking an array of arguments, you can actually pass the hash table or any sort of dictionary like object and sort of splat that into there.
So you don't have to rely on positional arguments and another command that we have is in invoke remote Coward session.
So this is sort of like a parallel to new PPS session.
The same thing, it's just sort of an overlay of that other sort of plugins to use to easily create peer sessions from their implementations and then sort of the last main one that's out there is enter remote.
So this is like Entropia session, but in this case here we've just got that support for the forge side with the forges.
Right now I've got the two that's provided built in, so there's SSH and WS man, so that's just sort of the built in ones that are provided by (PowerShell.
But in this case here I've been playing around with a few different sort of Alternate SSH options.
Is sort of the demo, so HVC is a custom one that's designed for Hyper V Connect.
So you can connect and run things on a Hyper V VM or without any networking setup which is quite useful.
SSHV 2 is sort of just the SSH built in stuff with a few extra features like password Support that you don't currently have.
It's SSH.
Net is sort of one that isn't working right now, but it's designed to be using the net assembly SSH .net.
So instead of spawning a new process that's using a built-in library which opens up a few new features that isn't currently available, so using them is quite simple in this case.
Here I wanna enter remote.
And face I've just got a Linux VM under this IP address.
I'm currently not actually.
I don't trust that the host key, so I do need to supply a credential and there.
Sorry, an option to sort of skip that and then I wanted to provide explicit password credentials.
So in this case here it's just going to prompt me from this one here.
So in this case, it's just a very simple password.
Now I've sort of entered that session.
If I was to use so let's go into PSTN, it's name.
After this form here will actually ask me prompted trust the key and it'll prompt me the password in there which actually have to do physically through the terminal, but I can actually do a get credential.
I can talk.
I can create a credential object so I can get some start from anything that I want.
Like Secret store or anything and I can actually provide that explicitly and actually connect to that without any prompts from SSH.
That's sort of one of the extra features that I've been playing around with now in this case here, I've connected to a remote VM and with PS Remoting it doesn't actually have a console for applications to prompt back on you.
So if I was to do pseudo, it's going to fail because there's no terminal to actually prompt me for the password.
In this case here I've got a new forge in this section here. Sorry.
Import module first.
I'm in this case here.
It's added a new one and what I can do is I can use invoke for emote.
I specify the forge ID.
In this case he is Sudo doesn't have any extra options, but I could do like first name if I was connecting to a specific host, but I just don't have anything.
Who am I?
Just for a quick test, it's Katie is actually going to put me for the sudo password, even though there's no Console that's using PowerShell Remoting's post sort of prompting to get that working there.
So I'll just type that one in and it works ohh clear the cache and once again we can also use a credential for it.
Been prompt me for that one there and I can go.
New Sudo.
We caught.
Prudential.
And they can do Sudo without actually prompting me for it.
So it has a whole bunch of abilities to do various things.
One thing that I'm not 100% sort of happy on is the fact that you actually have to provide this new pseudo forge or new such and such forge info object actually create it for every connection that you were using.
I'm wondering whether it's a better idea to pass in options parameter in this command that gets passed to those forges to be able to create a similar to what sort of secret secret management does, but there's a few things I'm sort of playing around with to trying to improve the UX around that, but like in this simple case here, the idea is that you should be able to specify just sort of a forward ID that you're using.
So in this case it's Sudo and then sort of extra options that you were on top of there.
So it's like it was SSH.
I could do SSH for its name or SSH.
Umm, it hurts?
Name in in such and such.
But for very sort of more advanced scenarios where it doesn't cover the common options, that's what these commandlets that you'll module would offer.
And that's sort of just a read only object.
So in this case here, if I was to output that itself.
So it's very simple object that didn't always have to create the connection from there.
The last option, so if I will exit out of that session, there's another one which we have the HSV HVC one.
Would infer I wanna go to when VM that's the name of my virtual environment and I was going to skip those key check.
Uh, I believe this is actually gonna fail cause I've got a bug on it.
Maybe not.
That is why.
All right, wrong username and then debugging doesn't help.
Yeah.
And so it's connected to an actual Windows VM that I have here and what you can do as well if you wanting to go in parallel I can do.
Thank you.
Connection too.
I also see I've got eight through there, so I need to fix that.
This.
I can't remember the IP address.
Like the connection fix up this typo and then if I wanting to run it in parallel I can just provide those connection options as an argument and then it should run that.
Alright, once again I need to fix that.
As you can see, it's still sort of beta and I'm working for the kinks flower met.
Just provide the password explicitly.
In this case here it's run it on both Al Hyper V VM which is connection one.
You can see the app port here and then also the Linux VM which is just the VM and it's called local host.
If we were to capture that output like you can within Burke commands, you can see that he is computer name is an option there.
So you can see that's from SSH info that's dependent on the actual object that's actually created.
Umm, I don't think I've got a walk.
Human friendly name sort of set up for that one there, but there's various options.
That you get from that.
So essentially like it's designed to be able to help people write their own connection plugins.
So I've got a few ideas and I'm wanting to work on like a Python one that you can execute Python scripts through invoke remote, but it's also designed to sort of extend some of the features that may be missing in the built in ones.
Like with SSH you have password Support or an easy way of supplying extra options that people might not know about.
And yeah, but that's essentially it.
I'm sorry.
I was sort of scrambling with the doing the demos here.
So.
Umm hopefully that can help.

Sydney Smith (SHE/HER)   49:33
Awesome.
Thank you so much, Jordan.
That was a really great demo.
Super appreciate it.
Feel free to follow up with Jordan in the chat if you have any comments or questions on the demo.
Umm, very cool.
From that we can open it up the floor to questions.
I'm gonna start with I'm thought 2 questions.
In the GitHub issue, but feel free to continue to add them to the chat.
Use the raise hand feature or at the GitHub discussion and so the first question that I thought we would go for is what is the time frame of cloud apps V2?
Well, the current version gets Support of components.
Jason, are you the right person for this question?

Jason Helmick   50:29
Hey I think I am.
So plenty PS V2, so let let's talk about this for a second.
We're currently doing work on Platy PS I apologize that for those of you that have been waiting it, it seems like it's taken a while, but here's the deal.
We're working with an internal partner.
The internal partner happens to be the internal partner that makes all of the docs that Microsoft and this is in coordination with Sean Wheeler, who has been a great contributor to this project.
So Planning PS right now as we are building it for an internal partner, we wanna make sure we can dog food it.
But we've rearchitected it.
Thanks to Jim Truher and DTF, they've helped us rearchitect and I mean it's it's really different than it was before.
Once we get that through our internal partner and it seems to improve the docs overall at Microsoft, then we're gonna reevaluate what we wanna ship as a public release and that will be released.
It may be released as I don't know if you noticed this or not.
Apply the PS isn't actually releases a version .1, so we may release this as the actual version one of Platy PS in the upcoming months, but it's gonna take us a little bit so towards the end of the year, I think I'll have more information towards a public release.
But what you're gonna find in this as we release it is the conversions to mammal, the conversions to YAML are much faster.
They're highly validated.
You're gonna find the architecture makes more sense that the commandlets make more sense, and I ioffer any moment to Jim Truher, whose happens to be here tonight to also speak up about this.
But I'm actually very excited about what we're gonna do.
So, hang on, we're almost there and I'm really excited about the release.
So thanks Sydney, thanks for the opportunity.

Sydney Smith (SHE/HER)   52:37
Yeah, absolutely.
Thanks for the answer Jason.
The second question I saw came from Thomas.
It's about semantic versioning.
I read into the issue a little bit.
Happy to give a response or, but did you wanna ask a specific question?

Thomas Nieto   52:50
Yes.
And I'm here if I can.

Sydney Smith (SHE/HER)   52:52
Yeah.
Do you want to talk a little bit about it?

Thomas Nieto   52:52
Yeah.
Yeah, I'll talk about it.

Sydney Smith (SHE/HER)   52:54
Yeah, that would be great. Awesome.

Thomas Nieto   52:54
So alright, so the background of the semantic versioning is issue.
That's LinkedIn.
The chat is the adding support for semantic version in the engine due to (PowerShell Git.
Since (PowerShell Git Support semantic versioning, so here's some of the things about it.
So the working group decision didn't have any context address any of the issues or consider alternate solutions from the verdict that was posted there.
As such, it indicates that semantic versioning is not supported in (PowerShell and by extension power source resource git and should be removed.
The scope of the work and the issue is allowing semantic versioning in the module COMMANDLETS, which could be accomplished by adding a 65 KB dependency on Nuget.
Versioning.
Here are some of the issues when PS Resource gets PS resource.
Get Support semantic versioning while the engine does not #1 prerelease binary modules cannot be updated to stable.
The largest impact of this is psreadline, as users who install prerelease versions will have to undergo workarounds to install the stable version, thus hurting testing.
Adoption 2 prerelease and stable versions cannot be installed at the same time.
3 The formatter displays a fake prerelease property, causing user confusion.
(PowerShell should either fully support semantic versioning in the engine, or remove it completely and in PS resource git my concern about working group verdicts and the lack of transparency could be addressed by working group meetings being recorded, having meeting minutes in a publicly accessible vote with a company and concurring and dissenting opinions.
This would help users know their issue has been earnestly considered, so I want (PowerShell committee to review the future of semantic versioning as (PowerShell for PS Resource.
Git should not support a versioning scheme that (PowerShell cannot.
You know, open up there.

Sydney Smith (SHE/HER)   54:39
Yeah.
Great.
Thank you, Thomas.
And I'm happy to if anyone else wants to chime in.
Happy to take other comments on that topic right now as well, but Dev yeah.

James Brundage   54:51
Yeah, just being really brief there, there's a lot in that area that could be desired.

Sydney Smith (SHE/HER)   54:54
Sure.

James Brundage   54:57
My most potent and painful memory is that it's not really possible to have a command or module properly with a semantic version name.
You have to keep alternating the name because the version field is locked to being a proper numeric version, and so it creates a lot of confusion.

Sydney Smith (SHE/HER)   55:08
Mm-hmm.
Mm-hmm.

James Brundage   55:19
It's something that almost every organization that I've worked with in the past seven or eight years has wanted to adopt Sember and has essentially used (POWERSHELL's failure in this area as a crucial against (PowerShell adoption internally.
So this is actually significantly damaging in my opinion, as a persistent thing, and it could be fixed very, very easily by essentially having either the version string or version be capable of being a a full string or the version be keeps augmented by a version and a string variant.

Sydney Smith (SHE/HER)   55:40
Mm-hmm.

James Brundage   56:01
So it's just a a little big thing, but it is something that I would estimate probably costs (PowerShell, I don't know 5 or 10 companies a year in terms of internal beachheads and battles.

Sydney Smith (SHE/HER)   56:02
Mm-hmm.
OK.

James Brundage   56:21
So sorry, but I do feel strongly as well.

Sydney Smith (SHE/HER)   56:22
Yeah, no.

James Brundage   56:24
Thank you, Thomas, for bringing this up.

Sydney Smith (SHE/HER)   56:26
Totally, totally appreciate the sentiment.
I am not going to respond on behalf of the engine working group as I am not part of it or on behalf of the team at the moment, but more than happy to kind of bring this back to the team and maybe we can talk about it again next call Umm and just kind of think about what the next step could be.
I know that there is this existing issue on semantic versioning.
Let me just pull it up and take a look at it is the issue is closed.
I'm wondering if we think it would be helpful.
There's good context in that issue.
Umm.
Or if it would be helpful if we started a fresh Discussion or issue on this topic.

Thomas Nieto   57:18
I would say leave the existing one open because it has about 100 comments that are very useful in the ultimate.

Sydney Smith (SHE/HER)   57:21
OK.
Good, good context that we don't wanna lose.

Thomas Nieto   57:26
Yeah, exactly.

Sydney Smith (SHE/HER)   57:28
OK.
OK, fair. Fair enough.
Umm.
Again, I yeah, not part of the engine working group so don't want to speak on their behalf, but happy to collect the feedback and appreciate you speaking up.
So thank you for that.
I did also see a question on in the chat around Speaking of working group schedule.
Is there a general schedule so working groups?
Yeah, typically scheduled themselves.
So I would say Cadence is different, I would say for all of the working groups, I'm a part of which is only a couple and we've meet on my on Mondays and we try and go through all the issues on Mondays, but kind of varies cause that's Community today for us.
We try and Triage on Mondays.
I thought, Jim, come off mute.
Did you want to say something, Jim?

Jim Truher   58:16
The engine working group meets every two weeks and we've discussed and we've discussed, uh, that that issue at length and they're all part of the working group notes, which are part of the discord.

Sydney Smith (SHE/HER)   58:19
OK, great.
OK. Umm.

James Brundage   58:36
He said there is a discord.

Jim Truher   58:40
Yes, I did.

James Brundage   58:42
Ah, I think circulating that link would be helpful.
Thank you.

Jordan   58:48
Is there like a known backlog like the working groups up to what they think is sort of where the timelines at or do they have sort of a large backlog that they need to work through?
It's just seems a bit opaque right now where I'm sure they've got a lot of work going on.
I don't want to see needy, but gets a bit frustrating when you sort of putting in some work and either you get silence or you get maybe one comment and then sort of nothing else beyond that.

Jim Truher   59:17
Different work groups have different backlogs.
The engine working group has the and not insignificant backlog.
We spend a lot of time discussing the issues during during, during the time that we meet, and then we we capture all the notes as part of the discord in the working group notes, and those are available on the discord.
I don't know about whether it's public or not.
I assume it is.
There are on the working on the engine working group.
Uh, it's half and half community members in Microsoft employees.

Sydney Smith (SHE/HER)   1:00:01
Yeah.

Thomas Nieto   1:00:01
I think it would be helpful if working group would publish like agendas like here's the issues that we're going to work through this meeting so that people can see in the next week or two or five.

Sydney Smith (SHE/HER)   1:00:01
The other thing I'm sorry guys.

Thomas Nieto   1:00:14
Hey, Jordan's issue is coming up in three weeks.

Jim Truher   1:00:14
Who?

Thomas Nieto   1:00:17
So he has an expectation of when that will be released.

Sydney Smith (SHE/HER)   1:00:24
Yeah, I think that's yeah, but for Jason.

Jason Helmick   1:00:24
If I could interject.
Umm I I just want to follow up on with a.
A Jim said that a lot of the working groups meet on a very timely schedule, but they also have very extensive backlogs.
This is the due to the size of mount of issues that we have and and directly to answer the the the question that just came in, I think it was from Tomas or whatever about producing a list of what issues are next.

Sydney Smith (SHE/HER)   1:00:38
Umm.

Jason Helmick   1:00:50
I actually produced that list and and many of the working groups do produce that list to the working group, but only with a couple of days of a of a of advance in other words.
E we're not at a point where we can do that like weeks out it it.
It comes down to taking things off the backlog and moving them in, and some of those discussions, just as an example, in the working group interactive one that we just had, we had an issue that took the entire time.
That was allotted to us that we're still not done with and we have to come back to a lot of these issues at this point are not just is it an issue that impacts you, but is it an impact isn't an issue that impacts the 2 billion users a month that we have and how does that impact it on that scale.
So it takes us some time to work through those issues.

Sydney Smith (SHE/HER)   1:01:46
Mm-hmm.

Jason Helmick   1:01:47
So we don't necessarily have that kind of detailed timeline to provide, although we are working to try to provide more details in that kind of thing as we go forward.
So thank you for letting me interject.

Sydney Smith (SHE/HER)   1:02:02
Yeah, I think one other thing I'll just say and then I'll, I'll, I'll let other folks speak as well if you're ever curious to see like the, if you wanna see the query that any working group is using for the most part to see like what are the total sum of issues that they're looking at and what they're pulling from, if you just go to the (PowerShell issues and use the tags needs Triage and then the working group label, that's what they're picking from.

Jordan   1:02:02
Yeah.

Sydney Smith (SHE/HER)   1:02:27
So that could be helpful, but I know I think Jordan you were about to say something before I said that.

Jordan   1:02:33
Yeah, like, I don't wanna sound ungrateful or anything like that.

Sydney Smith (SHE/HER)   1:02:37
Sure.

Jordan   1:02:37
This is sort of just like a suggestion, that is it possible that potentially that there could be like a lot of times carved out when you're meeting.
So maybe like 10-15 minutes to go through like new issues that have popped up, that might be a bit quicker to sort of go through rather than spending the entire time on some sort of older issues that might need to be going through or even focusing on PR's that people might have sort of put in umm, so to try and clear out some some of the stuff that's sort of appeared there?

Jim Truher   1:03:06
The actual fact is that that time that that efficiency of time does not exist because you will get into the middle of something.
Ohh this is easy and we'll talk about it for 45 minutes.
So so it it doesn't that that that is a that that is not.

Sydney Smith (SHE/HER)   1:03:20
Yeah.

Jim Truher   1:03:24
That's not how.
That's not how it works out.

Sydney Smith (SHE/HER)   1:03:27
He could think we sometimes say is all the easy problems are solved, but.
Yeah, I think I think uh, to to if it's any consolation, we do put a lot of thought and into every single issue, even if it feels like.
And so sometimes I think we could do better job of communicating that, but if anything, I think we we oftentimes could spend less, could have more efficiency.
But yeah, usually there's a lot of time spent into issues.
I'm Thomas.

Thomas Nieto   1:04:00
I think I.

Sydney Smith (SHE/HER)   1:04:00
I think you're trying to say something.

Thomas Nieto   1:04:01
Yeah, I think the main thing right is for the Community side.
If we can pull back the curtain and see the transparency of it and see Ohh Jim's meeting for the engine working Group spent 45 minutes on this one thing or they work through these five issues.
The more transparency that we can see from our side will alleviate those concerns of like this is just a black box that we have no input or say into.
We can Triage it, you know, say, tag it for the working group to look at and then may or may not get a response and ever.

Jim Truher   1:04:32
We can certainly bring to the committee whether or not we can release the the Working Group notes.

James Brundage   1:04:32
Yeah.

Jim Truher   1:04:37
I don't see that there's a problem with that.
There's nothing in there that is that is is.
Something that needs protection so I don't know that that I expect it and you'll see we keep.
We don't keep every word that is said and we're not going to record them.
That's that's a that's quite problematic actually in the industry and I, but uh, we do keep a a kind of a somewhat flow of consciousness as we discussed these issues and I don't know that there is any reason that we can't a release that although I'm not a lawyer and I don't speak I'm I'm not going to to pretend to be one so there may be other issues at at at at play here that I don't that I can't speak to but we can certainly bring it to the committee to find out if there is a.
Way we could very easily release the last if it's.
If it's allowed, it released the last 18 months of notes.
I don't that's doesn't seem to be a problem or release them every week.
We'll have to follow up.

Sydney Smith (SHE/HER)   1:05:44
Umm, we're just going forward maybe.

Jason Helmick   1:05:45
And and yeah, let me just let me just jump in real quick and say that, Thomas, I really appreciate this and we'll definitely work towards it.

James Brundage   1:05:46
Yeah, I I think that that would be helpful.

Jason Helmick   1:05:53
However, there are some legal requirements and some legal restrictions about the conversations that we have.
That's why we don't record them.
There are some intellectual property discussions that occur because of the nature of some of these issues.
These aren't.
If these were easy issues, they would have been handled by the Community much sooner than they reached.
One of the working groups, and so sometimes I'm not saying all the time I'm saying sometimes it is challenging us.
Uh, for us to be able to release, like recordings, as Jim pointed out, but I do appreciate what you're saying.
I really do, and I'd like to get more transparency into it, so maybe that's a conversation that we can keep going and seeing what we can do to make sure that we are providing you with the transparency that you would like to see.
I I I mean I'd love it to have you know, your issue comes up on Tuesday, you know, March 23rd or whatever that is.
I'd love to have that kind of scale, but right now we just don't because at it like Jim had pointed out, sometimes an issue that seems like it's gonna be easy takes us days to work through.
So we'll try to work at that, but it's a little bit challenging.

Jim Truher   1:07:10
Uh.
And there is something that that could be done with the with the data that we have today, because the all of this activity that happens in the GitHub repository against issues is is queryable, we could go through and provide statistics on on backlogs and a in and how long issues are are in need Triage and we can provide, we can look into into collecting those statistics if if you think that that would make you feel better, I can tell you that I can tell you that the from the working group that worked the engine working group.

James Brundage   1:07:10
OK.

Sydney Smith (SHE/HER)   1:07:10
Yeah.

Jim Truher   1:07:50
Perspective.
It is getting smaller.
Uh, the backlog and I know it is getting for some of the other working groups as well.
So.
So.
So it may be it may be opaque to you, but it I we can definitely see that the list is getting shorter as time goes on because we are spending more and more time doing this and.
Because we know that it's important.

James Brundage   1:08:17
If I may, for a second, I'm a restatement of the problem from the Community perspective and then a corollary and a suggestion.
The problem ends up being that squeaky wheels are supposed to get grease, and we don't know where to squeak like.
Aside from just filing an issue and hoping it's, it's hard to get traction, we don't really kind of know how to engage.
OK.
The corollary here is is drill as it might be a condo board.
I'm on my condo board.
We have to meet every month.
We make decisions that impact everybody and we produce minutes and every stakeholder in the Community gets those minutes because there are a stakeholder in the Community.
I think Jim suggestion of releasing those notes or minutes would go a really long way to the transparency concern, but I can say even on the scale of 50 people that people can get really angry if they don't feel like they have a say in the decision making body that rules over them.
So that I think is part of what's going on.
I think your suggestion is really great as far as releasing minutes.
I think the other thing that a condo board traditionally does that would be great to do for the working groups is essentially allow people to come to a given Forum, especially if they're issue is coming to the floor.
So if you know you're gonna be discussing an issue that I've been working on, you don't need to invite me into the whole working group to invite me to the working group for the five minutes to talk about the issue that's normally front loaded so that you get it out of the way so that you can get the person out and proceed with private business.
But I think the corollary works really well, and I think adopting the practices that you would adopt if you wanted your condo board to not suck would be a great experience here to sense over.

Sydney Smith (SHE/HER)   1:10:07
Great.
Well, we're a little bit overtime, so I think we'll start to wrap things up.
Do you appreciate the feedback from everyone?
And I am happy to kind of collect this feedback and bring it to the committee.
I'm not personally committee member, but I'm happy to kind of bring this to the committee and we can kind of move forward from there as we wrap things up.
If you have continued thoughts or feelings on this, or if you're watching the recording and have continued thoughts on and feelings on this umm Steven, maybe we can leave the Discussion open for this community.
Call for a little bit so folks have a place to voice that and then ohm.
What kind of move forward from there, but thanks for being here today.
Hopefully our little experiment of expanding to a new time zone was successful.
Thanks for all the folks who don't usually get to join who are here.
Today we released do appreciate you being here.
And we'll see you, if not next month.
We'll see you in September, but yeah, thanks again everyone, and thanks for bringing up this discussion topic as well.

Lionetti, Chris   1:11:19
And how many of you are going to be at the summit next next month?

Sydney Smith (SHE/HER)   1:11:22
Oh yeah, you're submit it.
I'll be there.

Lionetti, Chris   1:11:28
Sweet.
Thank you.

Jason Helmick stopped transcription
