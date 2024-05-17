# PowerShell_OpenSSH Community Call-20240516_092533-Meeting Recording

May 16, 2024, 4:25PM

Sydney Smith (SHE/HER)   4:27
Well, it is 930 here in Seattle, so we will go ahead and get started because we have a very full agenda today and hopefully a bunch of Community demos.
But before we get started, a few of the usual reminder first of all, this call is being recorded.
If you don't want to be on the recording, feel free to just drop from the call now and enjoy the recording.
The recording of this call will be posted to our YouTube channel.
We try and do it within two weeks, but usually the call is posted much sooner along with the note and all of that.
All, they're just a reminder of our code of conduct and we do take our code of conduct very seriously and it will be enforced the basics of it is we treat each other with kindness and also do you thin posted the agenda a reminder that we post our agenda to the discussions section of the PowerShell PowerShell GitHub page each month.
You can always host topic discussions and we really encourage Community demos.
They can be just something you found recently that you love, or something you created yourself.
It could be a wide range of topics, so we have a few today that we're excited about.
And with that, we can go ahead and get started if you would like to participate during the call, we really encourage that as well.
A few different ways you can participate.
One is by using the chat.
As you can see, people are already using the chat.
You can also continue to use the discussion that will continue to be monitored as well.
And then finally, feel free to use the raise hand feature and we'll try and moderate with that as well.
If you don't know me already, my name is Sydney.
I use she, her pregnant and I'm one of the PM's until team and I'll be moderating today. UM.
Alrighty that the checking on the comments.
So our first topic first today is conferences.
I think last month called, we really highlighted partial summit which was super fun, but we bought a number of conferences coming up over the next few months that we wanted to talk about and share our presence at as well.
So first up is Microsoft build, which is actually, I believe next week.
Ohm, I personally will be at Microsoft build staffing the cloud platform experts area in the afternoon.
I think each day, so if you are in person at Microsoft build, please do come and say hi to me and grab a PowerShell sticker and chat and I can answer any questions or just hang out.
So if you're at build, come say hi.
I think a few of us are hoping to be some of the MVP events at building as well.
The powers of team doesn't have like a specific build session.
However, we have to have a number of deliverables if it will show up in other sessions at build.
I don't wanna sort of give away any other teams, build deliverables or announcements beforehand, so we're probably highlight those more at the June community call.
But do you know that we were a part of build, even though we don't have a specific PowerShell session?
So you know PowerShell is a glue language and we contribute to a number of exciting scenarios.
Umm, we also will have an exciting UM hasn't at PS Comp EU and which is always a really, really fun conference.
I think there's gonna be a ton of Microsoft people there and people from the PowerShell team.
I really disappointed.
I'm not gonna be there for just for personal reasons.
I wasn't able to go, but a number of people from our team are gonna be there.
Steve, I'm gonna put you on the spot if you wanna.
If you wanna just spend a few seconds talking about, I know you're going.
About what?
The team is bringing to his company you.

Steve Lee (POWERSHELL HE/HIM)   8:34
What we're bringing, we're bringing ourselves, but I mean in terms of sessions or?

Sydney Smith (SHE/HER)   8:37
Yeah.
I just mean and ever you wanna say about PS come to you.

Steve Lee (POWERSHELL HE/HIM)   8:41
Yeah.
So I'm I'm really glad I'm able to attend again.
I'll be covering DSC and probably sharing some stage time with Stevens going and who else is calling on the PM side?

Sydney Smith (SHE/HER)   8:51
Singing.

Steve Lee (POWERSHELL HE/HIM)   8:52
OK.
And then Amber will be coming talking about partial gallery and PSE get type of topics and of course we'll be happy to answer any questions you might have at Psconf view.

Sydney Smith (SHE/HER)   8:53
Yeah.
Awesome.
Thank you.
I appreciate it.
And then Jason is going to talk a little bit about Tech mentor.

Jason Helmick   9:11
Hey everyone.
Yeah, techmen are.
So I know that umm and many of you have a attended detect manner in a past.
This is a solution oriented conference, really kind of focused around it.
I've been at many tech manners, really enjoyed them in the past.
I'm glad to see that several members of the PowerShell team, along with James Petty from powershell.org, he's going to join us as well.
We're going to do several sessions at the Redmond Tech matters.
Matter of fact, I'm going to drop a link in here if you're interested, so I'd like to invite you.
And hey, if for those of you that are on the East Coast Tech mentor also has a show in Orlando just a month or two after the Redmond show, to be honest.
So so you can experience a great conference there as well.
So if you're interested in coming to a great IT conference and hanging with the PowerShell team, we look forward to seeing you in Redmond.
If you can make it so, thanks Sydney.

Sydney Smith (SHE/HER)   10:09
Awesome.
Thank you, Jason.
Next up I I was just gonna mention that we did do a recently Apache release PS Resource guide.
This release contained a number of community bug fixes, so thank you so much to all the community members who contributed those.
Please keep them coming.
Keep the issues coming and keep the PR's coming.
Most of the PR's were actually related to fixes or using repositories other than the PowerShell gallery, so that was really helpful.
And then one other key things that also went into that patch release was a fix that we contributed related to constrained language mode.
So getting the module working better and constrained language mode.
So if you were having trouble with either of those types of scenarios, I can like the changelog and whatnot in a second or maybe somebody else will, but definitely check out that patch release.
I believe it's 1:05 that will go into the next preview release of partial 75 as well, and Amber, do you want to slot in right here?
I saw that you you wanted to do a talk a little bit about the performance update that you're working on as well.

