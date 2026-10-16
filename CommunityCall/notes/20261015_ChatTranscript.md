# PowerShellOpenSSH Community Call-20261015_093034-Meeting Recording
October 15, 2026, 4:30PM
44m 3s

Jason Helmick started transcription

Jason Helmick   0:35
Okay everyone, we'll go ahead and get started.
Hello, good morning, good afternoon, and good evening. Welcome to the October 2026 PowerShell Community Call. Everyone is welcome to join in and discuss the PowerShell project and its development. I apologize today for my voice. I know that's difficult to listen to, but just a little bit of a cold.
So we have our agenda posted in the meeting chat and a link to the meeting discussion. Please feel free to post questions. You can do it in the discussion or just right here in the meeting chat. And please check out and follow our code of conduct. Once again,
Once again, ladies and gentlemen, a happy 20 years to PowerShell this year. Technically, the birthday is next month. I'll remind you of that towards the end of the call. It's November 14th is the actual day when PowerShell 10 was officially released. So happy
20 years to that. Also, as normal, as we normally do this, this is the last community call for the year. The next couple of months, people are off on vacation, enjoying holidays, that kind of thing. So of course, we look forward to seeing you back next year.
And as we work on our 7.7 release and so forth, you'll see blogs and information come out from us from there. We also have an amazing agenda today, quite a nifty agenda. So let's go ahead and get started with the agenda. I think first up, and Justin, I haven't had the time to look for you. Are you here?
this morning, because we're going to get started with you with PSGallery Line and SQL Lite if you're around. Awesome. It's up to you.

Justin Chung   2:21
Yes.
I am here.
All right, this has been something that we've been working on for a little bit of time. There were other things that kind of held it up, as I'll get into. Let's see if I could share my screen. Can everyone see that PowerShell window?
Or Ken, okay.
All right, great, great. So I'm going to, oh yeah, I have this suggestion turned on. Let me turn that off for now.
Let me show you the new option that we added here or yeah.
Let's see, right now the history type is text, and we're going to change that to SQL Lite by...
Get PS read line, set PSGallery line option, history type SQLite. So what this does is it migrates the text history that you have into a SQLite database and it will be a one-time migration and after that you...
your database will be set up. There were some, there's some considerations with that. It deduplicates any repeated commands, like for example,
Uh, let's see, like, get add start maybe.
or get poll, you probably have several of those, or get child item or LRS. All of those will be combined into a single entry with a count number.
I'll give you a better idea, once I do that, and I type in...
See here, LRS. So now it shows you the times I did LRS.
I added in this view, in this view, I added the run count, the last execution time of this, and where it was executed. So I think this is a neat feature of having a SQL Lite database with the text history file.
is that it can give you context-based suggestions for the commands you ran in that location. So it keeps a running tally of...
how many times you've executed in that location.
So, normally we use the up arrow to do recall, and in the text base history file, the up arrow is a...
recency-based recall. So it's the most recent command that you put in in order of most recent to least recent.
And as you can see right here, there are no duplicated commands. Like if I type in LS multiple times, the LS will only show up once. Now if we press for the context-based,
feature here, I've added an alt up arrow and that gives me the most executed command in this context.
Right now I have my PowerShell prompt set up so we can't really know where we're at. So let's.
Oh.
Get locations. I think that's it. So right now we're in my just users, just my in my user directory. From at this point, my most recently, my most run command is into the PowerShell repo.
And once we go into the PowerShell repo, what is my most recent command is Git pull or most often executed command in this location.
This.
Yeah, so that's just like a...
One thing that we are working on here, and you can see I've run, I think this is my most run command is yet. There's several here. There's a lot of different ideas that what this window could be.
But let me show you briefly why this is taking so long to get in or for us to do.
Let me share another window here.
So this is the PSGallery line.
This is where the text history is normally stored. The database gets stored right next to the text file.
But the main thing here is that this PSGallery line, if you see, the size of it is 34 megabytes. It's very high. And the reason why it's so high is because SQL Lite.
that it needs a native executable for every single operating system that we ship to.
So in order to prevent PS read line from being 34 megabytes in every machine, we create, we made a filtering system on PS Psresourceget so that it will be able to filter this and only get, for example, for my machine, when x64.
We haven't, so mine hasn't been updated for that yet because PSGallery Line is not, this current version of PSGallery Line is not on the gallery. But yeah, that's what we hope to bring you soon.
Yeah.

