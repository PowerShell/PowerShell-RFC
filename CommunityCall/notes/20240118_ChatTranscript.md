# PowerShell_OpenSSH Community Call-20240118_092648-Meeting Recording

January 18, 2024, 5:26PM
56m 10s

Jason Helmick   4:19
So let's go ahead and get started with that.
Hello and Happy New Year.
So, good morning, good afternoon and good evening.

PowerCode joined the meeting

Jason Helmick   4:27
Welcome to the January 2024 PowerShell community call.
Hey everyone is welcome to join and discuss the PowerShell project and its development.

Damien Caro joined the meeting

Jason Helmick   4:35
Now look, we've got an action packed agenda for this brand new year and so I've posted the agenda in the meeting chat and a link to the meeting discussion.

Ayan Mullick joined the meeting

Jason Helmick   4:44
If you want to drop questions in there now the meeting is being recorded, so if that's not your thing, go ahead and exit now.

Nate Kasco joined the meeting

Jason Helmick   4:51
And please take a look at our code of conduct.
We care about that and it's a fun read.
So with that, let's go ahead and dive in again.
Welcome and Happy New Year to everyone.
We're looking forward to a great year.
I think first on the list today is to go to Sydney and I think I saw Sydney earlier.
Sydney, do you want to start us off by talking about VS code?

Sydney Smith (SHE/HER)   5:14
Absolutely.
Thank you, Jason.
So last week we had a stable release of our VS code extension.

Domansky, Mark joined the meeting

Sydney Smith (SHE/HER)   5:23
This release included a bunch of improvements to the LSP server and a major upgrade to our testing system, and also included some updates to Shell integration.

Vivian Thiebaut joined the meeting

Sarney, Ed joined the meeting

Sydney Smith (SHE/HER)   5:37
One thing to note about this release as well.
Or just the VS code extension plans in general is that this year you can expect to see about a quarterly stable release from us and then preview releases in between them.

Darren Kattan joined the meeting

Sydney Smith (SHE/HER)   5:50
If we need a stable release sooner than that, we'll roll one out, of course.

Ryan.Wakefield joined the meeting

Sydney Smith (SHE/HER)   5:55
But that's kind of the cadence you can come to expect from us this year.
I saw Andy is also on the call.
Andy, did you have anything you wanted to say specifically about this release?

Andy Jordan (THEY/THEM)   6:06
I'm I would just add that we're really happy having moved RCI over to GitHub actions.

Gill, Matthew joined the meeting

Andy Jordan (THEY/THEM)   6:11
It was kind of like a a December sort of fun thing to work on.

Chris Chisholm joined the meeting

Bartlett, James joined the meeting

Andy Jordan (THEY/THEM)   6:15
We struggled over the last year.
We're like, yeah, we had CI and you know us, we've been focusing on stability and writing a lot of tests, but they were always a little flaky.

Colin Webster joined the meeting

Andy Jordan (THEY/THEM)   6:22
So we'd try to fix a bug and we'd run our test three times until they finally passed in.

Patrick Wahlmueller joined the meeting

Brooks, Ron joined the meeting

Andy Jordan (THEY/THEM)   6:27
The flakiness went away.
I finally ported it all over to have actions we now can actually move on our pool request really quickly, which has been very nice and we even have like the end to end tests of the server side running against the daily updates to PowerShell.
So we're hoping that helps us catch like bugs in the future more quickly.

Aleksandar Nikolic joined the meeting

Andy Jordan (THEY/THEM)   6:47
So if PowerShell daily new change in PowerShell breaks us, we know before you tell us because we have actually run into that before.
In fact, we have code that lets you run the extension, finds the daily installed PowerShell.
If you're using that for all that beta testing, but yeah, it was really just an update focusing on the server, mostly with a lot of bug fixes.
We did find out that the Shell integration was broken for a bit and we just had to pull in the update to their script.
If you installed the January release, all those little symbols that they put in, like on the side of the shell, those work now.

Bryan Gritton joined the meeting

Andy Jordan (THEY/THEM)   7:20
If you have an error, it shows an error.

Michael joined the meeting

Andy Jordan (THEY/THEM)   7:23
Works as expected.
Yeah.
We appreciate people testing stuff like that, and he said we'll have another out in April.

Scott Ott left the meeting

Scott Ott joined the meeting

Ayan Mullick left the meeting

Andy Jordan (THEY/THEM)   7:29
But do you expect Prereleases to still come before then?
In fact, I think we're planning a pre release next week just because we have a number of changes that we're incorporating over the next few days.

Anam Navied joined the meeting

Andy Jordan (THEY/THEM)   7:39
So yeah, thanks for using the extension.

Jason Helmick   7:43
Thanks Andy.
Thanks, Cindy.
That's great.
That's great now.
Hey folks.
You you may have realized a traditionally a Christmas present.
We shipped PowerShell 7.4 long term service release for the next three years back in November.
That's awesome, but this is January, so it's time for us to once again start talking about our next release, which will be 75.
This will not be an LTS release, but with that.

Ayan Mullick joined the meeting

Jason Helmick   8:07
Hey Dumbo, do you want to say hello and talk about the upcoming previews?

Dongbo Wang   8:12
Yeah, of course.
So I'm gonna be quick.
Uh, we are planning on a a new uh, a release for 75 preview one which will be out hopefully next Tuesday.
So it's it's, uh, the release work is is undergoing right now.

Luke Murray joined the meeting

Jason Helmick   8:29
Well, that's it.
That that's awesome.
And so folks, we're starting that exciting preview role again.
And so we really appreciate you taking the opportunity, if you can, to download and check out those previews and to provide us feedback while we're in this critical development stage on it.
So thank you very much.
And next on the list, Steve, Improvements to Commandlets.

Steve Lee (POWERSHELL HE/HIM)   8:48
Yes.
And actually before I get to that, I actually remember one other topic which could have had an agenda that follows 75.

Jason Helmick   8:50
Yeah.

