PowerShellOpenSSH Community Call-20260917_112834-Meeting Recording
September 17, 2026, 4:28PM
39m 46s

Sean Wheeler started transcription


Sean Wheeler   2:06
Well, we're...
At time, there's a few people still joining, but we're going to go ahead and get started.
Good evening. Hello and welcome to the September 2026 PowerShell Community Call. I'm going to be your host today. Jason is out for some well-deserved vacation. We don't have a huge agenda today.
Um...
And so we'll probably keep this a bit short, but...
Just want to remind everybody about our code of conduct and our contributor guide. This meeting is being recorded and I see nicely everyone's muted and cameras turned off.
That helps us get a better recording. So unless you're speaking, we appreciate keeping those off. I've posted the agenda to the chat so you can see what we'll be talking about. And
Let's see, we're going to get started, I think, with Amber. Are you here?
Amber was going to give us an update on PS Resource Kit Preview 2.
I don't see her here yet.

Steve Lee (POWERSHELL)   3:43
Let's skip ahead and then we can get back.

Sean Wheeler   3:45
Yeah, we'll circle back.
So, Mikey, are you here?
Oh my.
Um, Steve, do you want to talk about this at all?

Steve Lee (POWERSHELL)   4:06
Oh, Amber just got back.

Sean Wheeler   4:07
Oh, Amber.

Steve Lee (POWERSHELL)   4:08
Let's go back, let's go back, and then maybe Mike will join, otherwise I'll talk to him to see.

Sean Wheeler   4:09
Yes.
Yeah.
Amber, are you here?

Steve Lee (POWERSHELL)   4:18
I just saw her image popping.
Unless you dropped.
No, no, she's still here. Amber, you're on mute if you're...
Saying something.
Maybe she's having technical difficulties.

Sean Wheeler   4:45
Yeah, so has Mikey joined yet?
Okay. Steve, you want to step in and...

Steve Lee (POWERSHELL)   4:59
Yeah, sure. So Mikey did ship DC 3.3 GA main general availability. Oh.

Amber Erickson   5:05
Hi, sorry, I had just joined, so I didn't realize. I think, Sean, you had prompted me to talk about PS Resource Gap.

Steve Lee (POWERSHELL)   5:11
Yeah, you were gonna be first.

Sean Wheeler   5:15
Do you want to continue, Steve, or let Amber jump in?

Steve Lee (POWERSHELL)   5:19
Well, we can let Amber do it.

Sean Wheeler   5:21
Okay.

Amber Erickson   5:21
No, I think I'm having some issues actually. I can't hear anyone, but...

Steve Lee (POWERSHELL)   5:24
All right.

Amber Erickson   5:30
Sorry, Steve, sorry if I interrupted.

Steve Lee (POWERSHELL)   5:30
We can, we can hear you if you can hear me.

Amber Erickson   5:37
Okay, I'm just going to do a quick update of PS Resource Kit. We have a release that was supposed to be out today, and it's not out yet. We're hoping to still get it out today. So I just wanted to update folks about that. We're going to have some new changes. All of the concurrency work is now done and going to be out.
We have an update that Justin had added to our PS modules path. We're a little bit late with getting that information over to Sean, but Sean will have docs for that up as well. So just be on the lookout for that. And that's pretty much it.

Sean Wheeler   6:19
Okay, thanks. Yeah, I've got a PR working on that, so hopefully we can get them out today as well.
All right.
Has Mikey joined yet?
Okay, so, yes.

Steve Lee (POWERSHELL)   6:40
It's all right, I can just give a quick update on DSC. By the way, I just noticed that Jim has joined this call. That's exciting. I haven't seen him in a while. Jim Truer. All right, yeah, so Mikey released DSC 3.3 either yesterday or this morning. I forget which, but so that's out. Yeah, so that is our last, our latest stable release.

Sean Wheeler   6:56
This morning.