Amber Erickson   11:18
Yeah, I actually had something long I wanted to go over, but I will.
Just super condense it to say that I've been working on.

Sydney Smith (SHE/HER)   11:27
You don't.
You can take your time if you wanna.
If you wanna take a few minutes, you can.
You can take your time on it.

Amber Erickson   11:33
OK, OK.
I was OK.

Sydney Smith (SHE/HER)   11:35
Yeah, either way.

Amber Erickson   11:37
I was just gonna say that I have been working on some performance updates for PS Resource get.
So for a little bit of context, PowerShell get V2 which is now in the process of kind of being semi deprecated in favor of PS resource.
Get it?
Had some significant performance issues that were never really a problem when people were installing a couple of modules.
One or two modules at a time.
Uh, but as we're starting to see more modules like Asian, AWS have dozens and dozens of dependencies, asy for example, has 88 dependencies.
We're starting to see these perf improvements increasingly become more common and just get exacerbated.
So with PowerShell get V2 we had these bottlenecks that really came about from how the request calls were being made.
They were very generic request calls that had huge payloads that were then filtered and processed.
Client side PS resource get improved upon this by rating much more specific queries to repositories to get back as little data as possible.
So we had a lot of perf improvements going from PowerShell Git V2 to PS resource get.
However, PS Resource get also had its own set of PERF optimizations that needed to be made.
I started working on a few of them.
There's possibly still a few more things left that we can do, but I'm just going to go over the couple of big things that we're done not yet released, but just that I'm working on umm so optimizations are adding concurrency and then deduplicating dependency calls.
So with the concurrency it was pretty straightforward.
I just used Nets parallel for each in both the find and install requests.
So install actually calls into find before actually making the install request, so this kind of doubly helped improve the performance for install.
Umm.
Write down.
OK.
Well, yeah, right now the parallelism really only happens if there's five calls being made.
So that's like 5 packages that are being found or installed.
So unless you are actually installing multiple packages, it's not really going to affect you, but it does.
Again, if you're installing large amounts.
So the other big optimization here was with handling dependencies.
Again, using AC as an example, it's got 88 dependencies.
Those dependencies all take another dependency, a sub dependency on AZ accounts.
So each dependencies so each request for a Z accounts previously was being made to the server to see first of all if that package exists and then to find the correct version and then later when we get to install only do we say OK we only actually need 1 version of this.
So to optimize this, we now have a dictionary that stores all the versions needed for a package.
As we're discovering the new dependencies, so essentially each time we see a new dependency version or version range, we resolve this version or version range with what already exists in the dictionary.
So that might be saying like we already have this version in the dictionary, so we're good or we now need to also search for this new version.
So let's add it to the dictionary and this helps us kind of continuously resolve the version as soon as we encounter them, so that we don't need to make these excess calls to find just to find out that we already have that information or that we don't need that information.
So really quickly, I'll kind of summarize everything.
I clocked some times yesterday for PS.
Sorry for PowerShell, get V2 specifically using version 225, the average time to install a Z and it's dependencies was about 15 minutes and 28 seconds.
I know people have complained about times that are much, much longer.
One of the times that I clocked was about 35 minutes for PS Resource get version 105105.
This one doesn't.
It's our latest release.
It doesn't have any perf updates.
The average time I should say it doesn't have the PERF updates that I've been working on.
The average time was about 6 minutes and 40 seconds, and then in PS resource get with these new concurrency and dependency updates which again has not been released yet, the average time was about a minute and 22 seconds.
So we're getting some huge perf improvements just with that alone.
And like I said, there's still more that can be made on top of that.
I just wanted to shout out Justin Grody and his module fast for kind of setting the bar for these perf updates.
His module, you know, it's a quick and dirty way to install packages, but also made us realize that there's a lot of room for improvement and PS resource get when I clocked the module fast install call, the average time was about 50 seconds.
So we're not quite there yet, but definitely getting a lot closer and just wanted to thank Justin for sharing your learnings with developing module fast and kind of giving us some more insight into that.
So that's pretty much it again, just like feel free to reach out on GitHub and the PS resource, get GitHub and leave us.
Any suggestions or comments that you have?

Sydney Smith (SHE/HER)   17:18
Awesome.

Amber Erickson   17:18
Thanks, honey.

Sydney Smith (SHE/HER)   17:18
Yeah.
Yeah.
No, thank you so much, Amber.
I really appreciate the explanation of everything you did and it's really cool to see the improvements.
I know make a big impact, especially if anyones using functions or situations like that or we can see time really matters.
Alrighty, move it along with the agenda.
I believe we have dog grow up next to talk about the PowerShell release.

Dongbo Wang   17:46
OK.
So we are going to have a a 75 preview Release Preview 3 the hopefully this week, if not early early next week.
So the servicing releases are, uh, pushed off to next month because we have we, we already have a few months without a preview release.
So that's our focus for this month.
That's about it.

Sydney Smith (SHE/HER)   18:12
Awesome.
Yeah.
So look forward to uh PowerShell 75 preview release coming hopefully soon, and then Sean, you're up next with the docs.

