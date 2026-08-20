PPowerShellOpenSSH Community Call-20260820_092930-Meeting Recording
August 20, 2026, 4:29PM
49m 14s

Jason Helmick started transcription

All right, folks. Good morning. Let's get started.
Hello, good morning and good afternoon to, or good evening. I forgot the good evening part. Welcome to the August 2026 PowerShell Community Call where everyone is welcome to join and discuss the PowerShell project and its development. Hey, look, we've got an amazing agenda this time. Post it in the meeting chat and a link to the meeting discussion. Please feel free.
Please feel free to do what? Oh, post questions in the discussion issue that we have there, or just right here in the chat and things like that. And please make sure to check out and follow the code of conduct. Once again, ladies and gentlemen, it's August. I'd like to say happy 20 years to the release of PowerShell. Twenty years ago.
in November is when it got released. And so once again, happy 20 years. And on this August, a 20 year celebration of PowerShell, I was asked to figure out a way to use the word fjords in a sentence. Well, I haven't figured it out yet, but
A big nod and appreciation to all the MVPs and the community folks in Sweden for that one. So, to our agenda today, we've got an action-packed agenda. We brought some demos along with us as well, and so I'm just going to go ahead and kind of go down the list. And first up is we've done some release updates to PowerShell, and I...
I see Justin here. Justin, would you want to say good morning and tell us a little bit about those release updates?

Justin Chung   3:47
Sure, Jason. Good morning, everyone. Can everyone hear me? Can you hear me, Jason?

Justin Grote   3:52
Yep, we can hear you, Justin.

Jason Helmick   3:53
Yep, sure can. You're all good.

Justin Chung   3:54
All right, perfect. Yeah, so PowerShell releases, I think we got 7419, 7510, 765 available. And as many of you might have noticed, our release tempo has increased and we're getting better at releasing every month. And I, what...
I think a noticeable improvement here that we're making. It's not this time around, but we started testing SIM shipping with.NET. So we're going to get PowerShell releases even earlier, hopefully soon, maybe.

Jason Helmick   4:29
Soon, I love that. That's good. I love that. Soon, soon.

Justin Chung   4:33
We'll see when that happens. But we, but this, these three releases made a definite step towards that goal.
A lot of good news there. Yeah, that's, I think that's it for me. Should I direct it to Steve?

Jason Helmick   4:49
That's perfect, Justin. Thank you so much for the update. And yeah, we're looking, folks, we've been working on the shipping cadence and getting close to SIM shipping. So big appreciation to the maintenance crew and all the folks that are doing that work. Thanks, Justin. I know we're going to see you again in a minute. Now,

Justin Chung   4:50
Yeah.
Yeah.

Jason Helmick   5:08
onto DSC version 3. Steve, do you want to talk about the upcoming GIA?