Steve Lee (POWERSHELL)   7:01
There is a blog post that Sean is showing. There's a link in the chat about the blog post. There's a bunch of new things in there.
DC 3.4 Preview 1 has already been released as well. I'll give a demo towards the end on some of the new stuff there. We do hope people are starting to use this in production. If you do have issues, please open them up in the DC repo. And also, if you're trying to use in production and there are challenges with some of the legacy.
Let's call it DCV1 resources. There's a, just on everyone knows like there's, okay, there's two limitations you should be aware of. If you're using the old stuff with Windows PowerShell, it does go through the LLC, the Local Configuration Manager, which requires you to be elevated because it goes through WinRM. All right, so that was.
Intentional design because it doesn't require you to install anything extra. You don't need a new version of PSDSE module for that to work. But it does require you to be either elevated or you have to modify the permissions to winner in, which I would not recommend for that to work. So some of those scenarios, I've been slowly converting those old resources to a new DCV3 resources, so you don't have to go to that path.
But if you're using anything with PowerShell 7 with class-based resources, they'll show all this work. So that's a better model by understanding that some legacy stuff still use the old model.
I think that's pretty much it on that topic.

Sean Wheeler   8:23
Okay.
Thank you, Steve. Next on the agenda, we had Ryan Yates wanted to talk to the community about getting some feedback on an issue he opened for new module manifest. Ryan, are you here?

Ryan Yates   8:39
Yeah, yeah, I'm here. Could I get presenter right so I could just share my screen quickly?

Sean Wheeler   8:44
Um...
Let me find you so I can elevate you. Let's see.
Um...
Steve, if you can find him before I do.

Steve Lee (POWERSHELL)   9:09
I'll try it. This is, is this even any kind of order? It doesn't look like it.

Sean Wheeler   9:15
There he is. Let's see, make presenter. There we go, change.

Steve Lee (POWERSHELL)   9:17
But.

Sean Wheeler   9:20
You should have it now.

Ryan Yates   9:25
Yeah, there we go.
Let me know if you can see my screen. Nice, nice and clear. Right, so whilst I was going through some of the older issues that we've had in the repository, you'll notice I've got here a manifest issues grouping. So I'm going to be looking at stuff that has got issues that come with the new module manifest and then

Sean Wheeler   9:32
Yep.

Ryan Yates   9:50
Test module manifest and then from PS resource get the new update PS resource manifest. PowerShell get version 2 has the similar update module manifest and I know this has been a pain area for module authors over the years. One of the ones that I've seen a lot of people do instead of using new module manifest is they create their own
hash table and then have their own export to a BSD1. So I want to just add a couple of little small enhancements to this and want to get some feedback from the community if this just makes sense for changing how the output was. So on the left you can see I've got
This issue for adding a new switch just to remove the descriptive comments from the manifest. So this would leave this shape, as you can see in the issue. And I've then got on the right just to remove out all commented out properties so that you would have this sort of shape
which is what I see quite a lot of developers.
going forward and having, because they don't use all the other properties that you see on the left. So I want to be able to give that as a, this is what you can get from this particular command that going forward. And then with one for update PS resource manifest or
whatever the actual name for that is. What we don't want happening is if you've got some custom data in, say, PS data in your manifest, we want to be able to import that and then just update things like the module version properly without having to do a full delete of the previous manifest.
because that's one of the things that I know that there's a lot of working around to do that.
So that's just what I wanted to bring up. So if you could have a look at these two particular issues, give them a thumbs up if you think they look good and I will look to try and get that in in October time.

Steve Lee (POWERSHELL)   11:57
Let me just quickly add, because Ryan is part of the commanded working group, this is where this discussion came up and we couldn't conclude within ourselves what is the most prominent pattern used by module authors. So this is where we definitely want to get some feedback of what aspects of this issue is most important to you that you would actually use.

Ryan Yates   12:14
Particularly on the left-hand one, the comments at the top of the manifest, so this particular section here, I know that some people will use that, but mass majority probably just go, yeah, that's useless, we don't need it.
Yeah, that's everything from me so far. Thank you.