Sean Wheeler   18:23
Yeah.
So the big news is that, Umm, PowerShell 7.3 officially ended support on May 8th.
Umm the so the docs have been moved.
You can see in the drop down here 7.3 is no longer listed but if you click on previous versions it'll take you to this site and you can find the the reference here and the the release notes as well.
Umm.
And I've also done some update to this support lifecycle document.
Uh, as well as others.
The other, uh big activity has been in.
Uh DSC.
V3.
Umm.
Which Steve is going to talk more about, but check out the change log.
This talks about everything that was released in the latest preview.
Umm, lots going on in the DC version 3 space.
Lots of hard work by Mikey Lombardi on keeping these stocks up to date as things change.
And I things.
That's it.
Turn it back to you.

Sydney Smith (SHE/HER)   19:46
Awesome.
Thank you so much, John.
Always appreciate the door update.
Our next topic could be a quick one, but add summer is approaching.
But how PM team is thinking about if there's opportunity for us to create more educational content or videos around PowerShell and things we've created?
So if there are any topic uh that you think would be particularly useful or interesting, please do let us know.
Feel free to, you know, leave a comment here or you know any other means of contacting us.
Just something we're thinking about, so if you let us know if there's topics that you think would be really interesting or you would find useful rather than whether for yourself or for your colleagues or people you interact with.
Next up, we have a very exciting introduction.
We have our summer intern recently joined the team.
We always love our interns.
And so Leo would like to introduce yourself.

Leonardo Cobaleda   20:46
Yeah, well, The thing is, Jenny, I love everyone.
And Leo, Leonardo.
However, you guys wanna call me, it's fine.
I'll be the intern for this summer.
That about me. I'm from.
I have to say this every time, but I'm from the University of sorta.
If there's any theaters in the call today, but yeah, I'll be that's SH intern for this summer and a lot of cool things in the works.
Yeah.
Thank you.

Sydney Smith (SHE/HER)   21:10
Awesome.
And then see you're up with DSC.

Steve Lee (POWERSHELL HE/HIM)   21:14
Yep, and hopefully maybe towards the end of Leo's internship, he'll be able to present what he worked on at one of the community calls.
But what?
I'm gonna talk about is some changes that are happening in DC V3, so let me share my screen here.
Just some quick demos.
This one right?
Don't we just move teams back over here so I can see?
OK, so let me cover just a couple of quick things.
Again, this is all in the main branch right now, so it is not in the preview we release last month and our next preview we'll probably the week before psconf you if everything goes right.
Basically, one of the things one of the feedback we got from one of our partners, one of our internal first party partners is this concept of include.
So if you kind of think about it, if you have this large configuration for system and you have different teams working on, for example, maybe configuration for IIS maybe different for network storage, whatever, then if you want to have a single config with the old DSC today you would have a single large document that describes the whole state that you want and that might be a hard to maintain.
Or if these different teams are making changes, you might have merge conflicts and stuff like that.
So what include does is basically allow you to include another configuration as part of a config, so this is if you look on the left side and vice code, this is just a very small example and these can be nested so it can have include include stuff.
Whether that makes sense, I don't know.
But you can do it, but here basically this is a configuration file in YAML for DCP 3 and I'm just gonna include this configuration file osinbajo that happens to have parameters and then I'm gonna pass in this parameters file to it.
And if I were to show what's in here real quick, uh, should be this one.
But that's the parameters I actually wanted to go to this one.
OK so basically this configuration that's gonna get included.
It's just going to get the OS info with the OS family and has some and the parameters is is more for testing here but it shows like you can have Windows, Linux and macOS.
You can kind of ignore this.
This is also just a test case to ensure that expressions and functions can be used as part of parameters and have a comment here to explain that.
So if I were to actually run this, so do you see big?
Let's do a get.
You see examples and this was include, right?
So you see, if I go back to this file real quick, right?
So this just basically included it's at runtime we basically have a new type of resource and I'll go in a lot more of these details at Bischoff U in terms of if you want to participate and develop this stuff for example, I can expect maybe the community could do an import type resource that gets it from the Internet for example, and whether that's a good thing.
Again, I'll leave it up to and use the figure out, but anyways here you can see the type was included, but you can see the result was actually from the included configuration, right?
So this ramp?
Another cool thing that is in the main branch is what if?
And this is one that we definitely want to get some more feedback on.
So here's an example restri configuration that just have two a keys in registry values.
There you know.
Very simple.
So if I just do, do you see config?
Let's be test and I think I already have it here.
So right now.
If I'm going to skip a whole bunch of the stuff, but basically the key thing is execution type is a new thing, so this is actually running the configuration.
I have new information being added like how long did it take.
You can kind of see I got my 2 instances of the registry.
I want it desire state to be this umm this key to exist and also I have a retry value called hello which is a screen called World and tells me right now this thing does not exist it is not in desired state.
So previously you would just do a blind set and that would actually apply it, but it now there's a dash.
Dash, what if 4-W you can use and so if you run this, it looks very similar to the previous output, but instead I'll tell you like.
Here's the before state, meaning the current state.
Here's what would happen if I were to actually set it, and it tells you what properties would get changed.
So here if I remove this, what if and I run it.
This is actually going to change my registry and now if I do test.
And it should tell me I am in desired state.
Of course, now I have to use uh, either regedit or I can I I need to create a new YAML to remove these things, but anyways you can kind of see that the utility of this is like.
Now you can find out what is actually gonna change on your system via.
What if now the thing that isn't in place yet that will be coming later and it may not make it in time for the next preview, is getting resources themselves to participate.
Right now all the what if logic is within the DSC engine.
The only other thing I mentioned before I go is there's also a bunch of work happening with caching, so if you were to do DSC resource.
Of the source list right?
That shows you all the DCB 3 resources.
One of the changes we had is now in fact, there's a short and long version is this adapter.
So by default we don't enumerate all the adapted resources, because when we hit PowerShell it's it can take time.
But here you can see it only took a few few seconds.
This is all the ones I have on my particular system right now that work with partial 7, but this is definitely much faster than if I ran the old like get DSC resource which also is gonna emit a bunch of red text.
But I'm more work will happen here to make this faster.
We are not going to backport the caching changes to the PS DSC module.
We're kind of moving ahead with the PowerShell adapter for DCP 3, and we're gonna keep adding new capabilities there, which we cross platform.
And that's pretty much it.
Back to you, Sydney.

