PowerShellOpenSSH Community Call-20260219_092934-Meeting Recording
February 19, 2026
50m 28s

Jason Helmick started transcription

Jason Helmick   0:03
Now the meeting recording has begun.
If you don't want to be recorded, you have to leave. Other than that, you're welcome to stay.
Hey, Justin, I see a bunch of folks are just joining up, so let's give it a minute.
Well, it it does look like we have quorum building rather quickly. So we've got a pretty action-packed agenda. So let's go ahead and get started.
Well, folks, hello. Good morning. Good afternoon. Good evening. Welcome to the it's February, right? Yeah. Welcome to the February 2026 PowerShell Community Call. Everybody is welcome to join and discuss the PowerShell project. We have our agenda posted in the meeting chat and a link to the meeting discussion. Please feel free to.
Post questions in the discussion issue or right here in the meeting chat and be sure to check out our code of conduct for the agenda today. We've like I said, we've got a pretty full agenda. So let's go ahead and dive in with our our first topic. The first topic is is is mine. First of all, I just want to say happy 20 years.
Years of PowerShell. This is the 20th year of PowerShell being, well, pretty much in service since its release back in November of 2020 or 2026 back in November of 2006. So happy 20 years of PowerShell and let me start off with a quick update on PowerShell 7.6.
First, I know the delay has been frustrating for some of you. I hear that. So the LTS releases matter quite a bit and people do plan around them.
Now look, nothing is missed and nothing is broken. This release is taking the time that it needs because of compliance work, infrastructure updates in our release pipelines and some very careful backporting selections. And as many of you know, for LTS, the quality bar is intentionally higher since this is our long term support release.
And we're holding to our principle. We ship on quality and not on dates where we are right now. The team is planning to publish the release candidate, the RC very shortly, maybe today, maybe tomorrow, maybe early next week is what we're looking at. We are in those final stretches right now.
After that we expect about a 30 day evaluation period before we go to GA, so about a month after the RC releases and this will give the community and the ecosystem time to to review that final build which is very important to us.
That's what the current plan is.
As always, we really, really appreciate the folks who test the previews and file issues and help us make the LTS release solid. This release is always very important to us, but before I.
I move on with this. I do want to acknowledge something that the community has raised about and this is an opportunity where the community can help us. So the community has raised some concerns around Lifecycle alignment with.net releases. This is a valid concern, especially for vendors and enterprises.
And their planning scenarios moving forward. The team understands that and it's part of our ongoing internal conversations to work through and and to better understand what the the future life cycles are going to be. For today though, the important update is where we are with the 76 LTS.
And on that path to RC and GA, what I'd like to ask is folks is there's a great discussion issue in the PowerShell repo started by Jay Borian and that's a great discussion issue to go into right now and tell us our ID and give us our your ideas on how we.
Can better communicate to you. We usually do our communications through like here in the community call, but there might be better ways, especially for those enterprises around their release planning that we might be able to do. So please feel free to jump into that issue.
And talk to us about some ideas and we'll see what improvements we can make. With that, did I mention that we have an action-packed agenda? So with that, let me turn it over to Steve, who'll talk about the planning blog post. Steve, good morning.