Steve Lee (POWERSHELL HE/HIM)   8:54
So you know, we have the annual team investments blog posts, so that is in progress being written.
I'm hoping it'll be published by end of this month, or if not, early February, so do look forward to that and I'll talk more about what exactly we are planning for the seven five release as well as other projects that we have on the team.
Anyways, the main topic I wanna talk to you guys about today is something came up in the uh Commandlet working group meeting yesterday and before I get into that, I again I wanna thank all of our Community working group members.
And by the way, one other topic that was brought up in the committee meeting, which is also happened yesterday, is that we're going to kind of improve the process of giving more folks involved who want to be participating in working groups to do look forward to a couple of PR's coming out and maybe we'll talk more about it next community call when we have those things in place.

Terry Dow joined the meeting

Steve Lee (POWERSHELL HE/HIM)   9:46
But anyways, the thing I wanna bring up is we have a number of issues that come up in the command working group of people wanting to improve the commands, which makes a ton of sense.
Umm, some of the commandlets need new capability.

Clint Rutkas joined the meeting

Steve Lee (POWERSHELL HE/HIM)   9:57
They need behavior changes and one of the big challenges that we have is we have a lot of people that use PowerShell and they have scripts deployed and a small change sometimes seems safe and it doesn't break people like we've had this happen with the same 4 release where I got notification from partners that they deployed 7, four and something broke because a small change in the cmdlet which seems safe at the time, introduce new behavior that was unexpected and it didn't work the way it's said to make fixes or birth changes.

Axel Katzur joined the meeting

Scott Schaaf joined the meeting

Brown, Derek joined the meeting

Bruno joined the meeting

Harms, Greg J joined the meeting

Steve Lee (POWERSHELL HE/HIM)   10:25
Anyways, there's other cases where people want to add new capability, and the big question is like, alright, how do we wanna do this without adding like you know, a lot of parameter sets or making it super complex or having parameters that have similar names but have different behaviors or having a switch to change like there's a lot of things that we discussed.

Garza, Joe joined the meeting

Steve Lee (POWERSHELL HE/HIM)   10:45
And So what I wanna bring up and I'll put a link to this GitHub discussion in the chat here, and I hope people join.
This is we're currently like gonna brainstorm phase.
OK, there is a proposed, so I put up there that we kind of discussed briefly.
It is not the only way to go.
That's not the way we're gonna go.
I wanna get some feedback.
I really like, alright, if we want to kind of get to a place where the commandlets are more modernized.
They're more designed for cost platform.

Januszko, Missy joined the meeting

Steve Lee (POWERSHELL HE/HIM)   11:12
I'll give you another good example because I seen a lot of these issues come up like get child item people don't necessarily understand all the complexities.
We're using a wild card and then having include, exclude and have recursion where things can happen that are unexpected, but we can't clinical fix those behaviors because people are likely dependent on those behaviors and if we change it, even if it's technically correct, it will break a ton of people, right?
And so how do you address those kind of things?
Anyways, this meta topic is and this discussion, so please putting your thoughts put in, not just you know if you have ideas on how to fix this but also identify problems, you know whatever.

Danny Maertens joined the meeting

Steve Lee (POWERSHELL HE/HIM)   11:47
And let's just have that discussion and maybe we'll come to some conclusion that we can kind of take forward. Thanks.

Jason Helmick   11:55
That's awesome, Steve.
Thanks.
And just to reiterate, one of the things that Steve was mentioning was about the working groups he was mentioning that we are working on a new process.

Michael Anstadt joined the meeting

Jason Helmick   12:03
We we realized that the the discussion list that we had was difficult for us to keep track of.
So we're going to have a more formalized application process where applications won't get lost and people won't get forgotten about for months.

Strong, Jeremiah joined the meeting

Jason Helmick   12:15
And so at the next community call, we anticipate having that conversation.
So thank you for your interest in working groups.
We're working on getting you in.
So thanks a lot.
And with that, let's go back to Sydney and talk about PS resource get in a servicing release.
Hey, Sydney.

Sydney Smith (SHE/HER)   12:33
Hey this should be a quick one too.

Amir Terani joined the meeting

Sydney Smith (SHE/HER)   12:36
Just an update on PS Resource get that we are working on servicing release so getting some bug fixes out to you.
We are hoping to get this release out later this month, but of course I can't promise anything around thought.

Damien Caro joined the meeting

Sydney Smith (SHE/HER)   12:51
I will link the project so if you wanna track the work that's going into that release, it's just a few.
A few bugs that we thought were important, so link that there you can see what's gonna go into that release.
So expect that release out in the next couple of weeks if you're curious about other things that are going on with PS resource get, you can check out our projects.

Franco, Alex joined the meeting

Sydney Smith (SHE/HER)   13:13
We keep those very up to date and you can.

Garza, Joe   13:14
I'm starting with your car.
I ain't touching that.
It's like 5 pages of shirt.
Gonna take this off and take this off.

Sydney Smith (SHE/HER)   13:21
Umm.
Anyways, you can see exactly what we're working on, so definitely check out projects there and keep testing it out.
No opening issues for us, but expect that to be our next release.

Patrick N joined the meeting

Jason Helmick   13:32
Thanks Sydney.
Now, now, Steven, I have not let the cat out of the bag.
I have not said anything about this exciting news, but so please come online and tell us about the March community call.

Garza, Joe left the meeting

Steven Bucher   13:47
Sure.
Yes.
So we have been getting some requests for a lot of community members in Australia and Asia that this time is not super convenient for them, for the Community call.
And so as a way to.
Some include them.
We are going to have the March and the September community called be at a different time.

Ili Metuky joined the meeting

Steven Bucher   14:09
We are still picking out the exact time right now, but we're hoping that the March and September community call kind of ongoing would be a good time for.

Friedrich Weinmann (CSA, POWERSHELL) left the meeting

Steven Bucher   14:18
We'll still have it.
It would just be at a different time, so if the time still works for you, you're everyone is more than welcome to join.