Jason Helmick   8:12
Justin, thank you. Let me ask you a quick question. When you say soon, do you have a timeline in mind?

Justin Chung   8:15
Yeah.

Jason Helmick   8:22
It's okay if you don't. I don't want to put you on the spot.

Justin Chung   8:23
Yeah, yeah, no timeline in mind. Maybe early next year, who knows? Yeah, yeah.

Jason Helmick   8:32
I think that sounds cool. That's, believe me, I think this is great. That's awesome. Thank you so much. I appreciate it.

Justin Chung   8:39
Yeah.

Jason Helmick   8:40
Folks, with that, we're going to move on to, and I'm also going to change, I think I posted the title of this, and I'm going to change the title. Brandon Poe, if you're around, he's going to show us managing Docker hosts in a lab with DSC. I think I posted something about machine Config, but
Um, Brandon, are you around?

Brandon Poe   9:01
I am here. Thanks, Jason. Yeah, so I do work for the Machine Config team, but today I'm here to talk about something completely unrelated to Machine Config, and this is just a couple personal projects I've been working on related to Dsc. So I figured I'd share and see if you guys...

Jason Helmick   9:02
Oh, awesome!

Brandon Poe   9:22
Might find it useful, and if so, you could potentially provide me some feedback.
So I have a small little lab where I have a few Docker hosts that I try to manage completely with infrastructure as code. I'm currently using Ansible to do that. And because of my close proximity to DSA, I would love to be able to see DSA take over these.
workloads. So I started doing some experimentation with that. One of the things that I...
created was a DSA resource that helps to manage apt repositories, because the first thing you need to do with a Docker host is get Docker set up. So that's kind of where I started at. The other thing I needed was some kind of continuous reconciliation. So I also have something to do with that.
We'll dig in. And yes, this is a DSE v3. This is not old PowerShell DSE. So to start with, what's the problem that I'm trying to solve first? The first is if I want to start getting my Doctor host set up.
I have to install some packages. And if any of you have worked with Docker much, Docker is a little bit like Microsoft, where we don't necessarily publish everything directly to the distro repositories. They have a repository that you need to set up on your machine to be able to install their packages.
packages. So if I just want a default Ubuntu machine, try to find container D, there's nothing found. I have to set up the Docker repository on here first. So that's the first problem I tackled. So I first we need to do a little.
bootstrapping, I need to install the Microsoft package repository, which is where DSC is going to come from. But instead of installing DSC directly, I'm going to install my package, which happens to have DSC as a prereq, so it'll get installed as part of that.
So we'll run that real quick. And this has already been done. So there's not really a, there's no, it's a no op operation here since they're both already installed. But that's the prereq that's needed for getting this running. And then we have the DSA resource that I created. It's called Dsc apt. I can share a link to my.
Github repo on that later. But basically we need to copy that to the machine and get it somewhere that's on the path. So I'm going to place it in the user bin folder. So let's do that.
Okay, so they are ready to go.
And, oops, let's...
Test this by having Dsc list the resources that it's able to find, if it's able to find this new one that I added. And sure enough, it's able to see that here. We can see a package of resource for managing app repositories. So the next thing I need, of course, is a configuration.
So this is my Dsc v3 configuration for managing this Docker repository. We can see some parameters that it'll take. Suite is basically the code name for the Linux version that I'm using. And then architecture, we're defaulting to AMD 64.
Ignore the metadata for now. And then we have a variable for the Doctor signing key. And then here we just have this single resource in here right now, which is going to configure a repository named Docker with the URL that we need and those parameters. And then the parameters
just using a single parameter here. This is the noble version of Ubuntu that I have. So, that's what our Dsc configuration will look like here. Very simple, just starting on this. So, it's my first resource to do. So, let's run.
Libs.
Let's run a Dsc to run the test of this configuration. So here we're running Config test with the Docker file and the parameter file.
So pretty fast, it comes back and tells me it's not in the desired state. So it's able to find that no, the Docker repository is not set up on this machine. So I could just go and run the same command again with set and bring this to the desired state. But like I said, I don't want to just run it once. I want drift correction.
So I want to do this in a way where if I ever come back and stomp on this configuration and remove the Docker repository, it will get auto-corrected. So that's one of the things I wanted to do. So to do that, I built something called DSCD. You might think it's DSCD.
daemon, but it's actually the dead simple reconciler daemon. Just kidding, it is just TSCT. Basically, this thing, what it will do, it will look in a particular directory on the machine, and any config that it finds in there, it will just run it on a schedule.
Every 5 minutes this thing wakes up, enumerates everything in that folder, and runs that configuration.
It has support for parameters. So if you wanted to do a parameter file, you just have, so if my config name is Docker YAML, my config, my parameter file just needs to be docker.parameters.yaml. It also supports Jason as well for either of these.
Yeah, Ryan, me too. I always struggle with that. Damon versus demon. I don't know.
Let me get through this part real quick and then I'll take some questions. So we can take a quick look here. Oops.
The right-click functionality always gets me.
Remember.
So you can take a look and see I have my daemon installed.
And if I...
Check the service. This is actually running as a service in system D.
And we can see in this directory here that I have.
Right now it's empty. So what I'm going to do is I'm going to drop my configuration into that directory. And the daemon will, it has a watch on the folder. So it will notice that I dropped it in and it'll actually run it immediately.
and it will drop any of the Dsc results into the folder as a results file, and we'll be able to see what it did. So since it has a watch, I'm going to copy the parameters file first because I want to make sure when it runs, it runs with the proper parameters. So when it sees that Config, it's going to try to run it.
So we'll drop the parameters file in the folder, and then I don't want it to see the file midway through the right, so I'm going to copy it to a temp folder first, a temp file first.
and then do a rename so that it just sees it as a full file.
So here we go, we're going to copy that file over.
It's copied. Now if I go back and do a list on the results folder.
We can see there is now a file there. So it actually picked it up and ran that. We can see with the journal.
We can see here it ran a configuration. The configuration was a test operation. And I'll explain a little bit about that in just a minute. And the outcome was succeeded. So now if I go and look at the actual result file.
Keep doing this.
Okay, so we can see the result file here. It says in the desired state is false, all that good stuff. We can see everything that it did. We can see that it was a test operation. So that's all great. And basically,
I said, don't worry about that metadata section before. We have this metadata section here. We don't always want it to do a full reconciliation every single time it runs, especially if it's a new config. You might want to just do a test first to make sure it's going to see this correctly and not just go nuts doing things.
So this, we're utilizing this metadata section that Dsc supports where we can put our own custom metadata in. And I can tell it to use operation test, in which case that instructs DSCD to not do the set. It'll do a test operation instead. I can change this to set.
Or I can just remove it and it will default to set. So to see what that looks like, let's make that change real quick. And again, since this picks up changes right away, we can see this right away. So here's our configuration.
I change that to set.
And as soon as I write this, Dsc is going to see it and it's going to execute it.
O, now if I look at this result file again.
I can see the operation was now set and it succeeded and there are changed properties. So I should be able to do that Docker list now and see.
the package that I wanted that's coming from the Docker repository.
Oops!
Sudo update.
Okay, we can see here, it's actually downloading from the Docker repository. So that is now there. And now if we run this here, the package is available.
All right, that's my demo.
Thank you.
Real quick, I'll just say that these are available to use if you want to give it a try. Here's the reconciler. I have a release available for that. It has
Support for a Debian package, an RPM package, and a Windows Msi, so you can install this all on Windows as well. And then I have a repo that I started here called Dsc Extras, and inside there is that apt package, and there is a zip file available.
that you can download and use on your machine as well.
All right. Thanks, everybody.