Steve Lee (POWERSHELL)   6:02
Hold on. All right, let me share my screen here.
This one.
All right. So for those of you who have not had a chance to.
Read this. This was published 17 days ago. All right. So basically, you know, every year I like to have a blog post about what we plan to do for the calendar year. And I will admit, you know, not everything gets done for various reasons. All right.
But let me kind of go through this and we can read this in detail themselves. First off, you know, I do want to continue to thank, you know, we do have a very active community and I do really appreciate all the work that people do on their own time. So please keep that up. And this is across all of our repos, not just the partial repo.
One of the things I do wanna call out and so there's a bunch of work that we have to do at Microsoft that is not publicly visible and it's sometimes it has to just do with security, not security in terms of like code changes, but just on how we secure our own supply chain.
Make sure that the way that we do releases meets new and ever changing compliance requirements. So we always have to do that. That takes priority and that takes priority over some of the feature work that we want to do. And not everything from a security perspective we know ahead of time. In fact, mostly we don't know ahead of time.
We do want to continue to kind of prioritize getting some of the bug fixes and community PRS into the product. Hopefully people have seen maybe a little bit more activity on that on that front over the last year. We'll continue to try to focus on some of that with the resources that we have. So specifically around PowerShell 7.7. So again, we're looking forward with the assumptions.
That work on the GA 76 soon when I say soon like this, hopefully this quarter PS user content password location. So this is something that was in the 2025 blog post and there's different reasons why we couldn't get it done. One thing I'll just also mention is like.
In terms of how we do feature work, especially for an LTS, right, 76 and LTS at a certain point and that that point is usually around, let's say around June time frame becomes more and more risky to make big changes into the product. So if we missed that window before then.
And then naturally those kind of changes will flow into the next release for us. So just keep that in mind. But to summarize, you know, this is where people have raised this concern a while back where by default on Windows, you know your your My Documents folder gets put on OneDrive and that can have a perfect impact for a lot of folks whether they know it or not.
So this is being having the ability to really move your content, the content meaning like your profile, where modules get installed, where update will help gets downloaded, things like that.
The next one here is non profile based module loading. So this has come from a lot of our internal partners where they don't necessarily ship a PowerShell module, but they ship something that a lot of PowerShell users use and they want to be able to expose like tab completers, feedback providers and things like that.
So the question is how do they do that and be able to have it installed and not muck around with somebody's profile? Because now when you do an uninstall, it becomes more complicated to kind of clean it back up. So there is an RFC on this already. So again, take an opportunity to provide that feedback.
Another um.
Hey, keep using more feedback. Another thing that users have raised to us is that you know we have the whenever we do a release, we push the update information to a basically an Azure Storage BLOB and then PowerShell notices that the version you're running is different than the version that's been pushed in terms of notification.
And the problem is, there's always a delay between us publishing on GitHub and everywhere else. You know, this includes stuff like packages, microsoft.com, snap packages, the store and things like that. It just even though we may publish it to those things, it takes time for them to do some of their own validation and then get them make it available to.
The public. So we're looking at doing some, maybe some delay here in terms of the notification. That way by the time you see the notification that you should be able to update.
Bashed out aliases. This is something that actually was brought up. I want to say when it was PowerShell Core 6, like that time frame like very long time ago. So we do have some revised interest into this from our side. So this is something we're gonna look at to see if we can.
Support such a thing as basically the idea of bash aliases and macros. So in the PowerShell world, an alias is really just another way to shorthand a commandlet, right? In the bash case, it's actually much more.
Rich in terms of features. So for example on my MacBook I might say my my alias for LS in in a bash case, not in a partial case. I might actually have a bunch of switches so I can get like you know, color output, wide format, stuff like that.
In the PowerShell case, what we've always told people say, if you want that, you write your own function and that's equivalent, but it's not beginner user friendly, let's say, or not. People come from Bash. We'll find that kind of odd. So anyway, so that's something we're looking into.
Obviously AI is still a big topic in the industry, so we do want to start some work publicly on producing a MCP server for PowerShell 7. And I think right now what we're focusing on is what I say here, like safety and security of using AI with PowerShell.
All right, moving ahead out of PowerShell 7 itself. ES read line.
One of the things that I personally get impacted a lot is, you know, I I love the predictive Intellisense. However, I work in different git repos and it'd be great if it knew that I was in a partial repo versus the open SH versus DSC and it had historical predictions based on what I was doing in those directories. So this is something we're looking into.
The second one probably won't affect users directly much initially. There's more of a platform change. So basically the way P3 line works today is there's a loop and basically there's like a it watches for keyboard input and there's a timeout and then it renders and then it just does that loop.
And basically that prevents us from doing some interesting stuff where we may want to say, hey, the user's idle, we can maybe pop up a tool tip, things like that. So there's other things that we could do there if we can update this fundamental changing this loop in PS3 line.
So this is a hard change because it's just the current loop is how PS realign fundamentally works on the partial gallery piece resource guest side. You know we started this MERACR and in general something called ORAS which I forget what the.
Abbreviation stands for like open something, artifact something. But anyways, it's that's the open way of talking to any type of container or now call artifact registry. So this should work across like Google Cloud, AWS and of course Azure.
But the idea here is that we want to make sure that we at least first party start publishing our stuff to MAR. So the marks of artifact registry would be trusted by default. So this is if you want to install the AZ module, the office module, stuff like that, you don't need to worry about whether or not someone had published something similar, made a typo deliberate on a gallery and stuff like that. So we're going to.
Work with our partners internally to make sure we can get all that stuff out there. I do see a bunch of questions popping on the chat, but I'll address those after I get to finish through this concurrency and performance improvement. So you know, module fast is great and that's something that we've had intending intent of doing for a long time.
I think now that we're freeing up a little bit of bandwidth on some of the other stuff that we've been doing on the gallery, I think we want to get back to that so that we can improve the performance of using PS resource, get for downloading something like AZ module or even the AWS module is also huge. Also just general improvements to Pasha Galler in terms of reliability, scalability, security. Yes, we do know that it's painful whenever.
The gallery is not available, so we continue to try to do those things in terms of both notification, monitoring and things like that. On the Windows Open SSH side, the big thing is we are and I think this is also mentioned in 2025, that is something aspirational, but it's something we're more committed to.
Some prototype work has started. I'll say that we want to support intra ID authentication support. There are technical complications here, but we'll we'll continue to work through those.
On the DSC side, we're getting closer to and out. Maybe I'll just combine this with the other item that we have in the agenda anyways. So do you see? Yeah, it's OK. So 3.2, we're getting closer to a GA as similar to like the PowerShell 76, although the the timers are not aligned, they're not intended to be aligned.