Steve Lee (POWERSHELL)   5:14
Yes, so the current plan is that we're going to ship 3.3 RC2, hopefully today, but if we can't make it today, it'll be probably Monday because we don't want to ship on Friday. And the main changes from RC1 and RC2 is we got a bunch of good feedback from a particular user on the GitHub repo around the new Windows Firewall resource.
So they wanted to be able to make sure they can use that firewall resource for both auditing and for setting rules for firewall. So they found they had some really good use cases, and that basically required making some design changes in the resource, and we're learning from that. So that's really good stuff.
The other thing I want to make sure people are aware of with the 3.3RC1 release, which is already out, we also started publishing to Microsoft packages, packages.microsoft.com. So these are Linux packages, so these are RPMs and devs. So if you're on Linux, you're trying to use DSCv3, you should pick it up from packages.microsoft.com going forward, or you can always of course get the targzip.
packages from our GitHub repo. The other thing I want to bring up real quick, and I can share my screen, is as the as while 33 heads source GA, hopefully early September, early to mid-september, we are already talking about 34 within the DSC working group.
So if you go to the repo, which should be on the screen now, we already have a bunch of issues that we've tagged as milestone 3-4 approved. So these are all the things that are currently being planned to be invested into as part of the 3-4 release. Again, we don't have a specific.
Date when 34 gets shipped, but roughly we're roughly aligned with about six months for new versions. So some things I'll just call out here, there's some fundamental stuff like you can see all these things tagged as open telemetry. This should really hopefully address some of the tracing issues that we've had open over time.
We also have stuff like this new thing called actions, so this again is based off of actual real user feedback, where, for example, we have some internal customer that was trying to install a particular application, and for them, one of the things they need to do is, if they need to install a new version, they actually...
because of the way Windows works, they might have to actually kill that executable. So this would be an action, right? So we don't want to have people produce these, let's call them artificial resources, which should really be more about state. And if there are actions that need to be taking, then like killing a process, potentially from rebooting the system, then we'll have a new type of.
I don't wanna overlook the word resource, but a new type of thing called actions that can be supported. There's also things around DSC as a SSH subsystem, and also I think the ORAS stuff is supposed to be in here, but I don't see it, but we're starting to think about how do we now.
expand DSC so you can do more management at scale. So rather than doing one machine to one machine, you can do one to many and using leveraging SSH and also or as OCI type of stuff is the direction we're heading. You can also see there's a Python adapter. There's already a PR draft PR up.
from some of our working group members from the community. So hopefully once that is published, then we can start seeing more Linux specific resources written in Python.
And there's some signing stuff as well, so...
Meaning that you can require that the resource manifest are signed. You can also require that the configurations are signed, things like that. So if there are any things that you need for your deployment, if you're starting to use TCS v3 and you'd like to see it in three four, I would say either open an issue or if there's an existing issue.
At mentioned, Jason might be the best way to go, and he can bring it up to the working group's attention to see if it's going to make it on three, four. If it doesn't make it in three, four, it can always make it in three, five, three, six, whatever the case maybe, so don't worry about that.
That's pretty much all I got. Yeah.

Jason Helmick   9:20
And Steve, did you, did you have a, I have a reminder here, did you want to mention Thomas Nieto's RFC? I think I have the link.

Steve Lee (POWERSHELL)   9:29
So his RFC is on the Python adapter, but Heist actually reminded me to mention a different RFC.
Let me show my screen again.
Because I'm also hoping, well, I think the PowerShell community, which, let's see, open DSC side is going to do the actual work, but where is it? This one. So I did create this RFC for the PowerShell authoring experience for writing in a configuration. So this is kind of like the replacement for the configuration keyword.
There's a bunch of text in here, but you can also skip to, there's two different experiences. One will probably happen sooner before the other one, but basically how do we enable like both tab completion, but also consistency with how you may write PowerShell scripts and it will generate A resulting.
DC compatible configuration file, probably a JSON. I would love to get people to look at this and give their feedback, especially for those who have prior experience working with PSDSC V2 and the configuration keyword. And then you can kind of also see a potential Pester-like experience that would be built on top of the prior experience, which is really about.NET types and stuff.
So do add your own comments in here. This is a work in progress. That's why it's in draft, but although it's in draft, we do it. I do want people to provide their feedback now, right?

Jason Helmick   11:01
Outstanding. Thank you. Folks, before we move on, I just want to, Ryan Yates, thank you so much for putting that in. So Ryan typed in, it's 10 years being open source as of this past Tuesday. That was a date I was totally unaware of. So I'm going to turn to my friend Sean and see if there's some way we can work with this to add it to our normal celebration list. So thanks for pointing that out.
Up next, I'm so excited about this. Andrew, Andrew Pla, the famous Andrew Pla from the PowerShell Podcast is here to talk to us about some news and stuff and community. So Andrew, thanks for being here. Take it away, sir.