Jason Helmick   21:04
Brandon, I love it. Thank you so much for sharing that. And folks, go out there and check that out. That's really cool. Boy, your personal projects are a lot neater than mine. Thank you so much. Folks, with that, Tess, if you're around, how about discussing a little bit about this OpenSSH 10.4 preview?

Tess Gauthier   21:25
Sure thing. Thanks, Jason. Just wanted to give a brief update on the 10.4 release. This is a little underwhelming because it's not out yet. I was hoping to have it out today, but I would say by early next week, if not later today, it will be out. In addition to the changes from upstream, this has fixes for a few specific Windows issues.
as well as some community contributions. I just wanted to note that I recognize that 10.4 is still going to be behind Upstream's latest release, which they published last week. It's 10.6. We are, you know, aware that their release cadence is increasing, and we are, you know, working to close that gap with some efforts to streamline the merge process.
and increase our test coverage, but that is going to take us some time. So if there are any issues with the release, please report them on Github. And we appreciate all the feedback in testing that out. So thanks.

Jason Helmick   22:24
That's awesome. Thanks, Tess. Thank you very much. And Shammu, I posted the form in the actual agenda, but you're looking for some gallery feedback?

Shammu Meenakshi Sundaram   22:36
Yeah, I can talk about that. So hello everyone. So we are looking for some feedback from the community about your experience with the PowerShell Gallery. So we are mainly looking for two things. When you want to install a package and you find multiple packages that solve a similar problem, so like how do you choose a particular package?
And second, like for those who publish packages, like what kind of information or insights you're looking for about your package after publishing it? And like what information you use today? So these are the information we are mainly targeting. And we put together a short survey, and as Jason mentioned, it's in the agenda, and I'm also going to put that in the chat.
So please take some time to participate in that and your feedback is going to help us identify the areas where improvement is going to have like most value for the community. So we really appreciate your feedback. Thank you.