Jason Helmick   14:32
Yeah, I did that to you again.

Steve Lee (POWERSHELL)   14:43
So I'm hoping that we'll have a release candidate next month or so and then maybe GA. So that means March and then April, maybe GA something like that and then we'll immediately move on to 3/3.
The Python adapter is something that's going to be. I'm not expecting it to show up in 3.2 as a preview although it might, but you know one of the things that we've talked about in the working group, in the community and also within our team is as much as we love PowerShell, we don't expect everyone to write the resources in PowerShell, especially on non Windows systems.
So we're looking at basically providing an equivalent experience of how you write a. Today you can write a partial class and that becomes a DC resource working with both V2 and V3. We want to figure out how do we make that also possible with Python and hopefully we'll get a lot more resources on Linux for example.
Yep, mentioned 3-3 a bit and that's kind of, you know, add again, there are other things that we have to do and other things that we want to do that I that are not here. So there's not necessarily complete exhaustive list to say, hey, here's everything that the team is only working on.
But these are things that I do want to make see if we can prioritize and get it available to users over this next calendar year. So all right, I will go through the questions in the chat and I'll answer in the chat and then we can move on to other things.

Jason Helmick   16:03
Thanks, Steve. Well, the next thing to move on to, since Steve already did kind of the DSC update as well would be, oh, let's talk about PS resource kit. Anam, are you around?

Anam Navied   16:16
Yeah, hi everyone. psresourceget recently had a 120 RC3 release and this release addressed an issue where if you were using psresourceget to install the latest version of AZ, a 500 error was being thrown for the dependency module AZ Container Registry.
And this was causing installation to fail for AZ and our fix basically improves how we determine dependency package versions and query for it. So if you're using PS resource get to install AZ, we recommend that you update to the latest.
Preview this 120 RC three. It is also production ready by virtue of it being an RC release, so we would appreciate feedback and if you're still seeing any issues with that, please let us know. One thing to note was that this issue did not repro with PowerShell get, so if you're using PowerShell get.
No kind of additional stuff there. Yeah, so would appreciate feedback on the RC, especially as we kind of go towards the GA 412 for PS resource get. The other thing I kind of wanted to mention more so relates to PS Gallery. So over the past couple of months, if you've gone to the gallery site, you might have.
Occasionally seen that when you search for a package, you get 0 packages returned message or an incomplete number of packages returned. Unfortunately, we are observing intermittent issues with our search indexing system that's causing incorrect search results to be returned.
And our guidance is that if you see this issue to use client commands like PS resource get to search for your package and to provide a full package name. So for example doing find PS resource dash AZ will mean that you still get your package back for search results. Unfortunately there are some.
Client commands that are impacted by this issue. So if you search for tags or if you're searching for a package with a wild card in the name like find PS resource, name AZ star, you may occasionally see incomplete results or empty results.
And we apologize for the impact and the frustration that this causes. But we do also want to just let the community know that we're actively working on this issue and it's definitely something the team is aware of, yeah.

Jason Helmick   18:37
You know, while I got you on the call and I'm not trying to put you on the spot or anything, but just kind of check my own memory here. There were folks, there were a couple of questions in the community call discussion issue around RHEL 10 and Debian.
12 and 13 RPM package support and and folks I I'm trying to get some some answers on that and we'll probably post back into that discussion. But Anam am I correct in that REL 10 actually we actually ran into a problem with that a couple of weeks ago and we're we're trying to figure that out and then we'll work on that.
And get that moving.

Anam Navied   19:15
Yes, yeah, definitely. And we'll post an update in that issue or discussion issue when we get a chance. Yeah. And I believe REL 10, I think we do have some, sorry for Debian 12, we do have some PowerShell packages published to PMC.