Sydney Smith (SHE/HER)   27:22
Awesome.
Thank you for the demo, Steve.
And then I think we're now through our teams addenda and on to community.
Christian, Are you ready with your PS Clippy?
No, I think we'll probably need to make you a presenter.
We're here, yeah.

Christian Ritter   27:42
So I want uh.
Thanks for having me.
You can hear me.

Sydney Smith (SHE/HER)   27:48
Yep, we can hear you.

Christian Ritter   27:49
Perfect.

Sydney Smith (SHE/HER)   27:49
I am making you a presenter right now, so you should be able to share your screen now.

Beckers, Dieter [GTSBE]   27:50
Yep.

Christian Ritter   28:04
Yeah. OK.
Perfect.
OK, so I want to talk today about a little bit of of my project, Pierre Sleepy, which is based off the awesome work from Justin Crowley who generated a module for the feedback providers.
So he's like creator and this PowerShell and I just took the chance and wanted to integrate this in some kind of way for general advices or cross platform advices into my current PowerShell session.
So that if whenever I do something there should not do there, I would be held that.
So basically the same thing.
The script analyzer does.
I want the same behaviour into get into my actual current running session, so then I came up OK.
Does this more or less?
No, some old fashioned AI away, so I thought Clippy would be proper name for this.
And yeah, I could demonstrate what I've achieved so far as all everything is still in the POG face.
So there's a lot of improvements which could be made to this, but yeah, to get your basic idea what is happening, there's some important module for a second.
So, and then what I've influenced the far is fit or left no equals check and also.
From it on the right.
So as you can see here we have some.
Yeah, not not bad.
Cold but cold, which could be better if we keep up to the standards.
So for example in this case, OK, try to get our child items off.
My 10th path was there object I asked for something extension of this text file and now I get a feedback.
That devices please filter on the left.
So basically OK what it shows me what have executed so far and what I should replace with this and the same goes for.
We'll go by various now, OK.
She's put no to left and also for formatting on the right, which was also an example Justin gave me so.
Uh, so here.
OK, please make sure that you if I'm not at the end.
So basically it's using.
Areas are already covered.
I'm planning to do some more, so I'm also.
Open for any PR request for rapporteur repository, which link that will show in a few moments.
Now if you like, we could just look look behind the curtain and say, OK, how is this implemented?
For example, fit left on here.
It's basically OK.
You have to perimeter inside of this function.
The script block in this case, for example here the context and then we say OK if there's any contacts right now, then I ask OK for the context for the common line AT.
I was OK.
Is there any match with Ralf tracked where or questionmark and it was the case I just make sure OK I also passed the ACT one more time.
I also OK if there's a common ask and this parameter contains and 30s per meter.
Then please give the advice.
So it's really Basic here on this.
In this case, the same goes also for formatting, as I only ask if there's any from a table whitelist all the aliases, and if that's the case and it's not at the end.
So the regular says it's not at the end.
Then also give me advice and afterwards this message will be written to the feedback item provider so that you can see this on the screen.
That's how Basic, thing behind this.
And as I said, when we just quickly had to the peers and one file, umm, like you said, it's all based on the Isenberg from Justin and.
You make sure that the experimental feature for the PSTN providers enabled, which is not the case.
OK, don't work.
Obviously and then adjusted right through all of the script blocks inside of the private function, we can see here and then just.
The thing is about the names, as you may have noticed, I just used the name of the feedback provider and the top as a comment, which is probably the best breakfast to do so.
But as a current, I'd say to the current POC state I didn't came up with any better solution yet.
So as mentioned twice, I'm open for pull requests or issues, so if you have any idea of what's about it, want to contribute to I would be happy about this.
So and then I'll basically say, OK, get all through all of the scripts gets to provide a name and then just register this feedback provider.
Andreas out the all feedback providers which the same name because maybe version is changed or something like that and afterwards we register them one more time and yeah currently it is basically the whole thing about it.
I just made up a little cool logo for this with designer Dot Microsoft.
Yeah.
And if you like, I can share also the GitHub repositories, one of PSP itself and one from Justin.
And yeah, there's probably Basic.
All have to show today.

Sydney Smith (SHE/HER)   34:25
Problem.
Thank you so much.
That was really great.
We really appreciate you coming to the call and showing that off.

Christian Ritter   34:28
You're welcome.

Sydney Smith (SHE/HER)   34:32
Love it.
Next up, we have Thomas Thomas.
I am also making you a presenter right now if you're ready to go.
So.

Thomas Nieto   34:43
Yep, I'm ready.