Andrew Pla   11:35
Thank you. I saw there's quite the agenda, so I will try and be efficient. Hello. There is this thing called the PowerShell Podcast. It comes out every single week. Been doing it for a while. There's some great episodes out there recently. I think Jake Hildreth, our boy in the chat here, Fred's another great recent episode. But what I'm looking for is other folks who would like to join, who haven't been on before.
So if you know anybody, please suggest it. I'm at Andrew Pla everywhere. Discord and LinkedIn are my preferred ways to hear from people. I'd also, I'm exploring something called like more of a conversation where it's me and multiple people, and maybe there's some topics you'd like to see addressed where it's not just a one-to-one interview. Maybe we're talking about different things. Same thing kind of applies for PowerShell Wednesday, which is a weekly stream that I do. Think of it like a conference talk, but kind of more interactive and lower.
open for some first-time speakers there or folks who haven't been on for quite a while. It doesn't have to be like a big polish, it can be just like a little demo, maybe we can code something together. We're here to help you. And along the lines of sort of helping people get involved in the community, branching out to the slightly more open source side of things, which I guess is good timing-wise.
Gilbert, make sure to get over here right now. But PowerShell, we have this new stewardship program for open source projects. So if you've been around PowerShell for a long time, you've probably seen some great projects pop up, and they've received a lot of passion and support. And as time goes on, like any kind of project, maybe the maintainer moves on or has other priorities.
and the plan that we have and we have been going forward with it. Gilbert, are you here? Are you ready to speak to it, my friend? Or do you want me to just roll?

Gilbert Sanchez   13:04
Yep, I'm here. Yeah, so we announced this back at Summit this year that we are starting this revival program. So we've set up sort of a governance model to try to figure out how to best continue to support community modules that are, you know, are super important to the

Andrew Pla   13:05
Yeah, what are we up to?

Gilbert Sanchez   13:23
to the community that the maintainer maybe is busy and they can't get to it, or maybe they've moved on from the language, or life happens, right? So, we were able to bring on PS Depends. Plaster is an example of something like that. Plaster saw a new version, PS Depend V.
came out the last two weeks, if not sooner potentially. And that's gone well. Andrew recently is going to start stewarding the PS Cohens module. And we have another one, PS Build Helber, which is also going to get stewarded. But this is essentially a call out to folks to, if you
see modules you'd like to get kind of revived and you know where you want to help participate. We're looking for stewards, so people who want to take on the work of managing. Not steward to steward. And so a steward is just essentially like a project manager. They don't have to be the person who does all the work.
And so, you know, don't worry if you're not super technical and you're scared of maybe getting into that to the weeds of the code. If you want to just, you know, help manage or, you know, and again, there's also other stewards who are there to support you. So we're trying to make sure we create, you know, all the
playbooks and templates to make things as easy as possible for everyone. And we're also doing things like identifying easy to solve or good tasks for new users or new contributors. So yeah, if you have questions or anything, just feel free to reach out.
A steward? Sorry, go ahead.

Jason Helmick   15:11
And Gilbert, Gilbert, this sounds, this, this is, this is, this sounds fantastic. This is great news and I can see a lot of benefit here. So if I wanted to sign up, who do I reach out to?

Gilbert Sanchez   15:22
So we actually, if you go to PowerShell, github.com slash PowerShell org, we have a link on the Remy, which I'll throw in right now. On there, we kind of talk about sort of the bar that we're looking for for new modules. But you can kind of see like what we expect from
stewards, what we expect from maintainers, how the entire governance is kind of spelled out there to make sure that it's very clear. But the main thing is that we want to give people support, right? We just want to make sure that people feel supported and are successful.

Jason Helmick   15:56
That's great. That's fantastic. Thank you. And yes, I'm looking forward to the link.
Is that it, Andrew? You got more?

Andrew Pla   16:05
That's it.

Jason Helmick   16:06
That's it. Well, thank you very much, sir. I hope to see you more in the future. Thanks for your time. So folks, as we move forward, I kind of, well, I'll just say it. I organized the agenda in an incorrect order. So what I'd like to do is before we get to the demos, and we got about 3 demos, and we're going to do those for you. Before we get there, I'd like to turn to Anam and Sean and see if we can