James Brundage   23:27
Can I give some really, really quick feedback about a breaking change here?

Shammu Meenakshi Sundaram   23:33
Sure.

James Brundage   23:33
Okay, so seems like about two or three months ago, the PowerShell Gallery shut off the downloads per package meter that most of the badges use. So obviously, as a developer of packages, I want to know how many times my package has been downloaded. That's already important.
But, you know, forgive the slight, you know, arrogance here, but I also want my little badge for downloads to show a greener and greener number and not be stuck at red zero, which is what the current experience of any module author is.
Please, please, please, please, please, please, please make sure that the data of the PowerShell Gallery surfaces does not degrade and that the current degradation to the data is fixed as quickly as you can. Thank you.

Shammu Meenakshi Sundaram   24:26
Sure, we definitely get that, like we are looking into that, STI, yeah.

Steve Lee (POWERSHELL)   24:26
James, yeah.
I was just saying like that's a great use case. I think you should be as detailed as you can when you fill out the form.

Shammu Meenakshi Sundaram   24:37
Yeah, that's going to help us.

Steve Lee (POWERSHELL)   24:37
Because it's not just the data, but how you use the data is what's important that we want to know.

James Brundage   24:37
I will.
I will do so. Thank you.

Jason Helmick   24:48
Perfect. Thank you. Thanks, Shammu. Thanks, James. Thanks, everybody. And now for the next thing is, we tried to have been mentioning this all year long, but Aditya, would you like to talk to us about the end of life for PowerShell 7.4 and 7.5?