Sean Wheeler   12:34
All right.
Thanks, Ryan. All right, so now I will get into my docs update real quick. Let me share my screen. Come back here, of course.
Oh.
As always, I publish each month an update of what's new in PowerShell docs, and I call out our community contributors. I want to thank everyone for that. If you haven't seen also the contributor Hall of Fame, this goes back to the beginnings of the documentation repo.
largest contributors. And we couldn't have the product we have without the contributors we have, so we really appreciate that. There was one new article I helped create for DSCV3, updated the install instructions here, and we have
This tabbed interface for each of the different supported operating systems, so you can figure out how to install the new DSC V3.
Both.
using package managers or manually from binary archives. And also wanted to point out, I updated my release info module this morning.
to version 1.4. And this is, if you remember, I demoed this several months ago. There's a bunch of commands in there that help you keep track of our releases and what versions of operating systems are still in support and so on.
I added this new find DSC package. It's similar to the find PMC package command that showed you what PMC packages have been published for PowerShell, but these are the published packages for DSC version 3.
And so you can get the release history. This query is GitHub to get this information. And you can get the, you can find what packages have been published.
And this will be changing right now. We only have the latest channel version published, but as new previews get published to the packages.microsoft.com, I'll...
I'll have to do a quick update, but those will show up here as well.
Um...

Steve Lee (POWERSHELL)   15:28
Sean, can I add something real quick on this topic? So what you see on the screen are just the initial distros and versions that are being published for GCV3. So if you, right now, by design, GCV3 actually has no external dependencies. So if you take the tar gzip file, for example.

Sean Wheeler   15:29
Sure.

Steve Lee (POWERSHELL)   15:48
Hypothetically, it could work on any distro of Linux, as long as you have the right architecture ARM or x64, because all the dependencies are built statically. So if you have a need for us to publish to a different distro or version on packages at marsa.com, you can just open an issue in a DC repo, and that's pretty easy for us to do.
We just need to know what distros and versions you care about.
So, it's not just to be clear, it's not limited to just the ones that are currently being published on.com.

Sean Wheeler   16:19
All right, thanks, Steve. And with that, Andrew Pla, are you here?

Steve Lee (POWERSHELL)   16:27
Hold on, Gilbert's got his hand up.

Sean Wheeler   16:29
Oh, yes, Gilbert.

Gilbert Sanchez   16:29
Sorry, Steve, so if there's no limitation, is there any reason not to just auto publish to all the existing distros that are that are available on that repository list?

Steve Lee (POWERSHELL)   16:40
One of the things, or the reason not to do it is that if it's there, we have to support it, quote un quote, forever.

Gilbert Sanchez   16:46
Yeah, I figured that would be it, okay.

Steve Lee (POWERSHELL)   16:49
Yeah, so I'd rather be selective on what people are actually using. So those are just the common ones we expect people to use. So if there is something you need, let me know. I'm more than happy to do it.

Gilbert Sanchez   16:50
Right there.
Thank you.

Sean Wheeler   17:01
Alright, Andrew.
Oh, Andrew had to drop. Thanks anyway, Andrew. We'll catch you later. And with that, Steve, you're up for a demo.

Steve Lee (POWERSHELL)   17:23
Sure, let's see.
All right, so I'm going to show just a couple of some of the new resources that are part of DSC 3.4 Preview 1, which is already out.
And as I was building this demo, I already found a bug that I submitted a PR to fix, which is why it's important for people to play with these resources and give feedback, because during development time, it all looks fine. And then we actually try to use it. It's like, oh, this thing doesn't work the way you expect. So there's three specific ones. You can see I got a bunch of stuff on my.
Development machine here, but if you look on the left side, there's the Microsoft Windows environmental variable. There's also the Microsoft Windows environmental variable list version, so these two are for managing, again, it's only under the Windows namespace 'cause these only work on Windows because for managing the persistent.
environmental variables for the user or for the machine. So this is, for those who are aware, these are literally stored in the Windows registry. So this is what they're manipulating. So I'm going to show that in this case, I'm going to retrieve the PS module path from the machine. So machine is all users. There's another one which is related to group policy.
which is also Windows only. Oh, this is actually, I should show it this way. So this is an adapter. So this is the Microsoft Adapter Group Policy template. So let me give a very brief, whoops, I didn't hit that. So for those who don't know, if you open up MMC,
This is the old Microsoft Management Console, or actually use your shortcut, you can go GP edit.
which will load in the group policy snap in and I will try to make this, can I make this bigger?
Actually, I don't know if this old thing, yeah, I can't make the font bigger.