Andrew Pla   16:10
Absolutely. Thank you.

Jason Helmick   16:28
do their thing. So Anam, can we talk about the PMC package update for, and Sean keeps correct, Ubuntu? I keep spelling it wrong.

Anam Navied   16:38
Yeah, hey everyone. Glad to be here again. I just had a quick announcement for packages.microsoft.com or PMC. So with the upcoming 770 preview for release, PowerShell will now have a package published to the Ubuntu 2604 repo.
So we'll support Ubuntu 2604 going forward from the PowerShell 770 preview releases and onward. So yeah, just that short and sweet update. Thanks, Jason.

Jason Helmick   17:09
That's perfect. Thanks, Anam. And Sean, while we're here, do you want to give us a docs update? And oh, did you see that thing by Ryan Yates? Can we write that down?
You're on mute, my friend.

Sean Wheeler   17:26
Yeah, I did see Ryan's note. I'm not sure where we'd put that, but we can chat. So let me, not a whole lot to talk about new content and docs, but in the past month, we've been focused on updating the release documentation for the new releases.
Not a lot of big changes there. Done more cleanup on the script analyzer rules. I'm really pleased with how things are going there. Every rule now has a section that talks about configuration. Even if the rule is not configurable,
There's a configuration section that tells you that.
And one of the bigger changes in 7.7 was the new GUID command now creates version 7 format UUIDs. Technically, this is a breaking change, but in practice, it shouldn't affect anyone.
The benefit of version 7 UUIDs is that they are time sortable, so they make good values for indexes and databases and things like that. So it's a nice update there. Previously, in older versions, still create the
version 4 IDs, which are completely random, but because they're completely random, they're not good, sortable, indexable entities. So again, call out a couple of contributors.
We appreciate quality contributions from the community and encourage more. And
I've been working on a lot of other feedback channels, improving the docs, so it's just been incremental updates. Not a lot new to talk about. Though I just did an overhaul of the...
What is it about?
Um...
The PowerShell Config.
Where?
clearer information about.
the scope of the settings in the Config file, and that's important to kind of get this stuff clarified because this is an area that will be changing.
in the next few releases as we start to look at moving things like where your data directory is for
your modules and profile scripts and all that, get it out of documents folder and make it relocatable. So anyway, that's all I had.

Jason Helmick   20:42
Well, that's awesome, Sean. And you all saw it, it's been recorded. He kind of blew me off on that thing about, you know, the, yeah, well, we'll figure it out. So, folks, let's go ahead and move to the demo phase, and Justin's going to come back, and he's got a couple of interesting demos that I've been really fascinated to see. Customizable, user-scoped,
content path and on PowerShell configuration with MSIX. So Justin, you want to take charge? Take it away.

Justin Chung   21:12
Yeah, that's a perfect place to leave off of where Sean was talking about. Customizing where our user scope modules can live. And let me see, I have like 3 different PowerShell instances open and I want to share the window only. Let's see if I get the right one.

Jason Helmick   21:16
Yeah.

Justin Chung   21:32
Okay, I think this is the right one. Can everyone see my screen?

Jason Helmick   21:34
Yay!