Pulk, Joseph joined the meeting

Steven Bucher   14:25
We just wanted to have a time that is not in the middle of the night for a lot of folks in Asia and in Australia.

Chris Wright joined the meeting

Steven Bucher   14:34
So if you know anyone, uh, in those time zones, or are you or in that time zone yourself?

Domansky, Mark left the meeting

Steven Bucher   14:40
And it's just very, very late for you.
Don't.
Please please let them know that we will be announcing the exact time very soon, but March and September will be a community called probably later in the evening for us in the Pacific Northwest.

Domansky, Mark joined the meeting

Steven Bucher   14:57
But yeah, so that's all I have on that.

Jason Helmick   15:02
And so on.
So Steven, I'm really looking forward to that.
And hey, folks, yeah, if it works for you, please join us.
It's gonna be the the the regular community call with all of us there, just at a later time.
So we can bring in more of the community and get more feedback and have even more exciting conversations.
So expect some great demos as we go through this.
Things that you normally wouldn't see because of the time difference.
So we're really excited about the expansion of this.
So looking forward to it.
Thanks Steven.
Damien and my friend, you want to talk about some changes in Wham?
I'm not sure what Wham we're talking about.
The band. What?

Damien Caro   15:39
Yes.
Thanks Jason.
Umm I I have actually a couple of slides that I wanted to to present and I hope everyone can see that.

Jason Helmick   15:45
Sure.

Damien Caro   15:49
SO1 stands for Web account manager.
Think of it as a different way of authenticating against Azure.
Uh, so far when you're using Azure CLI or Azure PowerShell to connect to Azure, one of the mechanism we have is called the web browser flow.

Tess Gauthier joined the meeting

Damien Caro   16:06
So you basically open that web page, authenticate and then you return to your command line.
One will reduce the need of launching the browser or remove the need of launching the browser, and we integrate with Windows blocker to do the authentication.
The benefits I have listed four of them.
It's an enhanced security mechanism.
It's a faster single sign on approach, so you're credential to.
I just had my Microsoft account here in the list in the in in the screenshot, but you can have other accounts here that are connected to your client.
It is integrated as the Windows account and we want to enable at some point token protection.
It's not there yet, but that's something that we have in the plans.
It is today supported in preview.
Uh, we have it in preview for CLI.
We have it in preview for Windows PowerShell and what I wanted to mention or talk about today is that we are looking at making it as the default authentication.
It's going to be supported only on Windows.
We don't have it yet and I insist on, yet we're looking at supporting it on Mac and Linux as well.
If you're running the client on the non supported version for one, we will fall back to default mechanism of of program flow.

Victor (External) (Guest) joined the meeting

Damien Caro   17:29
Non interactive flows service principal manager identities, which is kind of a service principle in the way are not going to be impacted by this change.
It's really about the interactive flow on me.
We have a tentative timeline and I wanted to to make sure that it's that everyone is aware of that I've switching to default for CLI.
We're looking at March and for PowerShell we're looking at May.
We are looking at a couple of issues at this point that can come in the way and and delay our timeline, but that's roughly what we are targeting March and May for, for CLI and PS I'm you may ask why do we have couple of months of of different one of the reason of that is that PowerShell is depending on the SDK as we had to wait for the SDK to be built and then we can build on PowerShell.
CLI is building directly on the libraries.
I can dive in the state in those technicalities, but that's probably the main reason of the of the of those different timelines.
If you want to know more, if you want to try out, you have in the docs for RCA and Azure PowerShell where the information is how to install, how to use the the the info and the feedback as usual on GitHub it is where we get the data and that's where we listen and fix issues.
Moving on, the next improvement, we're also doing some improvement on Azure PowerShell specifically here on the output of commands.
We heard from customers that some of the off command mostly so identity related get easy context get easy subscription, get easy tenant.

Elrod, Andrew (he/him/his) joined the meeting

Frode joined the meeting

Damien Caro   19:15
The output was kind of cumbersome.
We were missing customer.
We're missing the integration of a link of of infos.
Are you are getting a subscription writing some info that the number of columns were kind of hard to read.
So we've done some changes to to rationalize a bit how we display the output, try to make it easier to read and as well as easier to to handle in script.

Domansky, Mark joined the meeting

Domansky, Mark left the meeting

Damien Caro   19:42
It's not a breaking change.
We're really focused on the UI and not on the object model that is behind the object returned.
It's available today in preview on Easy Account version 2.42 DOT 14 Preview.
Uh, so you can download it and try it out.
Our plan is to release it as a GA on February 6, which is Tuesday the 1st Tuesday of February.
So that's two weeks from now.
Until then, you can try out if you find issues.
If you are in something that you don't like with that output, let us know we can make quick changes by by then.
I'm here.
You have a screenshot of the get easy context, and we've also looked at the get easy subscription.
I've tried to hide some ideas here, but but you get the idea of, I think a birth of of a table and now we we make it easier to read.

Chris Chisholm left the meeting

Damien Caro   20:33
The again try out report issues on GitHub and and that's that's kind of the two two updates for the Azure command line tools.

Jason Helmick   20:44
Oh, that's awesome.
I got a note here that you also want to talk a little bit about the Azure PowerShell UM Authentication command.
Was that part of the Wham conversation that you were having?

Damien Caro   20:53
That's the command I was talking about.
The command output get a discussion getting the context or.

Jason Helmick   20:54
Ah, awesome.
Ohh the output.
OK, that's perfect.
Thank you so much, Damien.
Appreciate it.
So I I I wasn't looking at the screen and I I I'm.
I'm hoping my friend Sean is here and can provide us with a docs update.
Are you around Sean?

Sean Wheeler   21:11
Yes, I am.

Jason Helmick   21:12
Ohh.