Jason Helmick   19:25
Great.

Anam Navied   19:35
I think 754 is the latest, but we can look into other versions also being there as well. But yeah, I'll post an update there when we get more information, yeah.

Jason Helmick   19:45
Thanks, Anel. And folks, thank you for putting that into the discussion list. We'll work on chasing some information down on that. So appreciate it. And with that, oh, wait a minute, we've already done Steve. So that takes me to Sean. How you doing, Sean?

Anam Navied   19:45
Yes.
No.

Sean Wheeler   20:03
Good. So let me share my screen here and I'll provide a link to the what's new. Not not a lot of new documentation to talk about, but lots of maintenance work over the past.
Last few months.
I did a a big rewrite on.
PowerShell get the documentation so updated the.
Install documentation for PowerShell git. This is targeted at the new new Windows install PowerShell 5.1 scenarios when you first.
Run PowerShell. It's using. Windows has very old versions of PowerShell get and you have to bootstrap the Nugent components and if you're connected to the Internet, all of that works fine.
It prompts you for the updates and then and it installs the necessary components and then you can run install module to get the new versions of PowerShell get and PS resource get.
So this documentation reflects all of that, hopefully makes the steps a little clearer. They they could be confusing for a lot of folks, but the bigger source of confusion is when you're in an offline scenario where you have a brand new Windows machine that you're.
Building and it's not connected to the Internet. It's in an isolated network. So how do you bootstrap all of that? And the previous steps we had for that were very convoluted. The short answer is.
Um.
Install the updated versions of PowerShell, get package management and PS resource get on an Internet connected machine and then copy those installed modules over to the new machine and then the only other thing that you have to do is.
Install Nuget.exe and I've got instructions for that. Nuget.exe is used by the old PowerShell get module for publishing. If you're using PS resource get, it no longer needs that.
So yet another reason to.
Update your muscle memory to use resource kit instead.
Um.
And I think that's it.

Jason Helmick   23:03
Oh, wow. Cool. Thank you very much, Sir. Appreciate it. And Next up is, wow, I've been looking forward to seeing this and I actually got to see it yesterday. So now I'm really looking forward to it. Andy, are you around? You want to do your DSC plus bicep demo?

Sean Wheeler   23:05
All right. Thanks.

Speaker 1   23:20
I am and I would love to. Let me go ahead and get this link shared so people can actually get to the extension in the docs as we do this because it there is now the extension up on like a public ACR for demo purposes. So you could do this demo yourself other than you need to build DSC until we have the next pre-release out but.
Really excited to show you what will happen with that. So let me get the screen shared.
All right, do we see V as code?

Jason Helmick   23:51
Yep, we sure do.