Aditya Patwardhan   25:07
Hello. So I just wanted to remind everyone that 7.4 and 7.5 is UAL as of next month, November 10th. So the last releases for them are out yesterday. So or day before yesterday rather. So if you have not updated to 7.6, this is kind of the last month you should. Otherwise after that it would be a not supported release.
So I encourage everyone to move to 7-6.

Jason Helmick   25:35
Perfect, thank you. I love that. Yeah, folks, those are going out. So it's time for 7-6. And that'll be, 7-6 will be good for the next couple of years. So it's until towards the end of 2028. So this is a great time for that. And don't forget to test out the previews and so forth on 7-7 as we're working to get that out the door as well.
So the next topic is the docs update. Now look, James Brundage is getting ready to do a demo on Wave and I am so excited to see that. But let's talk about docs updates for a second.
First of all, I don't have a Docs update today, but I'll have all that figured out when we return next year. Recent organizational changes mean we're saying goodbye to a couple of wonderful folks on the Docs team, and two of them are Sean Wheeler and Mike F. Robbins. Mike and I go back before either of us worked at Microsoft. We even wrote a book.
book together on PowerShell. So I know firsthand how much he cares about getting it right for the person on the other end who's just trying to learn something. You can see that in every page that he touched. And Sean, Sean's title said he was in docs, but his impact went far beyond that. He was the PM we never officially had and always needed.
I leaned on him more than he'd ever let me say out loud, and I'm going to miss that a lot, Sean. So thank you very much. To both of you, thank you. You made PowerShell a better place for a lot of people who'll never know your name, but we do. And so cheers to you both. Thanks for your contributions. And folks, when we start up next year,
I'll bring back the official docs update with me. And with that, James, I have finite time, but I really want to see this demo on Wave. I have my own nickname for it, but please, James, go for it. Enlighten us.

James Brundage   27:29
Alright, I believe I've already heard your nickname and I believe it's a very apt one, but mind if I repeat what you had sent me yesterday?

Jason Helmick   27:37
No, no, go ahead. I am...

James Brundage   27:41
So you'd call this a PS Moog. And for those that are not familiar, Moog was the original electronic synthesizer and was also apparently quite the discovery journey for its creator of, oh, look at what I can do with sound forms now. And that's basically kind of what I've been on all month.

Jason Helmick   27:45
Yeah.

James Brundage   28:02
So to give a little bit of lead in before I start playing notes, also let me make sure, hold up, let me stop sharing again, share, make sure we got audio enabled because this has happened multiple times.
Okay, so about a month ago, basically, somebody vibe coded a PowerShell Wolfenstein, then deleted it from the internet in the huff, and left me with a repo full of interesting, you know, curiosities, including how to make sound in PowerShell.
So, okay, here's like the 10 or so lines of to be able to create a wave directly in 8-bit audio. Let's extend that out. Let's figure out all of how waves work. How bad could that possibly be, right?
So I'm going to go re-import my module here for a second. Actually, I'm going to go pop out of my examples directory, re-import my module, and can everybody hear that?
Did that come through? Okay. Let's play that a little bit higher.

Bohren, Andres   29:05
Yep.