Sydney Smith (SHE/HER)   34:44
OK, cool.
Alright, I think you're sad.

Thomas Nieto   34:58
Alright.
Can you see my screen?

Sydney Smith (SHE/HER)   35:01
Yep.

Thomas Nieto   35:02
Alright, So what I'm demoing today is a small little module that I created called lock file.
What is goal is I've had a need over time is about having an always on script that's always running and it needs to be highly available, so I need it to run on multiple machines and but The thing is the catch is that it only can run on one host at a topping, it's only one instance of it.
So what this is is just a quick little module that I wrote for those scripts that need that.
You know always on capability and highly available state to have a log file, create that log file dialed at that log file is actually valid.
And also check across other nodes.
So what it will do?
Set log file.
He just creates a little Jason file with the.
Computer name, process ID and computer name and then you can easily just test it as well.
So this is on the same one.
So it's going to be valid and if that process ID were to go away, it would show us is false or if the process name would that same process ID is no longer valid as well and she'll false as well.
One of the other capabilities with it is you can pass it a.
Session app session.
So if you need to check the other computer to see if that that script is running there, you can do that.
So here's an example of like getting the log file configuration and then if the computer does not match the lock file that you're currently running on, then create a new session to that and since it's using a session, you can also use winrm SSH any of the other customer remoting protocols and then test that on there and they'll they'll validate and return true or false for you.
So it's just a quick little you know module that that does that for you and yeah, that's all I had to show off.

Sydney Smith (SHE/HER)   37:00
OK.
Well, thank you, Thomas.
We appreciate the demo.
Very cool.
James, looks like we have some time.
If you wanna jump in and do your demo.

James Brundage   37:11
Uh, sure, I will see how much the demo gods curse me.
I might need sharing permissions.
I might just be able to start dragging some stuff over.
Let me at least get rid of a few tabs first, because you know I have too many fantastic.

Sydney Smith (SHE/HER)   37:22
Bugs.
You're a presenter now, so you should be able to share, yeah.