Speaker 1   23:53
Awesome. So what I'm going to demo here is where we sort of landed on the integration of like authoring, you know, your DSC V3 config documents in Bicep. And really what this means is that you're orchestrating DSC resources using Bicep. Why? Because Bicep isn't just some DSL that was written. It's actually like it's a.
A whole project that is all about resource orchestration, originally written to orchestrate Azure resources. They've kind of gone a little bit back to the drawing board inside of Bicep and said, you know what, this is just plain resource orchestration. Let's make it support arbitrary resources and they provided an API, they call it local deploy or.
Like the local extension model for Bicep, where you, me, an author, like some developer can come along and write an extension to teach Bicep what to do for arbitrary resources and what is DSC V3 but a bunch of resources to be deployed. So the demo that I'm going to show here is how I've actually.
I've got a Bicep module right here. This is just plain Bicep code. This isn't using any special version of Bicep. This is Bicep 0.40.2 just that I installed from Winget and I've written a bunch of Bicep code where I say hey we're using an extension called DSC with target scope local because it is a local.
Extension. The only special stuff we're doing here, if it's even all that special anymore, is that I've configured Bicep to know what that extension is, the DSC extension right here and where to get it from. And this is the part that you could actually follow along now where Bicep will just auto restore what's been published there, which if I got into the details of, it's interesting and I'll.
I'll tell you about it a little later, but it's a thing that gets restored that Bicep runs and inside it is a binary that links into DSC Lib. So it's not executing like the DSC command line, it's actually just getting information about each resource that you're creating or getting or deleting or whatever you want to be doing with it and the properties and just.
It executes directly, so that gets restored automatically by Bicep and we are using the feature called Local Deploy. Yes, it's under experimental features. Bicep has said they are committed to this feature. We are not the only ones using it and partnering with them on that. It's just one of these newer things that they've started to do inside of Bicep because they're like, actually, hold up.
We're not just Azure. This is a great tool for resource orchestration. Let's expand it. We're one of the expansions. So I've got a bicep parameter file that is going to load my bicep module that we are at that windows dot bicep file and actually has specified inputs to the parameters in that file and this is where stuff is really.
Starting to be super cool because since this is just bicep by like in use as in of itself, this isn't some sort of hack into DSC to kind of be bicep, which was sort of our initial approach was like can we get bicep to give us arm and then we parse the arm and it just like how is all of this going to work?
This is bicep code. So if I delete that right there, I get my Intelliseps because it's a bicep parameter file and it's reading the bicep file over there where I said hey, there's some options for dark mode off on and toggle and packages knows that this is an array.
If I set it to something else, a string, it's going to give me a type error that hey, you actually said that parameter is a string. So I'm going to put that back here, but note these inputs. I'm toggling dark mode and that's that's kind of a fun visual of this demo, but I'm also just listing out some packages because the second-half of the demo is to use the win get.
AC resource to get me some package info. So let's step through the bicep file cuz I think that this is really interesting. This is essentially a high level or actually I think Mikey likes to put it a higher order tool for orchestrating resources. I'm gonna orchestrate a bunch of registry resources and Winget resources.
And these are DSC resources that I just grabbed from DSC. It will work when we do the next pre-release of DSC. I have to add them to the path right now off the dev build. Sorry, not sorry, just, you know, that's where we're at with things. But let's take a look at that. There's that parameter I was just talking about where I've got a string that I've defined called dark mode and I want.
To toggle dark mode or set it on or off. In this case I'm gonna use toggle and that's my default here. And to do that on Windows, just for demo purposes, it means executing well, setting these four registry keys and no, I can actually start using real full bicep code here. Like I didn't want to rewrite this string. This is a shared.
Prefix for all these registry keys. So I get to use bicep string interpolation right here and it just automatically works. So it fills that in right there for this bicep array of objects that I've made where it's name and path for accent color menu, start color menu, use light beam and system uses light name and that might tell you why.
Zero I think means to turn it on because it's actually use light theme, but I called my parameter dark theme implementation details of the demo. Don't worry about it. But here's where stuff starts to get interesting. There's no way to tell this like set to toggle it's it's it's a binary zero or a one, right? But.
Well, this is bicep code. I can actually do a git on that resource 1st and this is where things I think start to get great because like in a a YAML config document in DSC, you're just listing your resources and then you're sending that to the you're executing it with the DSCCLI to get all of those or set all of those. There isn't this.
Um.
Mix of resources in there or sorry, mix of operations against the resources. You're either getting all of them or sending all of them. But Bicep is a higher order tool. It actually totally can understand that, hey, I'm gonna use this resource called Current registry key. What is it? Well, it's the Microsoft dot Windows registry resource. You've got full typing and.
Sense here. If I delete this you can see that there's some all the other built-in resources registry update list. I'm going to go put that back to registry and Yep version 1. Oh and the keyword here, literally a keyword is existing. This is bicepism. If you just pull the bicep docs I'm like how to do a git on a resource and that says get me.
The resource that matches these parameters. And so this underneath the covers is going to go ahead and execute bicep code first and foremost. And what is it doing? It's going to the keys array and it's doing a reverse look up for the last item in that array. Yeah, I could have used the first one and just put a zero there, but I kind of.
I want to show off that this is just bicep straight up and so bicep has got the syntax of a caret one to go to last element right? And that just works because it's just bicep running at the top level. So it gets that last item there and then it gets the pieces off of it which.
Cool thing like it knows what that thing is. So if I hit dot I can fill out that I want dot path right? That IntelliSense is there. So what I'm doing here first is executing the registry resource to create a current registry key resource of like properties here that matched system uses light beam under.
Under the prefix HKCU software, Microsoft Windows current version beams personalized and then we're going to get the value off of it and things get really interesting here because we have full typing support of this. The project that I linked up there is doing type resolution.
For all of these DSC resources. So if I just move over to the little bit piece of TypeScript code here, I actually executed this against those built-in DSC resources. It went through the manifest of the registry resource, processed the type, knows what all of the things are defined to be as if that makes sense like it gets the strong typing for that.
And translates it into types that Bicep understands so that something like this happens. Well, current registry key, let me go ahead and delete this and do it out. What's on there? Well, value data is on there. What's on there? Well, nothing, because it could be null. So let's go ahead and say, well, it might be null, but if it's not null.
What's the item on it? Dark mode. I'm just going to paste because I don't know exactly how to type this again. So not dark mode D word and that was it. So dot value data dot.
Yes, yes.
Could be any of these items. There it is. Value data could be AD word. So as long as it's not null, let's go ahead and get the D word off of it and not mess up my demo. So paste again. Oh, that's why that wasn't completing. I'm so sorry. See, this is what happens when you do live demos and you can't type or copy paste.
you're in a Windows computer from a Mac computer.
Do that again. Current registry key dot fill it out. Value data dot could be null. Get the get the item off of it. Oh my God Andy, just revert to what I have saved. One second.
Discard file reloaded dot value data dot D word which is off of there. More interesting to this is that it knows that that might be null. So I have to do the null call less scene to say get the D word item off that field and if it's you know not available, if we didn't get a registry item for instance that match keep at the value name, we need to set a value.
for it because we can't have null values. So I do my more null coalescing to zero and then it actually knows when I go to use this item later in this code that well that item might be as small as -1 and that's too small for where it's assigned down here. This D word takes
The value data and it knows that that needs to be positive. They grab that from the type information. So what do you do if you have some random integer and you need to ensure that it's positive to satisfy a typing system? You wrap it in Max and you know, say it's zero or greater. So that gets us.
The number that was in the registry key.
Hopefully it says your or one it should have been, but you know we got that number in the registry key and now we can use even more cool. I think bicep code here where we've got support for bicep turn. Well no let me correct that. When we first started this project it was we need to support stuff like binary's ternary statement, the biceps ternary statement.
Because it meant that like we were processing the output sort of before it's been executed. We got an ARM file that had the equivalent functions that Bicep said, well, if you have a ternary here, you're actually doing all these logical functions in the ARM and then DSC needed to support those.
That's not the case anymore. This is Bicep code that's executing. Bicep sees the ternary statement. It does the logic that it needs to do of well, let me check what dark mode is. If it's toggle, let's go ahead and grab current whatever value that we got there, add 1 to it, and modify 2. It's executing that actual code first and foremost so that when we.
Get to where we're going to use the value we assign your value data. It's actually just been resolved. It's the zero or the one that we're then gonna send over to DSC. So DSC isn't getting functions out of Bicep, it's just getting, hey, here's a resource and here's the properties that need to be set or get on that resource. And that's the very.
But.
We're gonna do, we're going to go ahead and set a whole bunch of resources. We're gonna use a for loop in Bicep. This doesn't mean that DSC needs to support a copy statement inside of like a Jason file. It actually the little GRPC server that's executing against DSC Lib is going cool, let me get the first registry resource.
With all of this resolved and set it and then do it again and a third time and a fourth time because Bicep is actually doing the looping against the API, if that makes sense. We're calling just a few functions at a high level. I'll show you the Bicep dot proto, but let me actually visualize the demo first. So we'll set that key path to the value of key dot.
Path key dot name and the set value for value data and we're doing that for key in keys, which is what we define up here again. So we're going to do 4 registry resources and that's the first half of the demo. The second-half of the demo is a very simple demo of we're going to do a get again on a for loop for the parameter.
That we had of packages and we're going to do the win get package resource and we're going to output the properties of those resources once we've done a get on them. Now the last really interesting thing in here I want to point out is I don't have any sort of manually said depends on, but there is a dependency in here, right? We had to get the value of that first.
3.
To know whether or not it's zero or one. So we could flip the bit for toggling it and then set it. Well, this is Bicep. It's a high order resource orchestration tool. It knows it has figured out that value data got assigned from this piece of code here and that therefore it needs to execute this registry this resource first. So it does that get first.
The value back from it and then does the computation that we need and then does the sets. That's kind of the key thing here is you can write Bicep however you might want to write the Bicep code per the Bicep documentation. It all just works. So let's go ahead and demo it. Like I said, I need to set the path to the debug version.
Version of DSC. That is a temporary thing until we have a new release out and then you should just be able to do this with a built version of DSC where those resources get solved to your path. But right, you do need your DSC resources in your path and then I'm gonna run bicep local deploy against that Windows param file, that top level one that says hey use that module.
Dark mode to toggle and get me some input from some packages. So let me go ahead and execute that.
All right, it's processed everything. It's now executing against DSC lib and got a nice little table output. I'll go ahead and maximize this.
In the background we just you can see current registry key was the first one executed, got that value and then it did the sets on the other registry key. My color name changed to dark because I toggled it and then it executed that those two win get resources and we can see that what was the output? Well an array of two objects with data out of that win get resource of Microsoft dot by.
Use latest is true, version is 04/02, just like you'd get if you executed the Winget package resource against that. So that's kind of the visual part of the demo and the fun part to show you.
This README is the Microsoft the DSC extension for Bicep sort of top level project where you can see how all this is implemented. I go through a lot of what I've shown you in the demo here that we've got your type resolution happening in the GRPC server that gets bundled, but it has.
The workable bicep config that you can go use right now for demo purposes. This extension's hosted up here. It does only support Windows right now. And then what you would put at the top of your bicep file like I did for the demo and then the world's your oyster. You should just be able to use those DSC resources with full auto completion and typing.
As you saw it, so go ahead and play with that. And if you want me to get into some of the nitty gritty of like what it actually looks like underneath the covers, this is that API surface that I was talking about where we've implemented A Bicep extension. This is the DSC extension for Bicep. These are RPC calls. This is.
It's just like a protobuf API from Bicep itself of create or update a resource back, preview a resource back, get delete a resource back, which kind of sounds familiar, right? DSC get set or get with what is it? What if is the translation for preview and so you actually then have.
This is inside of the DSC project, the implementation. I think the set implementation or creator update is the one that's interesting to look at. A resource back comes through. Ignore some of the debugging here, but we get the resource type off of it, the version and a full properties bag.
We're not getting ARM Jason with a bunch of functions, right? We're getting the resource name, its version and resolved properties, and then we're running the DSC manager to find that resource for the resource type in the version. And as long as we have been found that resource, we're just executing this is DSC library API level.
Level resource dot set with the properties and then the execution kind in this case actual for the implementation of preview. That's where that differs and it's a what if and that's it. That is a GRPC server built into DSC or built by DSC and built into the extension that is uploaded that bicep resource.
That's the whole implementation. Cool. Thanks. I'll go ahead and run this again because I need my theme to be light during the day, but open for questions.