James Brundage   29:13
Let's do that in proper notation here for a second. C4, A4, E4, F4, or F4, E4.
This is directly generating the sounds with PowerShell. Let's get a little bit more complicated. And may the demo gods be kind to me. Let me know if you all recognize this piece.
It's going to take a little second to generate.
You might have heard it sometime before. I don't know, sometime since the 80s. There's a lot of fun stuff in here.
So...
Everybody know this one?
Or bikini, otherwise known as the Tetris theme. So we got a few fun examples already. In fact, we'll get to this one, but this is the first time I've ever actually had a PR into a module that isn't released yet. So this module is called Wave. It will be coming out when I'm good and ready.
hopefully in the next few days. But yeah, it's already gotten a bunch of interesting uptake in terms of interest from powerful community people that are musically inclined that have been helping me understand the theory of it. Kind of getting to the guts and nuts and bolts for a second here. I'm going to go crack open our wave PS1.
which has a few other examples. Doing the emoji alias things, we can also do wave, note, cafe, play. Or if we want something a little bit more fail note, we can just do an A&D. Let's make that one a little bit fun. Let's do a new.
register engine event. Let's call it uh-oh.
Action, wave notes.
AE play. Okay, do event source identifier. Uh-oh.
So just because it's PowerShell, we can actually already do event-based triggering of music. That's fun. That's fine. Let's do something else a little bit fun. Let's do wave note.
C Yeah, CCDCC3G3 play.
Everybody recognize that one?
So we're getting closer to something really cool here for a second. Okay, back to the code. What we basically got is a quick little trick, returning memory stream, we're decorating a memory stream. So we're basically creating a memory stream for a wave, and then we're saying it's a wave, and then we're extending it with properties and methods.
We can run in background jobs. We can go run any instructions on a wave. We can directly pipe in samples, which I'm not going to get to demonstrate right this second. And we can get to the raw construction of a wave a little bit further down here for a second. We've got to have an audio format.
number of bytes per block, number of bytes per second, channel rate, sample rate, channel count, and then we're pretty much done. Where's my nice code that actually does the raw construction? There we go. Wave file start with a riff, then have their length minus 8, then wave.
then a format chunk, which has 16 bytes, and that has an audio format, and a channel count, and a sample rate, and a bytes per second, and bytes per block, and a bits per sample. And then we have some data marker, and then we say how long our data is, and everything beyond that is just the wave.
So we can actually directly work with this in the object pipeline. I'm going to take this here for a second and we're going to actually go get its samples back.
So that's the samples that are actually playing when WaveNote Cafe plays. Quite a few samples. You can see why I didn't return double array in the object Pipeline, I think, from this, right?
So yeah, direct access to PowerShell on the object Pipeline. What other fun do we have here? We'll get to Flight of the Bumblebee in a second. We might even want to start writing it. I'm writing it now, but let's see.
We can do fun stuff too, like back masking. So we're going to keep doing our wave note cafe. We're going to do speed negative one play.
Speed negative 1.5, which is going to make it basically go up an octave speed.
0.5.
It's going to slow the way down, take it down an octave.
So I will mess with time. There are crazy interesting things that you can do with audio once you understand the basics. I'm also going to take you over really quickly to the core of our generator. We're going to see how PowerShell it is right now. So we're going to go to types.
Wave tone.
which is our polyphonic tone generator here.
It takes a series of frequencies, a duration, a volume, a start time will maybe have a step count.
It goes and creates 2 steps of an object pipeline, a poly sign in an envelope. This is basically what sound is again and again. You have a sign generator, you stuff it in an envelope. So our sign generator takes T, takes a bunch of angles, adds all the angles at P together, multiplies by their weight, keeps having their weight.
so that we don't overpower our overtones and undertones, then stuffs it in an envelope, which basically takes the time and slowly hushes the volume out and scales it. Again, that's all what sound is. So our samples for our tone are literally go to T, go from this range to that range, make our sign, stuff it in our envelope.
Sticking at the end of our wave, we're done. Let's do this directly here for a second, so we're going to do wave, wave.
And wave tone 440.
Play.
I'm actually going to hear two of these, because...
I'm using the same object. Let's do this a little bit more, or have a little bit more fun with this.
Or have the demo gods curse me? Either one.
Wave to.
Uh, we're looking at the demo godscur scenario here for a second. Let me try this. There we go.
So we got one tone. Now repeating our tone. Now let's fix our syntax and start to add some more stuff to this. So we got a tone. Let's add a triangle at the same set of frequencies.
And a square.
at the same set of frequencies.
and a sawtooth.
at the same set of frequencies.
So this is direct construction through dotting in objects. Here we go.
I, I will not.
debug all my demos live, I will not debug all my demos live.
Yep, we've got a slight problem somewhere and I'll come back to it later. Shucks.