Justin Chung   21:36
So what I'm doing, this is a custom built local PowerShell that I built off of the PR that's open currently on GitHub. It's been there for a while and we've been deliberating like what the best way to approach this is. And I just want to show off.
some of the features that I've been working on in this effort to.
give power to the users to have a say in whether user scoped PowerShell contents line. So we are introducing a...
commandlet here, get PS content path. It's currently only user scope, so this always will always return the user scope. I think there's plans to maybe expand on that further. But here it is.
Directory info object, and it has two important fields: full name and Config file. The directory info object, unfortunately, doesn't really show the full name very well in that table, so I'll just show it here. So, right now, my PS content path.
Is set to...
App data local PowerShell. Now, the code change routes everything, the scripts, the modules, the profile, and the help for user scope things all through this one location. And that location lives in the...
Config file.
The Config file, as you can see, that it is in also in that area. That's going to be a manual move. I moved it manually over there and currently my setup is, or I moved it manually there for the demo. My current setup is not this yet, but...
Um...
Yeah, let's see. Let's see what that content looks like.
Ohh.
Get content, something like that. So right now that Config file looks like this in App Data and it is showing an execution user scoped execution policy remote signed and experimental features will be there. And this is where the PS user content path.
Is going to be read from from that Config file, so we also are introducing a...
Set PS content path.
Let's see.
I want to do...
My documents.
I think that's where it lived before.
Maybe like some somewhere like here.
Do yes to all.
And I'll need to restart PowerShell perfect.
And, upon restarting PowerShell.
I'm also using a custom-built PS read line here. It's a little finicky switching between the modules because this custom-built PS read line is not in both places.
And now that user PS content path has changed, and you, and actually this error kind of shows where this it changed the location where the modules are being read from, so...
I'm excited to see this go in. I think it's in a good place. It's...
Yeah, there's a lot of nitty-gritty details here that I, it would be hard to demonstrate and hard to articulate here on this meeting, but yeah, feel free to.
discuss and raise questions in the PR. And yeah, I see some hands up. Can we, let's, Justin?

Jason Helmick   25:42
Yeah.

Justin Chung   25:43
Yeah, or should we save that for her? Yeah.

Justin Grote   25:44
And.
I can, since you have the issue up, I can look there. I was just wondering, you know, obviously you have to save this setting somewhere. So when you do this, is the new location, this app data local config file, is this just where the executable is now going to look for? Is that PowerShell.config.json? Is that like the hard-coded path where it finds where to get the new update?

Justin Chung   26:06
Yeah, that was a that was a tricky thing I wanted to get into, but I was kind of afraid, but good thing you brought it up, so the...
We've made it a tiered, like what is it, a fallback system. So it first looks, with this change, it will first look in this new location. And of course it won't be there because we don't migrate it for you. And then it will look in the OneDrive location, this path here.
And then if it doesn't find that, it'll fall back to, and if your config also isn't there, then it'll fall back to the defaults, which we are changing to app data.
And, but most likely, your your Config should be there.

Jason Helmick   26:53
Folks, feel free to type in questions as well so that Justin can answer. He's got another demo he's going to go do. But feel free to type in questions on this. What I really, really like is how demos sometimes, you know, show the cool stuff. So you obviously now know we're working on something with SQLite. Thanks, Ryan.

Justin Chung   26:55
Yeah.

Justin Grote   26:56
Yeah, and.

Justin Chung   27:02
Yes.
Yeah.

James Brundage   27:16
Real quickly, I would also recommend push and pop as verbs for this content path. And I was going to recommend that before you started talking about using like an internal stack to represent the fallback mechanism. Now I really think push and pop are appropriate verbs for that scenario.
The case that I really want them for is get work trees, so I can push and pop into the work tree directory and just import my modules normally. And I would have to remember which work tree I'm in, and to go to the right work tree root, and to import the module from there. Rant over, have fun. Next demo, please. Thank you.

Justin Grote   27:35
So, yeah.

Justin Chung   27:54
Thanks for the feedback, James. Yeah.

Jason Helmick   27:55
Yeah, that's interesting, James. Thanks.
Go ahead, Justin. Go ahead and go with your next one.

Justin Chung   28:04
Oh, okay. Yeah. I just was not seeing the hands down. Okay, so the next one.
Is.
Let's see if I could get the right one first. Okay, I'm not sure why, but this one is a little... Okay. Can everyone see my screen? It should be an admin. I'm running as admin here. So if you see the location of where I am, this is...
an MSIX installation of PowerShell.
And the amongst many things that are broken with MSIX, it's PS Home, which is the...
Machine-wide files are not configurable.
So I'm going to demonstrate that. I think maybe some people don't know. Oops.
Ohh, no.
Okay.
So if I do this command, set execution policy, local machine, if I just try to do that.
It.
Okay, there we go.
So I'm getting denied access to the path there, even though I'm running as an admin. So this is one of the problems we're trying to solve with this PR. I'll switch to the other screen, which is a custom-built PowerShell yet again for local machine only here.
So, here...
These are kind of acting funny to me.
Okay.
Now, here we have, let's see, get execution policy.
I could do that and get execution policy list.
Now, I should have done get execution policy before, but it was set to remote signed, and now I just set it to bypass, and it doesn't say, it doesn't give me that error of I don't have access to it, and the reason why, and the way we get around this is by changing also, again, we're changing the location.