James Brundage   37:31
OK.
So umm, for those that aren't aware, I have been having a lot of fun recently making uh PowerShell working containers and just to like let it pictures speak 1000 words.
This is PowerShell running on a container returning an SVG giving me graphics at whatever parameters I'd like to give it.
In this case, you're taking a pair of roses.
That's what these geometric shapes are called.
You're spinning one of them by an amount you're spitting couple or rotating couple of them by amount.
You're animating it over X number of seconds.
Let's make this a lot quicker so that it's a lot more pronounced.
So 4.8 seconds instead.
Infinite visual beauty out of PowerShell.
Ah, to kind of take you to another example here that I think just really pops.
Let's go ahead and do some repeated shapes.
And I want you to kind of note here, these are PowerShell scripts that are just getting parameters via get and catching them.
So we're going to go asked to repeated shapes that are roses and it will take a little bit of a SEC.
It's a lot of different repetitions, so let's drop the repeat count down to 40.
And again down to 20.
And just because I like to make things a little bit nicer, I'm going to go ahead and animate the opacity.
True or too.
So a hopefully some of these pictures are speaking 1000 words.
Let's go to some code here for a second, or at least see how the demigod's curse me.
Well, we've got running right now.
Is this Docker container that I had uploaded last night?
Let's go ahead and pull down new bits here.
This is a branch build, but this is the way all projects of mine are going to be working going forward.
Basically, if there is a reason to turn it into a container, I will and it will be a microservice and it will have every single build of every single branch that is successful downloadable as a package that you could try.
Uh, it's about done pulling that all back for me.
There we go.
Got a new image now to be fair, I did make him change or two, so we're gonna see if the demo gods curse me.
We're gonna go ahead and run. Detach.
And we're going to put this one on 8387.
Cool new image.
Let's take our other demo.
Just switch over a port.
Demigods are gonna be kind to me.
Yay, let's just prove the point more by, you know, upping the side count.
So I'm I'm pretty happy with the progress that this particular module has been making towards getting out the door and the nice aesthetic stuff I can do with it.
I could go head into PSG land for a bit, but I want.
I really wanted the communities feedback on was the layout of this.
So inside of the PS SVG directory that you see on the screen here you have basically the whole server or almost all of it.
Umm, first of all, does this seem like an obvious name of a method?
If I have a module and it has a script method called serve, that should be my server, right?
Am I talking crazy here or is that seem like a really obvious method name that would stick in everybody's brain?
Not getting feedback.
Moving on, I'm walking through.
What server does here?
First, we ignore favor icons because it's no longer the 90s.
Then we go ahead and basically, you know always return SVG cause PSG.
Make sure we've got this set up NPSG set up.
Check to see if we've cached or request.
If we did return the cached one, if the cached request was a bad status code, then let's give them a 404.
Speaking of which, let's go ahead and, you know, show that off for a second.
There we go.
Got a 404.
And great, now I gotta route the request.
Same sort of thing.
I wanna figure out basically whether I have a route already and if I do have one and it's status code then tough luck.
However, if it's command, then I basically just have to take whatever is in its query and splat it, and this will be turning into a script method too.
But then it's just framing in the results, so if it's like some L, go ahead, take the outer XML, frame it.
If it's a file info, make sure it's an SVG, then frame it and otherwise 404.
Tough luck to file info.
That's marked down great return the markdown inside of SVG if it's file on that's SVG great return SVG.
Otherwise, 404 are tough luck, so 100 odd lines.
Mostly redundant.
Mostly that would be the same for module and the module to serve up any content, and a Docker layer that can either have very animated 40 fours or infinite beauty.
Uh, questions.
Comments.
Thoughts.
I kind of rushed through these things a little too quickly.
I think so.
If people have things they want to talk about, I would love to hear more, but I'm just now looking back to chat and OK, so we got.
I wanna see this in a wasm container that is good.
I will be getting in classroom working probably in the next six months.
Uh, there's a couple of interesting projects on integrate with uh.
Dan Schroeder had said service.
What other apps use which is part of why I chose it.
Uh.
The approved verbs comment.
I would love if the approved verbs lips changed more often.
I would also love if PowerShell thought differently.
A little bit about verbs and nouns.
Umm, somebody had asked.
How am I running this inside of a container?
Does that mean somebody wants to see the Docker file?
Cause that's probably about all the explanation.
You'll need there.
So we grab the base image for Microsoft, wait till you're gonna install PSG, go tell your install some mail and packages and some modules.
Those go and install the packages.
Go copy the module into the right spot.
And set PowerShell as the shell cause we want it all to be in one layer and then create a profile and my module to the profile and install the other modules, add those to the profile and then say my profile also contains this micro service script.
And microservice script here is getting shorter by the day but it is basically.
Set yourself up for the first party lines.
Declare a couple of functions that their stuff used that we'll switch into a script method and eventually call serve.
It will become about four lines.
Uh, because it should basically be, if you're not already started, start me.
If not, serve me. Umm?
Somebody asked except for because you can.
What made you start this?
I will also accept because I can, but the practical reason that I've been interested in PowerShell and Wikification for a very long time is that a lot of average everyday users aren't going to ever step in front of a terminal.
So unless I can deliver a script and the clickable way to them, they're not going to appreciate the joy.
So web ification of PowerShell is very important to me over the long term because it's how one of the avenues to how we as a language expose ourselves to a larger community.
Uh, the other part of this, that's beyond because I can, is what I am jokingly calling the cheating in Spiderman principle.
That is to say, with any power comes responsibility.
And since I happen to have the technical prowess to be able to do these sorts of things, and the Community would benefit from having a lot of good examples and functionality that helps them, bootstrap container is rapidly I'm trying to build you all a fundamental basis so that we can all turn any PowerShell that we have into a web app like that.
I don't know.
Noise cancellation is capturing the snap, but like snap and yes, Azure functions, PowerShell, pode all support containers for more batteries included proach sort of type script in how is trying to basically sit atop all of those.
This is just one iteration of how you can actually publish it out, because two pipe script it doesn't particularly care, it's just metaprogramming or PowerShell producing other stuff.
In this case, part of the stuff that it's producing is a micro service.
But you know, to be fair for power script and this module itself.
The other thing that it was producing for PSG was the entire module before megabytes of code that it writes automatically from MDN, so that it ensures that it has a perfect copy of everything in the SVG standard, no matter what the SVG standard might ever include in the future.
So why I'm interested in doing all this stuff is that it is the great way for us to broaden our customer experience to everyone.
And why?
I'm personally interested in doing this stuff is no offense to anybody, but it's probably better me than the like.
I I'm gone through enough horrible technical rabbit holes that I can go down the web rabbit hole and come back with something.
Hopefully useful.
However, please let me know if this does not feel used fuel to you yet or you're not feeling the potential use.
There were a couple more points on start time and this is where I want to get a little braggy here for a second.
I'm let's go ahead and take this back up here.
And invoke rest method and pop that into a measure command.
Would really help if I didn't accidentally get rid of my eye there.
So.
This is a just astoundingly good time.
So you getting back from a web server especially for this complexity of response, you know you're getting back a response in basically quarter, second tops and this is doing a very long complicated response.
If I did something shorter, it would be faster, but you know, let's get a sense of that here for a second.
If I do this outer XML length, so that's returning about 1/4 of agony, and about 1/4 of a second, I'll live.
Uh, boot up times on this are about a second and average response times raw are between about 1/10 to 2/10 of a second, which is somewhere between in line with everybody else and faster.
So this should be a helpful way to get you all started in dockerizing your PowerShell.
If we dockerize our PowerShell, if we expose all of what we built in containers, then everything that we build can be usable to everybody on the face of the planet, not just everybody on the face of the eternal terminal.
And we can all see pretty pictures like these as the basis of a lot of richer websites in the future.
I feel like I've talked too much and I'm just kind of keeping up with chat.

Steve Lee (POWERSHELL HE/HIM)   50:21
Hey James, there is a I see there's one hand up right now from a guest didn't say a name.

James Brundage   50:28
Uh, sure, I was just trying to dismount and let somebody else talk for a second.
Anyway, who's got the hand up?

Steve Lee (POWERSHELL HE/HIM)   50:38
I don't know.
I only see guests.
This isn't verified.

James Brundage   50:42
But I'm happy to answer any other questions now if anybody's got them.

Steve Lee (POWERSHELL HE/HIM)   50:43
Uh, like hand is down.
Well, you're just you can respond to all the questions in chat as well.
I don't know if I'm sitting with other things to cover.

Sydney Smith (SHE/HER)   50:52
Yeah.
Yeah.
No, totally definitely appreciate the the demo James and yeah, tons of great discussion and comments in chat.
I think at this point we'll open up the floor just to any questions about anything we've made it through our agenda, so.
UM, it looks like Darren got a demo.
I'll pause for like another 30 seconds to to see if anybody had any other questions that they wanted to make sure that answered.
Today I'm team or by anyone, and if not, Darren, we'll we'll put you on deck for demo.