Sean Wheeler   19:11
It.

Steve Lee (POWERSHELL)   19:13
Don't worry about what this says. The key thing here is these are administrative templates that are shipped with Windows. If you're able to see, and I'm not going to use the Zoom tool because I really hate it, but there's a section for computer and a section for user, and one of the administrative templates that gets shipped is for Windows PowerShell.
So, it's down here, and this allows you to do stuff like turn on module logging, script log logging, stuff like that. Alright, so what this adapter does in DCV3 is it takes all these ADMX files that are on Windows and it adapts them as DCV3 resources. So, if I were to do...
See, like this, right? So this is saying, list all the resources that this adapter supports.

vukasin.terzic   19:59
How are we helping animals of the earth?

Steve Lee (POWERSHELL)   20:01
Alright, someone is not muted.
And you can kind of see that there's actually a large number of ADMX files on my system. And if you install different Windows features or roles, you'll get other ones as well. But basically, all of these are now available as a DC resource.
Right, so for the purpose of my demo, the other thing I'll mention is I prefix resident GPO. There's a reason for that in the future where we can optimize finding resources more easily with that name spacing. But I'm just going to use the Windows PowerShell one to show what's available. So I'll show that in a second. The last one is this file content resource.
This is really for, for example, in this case, you know, I work out of my Q drive for all of my Git repos, and I'm going to show the contents of the.gitignore. So if I were to run this as an export.
This one.
This is going to export these three things.
Take just a second.
All right, so if I scroll up...
So the first one is the emergency variable. Like I said, this is my PS module path, but it's for all users because I don't have it defined for myself. You can kind of see this is just a string. I open up an issue to turn this back into an array. But for the GP or the administrative templates, you can kind of see for my own testing, I did enable a few of these things.
at the user level. So you can have like module logging, transcription is not enabled, script log is enabled, stuff like that. So you can see that if you were to, this GUI was bigger, then you can kind of see if I go in here.
Like I have some test content in here, this is my comment. And then like, you know, it has, you're able to add like a list of modules to do module logging. And that gets represented in the Jason here or in the Xiangmiao representation. And then the last one is file content. Again, everything gets read in as a string.
UTF-8, so you can see the content of the git ignore in the DSC project is, you know, a bunch of rust stuff, code coverage, artifact stuff, tree sitter stuff, stuff like that, but you can also use it to generate the SHA-256 and SHA-512, so you can easily determine whether or not the file that you expect in the system is.
matching the one that you desire, right? I think we'll probably continue to enhance and have separate file copy resource if you're trying to like put the same file everywhere kind of thing.
And I think that's probably it for this. So you can definitely do a get set and test and export for all of these resources. Export has a problem that I fixed already, so that won't work in the preview one, but it'll be fixed in preview 2.
That's it for my demo.
Ryan, you have a question on this?

Sean Wheeler   22:53
All right.

Ryan Yates   22:55
Just, yeah, just on that, particularly the file content. Obviously, this is an export of the content that's currently on disk. Is there for the import process, can we say import it from an existing file?

Steve Lee (POWERSHELL)   23:03
Yes.
So you could do, so the answer is yes and no. So if you, if I were to do this and redirect this to, like I have everything in my test folder, because I can just delete it. Let's say exported.json, because it's going to be JSON by default, right? So if I ran this, it's basically going to have the same output, but it'll be in JSON.
Then I can take this JSON file, which is a valid DC configuration, I could do a set on a different computer, and it will apply this content onto the same path. All right, now your question, I think, is more like, can I take a file, let's say, from a network share and apply it? So that's where the file copy resource would.
Support that model, and that one one of those is it.

Ryan Yates   23:49
It's more the, it's more that when I look at these sort of files, I personally hate seeing PowerShell scripts in any YAML file. I want a pointer to a file that already exists so I can get all the Intellisense. Yeah.