Jason Helmick   30:10
Yeah.

Justin Chung   30:25
of the Config from PS Home to program data. There was much deliberation on where to actually set this configuration file, and I think program data is the right way. I think we're kind of settling down on that.
Yeah, this is an exciting change to go in. And with this, once we kind of carve out our space and program data for all the MSIX machine-wide settings files.
then a lot of those MSIX problems will be solved, hopefully.
Yes, are there any questions for this? Justin, is your hand raised again or?

Jason Helmick   31:09
I think it was leftover. That's awesome.

Justin Grote   31:09
Sorry, thank you.

Justin Chung   31:10
Okay, yeah, definitely.
Um, I maybe I could show you where this lies. Will this work? No, Yifan.
Um...
See, yeah, then Config.
Ugh.
Maybe.
Well, anyway...
I think that's, yeah, that is all for me. It lies in the program data under the package family name. It's going to be channel specific, so it's going to be, LTS is going to have their own channel, LTS channel, system-wide configs, preview is going to have its own, and stable is going to have its own.
So I think this solves it nicely for the time being. And yeah, the PR is open for this as well. I'll post the PR in the chat so that everyone can look at it.

Jason Helmick   32:11
That's awesome. Thanks, Justin. That is, that's fantastic. So folks, obviously our team is doing some work around the MSIX. There are also several other teams doing work around MSIX. We have a master MSIX tracking issue that's available on GitHub. I'll go find the link and put it back in here. I think I put it in a couple of times.

Justin Chung   32:12
Right, that's it.

Jason Helmick   32:31
already. And so Justin is demonstrating some of the things that we've been working on to solve for PowerShell with MSIX. We look forward to your inputs on the PRs and the future work. Thanks, Justin. It's looking great. Really interested in your SQLite thing. Maybe you can come back sometime and talk to us about that.

Justin Chung   32:50
Oh yeah, that one's, I think, a much more fun demo. Yeah, that'll be fun. Yeah.

Jason Helmick   32:53
There you go. There you go. Thank you. And with that, we've got one more demo left, and I think that's with Amber on PS resource get concurrency improvements. Is that right?