Steve Lee (POWERSHELL)   41:04
Handle questions in the chat, Andy.

Jason Helmick   41:06
Yeah. And I really appreciate it, Andy. This is really cool to see, especially, you know, where we started 18 months ago. This is, this is really impactful. So thank you very much for coming by and showing it. Oh, I'm glad you're back, James. So James, you want to go ahead and.
And do your demo. Appreciate it.

James Brundage   41:26
Sure, let's, uh, let's get this show on the road. You drag something over to the other monitor. Could you please give me sharing rights real quick?

Jason Helmick   41:36
Somebody can help me give him sharing rights.

James Brundage   41:37
So we got a bunch of good news, but only 5 minutes to give it in. So I'm going to do my best. You all might remember that I kind of took this old turtle thing and made it work in PowerShell, and you all might know that PowerShell has ways to do safe.
Data language execution. Well, for the holidays, I wanted to kind of give my nieces a good educational gift. Teach them a little bit about Turtle, teach them a little bit about Web Dev, teach them a little bit about PowerShell, but not get them hacked. So I went and made a web safe REPL for PowerShell.
And then, well, don't quite have the sharing rights yet, so I'm gonna gonna wait until I do. Oh, I guess I can at least click over to the page here while we wait. Actually, I can probably even highlight a little bit here.
So it's called Reptile. You can basically say here is an initialization script, here is a list of supported commands, and here is the REPL we're going to run.
Do we? Do I have sharing rights yet?
If not, I'm going to have to apologize for the resolution in the interest of time. So I went and clicked open that link. Actually, let's create a new one real quick just to show these are just running in the thread job. I've got a bunch of them now.
Go. Let's go to this one.
OK, so I said to my nieces. Are we ready to do full screen here?
Yes, OK, cool, I said to my nieces. There we go.