Steve Lee (POWERSHELL)   24:02
No, no, that's what the file copy resource is about, right? So rather than having the content in the actual configuration, file copy would say there's a source path and destination path, but that resource doesn't exist yet. That's coming later.
This one is more useful for if you already have a bunch of stuff set and you want to export it than if I were to open this, right?
Then you'll see, well, it's not gonna be, it's gonna be compressed. Actually, I can probably go like this and format document.
Then you can kind of see, like, um, it's truncated 'cause this, like, show so much, but it's all in here, and then I could just set it all or modify it as needed. The idea with the current one is you do export and then do a set to clinical import.
We can move on, Sean.

Sean Wheeler   24:59
That's the end of our official agenda.

Steve Lee (POWERSHELL)   25:03
I think, so if you have not been found chat, that is our official agenda, but I think...

Sean Wheeler   25:08
Yeah, there was somebody from the community you had.

Steve Lee (POWERSHELL)   25:12
Yes, I'm scrolling back up to find Fabian, I think.
Are you ready?
Do you need presentation?

Fabien Tschanz   25:24
I think so.
I will just quickly share my screen if that's okay.

Sean Wheeler   25:29
Um, yeah, what?
One of us will find you.

Fabien Tschanz   25:34
I don't really have a presentation.

Sean Wheeler   25:58
There we go.

Fabien Tschanz   26:01
OK, so I don't know, do you already see something?

Sean Wheeler   26:04
Yes.

Steve Lee (POWERSHELL)   26:04
Yes.

Fabien Tschanz   26:05
OK, awesome. So, as Steve already mentioned, I've been on the I've been on the Microsoft 365 DSC project, and we've now come to the to the result that we've now converted all of our script-based DSC resources that were around 535 of those DSC resources.
We've now been able to convert them all to class-based resources. So that's basically one of our greatest achievements we've probably done so far. And if somebody is interested in the entire story, I'm only just going to talk about it a little bit here. There's an entire three-part
blog series on the Microsoft 365 DSC.com blog. If you want to take a look at it, about the conversion, making it fast again, and comparing the configurations and some additional stuff with DSC v3. So if you want to take a look at it, a closer look at it, feel free to do so. And otherwise, I'll just...
quickly show a little bit about how it was before. So first, we all had this lovely structure here with, I'm just going to zoom in a little bit. So we've all had the PSM1 files with the PowerShell module files and the schema.mov, which describes how the things are structured.
So, if we take a look at this one here, then we can all can all see that that there's a huge param block for the for the get target resource function, followed by some more module code that we that we all had, and then if we scroll a little bit further down,
Yeah, I know it's a pretty large resource, but then we've got the set target resource function again with the entire param block, and well, this basically repeats over all of the resources. So you always have three functions, get, set, and test, and each and every one of those functions just displays
and defines all of the params, which isn't really that great in terms of lines of code and maintainability if you always have to scroll around 5,000 lines for each resource, or not just quite 5,000, but yeah, you get the point. And now the newer version with the clause-based resources is
The schema.mof is entirely gone, because everything lives in the PowerShell module file, and there you define the parameters just once as a DSC property. We've also added the system.component.description attribute, so that we can port over the schema morph descriptions that were there.
So that we still have some kind of description about all of the, for all of the parameters and properties here. And then you have the get method here, which returns a typed clause instance of that resource.
Then the entire part here is exactly the same, so I'll have to scroll a little bit further down to get to the next method. But then you'll have to have the set method, it's obviously a void because it doesn't output anything. And then further down you have...
The test method, which is coming up just right here, it's basically just one call to our M365DC resource base class, which we defined as our base instance, where every resource is inheriting from, so that we can just delegate some of the stuff to one.
Base resource and not have it to define over all of those other resources while time and time again.
That was basically everything about that.
If you have ever written one of those script-based resources, you probably have used PS bound parameters. So PS bound parameters is just one thing that only lives in script-based DS resource or better said in functions in PowerShell. It's only available in functions and defines which.
parameters were specified. And in class-based resources, there's pretty much no equivalent for that.
So we also had to find out a way on how we can mimic this behavior without changing everything in that module. And what we came up with is a little bit of a...
Difficult construct to wrap the head around, but it's we have here a read bound parameters method, which is just reading the CLR property of the underlying class instance being defined.
by the caller. So for example, if you have DSCV3 running and creating those classes and clause-based instances, it will always define the properties on that class, and we are then reading the CLR value.
Of of that of that of that underlying field that defines the property.
And why are we not just reading the property itself? Because the local configuration manager from Windows PowerShell sets all of those properties by reflection. So the actual property value is not even populated, it will always read null. So that's also one thing that we had to consider and test time and time again.
Steve, I think you probably wrote plenty of script-based DSC resources and have also used PS bound parameters and probably also a couple of others here in the community. Did you ever have something to do with the bound parameters when you converted them to class-based resources or something like that?