Sean Wheeler   21:13
So December was a slow month.
Most of us were gone for half the month, so no new content released.
Lots of minor updates.
And the the usual kind of fixes.
But.
Yesterday.
Mike, he dropped a a big update for the DSC V3 documentation for the A4 release.
Umm, I'll put a link in the chat there to the change log.
So you can see what's changed.
Mikey, are you on?

Mikey Lombardi   21:54
Yep.

Sean Wheeler   21:55
You want to give us a quick highlight of what's new?

Mikey Lombardi   22:01
Yeah.

Sean Wheeler   22:01
Uh.

Mikey Lombardi   22:03
That's a.
That's a good point.
So the big the big thing to keep in mind is that we did a major update to the schema between the last release and this one.
So the schema should be tighter and a little bit better.
And it's set up to support multiple schema versions going forward as we realize we're going to make breaking changes, particularly during alpha.
And then you'll want to be able to poorly the version that you're using of DSC with the schema that's compatible with it.
So that's doable now added a new well known keyword to kind of replace ensure one of the pieces of feedback we've gotten for forever is that ensure is pretty unclear to people and they get confused about what they should and shouldn't use and share for.
So we figured exist is a pretty clear.
Semantic option instead and it'll be easier to code around because it's just a Boolean true, false, no more enums and managing all of that across different programming languages.
A lot of minor updates.
One of the other big things is uh, input our global options for the DSC command previously had to just pipe everything over standard in.

Jake Daw joined the meeting

Mikey Lombardi   23:18
Now you can either save a config as a data BLOB or pass a reference to a file.

James Brundage joined the meeting

Mikey Lombardi   23:25
Lots of new stuff, shell completions turned on.
Now umm and logging is added and improving as we go, so that should be improved again in A5 I think.

Sean Wheeler   23:37
All right. Thanks.

Jason Helmick   23:40
That's awesome.

Sean Wheeler   23:40
Back to you, Jason.

Jason Helmick   23:40
Thank you both both Mikey and Sean.
That's great.
I love it.
We get a big update.
Hey folks.
A few months back I I don't know if you'll remember or not, but we had Clint Rutkas come in and talk about some of the work that they were doing on winget command not found.
I understand he's back, so I'm very excited to see what what's going on with Winget.
Clint, are you here? Awesome.

Clint Rutkas   24:00
Help.
I am.
Thank you.
Give Me 2 seconds to clothes 800 Windows I had open.
OK.
So everybody, I'm from powertoys.
I also am on a team called Engineering for developers on Windows, so our teams job is just to do great cool things and make your lives easier on Windows.
And one of those things was, hey, uh, I want to, you know, figure out how I can get an application that I thought was installed on my computer through command line and get it installed easier.
So I'm gonna share my screen and give a quick demo of this.
In theory, I was gonna go do this on a totally clean machine, but I'm not currently, so I'm going to go do some my desktop, so I'm going to use them.
Just give me one second to share my screen here.
Great, everything is not on there, so here you can see I have PowerShell 7.4 and I'm just type in them and you can see I don't have it installed on on this computer and go zoom in.
Here you can say the term vim is not installed gives a list of potential things because you can go look at the package managers and also go winget search to verify like is this the one you really really want.
If you typed in code, typically through command line, you'd be like.
Oh, I wanted Visual Studio code.
You'll see the visual Microsoft Dot Visual Studio code will come up as one of the options here also.

Paul, Ankita joined the meeting

Clint Rutkas   25:30
So the goal here is to get you where you need to get quicker on a brand new computer.
So here if I just want to win get.
Oops, let me fully copy it.
Copy it, paste it and just hit enter and it will fully execute.
So if I did.
Uh, try to think of another app that I don't actually proactively have in here, so I'm just gonna type Adam.
So same thing you see GitHub Adam.
These are all things, but if I type in code, Visual Studio code will actually boot up.
So this is the goal here is to get you where you need to go and we're looking for a lot of feedback here also of is this the pattern and practice that you all would want to get this installed?
It's super easy inside of power toys, so here we have a command not found applet page.
So we first we detect you have power Toys 7 four or greater installed.
Do you have the Wingate client module installed?
If you don't click install, click install, we'll auto install it at the bottom.

Nate Kasco left the meeting

Clint Rutkas   26:31
Here you can get your nice little a install logs to be sure it's there and you click install for installing this module and it just works and that's the goal is to get you where you need to go faster if you like the pattern and practice please tell us.
If you don't tell us, we love to understand more what your thoughts and patterns and practices are here.
So we have full documentation here as well, yeah.

Grote, Justin   26:54
Hey, Clint.
I just want to ask a quick question, is there, is this gonna be on the PowerShell gallery or is there a reason that it's not?

Clint Rutkas   27:02
Right now, this is literally our first release here, so we're trying to better understand what people's patterns and practices are before we go larger.
The other thoughts are click that we do other things, so we're we're still incubating.
We'd love to to start off a little bit small, a little bit more cautious before we go larger and understand the dependencies.

Chris Wright left the meeting

Clint Rutkas   27:25
So our team typically works in in actually applications.
So this is our first fourth tie into doing the module work.
So we're like, hey, are we doing the correct thing?
So the goal probably would be we put it on there, but we want to be sure, like I said, everyone is OK with what we're doing.

Grote, Justin   27:45
Sounds good.
And again, kudos to you guys for exploring PowerShell modules and adapting them as part of your workflow.
We really appreciate it.

Clint Rutkas   27:51
Yep.
Other cool things I while I am here is we are actually actively doing a DSC resource for power toys.
Also, to be sure that you're installing, once you install power tools, you can fully configure it.

Paul, Ankita left the meeting

Clint Rutkas   28:07
We're doing kind of some cool, clever things like reflecting off our settings files to then generate the actual DSC resource.
So here's our draft PR also.
So if anyone does have any feedback or thoughts and concerns for how we're doing it and things that we could be doing better were always up for for community feedback and community contributions as well.
So give that one a little bit of a plug.