Justin Grote   51:34
Uh Darren is saying he may want to do a demo.

Sydney Smith (SHE/HER)   51:38
Yeah.
Umm OK.
I'm not seeing any more questions coming in to Darren.
I'm gonna elevate you to presenter mode.
All right, you should be good to go, Darren.

Darren Kattan   51:53
Alright, let's see.
Cool, alright, so this is a been working on for a while and basically what I've done here is I've changed how dynamic parameters work a while back had demoed the ability to sort of build a form using a param block on the web here, which I was very excited about, but I immediately ran into a bunch of issues because of the way the dynamic parameters work.
You don't have the ability to get to condition off the value of a dynamic parameter, so I made some changes and what I'm able to do now is do things like this.
I could say new radio parameter, so watch if I refresh this it'll refresh that page there alright and you know maybe this is a guest user.
Maybe this is not a guest user, right?
And then this git new user parameters a local function here is going to kind of abstract it out.
The generation of these parameters, so it's producing that first name, that last name, or making a mandatory right.
If I were to take mandatory out here and I were to refresh this and then it would no longer be a mandatory field actually have to refresh it.
Will look at this.
It's got to parse the param block on this script.
Alright, Ohi also have to save that before I do.
That makes sense, right?
And she knows that decides to get his act together, and then this is going to no longer be mandatory.
But it's probably easier if I just went ahead and just brought those fields in here.
So let's say I wanted to do a new parameter and I wanted to name my first name whatever it needs to be, and maybe I don't make that one mandatory.
All right.
And maybe I'll just return here just so we could watch the form get sort of built out, alright and maybe I'll make that first name.
I make it mandatory here.
OK.
And I'll go ahead and do the same thing, but now I'll do the last name right.
And what's neat about this is we're able to actually condition off that.
So I might say I alright if dollar sign first name like I don't know Darren then of my do new help text parameter name.
Are you sure?
Whatever.
And then the help message here might be like, Are you sure your name is Darren?
Right.
So let's go ahead and refresh that and see what it does.
Alright, so I'll go ahead and put Darren here, alright and then it goes.
There you go.
So it's pretty neat because there's been a lot of people have tried to make that dynamic parameters work like this, but due to the way the underlying binder works, it only captures the the the parameters like in bulk right?
So at best you would get just, you know you could only condition off the value of a static parameter, right?
So whatever essentially done is I'm a I've I've I'm doing it Modi.
Modi did put in a PR for this, but the way I'm doing it is I inject a hook into the git dynamic parameters.
A.
A function right here.
So basically saying on PS script parameters get dynamic parameters and inside of here I basically recreate some of the same logic that exists in that method.
But the real trick is that I'm passing the parent parameter binder into my new parameter commandlet.
So if you look at what's new parameter command here.
Alright, so here's all the options.
You have to define that parameter is just like a wrapper around the new runtime defined parameter, and then in the end processing block.
What I do is I grab that that parameter binder called the incremental parameter binder and here I just call the bind runtime defined parameter method on that and then essentially I'm just calling back into the bind dynamic parameters methods that exist on the inter type and then I have uh and then I set a variable to.
So after I bind it and then take the bound value and I assign it to a variable that's and.
So what that does is it kind of works like out out variable or something like that where you just give it like the string name and then a variable with that name pops out.
So you can immediately use like the dollar sign name after so it works something like that, and that's how you're able to get the value of it and then use it immediately on the next line of code.
So I think this is gonna enable a lot of use cases, kind of like what James said.
You really a lot of the stuff that I see guys do.
You're right in PowerShell, but being able to get like really good inputs and sanitize those inputs sometimes requires reaching out into different systems and validating that there's not gonna be a name collision when somebody types in that name.
So I could have just as easily put a validate script that reaches in and checks Azure AD to make sure that there's not a colliding name right within this script block.
Here and so by the time it goes to execute, we know that everything is clean and the data is ready to go.
So yeah, that's what I've been working on.

Sydney Smith (SHE/HER)   57:05
Great.
Thank you so much, Karen.
Thanks for jumping in at the last minute.

Darren Kattan   57:09
Yeah.

Sydney Smith (SHE/HER)   57:11
Awesome.
Thank you everyone.
We have just a few minutes left.
Umm, Chris. Ohh.
I think that was the actual hand raise.
I'll pause once more to see if there are any final comments or questions before we close out today's call.

James Brundage   57:29
I mean, I'm just gonna be a little bit of a shameless pluggy friend here for a second and say, like, Darren was a little bit shy on his own product here.
Emmy Bot is an interesting and good offering, especially for MSP's that are trying to like get their PowerShell out to a broader audience.
And I'm sure there's more of the thing you could demo if you wanted to spend the time.
I'm sure a lot of other people would also be interested in seeing this, just like you know nobody giving you a commercial invite to do a demo.
And it seemed like you've spent all your time on the feature that you did.
That was really cool.
Rather than the product around the feature, which I personally think is also cool, but hey, that's just me and I'm not trying to like, you know, encourage you to shill too much.

Darren Kattan   58:09
I appreciate that I'm not trying to be a shell.

James Brundage   58:14
But just like if you wanted to chill a little more, I think like maybe some people might benefit.