Steve Lee (POWERSHELL)   32:00
I never had that myself.

Fabien Tschanz   32:02
Okay.
So, yeah, that that that CLR backing field, which we are just just populating and and retrieving the values from here is is pretty unique to just to just the LCM, because the SCV3 was working perfectly fine, but the moment we we we tested it with the with the LCM, it it all fell apart.
Unfortunately.
for the entire conversion of the resources over to class. Here we have one of our conversion scripts, which is absolutely huge.
But it did everything on its own.
And of course, I did not write that all by myself. I had AI assist me in that. Otherwise, it would not have been possible to do that in just about two to three months when we took about a year to specify how we wanted to have the solution look like at the end.
So the conversion itself was done quite easily, but the entire definition and how to use the base resource and so on, that took almost a year and quite some community Discussions to get everything wrapped around.
One thing to mention here is that class-based resources are way slower to import if you have many in your module. So, for example, importing the L365 DSC module before with the MOF schema files, that was just about
One and a half to two seconds import time, and the moment you import class-based resources from the same module at the same number of resources, it will spike up to five or six seconds, so that's one thing we experimented a couple of times with.
bucketing and just defining nested modules.
And that seemed to work better. So if you had all of those DSC resources in just one file, it would even peak at 18 to 20 seconds on my machine.
Don't know if ever if anybody ever heard of that, but that's pretty pretty nasty to get to get around.
And one more thing, the moment you want to compile configurations. So do I have any of those?
Tenant config. So, for example, if you want to define such a configuration here, that's an export of my of my Intune of my Intune environment on my tenant, that will probably take a couple of minutes to compile if you want to use class-based resources for a for a.
For MOF, it would take around 20 to 30 seconds.
And that is because the import DSC resource keyword that you see here is evaluated every single time one resource is being compiled by the PowerShell engine. We had to dig very, very deep and even come up with our custom.
Piler, let's say it like that, and that's...
Do I have it available here?
No, I don't think that I have it available here. But we came up with our own conversion logic, which pretty much mimics what the default compilation engine in PowerShell does, but it replaces a couple of things. All of those things are mentioned in the blog post.
But for short, we are doing some character replacement before parsing the file. So for example, configuration becomes C0 configuration. We remove the import DSC resource keyword.
So to just not trigger the slow import and compile time from the default PowerShell engine. So we are basically shadowing the PS desired state configuration module that's available in Windows PowerShell and in PowerShell 7 with the versions 1.1 and 2.0.7.
If you're interested in that, feel free to read the blog post to get everything out of it. And everything is also open source on the Microsoft 365 DSC.
GitHub organization if you want to take a look at that.
Any questions so far? I probably just just smashed everyone in their head.

Sean Wheeler   36:57
All right, thanks, Fabian. If there's no other questions, I think that's going to be a wrap. Thanks, everyone. Oh,

Steve Lee (POWERSHELL)   37:03
Hold on, I want to do one quick plug, and I know we're out of time. Sorry, James, maybe next time. I just want to make sure everyone is aware. PS Conf EU Minicon has an open call for papers now. I think call closes on September 25th, so about a week from now. So it is free to attend.
I'm submitting my session now because I just remembered, but I hope other people submit their sessions as well. I will see you guys there, at least part of it. All right.

Sean Wheeler   37:33
And happy 20th birthday to PowerShell.
and hope for many more. Thanks for coming.
Like.

Sean Wheeler stopped transcription