Jason Helmick   28:36
It's awesome.
Clint, congratulations on the work and the release.
This is this is this is really cool.
So it's very exciting.
I'm with that, folks.
We're ready to go to our our community demos and 1st up on my list is Justin Grody.
Now I've seen you out there, Justin, and I just heard you.
So I know you're here.

Grote, Justin   28:56
Just hiding, you know, I I had to ask.
I think so.
Hey, everybody, how you doing today?
I just wanna do a quick demo of module fast.
Is that a pretty significant new feature release?
It's sometimes I worked on the last couple months over the winter break and was able to get some good advances.
Can you see the release notes have up here on the screen?
OK, great.

Jason Helmick   29:14
It was shocking.

Grote, Justin   29:16
So starting of course with breaking changes, which is always important, uh, this is no longer a thing.
It did require Power 7 three because I tried to get fancy and use the clean block to do some fancy clean up.
Unfortunately, most of the places where this runs still aren't quite on 7/4 yet, so it now goes back to using 72, but that's an issue.
Another thing is that module fast.
Well, now evaluate all module folders.
You have in your PS module path, not just the destination.
By default, this makes it so that it can act even quicker if you've already got the module installed.
And also sorry to back up for a moment for a quick overview of module fast module fast is a module that optimizes the PowerShell module installation process to be really really fast as fast as possible.
Quick and dirty and allows a lot of really easy things.
It's really designed for getting modules into a CI build or anytime where you just need to pull down a bunch of modules really quickly and it's there is sort of a compliment to PS get and PS resource git for those kind of scenarios.
So one of the big new features is prerelease support, so module that's now completely supports pre releases.
You can do it by either specifying the pre release tab which is you see.
By default it doesn't get the latest prerelease, whereas if you do specify it, it will and these are live lookups.
So if that's how fast like your typical like find PS resource would be versus install module fast and then in addition there's a shorthand syntax.
Here we can just add an exclamation mark to the beginning or the end of the line, and that'll flag that thing is that you want to be a pre-release that's there to support certain kinds of scenarios in terms of the layout, and that brings me to my next point.
Is the shorthand string syntax, so one tricky thing with P3 source or excuse me with PowerShell get is that if you want to install a bunch of different modules, some of them having certain prereleases, some of them not any of those kind of specifications, you either have to use like the module specification hash table syntax which can get kind of unwieldly or you have to and there are certain combinations you just simply can't do.
So module fast supports a very straightforward syntax that lets you take individual rights.
Keep pausing on me.
I want to run but they felt three so it's real quick.
There we go.
So in support of stored and syntax here where you can say for instance if you want by default it's the latest version or say you want all modules that are less than two.
You could just say less than two if you want all modules that are, you know, version less than 2.3 that works.
Two, it supports wildcards.
It supports nugget syntax.
It supports again that preleased format as mentioned before.
But you can put the exclamation mark anywhere you want, and here's an example where you can specify the nugget syntax where you can say I want any module in between 2.2 and 2.4.
I get this little progress bar to go away.
You see here that it's getting that 2.3, umm.
In addition to that, module fast supports now spec files, so these are things like requires files etcetera module fast has its own version of.
A requires file that you can specify.
That's just really simply the name of the module, and that syntax that was mentioned earlier.
So you can define a file on this.
It will automatically.
You can just point and module fast to that folder.
It'll find that file and automatically install the modules that are associated with that in any of those kind of combinations, resolving all the dependencies, et cetera, and also now supports lock files.
If you've ever worked with NPM or PNPM in Nodejs, or if you've used net, you've known this.
When you specify module specifications, there's reasons like.
For instance, if modules update, what's even though you're specification says like modules, you know version two to version three.
If you have something you publishing to GitHub and you want other people to contribute to it, a new version of a module may come out that may break your module.
So just to be extra sure, using module fast you can use this CI command and that will commit a lock file to the to your repository.

Axel Katzur left the meeting

Grote, Justin   33:07
Which once you commit that whenever you run module fast with the dash CLI command inside that folder, it will only install the very same specific modules that you had when you were developing.
So anybody else who is doing development work will get the exact same modules you had, regardless of what your specifications say, and regardless what may have changed based on your repos and updates.

Victor (External) (Guest) left the meeting

Grote, Justin   33:27
So that makes it just to avoid some of those little inconsistencies and lets you do those kind of things.
Where?
Umm you it goes to do things like depend about type things where you can update specific things.
Paul, I saw you had hand raised Jeff, quick question.

Christian Piet joined the meeting

Grote, Justin   33:47
I can't see the shot right now.
I see like bring it up real quick.
There we go.
The the maybe missed it.
OK.
I'll move on.
I don't want to pick up too much time here.

Strong, Jeremiah left the meeting

Grote, Justin   33:58
A plan parameter has been added which basically does the same thing as the.
What if did it just because you can't suppress the?
What if output which I have a PR.
To fix that in PowerShell, but because you can't suppress what if output, you have a plan parameter, which is basically the same thing as.
What if that doesn't show the what if text, so that can be useful with module fast.
The plans that you generate, like if you do generate a plan, you can save that off anywhere.
Save it to a variable, save it to CLI XML, save it wherever and you can come back and pipe that back into install module fast our networks just like the log file where it will install only those modules based on that particular plan.
So this has kind of a very terraform like kind of view where you can plan with your modules are gonna be and then apply them and keep that state repeated.
Margo Fast also can read the requires lines of a script, so you can put in a path if you have a script that for instance has requires line that requires a module, you can point module fast.
Now directly to that script and have it install the prerequisites for that script.
There's also works for PowerShell modules, so if you have a module that has required modules and it's assembly and it's PSD one manifest, you can install module path and point it to that and it'll do the same thing the rest of these are mostly just a little bug fixes.

Steven Apel joined the meeting

Grote, Justin   35:08
If you were using Git module fast plan, that's been basically deprecated, it'll stay working, but it's not gonna get any real new features.
Basically, everything's rolled into the installed module.
Fast command that just has to do with the way that module fast bootstraps and such.
It just makes a lot more sense to have that than the separate get module, Fast Plan command and maintaining the API between the two is just too much work.