Jason Helmick   43:17
You should be able to go to full screen, yeah.

James Brundage   43:22
And I'm getting out my stopwatch here. I said I bet you I can draw 100 pretty pictures in a minute and said no, that that's not possible. So I'm actually going to click start on my stopwatch here and I'm going to click start a bunch of random times here.
Quick as I can.
And it's going to be generating a random image in a background thread job while I'm clicking each one of these and they're all going to just scroll up once we're done and the 900 and.
Can actually kind of see it go. There we go. Boom. So that's 21 seconds. You actually missed it all just standing up and going away for a second, Jason. So we basically have an interactive way to run PowerShell.

Jason Helmick   44:07
OK.

James Brundage   44:12
In the browser safely.
This is running PowerShell server side inside of a data block with only the supported commands that you allow. And if you try to do something like, you know, get process, it's going to yell at you and you're not going to be able to do that.
So this provides us a way to actually have a very quick, fast interactive experience around PowerShell and web, which actually brings me to the other real quick short joy. I threw it in chat earlier, but.
I had been looking at a site called Eleventy, a static site generator and they had this sort of humble brag. Hey, we're the fastest static site generator in the business. We can go through 4000 markdown files in X number of seconds and that acts as like two or three for them and the article they're linking to was.
One test done in 2022. So you know PowerShell person that I am, I just wouldn't go ahead and OK, I'm gonna see how fast PowerShell is compared to that. I'm gonna go build myself a repo with four KB worth of markdown files. So that's 4096 markdown files.
Each about four KB and I'm going to go see which is the fastest static site generator that I can find and I will take more PRS and submissions to this, but ain't that a sweet little graph there. So just running through PowerShell to build static sites.
Purely with PowerShell, fastest one is less than one second. There's 11 T down there at about 7.88. This is also a good module to know about that we're not going to have time to demo in 5 minutes, which is Mark X. That's a new markdown tool I built for PowerShell. That's just a little bit slower. This is the thing Cloudfare just paid a bunch of.
Money for Cloudflare. Could I please get a bunch of money for static site generation on PowerShell? Because clearly we are way faster than what you just paid a bunch of money for. So yeah, it's a really exciting time in PowerShell period.
It's really, really exciting for PowerShell on web development because at this point, just to kind of underline it, you just saw how fast we have server side PowerShell.