Amber Erickson   33:09
Hi, yes, that is correct. Let me share my screen.
Okay, awesome. So I'm just going to talk a little bit about some of the concurrency improvements that we've been working on in PSGallery resource get for the last couple of months. Right now with the latest release, 130 preview one, we do have like a small subset of the concurrency changes that are complete.
And that specifically is improvements or concurrency for package dependencies, specifically for V2 servers. So that would be, you know, primarily the PowerShell Gallery, or you can interact with some other servers with the V2 APIs. So right now that is out again in 130 preview one.
But since then, we've actually made improvements so that concurrency applies to all servers and that parent packages also both find and install concurrently. So I wanted to show some times for that and talk just a little bit about that.
So this is our last stable release, 111. And what I've done here is just like chose a couple of commands. Find, for example, for Azure, just because Azure is a single package that has a ton of dependencies. I think it's like
somewhere just over 100 right now. And with our latest, or sorry, yeah, our latest stable, finding the AZ module and the dependencies took almost one minute. Installing all of the AZ modules, parent package and dependencies took just over 3 minutes, 3 minutes and 32 seconds.
With these new concurrency improvements, we got down so that finding the AZ module and the dependencies takes like almost 6 seconds, just under 6 seconds, and installing all of these modules takes about a minute and 17 seconds.
So that's been a huge improvement that we're really excited about. Again, that's kind of just one parent package. So this is something that you can kind of like test out with the 130 preview one release. Since then, we've added the concurrency to parent packages. So this is just one example of.
some popular modules, finding these modules now, some of these don't have any dependencies and some of these like have a few dependencies. That took about 3 seconds, a little over 3 seconds.
And with the changes, that took like roughly half a second. So that's been a bit pretty big improvement as well. Installing all of those modules took almost 35 seconds previously with 111. Not as big of a perf improvement here, but it did take a little bit.
less. So really excited about these changes. Obviously, it's going to impact like some situations a lot more than others. And everything is still kind of contingent on server side performance as well. I did kind of want to run tests so that I could get the average of like 10 or so runs.
but didn't really have enough time to do that before this demo. So couldn't get a little bit more accurate measurements for everything, but this gives you a really good idea of kind of the perf improvements that have been made. And I will say a lot of this, it's definitely adding concurrency, but we've also made some like minor tweaks to help with.
Perf improvements, two of the big things are caching dependencies, which is why when a package has dependencies, there is kind of a more significant perf improvement there. And the other big thing is caching network calls. So those two things have also significantly helped with the performance.
So please try it out. We should get a release out like in the next few weeks or so. I'm not sure when exactly that'll happen, but very soon. And would love to get feedback on this, especially if you encountered any issues with 130 preview one, because again, we've kind of made some tweaks from that and added more functionality. So hopefully any bugs there are.
Fix now.

Jason Helmick   37:44
Wow, my mind was so blown. I didn't I didn't actually hear when you when you said you expected to to to see these updates roll out, but I think you just said in a in a few weeks, a few weeks.

Amber Erickson   37:55
Yeah, hopefully in a few weeks.

Jason Helmick   37:57
Yeah, that's really amazing. That's great improvements. Thank you so much, Amber. Really appreciate it.

Amber Erickson   38:03
Thank you.

Jason Helmick   38:05
And with that, folks, we've managed to come to the end of another wonderful community call. Now, if you still have some questions, feel free to type them into the chat. We'll go through and drop some answers in there. And also look out for the recording should be up. I always say two weeks, but I
I actually try to do it in about two days. So look for the recording and we look forward to seeing you next month for the next community call. So thanks, folks. Thanks for joining us. Oh, yeah, Ryan, go ahead.

Ryan Yates   38:32
Just, just very quickly, just very quickly for the recording. There was a type of squat type of squatting attack this week that was a lot of...
modules were published to the gallery. I reported this after somebody had tweeted it. So if you are one of these people that are like me and accidentally mistyped things from time to time, just be careful of what you are downloading from the gallery, because the packages
could have been credential stealers, I think is, and I've added into the chat the other
package managers that were attacked as well, which is MPM and Ruby. So we can kind of get, you know, the funny hat on our head of, yeah, we're worth attacking, if that makes sense from it. But yeah, I want to just call out Maurice and JE Cloud.
for his tweet the other day that led me to ping the team about this.

Jason Helmick   39:51
Hey, thanks, Ryan. I appreciate the awareness, especially since I'm a lousy typist. Just ask Sean or anybody else on the team. So thank you. Appreciate the notice.

Ryan Yates   40:00
I can't type either. We're in the same boat there, Jason.

Jason Helmick   40:05
Yeah, there we go. Alrighty, folks, once again, thank you so much. Have a wonderful month and look forward. Yeah, sure, Steve.

Steve Lee (POWERSHELL)   40:10
Hey, Jason, let me let me just respond really quick. Thanks, Ryan, for bringing that up. Like, just to be clear, the partial Gallery team is working through that situation and also future ways to remediate it so we don't have it happen again.

Jason Helmick   40:26
Alrighty folks, thank you very much. Have a wonderful month. Look forward to seeing you next month. Cheers.

Ryan Yates   40:27
Thank you.

Jason Helmick stopped transcription