Bryan Gritton left the meeting

Grote, Justin   35:31
There's a bunch of performance improvements, bunch of documentation updates, so I'll show that real quick.

Ayan Mullick left the meeting

Grote, Justin   35:37
So if you want to get started the module fast, the easiest thing you can do is just simply do iwr Bitly slash module, just IX.
And even if you don't module fast loaded or installed, that'll touch the latest version of the module and start running it on your computer.
The the reason this bootstrap exists is to remove a dependency on, say, PS, Git or PS resource git where they have to go out fetch the module, may take awhile, and the goal here is that if you're in like GitHub actions or something, you want to get module fast.
Bootstrap as fast as possible so you can start installing your dependencies.
If we do the help on install module fast, we will see that we have a whole bunch of help in here.
So the module command itself is a lot of detailed information as well as all the different parameters that it does on what they all do and lots of individual examples to show you the different ways that module fast could work.

Pulk, Joseph left the meeting

Grote, Justin   36:26
And just to show an example of that module demo thing I got edited there, let's see the changes to that.
So we have our module file here for instance, that has a a module this where's my have acquired about this?
Parlog tracker anyhow, ohh.
As of the very latest feature there is, there is support for PS depend files and and require module files.
If you use J Coles module and it also supports PS resource git manifest.
So if you've done, try it out, the new PowerShell get version three resource required manifest where you can write a file and list out the different modules that you want it.

Janne Kujanpää joined the meeting

Grote, Justin   37:15
Now you can use module fast to install those quickly too and I'm I'm way over my time as I knew I would be so I'm just going to show one last thing here.

Vivian Thiebaut left the meeting

Grote, Justin   37:22
So I ran a test of running module fast to install all of the all the AZ modules.
All the Microsoft graph modules and all the VMware Power CLI modules and in order to do this it can do it in 7.7 seconds from scratch, no cache, anything, find all the modules, resolve the dependencies, download them, install them and be done by comparison.
PS Resource Git takes about 103 seconds, so about you know about 10 times faster and you know and then recurring occurrences, it only fetches the modules that may have changed or have updated.
So very quick and in fact I had to run this in an extremely big, powerful.
And run this in a really big powerful Azure virtual machine because it fans out all all the decompression work that has to happen to unzip all those Nuget modules had to be fanned out in order for this to actually like, not get bottlenecked.
So I'll go ahead and stop there.
Thanks very much for your time and check out module fast.
Thank you.

Jason Helmick   38:19
While Justin outstanding, I need to let the dust settle in my head for a second.

Nieto, Thomas left the meeting

Jason Helmick   38:24
That's that's pretty incredible.
So folks target your questions towards Justin on this.
It's amazing.
Great work man.
Thank you.
Hey, Ryan. Wakefield.
Brian, my friend are would you like to?
We've got the time.
So would you like to do your demo on a new version of the original test pending reboot?

Ryan.Wakefield   38:43
Yes, I will definitely do so here.

Nieto, Thomas joined the meeting

Ryan.Wakefield   38:45
It's, uh, I'll admit a little nerve.
Nervous.
Just because I've never done this so I'm like, OK.

Mullens, Justin left the meeting

Jason Helmick   38:49
Welcome.
It's all good, man. Welcome.

Ryan.Wakefield   38:52
But yeah, I can say if you if I can share my screen here I can show you guys kind of some of the rework that I've done to it.

Jason Helmick   38:59
Sure.

Ryan.Wakefield   39:02
But but those of you don't know that you know Adam, you know Adam the Automator.
He had a script out there that he put out there is on the PowerShell gallery test dash pending reboot.
You know, it's been, you know, one of the most typical staple test pending reboots.
I know there's the DSC resource as well, but with some recent work that I've been ironically enough been working to use the PowerShell Universal.
I have found some gaps in issues with the current resource, so with that being said, UH says not a presenter there Jason.

Nieto, Thomas left the meeting

Jason Helmick   39:41
Ohh, I may have to ask one of my one of my one of my friends internally to give you a presenter rights because I don't have permission to do so.

Steve Lee (POWERSHELL HE/HIM)   39:47
You just did it.

Ryan.Wakefield   39:48
Ah, there we go.

Steve Lee (POWERSHELL HE/HIM)   39:50
Try it now.

Ryan.Wakefield   39:50
Thank you.

Jason Helmick   39:51
Thanks Steve.

Ryan.Wakefield   39:51
Alright, I will shoot. Yep.

James Petty joined the meeting

Ryan.Wakefield   39:55
Alright, you should be hopefully seeing it here.

Jason Helmick   39:57
Yep, we sure can.

Ryan.Wakefield   39:58
Alright, let me clear that out.

Jason Helmick   39:59
If you can make it a little larger, that'd be great, but we could see.

Ryan.Wakefield   40:01
Yes.
So yeah, that's what I was gonna get to here.

Jason Helmick   40:03
Oh, wow, nice.

Amber Erickson joined the meeting

Ryan.Wakefield   40:05
I I'm game with however large you wanna go.
So yeah, see one of the biggest gaps that I noticed was when you went to do it, when you with the original one, when you actually ran it with in the service.
I've got 2 servers that are just.
They're dummy.
They're just jump names, and apparently it is not.
Is going to be that kind of a day with me.
Gotta gotta love it.
How it does this on live sessions but the biggest pain point is you know it's it's gonna I have, you know the server names are jacted, but what's gonna happen is as soon as it tries to connect to one of the error ones, it's just gonna kill it.
Yeah, it stops.
So if that error one was up at the very first, it would have actually stopped right away.
So obviously it wasn't handling errors well, so went through and did some work in that and so I did a little measuring with the original one and was able to get the errors handled.
So it's not, it's not going, you know, with this one, you know it's giving about 7 seconds which, you know, not too bad, pretty good against.
You know there's 86 servers technically, and granted I'm running through a monkey, so I've got a little bit of a bottleneck with, you know, some hardware VPN stuff.