Jason Helmick   46:28
Mhm.

James Brundage   46:34
Which is basically about as fast as I can click it. We just saw how fast you have client side PowerShell, which is best in the business too. So most people still are on this page or oh, PowerShell's just this, you know, little terminal language that I can only use for Windows stuff.
Oh no, no, no, no, no. It is a cross-platform beautiful beast and while that and 5 bucks will get you a cup of coffee, it is currently the fastest web language in the world, both for static site generation and for server side code.
So if you want to know more about this, please do the follows on social. Please reach out. There's a new organization called Posh Web where I'm kind of trying to gather all the people trying to build a better web on top of PowerShell or with PowerShell.
Please reach out if you're interested in joining and hey, PowerShell team, let's let's start actually figuring out how we can highlight this scenario and integrate it better with the product line. Because at this point, why shouldn't we be really, really freaking proud of ourselves?
I have no microphone to drop, so I'm just gonna click stop sharing and was that less than 5 minutes?

Jason Helmick   47:52
It was not less, but I want to thank you for two things. You you were just about like 2 minutes over. It's it's so I I just want to thank you. First of all, great demo and thanks for the shout out on the PowerShell stuff. I really love it and #2.

James Brundage   47:54
How many seconds over?
Ah.

Jason Helmick   48:09
Thank you for the timing. Because of the timing, we get to have one more announcement before before we end the call for today. So Amber, you wanted to come up and talk about, I forgot what you wanted to talk about.

Amber Erickson   48:21
Yes, the gallery migration to AKS. So I just wanted to provide a quick update for this. So I know there's some concerns about this work. It's still a top priority for us. We've been working on this for about a year.
And are done with the initial stages of it. Um, we're gonna be slowly testing in our Canary environment and then moving over to prod. Um, we're still quite a few months out for that to happen, but I just wanted to say that um.
This is still definitely a very high priority for us and moving over to a KS, well, we're hoping help alleviate some of the scaling issues that we've been experiencing for the past year and a half or two years. But this is a really, really big project. We've had to rewrite the.
Entire, well, not the entire, but a lot of the gallery code base. We've had to build out entirely new infrastructure. So this is quite a bit of work. We're moving very cautiously because we want to make sure that like as this migration happens, especially in prod.
It doesn't impact users and that the experience is actually better than what it currently is. So just a quick update on that. If there's any questions, feel free to drop it in the chat and I will answer those.

Jason Helmick   49:43
Great. Thank you, Amber.

Amber Erickson   49:43
OK.

Jason Helmick   49:46
Thank you, Amber, and thank you everyone. This will bring our our February community call to a close. Please, please feel free to keep typing in questions into the chat and stuff like that and into that discussion in the the the PowerShell repo.
That we were talking about at the beginning of the call. I'll try to get the call up as soon as I can within the next few days. So with that, thank you everyone. Have a wonderful month and we'll see you back in March. Cheers.

Jason Helmick stopped transcription