Darren Kattan   58:22
I mean, I've got.
Yeah, I've done all kinds of stuff with the web interface like I've overridden the right progress messages to produce progress bars.
It works in a nested format, so essentially just kind of bringing PowerShell to the web, but it kind of has the same sort of need as James with a container thing where I need a safe place where I can execute PowerShell and cuz I'm allowing as you can see in here people to execute PowerShell, but I need a place to do it that's safe and being able to do in a container is great, but containers are still slow to start up.
That's why I'm interested in getting it into wasm.
Because you could just get that instance start time, right?
You could capture the memory after it's initialized and then just put that in a cloud flare worker and just go.
That's why that's why it really wanna see this go.

James Brundage   59:04
There I am realizing I didn't show the kind of crazy cool container demo that talks to security.
Would everybody like to see this really quickly just to understand a couple more things?

Darren Kattan   59:17
Umm.

Sydney Smith (SHE/HER)   59:18
We finish it in a couple of minutes super fast and you wanna save it for next month.

James Brundage   59:21
I think so.
I think so.
I think I can do it in 2 minutes.

Sydney Smith (SHE/HER)   59:26
Alright, 2 minutes.

James Brundage   59:27
We'll see two minutes on the clock.

Sydney Smith (SHE/HER)   59:27
You.
You got it.
OK. OK.

James Brundage   59:30
Alright, I'm gonna do the screen share.
OK, so here we're going to get extra crazy.
We're going to go Docker run, we're going to publish a different port going to do interactive.
TTY.
28388.
OK.
Yeah, we've got a new server up and running.
Cool.
In 321 homana, there we are.
I'm gonna control C so I have my terminal here in my container.
First thing I'm gonna do is I'm gonna do NB look how little information is there.
If you broke into my container, congratulations, you won nothing.
There are no secrets.
Have fun.
But that's not even the really crazy part.
That's get job.
This is the PS node job that is powering the web server and one of the coolest things that I can do with it is I can change its entire action.
So I can say our job is get process ID hit.
And now I'm going to go to you 8388 and it won't matter what I pass it, because it's just going to return me, my pit or my process.
But this is where we get crazy.
So now that I've changed the action once, let's do something that's going to terrify everybody.
This seems like a really bad idea, right?
It didn't die.
When you're running in a container, you're running in an ultra secure process, and so even though my web server is telling itself to die every single time I'm clicking control R, it just won't freaking die.
Containers are great. What?

Darren Kattan   1:01:31
What if you add dash force?
But if you ask add dash force.

James Brundage   1:01:36
Let's try.
I don't think it'll die then either, but you know, I could always be pleasantly unsurprised.

Darren Kattan   1:01:44
Oops.

James Brundage   1:01:45
Parameter cannot be found with dashboards, so parameter binding issue there isn't.

Darren Kattan   1:01:47
Ah, there you go.
You got me.
You got me.

James Brundage   1:01:50
Stuff for us, I stash or another isn't.
Get processed dash force.
So it didn't die because there's not a get process stash force.

Darren Kattan   1:01:54
Uh, OK, alright, I'll see.

James Brundage   1:01:58
Let's go ahead and add dash force here.
Still didn't die.

Darren Kattan   1:02:03
OK.
Umm yeah, it's mostly about the cold start time.

James Brundage   1:02:05
Still didn't die.
I hear that and also I can just as easily clone a repository into this existing container and start serving your request from there.
I don't have to care about cold starting.

Darren Kattan   1:02:21
Sure.

James Brundage   1:02:21
I have to care about the security.

Darren Kattan   1:02:22
But then you're then you have like, you know, cross pollination on the file system.
If people are putting stuff on there, seeing each other stuff, you know, connecting to the name pipes that get exposed from the debugger or things like that.

James Brundage   1:02:29
I guess let's do one more thing here, which is like, you know, let's see if this file stays.
Test dot text and I'm an exit my container.
And I'm gonna go run new one.
And wait for my thing.
Control C dur passed up text.
Ohh it's not there because every time I come to new container.

Darren Kattan   1:02:57
Right.
But how long did it take for you restart the container?

James Brundage   1:03:02
A second or two that most customers would not notice honestly.

Darren Kattan   1:03:05
Yeah.
Yeah. OK.

James Brundage   1:03:08
And you know, I'm not trying to rain on parades here.
I'm just trying to say like Docker is a very secure environment to be running in and for me that's worth a second or two.
So that was my I promised I'd be less than two minutes and I think I still am keeping that promise.
I'm stopping sharing now, but Docker is so bloody secure that if you try to kill the process you're running in, it just won't die.

Sydney Smith (SHE/HER)   1:03:28
OK, perfect.

James Brundage   1:03:34
And I love that about Docker.
I absolutely love the Docker ecosystem.
It is fantastic.
Oh yeah, one more bonus point, every single cloud you can upload your Docker image to and get hosted of course, including assures Kubernetes service.

Sydney Smith (SHE/HER)   1:03:51
Alrighty.
Well, thank you, James.
Thank you, Darren, and thank you everyone for being here and for presenting today and hope to see you next month at our June Community Hall.
It would be really great to get some new folks demoing as well as we love our folks who demo often and we really do appreciate you all, but I challenge each of you on the call today to to think about what maybe you would like to show next month.
Uh, but thanks again for being here and have a great rest of your day.

James Brundage   1:04:30
Thanks.