Josh Corrick joined the meeting

Ryan.Wakefield   41:22
But with my newest one I will give you guys kind of a preview.
I will actually pop this one out and hey, this is the move to new window type of feature that just came out recently.
So I'm sorry I had to do that, but some of the stuff that I worked to do is, you know, obviously I create all the sessions first.
I make sure to do that and then I invoke it as a job and the jobs have actually speed this up a decent amount to where I've actually got it.
I think I dropped another two to I think 2 to 3 seconds which you know off the seven seconds.
Not too bad, so I will drop that measure in here.
Oops, that's the beach too.
I need the V2 I guess. Yeah.
So it's about, so it's about 4.398 seconds, which, you know, compared to this.

Jason Helmick   42:10
Umm.

Ryan.Wakefield   42:14
So it's at almost three full seconds off of it.
Now that's off the six servers I've got this actually where I'm gonna be doing this against 2030 different servers and we're, you know, just to kind of keep alive monitor of some of this stuff and then you know, obviously that's gonna drastically you know once you start scaling it will drastically reduce.

Silva, Rodrigo N left the meeting

Ryan.Wakefield   42:35
So I do not have this on GitHub yet.
I'm working on it.
And I needed to get all my testing done, so once I get the GitHub, you know up in public I will do so. Obviously.
I'll contact Adam with respect to him.
Of course, before I post anything I want to give him his respect and due diligence, but this is just kind of some work that I've been doing and playing around with and having fun so.

Jason Helmick   42:58
Hey, thanks, Ryan.
Really looking forward to your publishing this on GitHub and yeah, and kudos to Adam originally for putting this together.
But Ryan, congratulations.
You just presented a great demo at the PowerShell community call, so thank you very much.

Ryan.Wakefield   43:13
Well, thank you.

Jason Helmick   43:13
We really appreciate you come back and do some more.
Umm.

Ryan.Wakefield   43:17
Yep.

Jason Helmick   43:17
Folks, with that, we do have a few extra minutes left, so I do want to turn to my friend, Mr Brundage.
James, do you wanna do a demo this month or next month?

James Brundage   43:28
I got a quick one for you.

Jason Helmick   43:28
We got some time.
If you wanna go a quick one would be great.

James Brundage   43:30
If you wanted to give me some sharing rights.

Jason Helmick   43:33
Ohh.
OK, we'll get you some sharing rights.

James Brundage   43:37
I'll do the uh, pixelated version of things first.
Umm, so this is actually something I demonstrated briefly at the holiday community.
Call the one the community ran for anybody that missed that, though this has been a big, interesting development in well, PowerShell and TypeScript land over time.

Domansky, Mark left the meeting

Steve Lee (POWERSHELL HE/HIM)   43:59
James, you should be able to present now.

James Brundage   43:59
Sorry.
Yep.
OK.
So let me let a picture speaks 1000 words first.
You see that?
I just ran a JavaScript file from PowerShell out.
Yes, I got its results from there.
But you know why?
Stop that.
Let's go ahead and run a Python file too.
So yeah, the short version of implicit interpretation is exactly what's on the tin.

Nieto, Thomas joined the meeting

James Brundage   44:35
It is now possible to define a language defined that that language has an interpreter and just just like that, have it wired up in PowerShell with pipe script loaded.

Nieto, Thomas left the meeting

James Brundage   44:46
So yeah, to kind of go there and prove the point and Hello PowerShell community call.
Look this.
Is a live demo.
And just a rule of not lying here.
There we go.
So ah, this should start, you know.
Umm.
Hopefully setting off some nice.
Alarm bells and everybody's heads about what PowerShell might now be capable of for the last 1617 years, it's been PowerShell as a .net scripting language, and at the edges of the .net ecosystem, PowerShell sort of starts to not be as helpful.
Well, that's gone now.
At this point, we can now pretty solidly break down the walls between the PowerShell ecosystem and any other ecosystem that might hypothetically exist.
Uh, it's actually done in a very small series of files.
Looks like we do have enough time on the clock to get there, so I'll just walk through how this is working.
OK, so I I guess I have one more to open up and that is the PS1.
All right.
So let's start with the JavaScript language definition.
One of the big things and you pipe script land is you can now define namespace functions.
This is actually going to define a function called JavaScript DOT language, which by the way, the PowerShell Gallery does not complain when you import that and it will be really nice if it stays that way.
Umm JavaScript Dot language is just basically a property bag here and all I've got here is file patterns and metadata basically saying hey, I'm a case sensitive language, some information about how I template and then the big thing here is just one line saying hey, my interpreter is node and this is a really major important point here.

Clint Rutkas left the meeting

James Brundage   46:46
I'm not building an interpreter for each of these languages, I'm just letting PowerShell implicitly call down to the ones that exist.
Big deal.

Nieto, Thomas joined the meeting

James Brundage   46:57
This foreach object is what's going to happen with each output inside of a template, there should be another one coming for inside of interpreters.
Ah, this should also, at least in theory, work with all the feedback provider stuff we have going on, because it's just making hello Dot JS an actual workable command.
How is it doing that?
Well, the first part of the pairings here, what's actually going on is, and please don't break this PowerShell team.
We've got a pre command lookup action here.
Recommend lookup action happens before you call any command.
See the problem is without this hello dot JS is actually a PowerShell command.

Nieto, Thomas left the meeting