Jason Helmick   37:21
Well, why don't you, James, why don't you give us the flight of the bumblebee and tell folks where they can maybe check this out in more detail?

James Brundage   37:30
I will after one other thing. I got to wish PowerShell a happy birthday first. So let me let me happy birthday, bring up a link, then fly to the bumblebee, and then I'll sing myself out.
Well, play myself out. This is happy birthday in Dex, because you know, it's PowerShell, so I love basically being able to take other audio formats, bring them into waves. Dex is an ancient audio format for console on the VT-520.
And PowerShell is about to have a birthday, so you know, happy birthday to PowerShell.
All right, now huge shout out to Steven Bailey here, who actually gave this pull request. He had written chromatic runs. I've actually since added this as range, so if I take in his newer pull request, it's going to get up on shorter. But Flight of the Bumblebee is basically made of a series of
scales from, you know, point A to point B, and as he puts it, you remember when the scales change and how many notes. And so all Flight of the Bumblebee is mostly these series of note runs, which all turn into letters between and B. Let's see if demo gods are still kind.
Let's go ahead and declare our functions here for a second.
Fate.
So if I wanted a note run from there, that's all the notes between there. Now let's play fight to the bumblebee and see how long it takes to generate. It's only like 600 notes. We'll see.
I also probably need to include my play after this. So let's control C that and play that.
It's also caching every note as it generates it, so that if you try to generate the same note twice, you will get a nice performance boost. And you'll see this progress bar move faster and faster as we go, because there are lots of repeated notes in music. At least that's what all the music theorists keep telling me.
Unfortunately, there's also a little bit of dead air to fill up before Flight of the Bumblebee finishes. And let's see if I can time it right. But it is quite a piece. We've got 656 notes being generated. Let's see how 656 notes sound coming out of PowerShell.
I think they sound pretty good. I'm also trying to make this as good of a reference implementation as I can make for audio, so this is all easy to understand in open source.
And...
So yeah, we can make music in PowerShell now.
We got music of the object Pipeline. You can find out about this project at Github.com slash posh web wave, and I'm throwing it into chat. It is still in alpha and not released. Please do not be rude and steal my module name on the gallery. In fact, maybe just embargo it until it's out. I half kid.
But no, this is a really, really fun project. It's been incredibly eye-opening. It's also really been really interesting to see just exactly what sound is, how it's made. And a huge shout out to audio engineers in Microsoft that I will never know or meet who wrote this file format almost 30 years ago, no more than 30 years ago. So yeah, this is how audio works.
actually works, how audio is actually encoded, and now we can do this inside of PowerShell. And for those keeping track, if you combine that with Turtle, and you can combine that with the ability to create 3D objects in PowerShell, congratulations. We can now treat PowerShell as a fully
Audio-visual language in two and three dimensions. We can mess with time and space and sound in PowerShell.
It's a hell of a language, and you should all be trying it more. And I know we're all on the PowerShell community call, but come on, make some tunes with PowerShell, make some GUIs with PowerShell, do things with PowerShell people never thought was possible. I promise you, it's fun.

Jason Helmick   42:44
Well, thank you, James. This was amazing. I'm just so happy with Flight of the Bumblebee. It worked out so well. So folks, with that, once again, thank you so much. Happy 20 years to PowerShell. And you know, next month is the actual birthday. And what I encourage everybody to do next month is do something fun.

James Brundage   42:45
Shepherd's crooks on.

Jason Helmick   43:06
like James has done, and do something fun with PowerShell. Just enjoy the language and what it does. Take it out of work for a minute and just have something fun. Program your Tesla, do whatever it is that tickles you. So have a wonderful and safe holiday, and we'll see you back here next year as the journey continues. Thanks so much.
Cheers.

Jason Helmick stopped transcription