James Brundage   47:37
It's just a PowerShell command that's gonna open up hello Dot JS with a default file association.
This is giving us the ability to have a file association for PowerShell, specifically cross platform.
Alright, so in the pre command lookup action I have to do a little bit of smart trickery here because I don't wanna end up in a recursive hell so I have to actually look up two commands I'm gonna need new module import module now for those that aren't aware you can use these two to do a post loading trick to basically create commands on the fly.
Also have to figure out the command didn't already exist, so I'm just gonna grab a pointer to get all potential commands.
Then I have to figure out hey, do I have interpreters?
Cool.
OK.
Do I have one for the file?
Yes, all right.
If I can do that, then create if the command does not exist, go create an alias, export the Member and go.
Say I have an interpreter and go return the new interpreter's name, which is always going to be invoked interpreter.
So let's go look at invoke interpreter.
It's a really simple function.
I mean like, what are we 45 lines and all it's really saying is that look, figure out what I was called when I went was called.
If I was called by Ryan, real name, forget it.
There weren't interpreters.
Forget it.
If I couldn't find a match, forget it.
Otherwise, pick out my interpreter command.
Any leading arguments and call me with Splatting.
That's it.
In fact, if you actually crack open like a node module and you look at any of the PS1 files in their directories, this is basically what they have.
If you run NPM, this is the PS1 that's running basically, so this is a clean, you know, officially sanctioned bridge into whatever other language ecosystems there are.
4 file is also pretty stupid.
It basically has a couple of ways to exclude, you know because you don't wanna always do this.
You might want to exclude a pattern or path and then just basically goes and checks OK do I have a language name?
Do I have a file pattern doesn't match cool.
All right.
You might be an interpreter.

Brosent, Dominik left the meeting

James Brundage   49:40
There is also got a little bit cheeky and created 2 new variables that are PS interpreters.
Look at that nice list, and if that wasn't enough languages to play with already, there are also is PS language.
Look at that nice list.
These are all the languages TypeScript can now template.
I didn't know.
I had a nice base two number right up there.
So yes, Luo was on the list.

Grote, Justin left the meeting

James Brundage   50:09
Uh, I think you know this flash by really quickly at this point.
These are mostly public bits.
I say mostly because there is one bug fix RE templates and interpretation that is in the hopper come out and be next.
Umm, I think also if I were to take a little feature that should move.
I don't know whether to call it's upstream or downstream from pipe script to PowerShell.
I think languages support might be it, because I think in listen to her, predation should be huge for all of us and again underline the point for our whole careers and PowerShell.
We've been boxed into the .net ecosystem and that's gone now.

James Petty left the meeting

James Brundage   50:57
We can now talk to every other ecosystem that exists fluently.
Questions.
Comments, feedback.
Cheers.
Cheers.
What have you done?
My God, or at least let's go for the, you know, real key one, which is all your base belong to us.
And one more time for good measure.
Ohh I gotta save the file first.

Jason Helmick   51:28
Hey.

James Brundage   51:30
All right. Good.
We good.

Jason Helmick   51:33
We're great.
Thanks, James.
This is awesome, boy.
Also thanks to Ryan Yates for putting in the link to the GitHub for pipe script.
This is amazing and I I I just have to say it's been a really amazing day for, for demos.
Today I'm kind of stunned.
This is awesome, James.
Thank you so much for taking the time to demonstrate this for everyone and go ahead and check out pipe script folks.
There's a link in the chat as we start to wind down this first community call of the year.

James Brundage   52:00
Yeah.

Jason Helmick   52:05
Please, we've been answering questions in the chat as we were going along, but if you have an additional question, please drop it in.
And also like to ask the members of the PowerShell team if there's anyone that has anything else that they'd like to say as we start to wrap up today.

James Brundage   52:25
I guess I could go another 30 seconds if you want your stocks fully blown off.

Jason Helmick   52:31
30 seconds go.

James Brundage   52:33
OK, not counting the time to reshare.
Uh.
Or get back open.
That's ohh one other fun thing that there is running in there is Docker support and support for starting custom jobs.

Josh Hendricks left the meeting

James Brundage   52:51
So I just run this one a little little recently.

Darren Kattan left the meeting

James Brundage   52:53
Let's see if my history is with me.
Uh, looks like no, so let's go ahead and get job.
OK, fine.
So here we've got this job running on localhost.
Which is probably going to complain.
OK, there is my little image coming back.
I'm going to go open up a window here.
So this is a Docker service running a micro server or a Docker image running a microserver, running pipe script, running PowerShell, and for really crazy bonus points, let's see, we've got our job.

Nieto, Thomas joined the meeting

Patrick N left the meeting

James Brundage   53:41
Let's go ahead and look at our PS node action.

Greg Malleus left the meeting

James Brundage   53:44
That's all it's gonna be rung.
Let's do something crazy and brave.

Terry Dow left the meeting

James Brundage   53:48
Hey community call.
Yep, process.
Did did piped you stop process?
OK, so I'm now changed my web server to do this.

Amber Erickson left the meeting

James Brundage   54:06
Ohh let me do one more thing which is.
Yes.
No right output.
Let's give it a random number.
OK.
So we're going to say hi to the community call when we hit this web service writing output from our job, which we're running. OK.
And then we're gonna try to stop process.
So I'm gonna go open that up again.
Hey, community call my hit refresh.
It's not dead.
It's not dead.
And if I get job ID to receive job keep.

Greg Malleus joined the meeting

James Brundage   54:49
So not only do you have microservice, not only is it a background job that can actually allow us to return both the client and the server, but if you throw it into a Docker image and you tell it to kill itself, it just won't die.
Socks on anybody.
Socks off now.
Alright, we good.

Michael Anstadt left the meeting

Jason Helmick   55:15
That's awesome, James.
Thank you.
It was worth the additional 30 seconds or so that was pretty screaming.
All righty, folks.

James Brundage   55:23
Thank you.

Jason Helmick   55:24
Last call for some questions and to the PowerShell team that they have anything they'd like to say.
I'd like to thank everyone for attending.
We're getting epic audiences at these, so please tell your friends come.
We want to hear from you when we'd like your feedback so.
Well, this is awesome.
Thank you, everyone.
And with that silence, let's go ahead and go with this in in the call, you have a great January month.

Christian Piet left the meeting

Jason Helmick   55:50
Thanks everyone for attending the PowerShell community call and we'll see you again next month in February. Chairs.

