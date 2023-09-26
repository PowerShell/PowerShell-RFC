[Thursday 9:24 AM] Jason Helmick

Hello and welcome to the Spetember 2023 PowerShell Community Call -  where everyone is welcome!  We will get started shortly.

This meeting is recorded. Recordings will be upload about a week after the call.  You can find our past Community Call’s on our YouTube channel: https://www.youtube.com/channel/UCMhQH-yJlr4_XHkwNunfMog

Please remember our Code of Conduct: https://opensource.microsoft.com/codeofconduct/

Please review our contributing guide: https://github.com/PowerShell/PowerShell/blob/master/.github/CONTRIBUTING.md

Team Room 40/3125 (8)
Welcome everyone to the September community, PowerShell and S open SSH community call.
0:2:12.750 --> 0:2:15.800
Team Room 40/3125 (8)
You can see all of us joining from a conference room here.
0:2:15.810 --> 0:2:21.130
Team Room 40/3125 (8)
So we'll be passing, passing it to each other for the agendas.
0:2:22.40 --> 0:2:25.580
Team Room 40/3125 (8)
Like I said earlier, the agenda is posted in the discussion tab.
0:2:27.650 --> 0:2:31.260
Team Room 40/3125 (8)
And we have a pretty busy agenda today.
0:2:31.270 --> 0:2:33.780
Team Room 40/3125 (8)
So I guess we'll probably jump right into it.
0:2:33.790 --> 0:2:38.120
Team Room 40/3125 (8)
The first thing we wanna cover is the PowerShell adapters release.
0:2:38.130 --> 0:2:44.470
Team Room 40/3125 (8)
So this is you may have previously seen this as being referred to as JSON adapters.
0:2:44.480 --> 0:2:48.70
Team Room 40/3125 (8)
We have changed the name and released it to be under the PowerShell.
0:2:48.110 --> 0:2:55.710
Team Room 40/3125 (8)
The Microsoft Dot PowerShell dot PS adapters name for the module and I will just do a quick demo of this.
0:2:55.720 --> 0:3:0.240
Team Room 40/3125 (8)
Again umm, if I can find my teams window.
0:3:0.250 --> 0:3:0.980
Team Room 40/3125 (8)
Hang on one second.
0:3:4.440 --> 0:3:10.280
Team Room 40/3125 (8)
So this is good being in the conference room, I can verify that I am actually sharing my screen so.
0:3:12.160 --> 0:3:15.30
Team Room 40/3125 (8)
So the PowerShell adapters is very similar.
0:3:15.100 --> 0:3:32.200
Team Room 40/3125 (8)
Not much has changed for the experience with the module, but using the latest PowerShell preview you can come on import so you can see this is now the full name of the module and just a reminder so this uses.
0:3:34.870 --> 0:4:2.800
Team Room 40/3125 (8)
This identifies these things called adapters, which are just PowerShell scripts in your path that can be used to convert native commands to PowerShell objects, and so if I do something like you name dash A you can see the feedback provider triggers and kind of gives me a suggestions that says hey you can use JC which is a JSON converter tool that I have installed on my system as well as a you name adapter which I have created myself.
0:4:3.530 --> 0:4:6.890
Team Room 40/3125 (8)
So then simply you can see what the.
0:4:10.890 --> 0:4:13.830
Team Room 40/3125 (8)
Adapter looks like it's very simple.
0:4:13.840 --> 0:4:19.920
Team Room 40/3125 (8)
I just take an input, pass it to JC again and convert it from JSON, but you can make more complex adapters.
0:4:22.420 --> 0:4:27.250
Team Room 40/3125 (8)
You know, as you wish, just using a very simple PowerShell scripting stuff.
0:4:27.380 --> 0:4:35.120
Team Room 40/3125 (8)
We've posted a number of blog posts about this, so I suggest if you are interested in learning more about them, to check out those blog posts.
0:4:36.830 --> 0:4:38.340
Team Room 40/3125 (8)
But yeah, just wanted to announce that.
0:4:40.990 --> 0:4:41.260
Victor
Yeah.
0:4:38.350 --> 0:4:41.620
Team Room 40/3125 (8)
So going forward, we referring to them as PowerShell adapters.
0:4:44.450 --> 0:4:49.40
James Brundage
To clarify, is this like fully baked and released in your mind at this point?
0:4:49.850 --> 0:4:52.280
James Brundage
Is this still marked as an experimental feature?
0:4:52.290 --> 0:4:52.580
James Brundage
Are there?
0:4:52.590 --> 0:4:55.400
James Brundage
Are there any other buy INS anybody needs to do to try it out?
0:4:56.300 --> 0:5:5.510
Team Room 40/3125 (8)
You so this is reliant on feedback providers, which are an experimental feature still, and there's still gonna be an experimental feature through 74 umm.
0:5:5.920 --> 0:5:16.660
Team Room 40/3125 (8)
And as far as I guess the module itself, we feel like it's in a pretty good state where, I mean we're we're we're still very open to changes in the future around it, but uh.
0:5:18.300 --> 0:5:19.450
Jim Truher
We still have work.
0:5:19.500 --> 0:5:22.870
Jim Truher
I still have work to do on on this module.
0:5:22.940 --> 0:5:30.950
Jim Truher
Remember, it's a module, so it relies on on the the predictors and feedback functionality.
0:5:30.960 --> 0:5:34.490
Jim Truher
And as Steven said, it is a an experimental feature.
0:5:34.500 --> 0:5:37.950
Jim Truher
This module is of course always going to be optional.
0:5:38.60 --> 0:5:39.460
Jim Truher
That's our current plan.
0:5:40.160 --> 0:5:41.160
Jim Truher
So you can use it or not.
0:5:44.490 --> 0:5:44.760
Team Room 40/3125 (8)
No.
0:6:0.460 --> 0:6:0.990
Jim Truher
Perhaps yeah.
0:5:44.160 --> 0:6:8.40
James Brundage
OK, I think it might be valuable to see if we can set up some time, Jim, to kind of go over some of the feedback that I've already had and Steven to do a separate feedburner follow up outside of this meeting just because, yeah, don't want to burn all the time, but I do still want to, you know, make sure that this comes out as perfectly as it can be.
0:6:8.50 --> 0:6:10.900
James Brundage
I appreciate the rename quite a bit.
0:6:11.370 --> 0:6:14.600
James Brundage
It also seems like it was pretty well received by the community.
0:6:21.780 --> 0:6:25.500
Team Room 40/3125 (8)
I always after like I would.
0:6:14.990 --> 0:6:27.640
James Brundage
I would also just love it if we can get to the point where it is at least possible to say, you know, I always want to use the adapter like I don't ever want you name to return naturally.
0:6:35.840 --> 0:6:36.10
Jim Truher
Yeah.
0:6:27.650 --> 0:6:36.100
James Brundage
I always wanna call the adapter cause I always wanted it as objects, so I think that would be a good thing to kind of get into offline and I'm shutting up now.
0:6:38.80 --> 0:6:38.510
Team Room 40/3125 (8)
Cool.
0:6:38.520 --> 0:6:38.810
Team Room 40/3125 (8)
OK.
0:6:38.820 --> 0:6:40.950
Team Room 40/3125 (8)
Well, yeah, we'll follow up offline about that.
0:6:41.490 --> 0:6:45.170
Team Room 40/3125 (8)
Umm, the next thing on our agenda, see.
0:6:46.140 --> 0:6:47.160
Team Room 40/3125 (8)
See, Carlos is here.
0:6:47.170 --> 0:6:49.430
Team Room 40/3125 (8)
Carlos, are you and and Clint.
0:6:50.250 --> 0:6:50.860
Carlos Zamora
Yeah, we're here.
0:6:51.970 --> 0:6:52.340
Team Room 40/3125 (8)
Cool.
0:6:52.390 --> 0:6:53.110
Team Room 40/3125 (8)
I'll take it away, guys.
0:6:53.120 --> 0:6:54.40
Team Room 40/3125 (8)
I'll pass it over to you.
0:6:55.790 --> 0:6:56.520
Carlos Zamora
Awesome.
0:6:56.630 --> 0:7:4.370
Carlos Zamora
So yeah, I'm here to talk about a a new PowerShell module we've been working on called command not found.
0:7:6.430 --> 0:7:11.660
Carlos Zamora
And basically the idea behind it is, you know it well, I'll just show it to you.
0:7:11.670 --> 0:7:21.260
Carlos Zamora
It's a lot easier that way, so the idea is just if if you type in a command and guess what, you don't have it installed like Visual Studio VS code.
0:7:21.900 --> 0:7:32.990
Carlos Zamora
Umm, we actually just recommend three wind gut what to install and so we kind of do a quick query on the back end and try to get that to you.
0:7:33.620 --> 0:7:35.190
Carlos Zamora
And so we're taking advantage of the.
0:7:36.380 --> 0:7:43.30
Carlos Zamora
Feedback predictor and the no the feedback provider and the command predictor.
0:7:43.740 --> 0:7:50.920
Carlos Zamora
So when you start typing in your result, it actually autocompletes it for you, which is pretty cool.
0:7:51.830 --> 0:7:52.360
Carlos Zamora
Umm.
0:7:53.470 --> 0:7:53.640
Team Room 40/3125 (8)
Thank.
0:7:54.810 --> 0:7:55.210
Team Room 40/3125 (8)
Go on.
0:7:53.110 --> 0:7:58.870
Carlos Zamora
Then I just wanted to show off a little bit of that too, like ohh yes, go ahead Ryan.
0:8:1.630 --> 0:8:2.640
Ryan Yates
So just a quick one.
0:8:2.650 --> 0:8:14.860
Ryan Yates
Well, this also then tie into the PowerShell gallery because not not all of these names are gonna be stuff that we're gonna have from winpcap will be stuff from the PowerShell country as well.
0:8:17.790 --> 0:8:17.980
Carlos Zamora
Yeah.
0:8:17.990 --> 0:8:18.260
Carlos Zamora
Go for it.
0:8:15.450 --> 0:8:22.250
Steve Lee (POWERSHELL HE/HIM)
Look, can I answer that one PowerShell PS resource get will have this capability in the future.
0:8:25.290 --> 0:8:26.760
Team Room 40/3125 (8)
As today or a.
0:8:22.890 --> 0:8:34.380
Steve Lee (POWERSHELL HE/HIM)
What is missing that Winget has today is a offline ability to query, so look for that in the future, but this is this will be set when GIT side will be separate from the gallery side.
0:8:36.500 --> 0:8:36.990
Carlos Zamora
Yeah.
0:8:37.30 --> 0:8:42.870
Carlos Zamora
And so there's this is just doing everything on the win get side and we try to streamline as much as possible.
0:8:42.920 --> 0:8:46.610
Carlos Zamora
So, like if we did a win that search code.
0:8:51.820 --> 0:8:52.60
Victor
Other.
0:8:48.960 --> 0:8:52.150
Carlos Zamora
Yeah, you can see we got a lot of options here.
0:8:52.160 --> 0:9:12.160
Carlos Zamora
So we try to leverage the metadata that win get has to give you kind of the best results possible and this also works with the some of the like if you actually use it as a command like NPM install, it knows that you were talking about NPM specifically and so we're able to pull that up.
0:9:13.840 --> 0:9:15.260
Carlos Zamora
Ohh and I can do a terminal one too.
0:9:16.300 --> 0:9:18.120
Carlos Zamora
Yeah, would give you a good amount of results.
0:9:19.560 --> 0:9:26.990
Carlos Zamora
I'm but one of the few things I just wanted to say on it is this is still a work in progress right now.
0:9:27.0 --> 0:9:36.670
Carlos Zamora
We have it a draft PR open in the power toys repo so you can take a look at all the code that's behind all this and what our fall back system is.
0:9:37.230 --> 0:9:39.240
Carlos Zamora
Umm, it's pretty straightforward.
0:9:39.250 --> 0:9:42.810
Carlos Zamora
We'd love to get some feedback from the community on it.
0:9:45.680 --> 0:9:46.180
Team Room 40/3125 (8)
He wanted to.
0:9:44.80 --> 0:9:46.470
Carlos Zamora
Client, is there anything you you wanted to add on this?
0:9:48.720 --> 0:9:48.900
Team Room 40/3125 (8)
Yeah.
0:9:48.410 --> 0:9:53.120
Clint Rutkas
Yeah, I think the big thing for us is understanding what, what could we do better.
0:9:53.130 --> 0:9:54.480
Clint Rutkas
We want to be learned at ALS.
0:9:54.490 --> 0:9:56.30
Clint Rutkas
We're coming in humble.
0:9:56.70 --> 0:10:4.180
Clint Rutkas
If there's something that's not working correctly or like we're not doing it best practice, please mix aware there is an open pull request here.
0:10:4.190 --> 0:10:15.140
Clint Rutkas
We're waiting for a .net aid to come out and then so we're targeting maybe the November, December release maybe for for this being a inside of power toys.
0:10:16.170 --> 0:10:25.560
Clint Rutkas
So whatever we can do to make this the best thing for for everyone, we wanna learn from from you all because you are the the experts are PowerShell.
0:10:27.760 --> 0:10:31.80
Clint Rutkas
We just play one on TV correctly.
0:10:30.600 --> 0:10:31.90
Carlos Zamora
You.
0:10:31.530 --> 0:10:32.40
Clint Rutkas
I didn't think.
0:10:34.330 --> 0:10:39.700
Clint Rutkas
And if this is a great way of of interacting like, please let us know.
0:10:39.710 --> 0:10:42.630
Clint Rutkas
That's the great thing with power toys is we can incubate really quickly.
0:10:43.430 --> 0:10:43.590
Carlos Zamora
Yeah.
0:10:45.170 --> 0:10:53.640
Steve Lee (POWERSHELL HE/HIM)
So Clint and Carlo, one comment for you guys and you guys have to decide if you wanna do this, but .net releases are release candidate releases or go live.
0:10:53.970 --> 0:10:57.190
Steve Lee (POWERSHELL HE/HIM)
So they technically supported in production, so you don't have to wait for the.
0:10:56.310 --> 0:10:57.220
Team Room 40/3125 (8)
And so you don't like.
0:10:59.30 --> 0:10:59.310
Victor
Yeah.
0:11:0.130 --> 0:11:0.360
Team Room 40/3125 (8)
It.
0:10:59.840 --> 0:11:1.360
Clint Rutkas
The day I learned.
0:11:4.480 --> 0:11:4.800
Clint Rutkas
Uh.
0:11:4.450 --> 0:11:11.210
Steve Lee (POWERSHELL HE/HIM)
So it's it's really up to you, but they I, unless they've changed their plans, they're release candidates have always been go live so.
0:11:14.130 --> 0:11:14.830
Darren Kattan
The question.
0:11:13.660 --> 0:11:15.260
Clint Rutkas
I did not know that there.
0:11:17.500 --> 0:11:17.600
Carlos Zamora
I.
0:11:17.820 --> 0:11:18.110
Darren Kattan
Hello.
0:11:18.120 --> 0:11:19.370
Darren Kattan
Question call.
0:11:17.70 --> 0:11:20.360
Grote, Justin
Yeah, for .net I I've seen release notes.
0:11:20.370 --> 0:11:21.100
Grote, Justin
This one it.
0:11:21.50 --> 0:11:21.240
Sid Moore
That's.
0:11:21.110 --> 0:11:23.400
Grote, Justin
Is all the that that's still true so.
0:11:27.290 --> 0:11:28.560
Steve Lee (POWERSHELL HE/HIM)
They don't have to wait till November.
0:11:30.870 --> 0:11:31.180
Carlos Zamora
Awesome.
0:11:36.620 --> 0:11:36.750
Darren Kattan
No.
0:11:32.700 --> 0:11:57.970
Carlos Zamora
But yeah, we're we're working on getting packed toys up to .net 8 and we're hoping this will give a good clear story from, you know, being able to install your whatever packages you're missing, using winget to export your configuration, and then hopefully you're able to import it using Dev home and that whole scenario just creating a better ecosystem for y'all.
0:12:0.630 --> 0:12:0.890
Darren Kattan
Uh.
0:11:59.690 --> 0:12:1.200
James Brundage
Yeah, it looks really good.
0:12:1.630 --> 0:12:4.920
James Brundage
I hope there will be beyond winget stuff.
0:12:4.930 --> 0:12:12.490
James Brundage
A human editable form of what suggestion possibilities might exist for commands and modules.
0:12:13.250 --> 0:12:26.610
James Brundage
Umm that is like it'd be nice if we have a reference file somewhere that and be programmatically looked at and individually edited to say I know what blah.exe is.
0:12:26.620 --> 0:12:32.170
James Brundage
I know how you can install it, so yeah, not everything's going to be win again.
0:12:32.180 --> 0:12:34.710
James Brundage
Not everything's gonna be gallery as long as it's editable.
0:12:34.780 --> 0:12:35.900
James Brundage
This should be fantastic.
0:12:37.890 --> 0:12:38.130
Darren Kattan
It.
0:12:36.830 --> 0:12:45.790
Clint Rutkas
I would love to to talk to you in the future and like kind of like how you'd work with this, not in a giant setting like this, but the like.
0:12:49.370 --> 0:12:49.500
Team Room 40/3125 (8)
What?
0:12:52.230 --> 0:12:53.10
Team Room 40/3125 (8)
But perfect.
0:12:45.800 --> 0:12:54.500
Clint Rutkas
You're raising some interesting scenarios like we're thinking in terms of, hey, I set up a new computer and I need VS code.
0:12:54.510 --> 0:13:9.900
Clint Rutkas
You type in terminal code cause that you'd expect that to pop up and you don't get it, so it's just like, uhm, how do we get you up and going as fast as possible is one of our teams internal mantras.
0:13:10.230 --> 0:13:16.630
Clint Rutkas
So I'd love to to to understand how you want that file, how that file would work like.
0:13:15.770 --> 0:13:18.530
Team Room 40/3125 (8)
Work, but the first.
0:13:16.640 --> 0:13:19.820
Clint Rutkas
How would it get on your computer in the 1st place for this kind of scenario?
0:13:21.200 --> 0:13:22.820
James Brundage
Uh, fair enough.
0:13:25.190 --> 0:13:25.370
Clint Rutkas
Yep.
0:13:23.20 --> 0:13:30.470
James Brundage
If you wanted to take that offline, you can drop your email or something in chat or something any other way I can contact you.
0:13:30.480 --> 0:13:30.670
James Brundage
I'll.
0:13:30.950 --> 0:13:31.150
Clint Rutkas
Yep.
0:13:30.680 --> 0:13:31.300
James Brundage
I'll get in touch.
0:13:33.130 --> 0:13:39.610
Darren Kattan
Do feedback providers only pop up after you've like entered a command and like tried to execute it?
0:13:42.250 --> 0:13:43.380
Steve Lee (POWERSHELL HE/HIM)
So let me answer that one.
0:13:43.390 --> 0:13:45.620
Steve Lee (POWERSHELL HE/HIM)
So the way so there's two different things.
0:13:49.300 --> 0:13:50.270
Team Room 40/3125 (8)
And before they get into.
0:13:46.150 --> 0:13:55.600
Steve Lee (POWERSHELL HE/HIM)
Feedback providers require something that happened before they get into, but predictors are the opposite where they can show an inline prediction.
0:13:55.610 --> 0:13:58.390
Steve Lee (POWERSHELL HE/HIM)
So what what scenario are you trying to figure out?
0:14:0.230 --> 0:14:2.360
Darren Kattan
What more?
0:14:4.570 --> 0:14:6.160
Darren Kattan
Out, you know.
0:14:6.170 --> 0:14:6.400
Darren Kattan
Right.
0:14:6.410 --> 0:14:11.620
Darren Kattan
Right now, the state of the existence is that MONICO has its own sort of like Intellisense.
0:14:11.710 --> 0:14:19.960
Darren Kattan
Then we've got PowerShell editor services that's using your input completions, and then we've got this feedback provider thing that's also doing it.
0:14:20.70 --> 0:14:30.200
Darren Kattan
And there's copilot involved, and sometimes it gets really confusing when I'm like in a script and then, like, you know, editor services pops up, but its thing, but then the shadow text pops up.
0:14:30.290 --> 0:14:40.680
Darren Kattan
And I'm just wondering if there any sort of like cohesion planned or is that kind of the purpose of what you guys are doing is to try to maybe bake this copilot stuff.
0:14:40.690 --> 0:14:46.480
Darren Kattan
They're ultimately underneath this new feedback provider interface with the completion provider interface.
0:14:46.490 --> 0:14:49.670
Darren Kattan
So that way it's not like VS code copilot.
0:14:49.680 --> 0:14:52.530
Darren Kattan
Extension fighting with editor services.
0:14:52.550 --> 0:14:52.790
Team Room 40/3125 (8)
So.
0:14:55.700 --> 0:14:57.480
Steve Lee (POWERSHELL HE/HIM)
So VS code specifically.
0:14:58.830 --> 0:15:1.170
Steve Lee (POWERSHELL HE/HIM)
Umm, I think that is going to be a question for them.
0:15:1.240 --> 0:15:14.790
Steve Lee (POWERSHELL HE/HIM)
If you're concerned is how to present all these different ways to get assistance, I can tell you that one of the things that we are we decided on the feedback provider interface model is that we're not supporting copilot or AI type of thing.
0:15:14.800 --> 0:15:20.380
Steve Lee (POWERSHELL HE/HIM)
It just takes too long and we we don't wanna do is hold the prompt for too long before we turn our result.
0:15:21.490 --> 0:15:26.510
Steve Lee (POWERSHELL HE/HIM)
Umm, there are other things that we're not talking about today about AI, but maybe in the future.
0:15:28.290 --> 0:15:28.620
Team Room 40/3125 (8)
Yep.
0:15:27.960 --> 0:15:28.690
James Brundage
Yeah.
0:15:28.500 --> 0:15:30.120
Darren Kattan
OK, that's that, that.
0:15:30.730 --> 0:15:31.760
Team Room 40/3125 (8)
That here.
0:15:28.700 --> 0:15:57.150
James Brundage
I don't mean to beat the dead horse here, but if if you are looking at that sort of delay based problem, this is where we're having an event oriented feedback provider might be more helpful just because at this point if you are saying you're gonna have to Pend prompt for all the feedback and not have a channel that can come back with more feedback, then we're only going to be able to give feedback that we can give just like that.
0:15:57.680 --> 0:16:1.230
James Brundage
So yeah, let's let's revisit that again too.
0:16:2.180 --> 0:16:6.50
Team Room 40/3125 (8)
Yeah, I'll reach out to you, James, and we can set up some time to to chat more about this.
0:16:8.430 --> 0:16:8.590
Team Room 40/3125 (8)
Cool.
0:16:9.350 --> 0:16:16.940
Team Room 40/3125 (8)
So one, so one question, while we're still on the win, get feedback provider topic about do you guys plan to post this on the gallery on the PowerShell Gallery?
0:16:19.280 --> 0:16:20.610
Carlos Zamora
Yeah, definitely.
0:16:21.40 --> 0:16:22.750
Carlos Zamora
Right now it's development.
0:16:22.760 --> 0:16:27.560
Carlos Zamora
We're wrapping up the uh PowerShell module itself.
0:16:28.260 --> 0:16:31.510
Carlos Zamora
The next step is to work on that.
0:16:31.520 --> 0:16:38.580
Carlos Zamora
Polishing the power play side of it so that we have a clean like installation method through that, but either way it's gonna be on the gallery.
0:16:40.690 --> 0:16:40.960
Team Room 40/3125 (8)
Awesome.
0:16:41.840 --> 0:16:45.230
Team Room 40/3125 (8)
Uh three, I will.
0:16:47.400 --> 0:16:48.840
Steve Lee (POWERSHELL HE/HIM)
Yeah, I'm.
0:16:45.240 --> 0:16:49.290
Team Room 40/3125 (8)
So then I think pass it the next day item is Steve.
0:16:53.500 --> 0:16:55.100
Steve Lee (POWERSHELL HE/HIM)
But I think we're about experimental features.
0:16:55.340 --> 0:16:55.670
Team Room 40/3125 (8)
Yeah.
0:16:55.680 --> 0:16:56.40
Team Room 40/3125 (8)
Yeah, sorry.
0:16:58.70 --> 0:16:58.680
James Brundage
The fun stuff.
0:16:58.400 --> 0:17:0.750
Team Room 40/3125 (8)
Experiment features for seven four what?
0:17:2.300 --> 0:17:6.690
Steve Lee (POWERSHELL HE/HIM)
Going to show that, alright, so this is an issue and an issue.
0:17:6.700 --> 0:17:12.600
Steve Lee (POWERSHELL HE/HIM)
This is a discussion in the PowerShell repo of the results of the PowerShell community triaging.
0:17:14.610 --> 0:17:17.910
Team Room 40/3125 (8)
It's critical 74 million, something crazy.
0:17:13.520 --> 0:17:18.220
Steve Lee (POWERSHELL HE/HIM)
What was experimental in seven, four and what we intend to make stable.
0:17:19.260 --> 0:17:21.190
Steve Lee (POWERSHELL HE/HIM)
Uh, can the conference room mute for a second?
0:17:21.340 --> 0:17:22.610
Steve Lee (POWERSHELL HE/HIM)
And I'm getting echo from you guys.
0:17:25.640 --> 0:17:25.820
Victor
Yeah.
0:17:23.140 --> 0:17:25.970
Steve Lee (POWERSHELL HE/HIM)
OK, so I'm gonna just quickly go over this.
0:17:25.980 --> 0:17:34.240
Steve Lee (POWERSHELL HE/HIM)
And by the way, if you guys have feedback, if you think something's not ready or if you have questions or whatever, feel free to just add it as comments in this discussion.
0:17:34.250 --> 0:17:44.550
Steve Lee (POWERSHELL HE/HIM)
We'll take a look at that, but currently I'm quickly go over these real quick so constraint audit logging so this is where if using so WWDC is the Windows Defender application control.
0:17:44.740 --> 0:17:50.600
Steve Lee (POWERSHELL HE/HIM)
So if you're using that one of the problems was that with the going without going to the weeds.
0:17:50.610 --> 0:17:59.520
Steve Lee (POWERSHELL HE/HIM)
Basically we have this thing called constraint language mode and we have lockdown mode and people get confused by these two things and it's not simply that if you sign your script, things just magically work.
0:18:0.400 --> 0:18:0.970
Victor
Wrong.
0:17:59.530 --> 0:18:3.930
Steve Lee (POWERSHELL HE/HIM)
There are other how your script interacts with other things, whether they're signed or not.
0:18:3.980 --> 0:18:19.870
Steve Lee (POWERSHELL HE/HIM)
Also has behavior changes, so the the logging here basically allows you to have WDAC audit mode and then what will happen is that everything will work because it actually runs in full language mode, but we will actually do the additional check and say if you were actually in lockdown, what would have actually happened.
0:18:20.240 --> 0:18:28.840
Steve Lee (POWERSHELL HE/HIM)
We have prevented this from working stuff like that, so this should make it easier for administrators trying to enable WWDC enforcement mode to know what actually is gonna happen via audit.
0:18:29.700 --> 0:18:32.700
Steve Lee (POWERSHELL HE/HIM)
Umm this is the table header.
0:18:32.710 --> 0:18:47.900
Steve Lee (POWERSHELL HE/HIM)
So basically for any partial table, if the heading is actually not a property on the object, but it's something that's synthetic for formatting, then it has a different rendering and so in this allows you to change that declaration as well.
0:18:48.870 --> 0:19:2.500
Steve Lee (POWERSHELL HE/HIM)
Uh, the PS Windows Native command are passing has gone through multiple iterations trying to fix things that come up, but basically based on looking at telemetry, it looks like people have not been changing it back to legacy.
0:19:2.510 --> 0:19:5.870
Steve Lee (POWERSHELL HE/HIM)
So we feel like this is stable enough that we want this to be default in seven, four.
0:19:7.450 --> 0:19:22.970
Steve Lee (POWERSHELL HE/HIM)
Umm, the Native Action command or action preference is also something that seems like we've had enough people testing it that this is where if you have a native command and it emits a non 0 exit code, then we interpret that as failure and we will synthetically generate a PowerShell error.
0:19:22.980 --> 0:19:27.40
Steve Lee (POWERSHELL HE/HIM)
That way you can handle a native commands and commandlet error handling in a similar way.
0:19:27.750 --> 0:19:46.360
Steve Lee (POWERSHELL HE/HIM)
Keep in mind that although this feature is going to be moved to stable, you'll still be false by default, meaning that by default there's no behavioral change and you have to actually explicitly set the reference variable to true so that native non 0 exit goes act the same as partial errors, and then the last one is the native pipe.
0:19:46.850 --> 0:19:50.840
Steve Lee (POWERSHELL HE/HIM)
So this will allow you to do something like curl, pipe to tar and we'll now work.
0:19:51.70 --> 0:19:59.210
Steve Lee (POWERSHELL HE/HIM)
Whereas before PowerShell was in the middle of this and tried to interpret binary output as text and it's lossy, so that didn't work.
0:19:59.220 --> 0:20:6.330
Steve Lee (POWERSHELL HE/HIM)
So these are the ones we've looked at based on the feedback based on telemetry and we believe these are stable enough to these become stable.
0:20:6.340 --> 0:20:17.850
Steve Lee (POWERSHELL HE/HIM)
And for anything that's not in this list, including feedback provider, for example, feedback provider interface will stay exponential through some four, and we'll reevaluate 375, which will be next year or maybe December.
0:20:17.860 --> 0:20:19.490
Steve Lee (POWERSHELL HE/HIM)
We'll see that.
0:20:19.500 --> 0:20:21.100
Steve Lee (POWERSHELL HE/HIM)
I think that covers and this is.
0:20:21.130 --> 0:20:26.20
Steve Lee (POWERSHELL HE/HIM)
A questions I think that covers my section here.
0:20:27.490 --> 0:20:27.680
Victor
The.
0:20:31.360 --> 0:20:31.770
Team Room 40/3125 (8)
Cool.
0:20:32.520 --> 0:20:38.120
Team Room 40/3125 (8)
I think we got now some updates from PS Resource, Git awesome.
0:20:38.520 --> 0:20:44.870
Team Room 40/3125 (8)
I'm going to paste building to the blog post in the chat, but on September 7th.
0:20:44.880 --> 0:20:49.790
Team Room 40/3125 (8)
So I think 2 weeks ago we released the RC for PS resource debt.
0:20:50.260 --> 0:21:1.160
Team Room 40/3125 (8)
This has been a very long time coming, so the huge thank you to all the communities, support and issues and feedback that has gotten us to this point.
0:21:1.400 --> 0:21:3.990
Team Room 40/3125 (8)
This release was just bug fixes.
0:21:4.140 --> 0:21:5.430
Team Room 40/3125 (8)
Really minor bug fixes.
0:21:10.860 --> 0:21:11.180
Victor
What?
0:21:5.440 --> 0:21:16.270
Team Room 40/3125 (8)
At this point, we weren't really taking any major changes at this point because we really want to, uh, make this final push to GA and have this stable of a release.
0:21:17.630 --> 0:21:29.200
Team Room 40/3125 (8)
I'm so you can check out the change log there, but just some what we thought were what we saw as important but low risk kind of that matrix of bug fixes are the things that went into this.
0:21:29.250 --> 0:21:40.670
Team Room 40/3125 (8)
RC, we have a few more bug fixes that are kind of going into the GI release at this point in time, but I expect a GA to be coming soon.
0:21:41.190 --> 0:21:43.850
Team Room 40/3125 (8)
Umm, so look forward to that.
0:21:44.50 --> 0:21:48.660
Team Room 40/3125 (8)
But definitely keep trying out the release and keep the bug reports coming.
0:21:48.760 --> 0:21:53.270
Team Room 40/3125 (8)
We already already started building the project for our first path release.
0:21:53.630 --> 0:22:3.680
Team Room 40/3125 (8)
It's gonna come out after GA, so the work is not stopping and in fact I expect maybe the patch release to get into like 7 four.
0:22:3.690 --> 0:22:6.530
Team Room 40/3125 (8)
So the work is going to continue on PS resource get.
0:22:6.620 --> 0:22:9.900
Team Room 40/3125 (8)
Keep trying it out like keep filing those bad reports for us.
0:22:10.40 --> 0:22:26.500
Team Room 40/3125 (8)
But just like huge shout out to the to both the community and the engineering team on this, because it's been a really big effort to get us to this, we in our CI also wanna thank Sean for all the documentation help on this module.
0:22:27.210 --> 0:22:33.90
Team Room 40/3125 (8)
He's been so so on top of it in terms of like updating our docs with every single preview release and all of that.
0:22:33.180 --> 0:22:36.330
Team Room 40/3125 (8)
The docs are also kind of go live at this point.
0:22:36.600 --> 0:22:39.350
Team Room 40/3125 (8)
Our C is live as a go live release.
0:22:39.420 --> 0:22:42.210
Team Room 40/3125 (8)
It's fully supported, so are the docs kind of at this point.
0:22:42.220 --> 0:22:44.560
Team Room 40/3125 (8)
So please check out the docs and give us feedback on that as well.
0:22:45.550 --> 0:22:48.60
Team Room 40/3125 (8)
Umm that those would be my two plugs.
0:22:48.110 --> 0:22:52.770
Team Room 40/3125 (8)
I'll pass things to Anam in case there's anything I missed that you'd want to mention about the RC release.
0:22:55.590 --> 0:22:58.40
Anam Navied
No, I think you covered it all perfectly, Sydney.
0:22:58.90 --> 0:23:4.790
Anam Navied
I also did want to mention that we did a compat PowerShell get release and I can drop the link for the blog post for that.
0:23:4.160 --> 0:23:4.910
Team Room 40/3125 (8)
Oh yeah.
0:23:5.340 --> 0:23:5.720
Team Room 40/3125 (8)
Thank you.
0:23:5.670 --> 0:23:6.10
Anam Navied
Uh.
0:23:5.900 --> 0:23:6.960
Team Room 40/3125 (8)
Ohh I'd.
0:23:6.430 --> 0:23:10.760
Anam Navied
And but that will allow users to keep using their.
0:23:13.10 --> 0:23:13.460
Team Room 40/3125 (8)
We should be.
0:23:11.410 --> 0:23:17.60
Anam Navied
I'm PowerShell get V2 fully qualified type commands or command.
0:23:17.70 --> 0:23:32.760
Anam Navied
Let's while getting the benefits of the PS resource get code and the performance improvements that we have so also recommend checking that out if your scripts or code are using the PowerShell get V2 kind of commandlet syntaxes, yeah, that's all that I had.
0:23:32.770 --> 0:23:33.990
Anam Navied
I think you covered arc perfectly.
0:23:39.940 --> 0:23:40.960
Anam Navied
Oh, I think your meat, Sydney.
0:23:44.450 --> 0:23:44.990
Team Room 40/3125 (8)
That happened.
0:23:45.610 --> 0:23:47.0
Team Room 40/3125 (8)
I was like, just thank you.
0:23:47.780 --> 0:23:48.40
Anam Navied
Yeah.
0:23:47.10 --> 0:23:50.140
Team Room 40/3125 (8)
Think I don't know how we don't have on you anyways?
0:23:50.210 --> 0:23:50.460
Team Room 40/3125 (8)
Yeah.
0:23:50.470 --> 0:23:56.600
Team Room 40/3125 (8)
So if you were still using PowerShell get previews and you didn't upgrade or change to PS resource get.
0:23:56.970 --> 0:24:9.840
Team Room 40/3125 (8)
If you upgrade your PowerShell get to the latest preview now, you'll notice that the commandlets are no longer the install PS resource commandlets.
0:24:9.890 --> 0:24:22.20
Team Room 40/3125 (8)
They've now switched back to install module, so the latest PowerShell Gap Preview is now like the compatibility module and like moving forward PowerShell get will continue to be the compatibility module.
0:24:22.70 --> 0:24:32.290
Team Room 40/3125 (8)
So that's just an important thing to know with the compatibility module, but yeah, we'll link the blog post if you want to read more about that there and I can pass it off to Steven with the next.
0:24:33.140 --> 0:24:35.750
Team Room 40/3125 (8)
Yeah, I think, uh, next we got docs updates.
0:24:35.760 --> 0:24:38.280
Team Room 40/3125 (8)
Uh, Sean, are you here in available to?
0:24:40.330 --> 0:24:43.380
Sean Wheeler
I am let me share my screen.
0:24:45.550 --> 0:24:48.390
Sean Wheeler
And drop a bunch of links here so.
0:24:49.550 --> 0:24:50.10
Team Room 40/3125 (8)
Umm.
0:24:53.140 --> 0:24:53.530
Team Room 40/3125 (8)
Want to talk?
0:24:52.820 --> 0:24:54.920
Sean Wheeler
Want to talk about some changes to the platform?
0:24:57.570 --> 0:25:1.400
Sean Wheeler
On docs we recently changed how search works.
0:25:1.450 --> 0:25:19.170
Sean Wheeler
So umm, in the table of contents here you have this filter box and what this allows you to do is you can start typing and this acts as a filter and it's filtering on the title of the article.
0:25:20.230 --> 0:25:20.660
Sean Wheeler
Umm.
0:25:21.120 --> 0:25:24.130
Sean Wheeler
Or this last item here is you can search.
0:25:24.140 --> 0:25:26.290
Sean Wheeler
So let me give you an example.
0:25:27.200 --> 0:25:29.600
Sean Wheeler
Search for a term like item potent.
0:25:33.140 --> 0:25:39.350
Sean Wheeler
There's nothing in the table of contents that has that word in it, but I can search all the PowerShell documentation.
0:25:39.640 --> 0:25:56.70
Sean Wheeler
This is called a search scope and not every doc set on the learn platform has a search scope defined, but I have that for PowerShell and you can see we got back nine articles, but if I search for the same thing over here.
0:25:57.700 --> 0:26:6.460
Sean Wheeler
In the top right in the site level NAV bar you see we get a lot more results.
0:26:6.810 --> 0:26:20.990
Sean Wheeler
So this is searching the entire learn side, not just the doc set, so that's that's the change they made the the site level search is now global instead of scoped.
0:26:22.960 --> 0:26:27.500
Sean Wheeler
Umm, the other thing I wanted to show that's coming soon.
0:26:29.560 --> 0:26:34.530
Sean Wheeler
Hopefully Monday of next week is a new feedback control at the bottom of the page.
0:26:34.760 --> 0:26:41.990
Sean Wheeler
So today, uh, what this looks like is is this you click the yeah.
0:26:43.770 --> 0:26:54.930
Sean Wheeler
Give feedback on this page and it opens a new uh GitHub issue with some information pre populated and you add your notes here.
0:26:57.480 --> 0:27:5.30
Sean Wheeler
But it's easy to delete this content, and, UM, uh, it it.
0:27:5.40 --> 0:27:11.70
Sean Wheeler
Also, we don't get a lot of quality feedback this way in the new experience.
0:27:13.100 --> 0:27:14.550
Sean Wheeler
Uh, you'll have.
0:27:14.610 --> 0:27:33.890
Sean Wheeler
You'll see this at the bottom of the page and when you click this to open a new documentation issue, it's taking to you to a new, umm issue template with the metadata prepopulated metadata about the article you were on prepopulated.
0:27:34.210 --> 0:27:35.900
Sean Wheeler
So the only thing you need to do here is.
0:27:37.550 --> 0:27:42.590
Sean Wheeler
Had a descriptive title and then a description of your feedback.
0:27:44.230 --> 0:27:48.430
Sean Wheeler
Anything with the red asterisk on it is required data.
0:27:50.510 --> 0:27:52.120
Sean Wheeler
You don't want to change any of this.
0:27:52.300 --> 0:28:6.270
Sean Wheeler
The uh data that was pre filled in for you but adds your information here and then there's optional boxes for more details and to add environmental information, give us some version information.
0:28:8.220 --> 0:28:9.250
Sean Wheeler
To help us out.
0:28:9.440 --> 0:28:12.0
Sean Wheeler
So that's coming next week.
0:28:14.870 --> 0:28:25.40
Sean Wheeler
And then, as always, wanna talk about what's new in documentation, there's been a ton of work done in the DSC V3 space.
0:28:25.390 --> 0:28:26.810
Sean Wheeler
We'll look at that here in a second.
0:28:27.100 --> 0:28:27.510
Sean Wheeler
Umm.
0:28:28.180 --> 0:28:33.610
Sean Wheeler
Crescendo 1.1 released and the docs have been updated for that.
0:28:34.710 --> 0:28:42.650
Sean Wheeler
Actually, in August it was a RC1UH and it's well, it's it's gone to GA release now.
0:28:44.500 --> 0:28:48.220
Sean Wheeler
Umm of course updates for the new preview lease and.
0:28:53.590 --> 0:29:1.70
Sean Wheeler
We also have this new article about feedback providers how to create your own feedback provider with sample code here.
0:29:5.330 --> 0:29:9.190
Sean Wheeler
And ah, here's the new crescendo docs.
0:29:11.310 --> 0:29:12.550
Sean Wheeler
All of these links are in the chat.
0:29:14.140 --> 0:29:16.10
Sean Wheeler
Umm in PowerShell gets.
0:29:16.900 --> 0:29:18.370
Sean Wheeler
I've updated the documentation.
0:29:18.380 --> 0:29:25.400
Sean Wheeler
Now reflects the new status of PowerShell get, umm beta 22.
0:29:25.510 --> 0:29:29.980
Sean Wheeler
This is the proxy module proxy commandlet module.
0:29:29.990 --> 0:29:35.10
Sean Wheeler
So you still have all the old commandlets, but these are proxy.
0:29:36.960 --> 0:29:41.520
Sean Wheeler
Functions that will call the new commandlets in the PS resource get module.
0:29:43.720 --> 0:29:48.150
Sean Wheeler
Umm, there are differences between the old module and the new module.
0:29:48.500 --> 0:29:51.890
Sean Wheeler
There's some parameters that no longer apply.
0:29:53.290 --> 0:29:55.950
Sean Wheeler
And those will get silently discarded.
0:29:57.900 --> 0:29:58.70
Victor
But.
0:29:57.640 --> 0:29:59.360
Sean Wheeler
Some parameters get transformed.
0:30:0.430 --> 0:30:8.570
Sean Wheeler
Umm, so mapped to the new parameters on the commandlets and in other cases they pass through to.
0:30:9.290 --> 0:30:13.470
Sean Wheeler
Uh, the same parameter name on the new commandlet?
0:30:16.660 --> 0:30:21.630
Sean Wheeler
So you can read about all of this and all of the updates to PS Resource Git.
0:30:23.550 --> 0:30:24.970
Sean Wheeler
For the.
0:30:26.710 --> 0:30:28.610
Sean Wheeler
The latest release, the RC1 release.
0:30:31.460 --> 0:30:36.640
Sean Wheeler
And DSC space over the past couple of months, a ton of new documentation.
0:30:40.400 --> 0:30:43.210
Sean Wheeler
I think we're up over 70 new documents here.
0:30:43.220 --> 0:30:46.130
Sean Wheeler
So you can see I've expanded the the table of contents.
0:30:46.740 --> 0:30:49.510
Sean Wheeler
Lots of great work here by Mikey.
0:30:51.190 --> 0:31:1.640
Sean Wheeler
And he's created a sample code for implementing DSC resources with full documentation over on this new site.
0:31:2.150 --> 0:31:4.10
Sean Wheeler
Again, these links are in the chat.
0:31:5.820 --> 0:31:6.260
Sean Wheeler
Umm.
0:31:8.290 --> 0:31:15.570
Sean Wheeler
In the Azure PowerShell space, there's a new feature that we released for notifications.
0:31:16.610 --> 0:31:22.820
Sean Wheeler
So now when you're using uh, Azure PowerShell, if you're if you have an older version installed, you can.
0:31:22.930 --> 0:31:35.920
Sean Wheeler
It'll give you a sort of a pop up message in your terminal telling you that you can upgrade, but this is configure configurable feature.
0:31:36.10 --> 0:31:44.730
Sean Wheeler
It is off by default and the link in the chat takes you to this documentation that explains how to turn it on if you want.
0:31:46.900 --> 0:31:48.750
Sean Wheeler
And that is my docks update.
0:31:50.870 --> 0:31:52.280
Steve Lee (POWERSHELL HE/HIM)
Isha, there's some chat.
0:31:52.450 --> 0:31:54.540
Steve Lee (POWERSHELL HE/HIM)
There's some discussion in the chat regarding docs.
0:31:54.550 --> 0:31:55.700
Steve Lee (POWERSHELL HE/HIM)
Can you respond?
0:31:56.300 --> 0:31:59.480
Sean Wheeler
Yes, I saw that and I was about to respond.
0:32:2.900 --> 0:32:3.280
Team Room 40/3125 (8)
Cool.
0:32:3.520 --> 0:32:4.670
Team Room 40/3125 (8)
Thanks for the please, Sean.
0:32:5.520 --> 0:32:8.530
Team Room 40/3125 (8)
Now Sydney's gonna present some updates on the VS code extension.
0:32:9.410 --> 0:32:10.0
Team Room 40/3125 (8)
Great.
0:32:10.650 --> 0:32:12.380
Team Room 40/3125 (8)
Yeah, I'm gonna keep this one pretty short.
0:32:13.50 --> 0:32:16.120
Team Room 40/3125 (8)
I'm going to paste just a link to the change log.
0:32:16.130 --> 0:32:25.820
Team Room 40/3125 (8)
Nothing too exciting and chat since the last community call, we've had two preview releases of the PowerShell Extension and DSC code.
0:32:25.860 --> 0:32:40.60
Team Room 40/3125 (8)
These have primary, umm, primarily pen bug fixes, bug fix focused and we've also been picking up the PS read line betas to get some extra testing on those as they moved to GA.
0:32:41.460 --> 0:32:52.470
Team Room 40/3125 (8)
There have also been some Community contributions in there, but just thanks to Andy Patrick and I think Justin had a few PR's that go went in there as well for making these releases happen.
0:32:52.640 --> 0:32:59.90
Team Room 40/3125 (8)
In terms of next steps that you can expect for the extension, we have another preview release.
0:32:59.100 --> 0:33:3.250
Team Room 40/3125 (8)
It's kind of ready to go that we're expecting to get out early next week.
0:33:3.400 --> 0:33:11.170
Team Room 40/3125 (8)
And then from there, barring no major issues, we're gonna roll that into stable probably later next week.
0:33:12.500 --> 0:33:13.30
Team Room 40/3125 (8)
Umm.
0:33:13.410 --> 0:33:16.440
Team Room 40/3125 (8)
And we'll be calling that kind of like our full.
0:33:18.280 --> 0:33:27.30
Team Room 40/3125 (8)
VS code stable release, so expect A blog post probably later next week and a stable release.
0:33:28.10 --> 0:33:36.720
Team Room 40/3125 (8)
Kind of bringing together all of our kind of preview releases from the summer and that will include the new GA of Psreadline, which I think Steven is going to talk about.
0:33:37.360 --> 0:33:38.850
Team Room 40/3125 (8)
I don't know if that's the next agenda item.
0:33:38.860 --> 0:33:40.330
Team Room 40/3125 (8)
It's not. It's not.
0:33:40.570 --> 0:33:44.390
Team Room 40/3125 (8)
That is later you know what about too good of a segue if it was.
0:33:46.290 --> 0:33:46.950
Team Room 40/3125 (8)
Yeah.
0:33:46.990 --> 0:33:50.640
Team Room 40/3125 (8)
First we got some updates on DSC A2.
0:33:50.650 --> 0:33:52.280
Team Room 40/3125 (8)
I think, Steve, you're gonna take those.
0:33:53.150 --> 0:33:53.880
Steve Lee (POWERSHELL HE/HIM)
Yes.
0:33:53.890 --> 0:33:57.730
Steve Lee (POWERSHELL HE/HIM)
So I'll just kind of be quick because I don't know if we're ahead or behind.
0:33:57.740 --> 0:33:59.110
Steve Lee (POWERSHELL HE/HIM)
I think we're behind time with a little bit.
0:34:0.620 --> 0:34:1.340
Team Room 40/3125 (8)
Uh second.
0:34:4.470 --> 0:34:5.130
Team Room 40/3125 (8)
I miss you.
0:34:7.740 --> 0:34:8.220
Team Room 40/3125 (8)
Just calling.
0:34:0.200 --> 0:34:9.640
Steve Lee (POWERSHELL HE/HIM)
Uh, so anyway, we're we're making steady progress through DC3 and I'll just call out a couple of the big changes in the A2.
0:34:10.70 --> 0:34:22.190
Steve Lee (POWERSHELL HE/HIM)
So and these are primarily being requested by some of our partners, but basically you know instead of searching by the path and Marko verbal, if you have a DCM score resource, underscore path, we'll use that instead.
0:34:23.380 --> 0:34:26.850
Steve Lee (POWERSHELL HE/HIM)
Umm, depends on is implemented in the configuration.
0:34:26.860 --> 0:34:32.650
Steve Lee (POWERSHELL HE/HIM)
So if you have specific sequencing of how you want resources to be invoked, you can use depends on.
0:34:32.660 --> 0:34:36.90
Steve Lee (POWERSHELL HE/HIM)
Now other than that, ohh.
0:34:36.100 --> 0:34:41.830
Steve Lee (POWERSHELL HE/HIM)
The other thing is I don't remember if export came in each of the shell.
0:34:41.840 --> 0:34:43.380
Steve Lee (POWERSHELL HE/HIM)
Look at the release notes.
0:34:44.160 --> 0:34:44.820
Steve Lee (POWERSHELL HE/HIM)
Uh, yeah.
0:34:44.830 --> 0:34:46.230
Steve Lee (POWERSHELL HE/HIM)
So expert was the other big thing.
0:34:46.320 --> 0:34:54.340
Steve Lee (POWERSHELL HE/HIM)
So this is like the kind like the reverse DSC, where if you had mainly configured a system you wanna suck out all the configurations you can apply somewhere else.
0:34:54.380 --> 0:34:55.990
Steve Lee (POWERSHELL HE/HIM)
That's what export is gonna enable.
0:34:56.160 --> 0:35:5.10
Steve Lee (POWERSHELL HE/HIM)
One thing I'll call out is that you can't directly support export via PowerShell class yet we have a separate issue open to add that support in the future.
0:35:5.820 --> 0:35:12.30
Steve Lee (POWERSHELL HE/HIM)
There are workarounds where if you had a partial script you can actually because those you can execute those.
0:35:12.40 --> 0:35:27.30
Steve Lee (POWERSHELL HE/HIM)
The other cmdlet you can also just have those be your resource, so there's ways around that if you want to play around with the export debility, but for an export configuration to work, every resource that's in that configuration also needs a support export, so hopefully we'll get more of those being developed in the future.
0:35:28.270 --> 0:35:32.470
Steve Lee (POWERSHELL HE/HIM)
We are working towards an A3 which hopefully will come out next week.
0:35:33.630 --> 0:35:36.780
Steve Lee (POWERSHELL HE/HIM)
I guess that's all I'll say about that for now and that's Mike.
0:35:36.790 --> 0:35:37.600
Steve Lee (POWERSHELL HE/HIM)
I think Mike is there don't.
0:35:37.610 --> 0:35:38.840
Steve Lee (POWERSHELL HE/HIM)
If there's anything to add on to that.
0:35:39.290 --> 0:35:39.860
Mikey Lombardi
Yeah.
0:35:39.870 --> 0:35:45.640
Mikey Lombardi
The only other thing I wanted to say is that there's a new article specific to DSC that people might find useful.
0:35:46.250 --> 0:35:53.740
Mikey Lombardi
We've been doing some work to enhance the schemas, so if you're playing around, there's a new article around using the enhanced schemas.
0:35:53.750 --> 0:36:6.930
Mikey Lombardi
These are non canonical, they're they do the exact same validation, but they include a bunch of extra VS code specific keywords that really push up the authoring experience.
0:36:7.920 --> 0:36:10.200
Mikey Lombardi
Uh, so you'll get better Intellisense.
0:36:10.210 --> 0:36:12.810
Mikey Lombardi
You'll get links to the online docs when you hover on stuff.
0:36:13.540 --> 0:36:20.320
Mikey Lombardi
Enums get descriptions, so you can see like, well, what is this value actually mean without having to go breathe the docs?
0:36:20.970 --> 0:36:21.420
Mikey Lombardi
Umm.
0:36:21.770 --> 0:36:23.260
Mikey Lombardi
And we're gonna keep improving those.
0:36:23.270 --> 0:36:25.190
Mikey Lombardi
There's new snippets and a bunch of other stuff.
0:36:32.790 --> 0:36:33.220
Team Room 40/3125 (8)
Cool.
0:36:33.530 --> 0:36:34.130
Team Room 40/3125 (8)
Thanks, Steven.
0:36:34.140 --> 0:36:47.880
Team Room 40/3125 (8)
Mikey, I'll keep this short and sweet, but we have GA, PowerShell or PS read line 2.3 dot three will be having a A blog post coming up pretty soon, highlighting some of the features.
0:36:47.890 --> 0:36:54.900
Team Room 40/3125 (8)
Not a whole lot of changes between the latest beta and this release, just a couple of bug fixes.
0:36:55.430 --> 0:37:4.230
Team Room 40/3125 (8)
Primary the primary feature changes are with list view and predictors which we've posted about in our blogs already with the previous releases.
0:37:4.610 --> 0:37:9.890
Team Room 40/3125 (8)
Umm, so we'd love for everyone to give it a try and let us know how it's working for you.
0:37:10.150 --> 0:37:18.170
Team Room 40/3125 (8)
It should, like Sydney mentioned, it should be in the next stable release of the VS code extensions, so that will be good.
0:37:18.180 --> 0:37:21.510
Team Room 40/3125 (8)
And yeah, I think that's all I gotta say about that.
0:37:22.240 --> 0:37:24.0
Team Room 40/3125 (8)
Cindy, do you want to talk about some gallery stuff?
0:37:27.570 --> 0:37:28.480
Team Room 40/3125 (8)
I would love to.
0:37:29.10 --> 0:37:31.980
Team Room 40/3125 (8)
So a couple of quick updates about the gallery.
0:37:31.990 --> 0:37:43.270
Team Room 40/3125 (8)
So I think one of the agenda items was referencing a potential down time on the gallery due to a change we were making with Azure front door.
0:37:43.410 --> 0:37:50.730
Team Room 40/3125 (8)
And I want to say that that change actually happened last week and was very successful and we didn't as far as we know, do our monitors.
0:37:51.290 --> 0:38:2.380
Team Room 40/3125 (8)
I'm nobody experienced any downtime and so actually nothing to worry about there, but I one thing I will know just as a practice that can we kind of used with.
0:38:5.300 --> 0:38:8.430
Team Room 40/3125 (8)
That was umm, we used the banner feature on the gallery.
0:38:8.500 --> 0:38:21.390
Team Room 40/3125 (8)
We we knew we needed to make this change and so we 48 hours in advance, we knew that the down time would be only about 5 minutes if anything were to occur or less because we could have an immediate rollback.
0:38:21.980 --> 0:38:25.270
Team Room 40/3125 (8)
But we put up the banner on the gallery about 48 hours in advance.
0:38:25.560 --> 0:38:28.730
Team Room 40/3125 (8)
Left it on there and then kind of made the change.
0:38:28.740 --> 0:38:29.850
Team Room 40/3125 (8)
So what?
0:38:29.860 --> 0:38:33.150
Team Room 40/3125 (8)
Expect that to be the pattern rolling forward going forward.
0:38:33.160 --> 0:38:42.200
Team Room 40/3125 (8)
If there are changes like that needed to be made in the future, and if we do have the timing where we can make it an alternative community call we, of course we'll make that announcement as well.
0:38:42.210 --> 0:38:51.380
Team Room 40/3125 (8)
But I think we have a pretty good plan in place at this point in time where we can really quick rollbacks to a known good release.
0:38:51.530 --> 0:38:53.830
Team Room 40/3125 (8)
So good stuff on that.
0:38:53.990 --> 0:39:15.650
Team Room 40/3125 (8)
One other thing I was going to mention about the gallery is many folks have reported we're well aware that thanks to many reports that stats were down for a period of time on the gallery, I'm very happy to announce that as of this morning we have resolved this issue and stats are officially fully working again.
0:39:15.920 --> 0:39:22.290
Team Room 40/3125 (8)
So Many thanks to the gallery team for getting that back up and working again.
0:39:22.800 --> 0:39:30.550
Team Room 40/3125 (8)
It root cause came down to a certain rotation issue, but that has been resolved and stats should be up and working again.
0:39:30.600 --> 0:39:34.830
Team Room 40/3125 (8)
So thank you to everyone who kind of worked with us on that and reported the issue.
0:39:35.700 --> 0:39:40.140
Team Room 40/3125 (8)
I think that's all I got on gallery for the time being. Cool.
0:39:39.590 --> 0:39:43.260
James Brundage
Being brief with my thanks, that was a very welcome warning.
0:39:43.270 --> 0:39:50.640
James Brundage
Banner as somebody is spreading publishing modules all the time, it was great to see the heads up and I can indeed confirm stats are working again.
0:39:52.170 --> 0:39:52.820
James Brundage
So thank you.
0:39:51.700 --> 0:39:53.360
Team Room 40/3125 (8)
Awesome, yeah.
0:39:55.670 --> 0:39:56.320
Team Room 40/3125 (8)
That's awesome.
0:39:57.810 --> 0:39:58.160
Team Room 40/3125 (8)
Cool.
0:39:58.170 --> 0:40:0.620
Team Room 40/3125 (8)
And so we're just a quick plug.
0:40:1.170 --> 0:40:6.120
Team Room 40/3125 (8)
So next thing is Microsoft Ignite, Microsoft Ignite is coming up in November.
0:40:6.130 --> 0:40:8.270
Team Room 40/3125 (8)
I believe the dates are the 14th through the 17th.
0:40:8.900 --> 0:40:17.250
Team Room 40/3125 (8)
Uh, I think this is the correct link, but just wanted to see if any kind of folks are planning to attend Ignite.
0:40:17.260 --> 0:40:18.560
Team Room 40/3125 (8)
Well, we'll have some.
0:40:18.800 --> 0:40:21.550
Team Room 40/3125 (8)
We're looking at having an event around the.
0:40:21.560 --> 0:40:22.580
Team Room 40/3125 (8)
Or do you want to talk about?
0:40:22.780 --> 0:40:36.50
Team Room 40/3125 (8)
So I'm I'm working on so many of you who have attended ignite in the past, May know that there's oftentimes been a PowerShell community event at Ignite, and I'm working on planning that right now.
0:40:36.200 --> 0:40:43.160
Team Room 40/3125 (8)
And so I'm just like trying to get a sense of, like, what scale that might look like without sort of thing in terms of like reserving an event space.
0:40:44.50 --> 0:40:56.660
Team Room 40/3125 (8)
So if I could get any sense of like, hey, I'm thinking about attending in person, that's super useful to me in terms of like starting to kind of make those plans and just like letting you know that there will be a PowerShell presence there.
0:40:56.670 --> 0:41:0.680
Team Room 40/3125 (8)
I don't believe that this schedule has come out yet.
0:41:1.690 --> 0:41:2.590
Team Room 40/3125 (8)
Umm.
0:41:2.770 --> 0:41:4.550
Team Room 40/3125 (8)
But hopefully that will come out soon.
0:41:4.630 --> 0:41:7.360
Team Room 40/3125 (8)
But just letting you all know, there will be, the PowerShell team will be there.
0:41:7.370 --> 0:41:13.0
Team Room 40/3125 (8)
There will be a PowerShell presence and in addition to that I'm working on planning a PowerShell community event.
0:41:13.10 --> 0:41:21.460
Team Room 40/3125 (8)
So if you would be interested in potentially attending, obviously not a commitment, it's helpful for my planning purposes just to start to get a sense of like what that might look like.
0:41:21.510 --> 0:41:28.630
Team Room 40/3125 (8)
So yeah, attending in person, attending in line online either way would be great for us to know just for planning purposes.
0:41:29.690 --> 0:41:37.240
Team Room 40/3125 (8)
Umm, I think that's all we've got for a night at the the moment, but more to come around Ignite.
0:41:37.390 --> 0:41:47.710
Team Room 40/3125 (8)
And while we're on the conference topic, I'll also just mention as well coming up next month on GAIL did post in the chat there is PS mini comp that is an online event.
0:41:47.810 --> 0:41:57.510
Team Room 40/3125 (8)
I believe it's on October 24th in European time zones, but I'm I'm sure you can get the exact times through the link.
0:41:57.890 --> 0:41:59.900
Team Room 40/3125 (8)
Uh, breaking news.
0:41:59.910 --> 0:42:7.920
Team Room 40/3125 (8)
I am doing PS resource get session at that and I think many other folks from the community will be doing awesome sessions as well.
0:42:8.310 --> 0:42:10.710
Team Room 40/3125 (8)
So hope to see you, folks.
0:42:10.720 --> 0:42:12.310
Team Room 40/3125 (8)
I thought of it as well.
0:42:14.800 --> 0:42:16.890
Team Room 40/3125 (8)
Yeah, I think that covers everything at conferences.
0:42:19.50 --> 0:42:24.420
Team Room 40/3125 (8)
And I believe I believe there there's a pretty large online presence for those who can't make it in person at ignite.
0:42:24.430 --> 0:42:29.340
Team Room 40/3125 (8)
So definitely keep an eye out for for new content coming out around that time too.
0:42:29.410 --> 0:42:30.300
Team Room 40/3125 (8)
Yeah, absolutely.
0:42:30.850 --> 0:42:32.120
Team Room 40/3125 (8)
But I think that covers that.
0:42:32.550 --> 0:42:36.0
Team Room 40/3125 (8)
Uh, another quick topic.
0:42:41.370 --> 0:42:41.790
Steve Lee (POWERSHELL HE/HIM)
That's me.
0:42:36.10 --> 0:42:41.800
Team Room 40/3125 (8)
The November end of year community call second Thursday of the month, you.
0:42:43.670 --> 0:42:51.210
Steve Lee (POWERSHELL HE/HIM)
Alright, so uh, First off, I appreciate every all the community members and all the team members that participate in these monthly community calls.
0:42:51.220 --> 0:42:52.760
Steve Lee (POWERSHELL HE/HIM)
You know, it's always that there Thursday of the month.
0:42:53.350 --> 0:43:4.750
Steve Lee (POWERSHELL HE/HIM)
Umm, over the last few years we've been doing this, we've always cancelled November because it coincides with Thanksgiving, which is highly celebrated in America, specifically United States. Umm.
0:43:4.970 --> 0:43:11.420
Steve Lee (POWERSHELL HE/HIM)
And then December would typically also cancel because people on the team are taking off time off for the holidays.
0:43:11.430 --> 0:43:13.500
Steve Lee (POWERSHELL HE/HIM)
Christmas and British news and stuff like that.
0:43:13.750 --> 0:43:37.440
Steve Lee (POWERSHELL HE/HIM)
So the thing that we were discussing as a team is maybe what we should do is have the 2nd Thursday of November be our end of year community call and would extend it to I think we're discussing like 2 hours and also encourage people in the Community to bring in stuff they wanna cover as well, like maybe different user groups can kind of maybe do a summary of end of year uh different conference and stuff like that.
0:43:37.820 --> 0:43:51.150
Steve Lee (POWERSHELL HE/HIM)
And also I think as far as I recall and I wanna be on the spot, Sydney, I think you were recording some working group stuff, but maybe we can ask the working group members, one representative from the working group to kind of also cover how the year one for them and stuff like that.
0:43:51.770 --> 0:43:58.480
Steve Lee (POWERSHELL HE/HIM)
So I think we're trying to still figure out the agenda, but I think we're pretty close to saying, yeah, let's have the 2nd Thursday.
0:43:58.490 --> 0:44:0.60
Steve Lee (POWERSHELL HE/HIM)
So we'll probably have a separate invite.
0:44:0.70 --> 0:44:6.130
Steve Lee (POWERSHELL HE/HIM)
What to update the, umm, the uh PowerShell RC repo where we hold the community call stuff?
0:44:6.960 --> 0:44:12.170
Steve Lee (POWERSHELL HE/HIM)
Uh, but we'll try it this year and we'll see how that goes and maybe this will be a regular recurring thing.
0:44:19.280 --> 0:44:19.590
Team Room 40/3125 (8)
Cool.
0:44:19.600 --> 0:44:21.320
Team Room 40/3125 (8)
I think that covers everything.
0:44:21.330 --> 0:44:23.910
Team Room 40/3125 (8)
Did you cover what you wanted to on the December community call, Steve.
0:44:24.800 --> 0:44:26.520
Steve Lee (POWERSHELL HE/HIM)
Ohh, I guess you'll.
0:44:26.530 --> 0:44:27.870
Steve Lee (POWERSHELL HE/HIM)
Ohh yeah, I forgot about that part.
0:44:28.60 --> 0:44:47.910
Steve Lee (POWERSHELL HE/HIM)
So the other thinking was because the N1 would be the last community call for the officially from the team that leaves open if the community themselves wanna do a call in December for whoever is available, that will be something that we would be interested in helping the coordinate.
0:44:47.920 --> 0:44:51.10
Steve Lee (POWERSHELL HE/HIM)
Although keep in mind that many team members will probably be on vacation.
0:44:51.660 --> 0:44:56.870
Steve Lee (POWERSHELL HE/HIM)
But however is interested in maybe doing that I say reach out to Steven or Sydney or Jason.
0:44:57.790 --> 0:44:58.410
Steve Lee (POWERSHELL HE/HIM)
Any of those 3.
0:44:59.140 --> 0:45:2.870
James Brundage
I mean, I'll be in town and I wouldn't mind having December community call.
0:45:2.880 --> 0:45:4.960
James Brundage
It's like a PowerShell Christmas presents, right?
0:45:5.620 --> 0:45:7.620
Steve Lee (POWERSHELL HE/HIM)
Yeah, exactly.
0:45:11.720 --> 0:45:12.90
Team Room 40/3125 (8)
Cool.
0:45:12.960 --> 0:45:15.850
Team Room 40/3125 (8)
I think that covers everything from our agenda.
0:45:15.860 --> 0:45:20.960
Team Room 40/3125 (8)
One thing I did see in the chat was the call for.
0:45:24.430 --> 0:45:30.480
Team Room 40/3125 (8)
The CFP for the PowerShell Dev OPS Global Summit opens on October 1st, so keep an eye out for that.
0:45:30.490 --> 0:45:39.480
Team Room 40/3125 (8)
That's usually held April, and I'm sure they'll be more announcements once once things are finalized later, but wanted to call that out because I saw that in the chat ohm.
0:45:39.520 --> 0:45:42.20
Team Room 40/3125 (8)
I think that there are covers our main agenda, ohm.
0:45:42.320 --> 0:45:45.850
Team Room 40/3125 (8)
Let me check she didn't 50.
0:45:47.830 --> 0:45:48.700
Team Room 40/3125 (8)
Rapid fire.
0:45:50.510 --> 0:45:51.250
Team Room 40/3125 (8)
Umm.
0:45:51.380 --> 0:45:56.380
Team Room 40/3125 (8)
So yeah, maybe, maybe, James, if you wanted to talk about the Pacific PowerShell user group.
0:46:2.460 --> 0:46:2.740
Team Room 40/3125 (8)
Perfect.
0:45:58.940 --> 0:46:3.50
James Brundage
Uh, since I just chat spammed, I will go ahead and do that so.
0:46:4.930 --> 0:46:8.20
James Brundage
Finally, set up a Pacific PowerShell user group.
0:46:8.70 --> 0:46:9.880
James Brundage
We had our first meeting last month.
0:46:9.950 --> 0:46:16.80
James Brundage
We actually had about half of the meet up sign UPS join, so that was pretty great.
0:46:16.90 --> 0:46:20.320
James Brundage
Actually we got our next meeting coming up the second Wednesday of every month.
0:46:20.630 --> 0:46:27.450
James Brundage
So we'll have one with Anthony Howell talking PowerShell on .net in on October 11th at 6:00 PM.
0:46:28.300 --> 0:46:32.430
James Brundage
Umm, I believe the month after that we're gonna have Bruce Payette talking about Braid.
0:46:32.860 --> 0:46:37.760
James Brundage
I think I'm gonna be doing December hint hint nudge, nudge, PowerShell team members.
0:46:39.170 --> 0:46:41.740
James Brundage
You got releases coming up right after the new year, right?
0:46:41.750 --> 0:46:44.260
James Brundage
You wanna talk about them and the subsequent months?
0:46:44.270 --> 0:46:45.420
James Brundage
Let's get you guys booked now.
0:46:46.440 --> 0:46:47.60
James Brundage
Umm.
0:46:47.480 --> 0:46:49.830
James Brundage
But yeah, the first meeting went better than expected.
0:46:49.840 --> 0:46:52.810
James Brundage
I hope the second meeting also goes better than expected.
0:46:52.820 --> 0:47:1.200
James Brundage
It's really nice as a West Coast PowerShell user to have something that isn't, you know, stop the middle of my work day to go attend the meeting.
0:47:2.790 --> 0:47:8.280
James Brundage
Although it is, you know, maybe have a late dinner during your meeting now, umm.
0:47:8.360 --> 0:47:14.170
James Brundage
But yeah, it was a real fun first instance and I hope to see people more in the future.
0:47:14.720 --> 0:47:17.600
James Brundage
So anybody that wants to sign up is welcome to him.
0:47:19.280 --> 0:47:19.890
James Brundage
That's about it.
0:47:19.900 --> 0:47:23.730
James Brundage
On that one side from that I you know, have a general smattering of new toys.
0:47:24.740 --> 0:47:25.910
James Brundage
Anybody wanna see new toys?
0:47:32.210 --> 0:47:32.410
Team Room 40/3125 (8)
Sure.
0:47:33.840 --> 0:47:35.690
James Brundage
OK, well, now let me get into demo mode.
0:47:34.460 --> 0:47:36.800
Team Room 40/3125 (8)
Your time I want to.
0:47:37.570 --> 0:47:42.510
Team Room 40/3125 (8)
I guess while you're doing that, I'll just address the other question in the community call.
0:47:42.650 --> 0:47:48.50
Team Room 40/3125 (8)
I think from JP Lewis, we'll we'll follow up on that issue.
0:47:51.270 --> 0:47:51.470
Team Room 40/3125 (8)
Yeah.
0:47:47.780 --> 0:47:54.270
Victor
Internal climate activist can load themselves on the ground in order to cool the Earth and it's.
0:47:55.800 --> 0:47:56.60
Team Room 40/3125 (8)
OK.
0:47:59.20 --> 0:47:59.280
Team Room 40/3125 (8)
Cool.
0:47:59.300 --> 0:48:0.330
Team Room 40/3125 (8)
If there's any other questions.
0:47:59.920 --> 0:48:2.0
James Brundage
That was an interesting audio distraction there.
0:48:4.480 --> 0:48:15.470
Team Room 40/3125 (8)
There's any other questions or other people that would like to to do any sort of quick demos, something cool they found, something they've been working on right, anything at all.
0:48:15.480 --> 0:48:18.20
Team Room 40/3125 (8)
Feel free to drop it in the chat or the git hopelink.
0:48:27.200 --> 0:48:28.760
Team Room 40/3125 (8)
You're good community call round in December.
0:48:31.580 --> 0:48:50.350
Team Room 40/3125 (8)
Yeah, that's a good call, Ryan, to get a discussion set up for the December call, would it, I guess, I guess, probably be good to have it in the same spot as all the community call discussions, so discussion, OK, just more like Community focused rather than like us starting with the agenda like folks bring what they wanted to talk about more so.
0:48:51.80 --> 0:48:51.450
James Brundage
Yeah.
0:48:51.520 --> 0:49:9.350
James Brundage
I honestly also could be the start of a good tradition, like if you guys no offense at all met, but if you guys get to set agenda for 11 months of the year on the PowerShell community call, you know giving the floor back to the community for the last meeting of the year could be fun and could be you know.
0:49:8.690 --> 0:49:10.540
Team Room 40/3125 (8)
Listen, we're always happy to give you the floor.
0:49:14.910 --> 0:49:15.110
Team Room 40/3125 (8)
No.
0:49:11.590 --> 0:49:15.550
James Brundage
Ohh no, no, no, no, I don't need it that much. I'm.
0:49:17.630 --> 0:49:17.790
James Brundage
But.
0:49:17.850 --> 0:49:18.910
Team Room 40/3125 (8)
Things ring for opening it up.
0:49:17.840 --> 0:49:21.650
James Brundage
But the yeah, alright.
0:49:22.30 --> 0:49:23.530
James Brundage
Uh, may I share?
0:49:24.930 --> 0:49:25.240
Team Room 40/3125 (8)
Sure.
0:49:27.480 --> 0:49:28.480
James Brundage
It appears that I do.
0:49:25.250 --> 0:49:29.390
Team Room 40/3125 (8)
Do you need a permission or OK?
0:49:32.980 --> 0:49:34.50
Team Room 40/3125 (8)
OK, one second.
0:49:39.920 --> 0:49:40.610
Team Room 40/3125 (8)
Yeah.
0:49:35.690 --> 0:49:41.240
James Brundage
So I'm just going to go for the fun points one, just because it's probably the most visually stimulating.
0:49:41.40 --> 0:49:42.710
Team Room 40/3125 (8)
Ohh yeah, do you need me to do this?
0:49:43.930 --> 0:49:45.840
James Brundage
And yeah, I didn't say mother.
0:49:45.850 --> 0:49:46.450
James Brundage
May I share?
0:49:46.460 --> 0:49:47.50
James Brundage
But I did say.
0:49:49.150 --> 0:49:49.340
Team Room 40/3125 (8)
Yeah.
0:49:47.60 --> 0:49:55.600
James Brundage
May I share not can I share, although technically on a pure grammatical level I cannot share because the share icon disabled so.
0:49:55.990 --> 0:49:56.430
Team Room 40/3125 (8)
Yeah.
0:49:56.440 --> 0:49:56.730
Team Room 40/3125 (8)
Sorry.
0:49:56.740 --> 0:49:57.760
Team Room 40/3125 (8)
Give me a second.
0:49:57.860 --> 0:49:59.650
Team Room 40/3125 (8)
Having trouble just made you a presenter.
0:49:57.460 --> 0:50:0.690
James Brundage
This is one of those few cases where can would have been the right one.
0:50:0.790 --> 0:50:1.550
James Brundage
There we go.
0:50:1.560 --> 0:50:2.100
James Brundage
I could do it.
0:50:1.200 --> 0:50:2.870
Team Room 40/3125 (8)
They yeah, I think you should be good now.
0:50:3.20 --> 0:50:3.580
Team Room 40/3125 (8)
OK, cool.
0:50:6.30 --> 0:50:8.450
James Brundage
But I appreciate the grammar lesson from the chat.
0:50:10.70 --> 0:50:12.540
James Brundage
OK, let's see if I got the right screen.
0:50:13.150 --> 0:50:14.450
James Brundage
Maybe, maybe not.
0:50:15.70 --> 0:50:18.930
James Brundage
Am I sharing a code window or am I sharing another code window?
0:50:19.180 --> 0:50:19.480
James Brundage
Yeah.
0:50:19.490 --> 0:50:20.900
James Brundage
No, we got we got the right one PSA.
0:50:21.470 --> 0:50:21.680
Team Room 40/3125 (8)
Yep.
0:50:22.230 --> 0:50:22.350
Steve Lee (POWERSHELL HE/HIM)
Yes.
0:50:22.200 --> 0:50:26.70
James Brundage
OK, so who has gotten on blue sky yet?
0:50:29.700 --> 0:50:30.10
James Brundage
Come on.
0:50:30.20 --> 0:50:33.700
James Brundage
PowerShell team knows new social network set up some handles before other people camping.
0:50:33.770 --> 0:50:33.890
Team Room 40/3125 (8)
Yeah.
0:50:36.710 --> 0:50:37.310
Team Room 40/3125 (8)
One of the thing.
0:50:55.970 --> 0:50:56.500
Team Room 40/3125 (8)
Look at this.
0:50:58.560 --> 0:50:59.470
Team Room 40/3125 (8)
Get a whole bunch of people.
0:50:34.510 --> 0:50:59.810
James Brundage
I one of the things that that I keep needing to do as a developer is get to the point where I can basically publicize releases as they come out and one of the things I love to do is developer is go take a new area that I could kind of, you know, look at its metadata and turn its and commandlets and get a whole bunch of new functionality.
0:51:0.420 --> 0:51:14.60
James Brundage
So last week I set my sights at PSA at Blue Sky and it building a general announcement module for PowerShell and well, let's let a picture speaks 1000 words if you know it's so nice to me.
0:51:14.70 --> 0:51:15.900
James Brundage
So let's go ahead and get my feed timeline.
0:51:21.80 --> 0:51:25.400
James Brundage
It is going to take a second because is somebody else's server and it is sometimes slow.
0:51:26.990 --> 0:51:28.530
James Brundage
Hamina Hamina, Hamina.
0:51:30.170 --> 0:51:31.250
James Brundage
There we go though.
0:51:34.170 --> 0:51:37.720
James Brundage
So this is every post made to my own timeline Blue sky.
0:51:38.860 --> 0:51:39.240
James Brundage
Let's see.
0:51:39.250 --> 0:51:40.810
James Brundage
There are a couple of ones here.
0:51:41.360 --> 0:51:44.490
James Brundage
It's today's horrifying thought we've all probably ignored a greater volume spam.
0:51:44.500 --> 0:51:47.290
James Brundage
The N corresponds cherished thanks to PS style.
0:51:47.300 --> 0:51:53.250
James Brundage
I can actually click this link, open this up, get right to the post status.
0:51:56.880 --> 0:51:58.560
James Brundage
But you know why stop there.
0:52:0.50 --> 0:52:3.420
James Brundage
I can go ahead and say, you know, get blue sky.
0:52:3.660 --> 0:52:5.440
James Brundage
Not like I have a variable set up for this.
0:52:5.450 --> 0:52:6.900
James Brundage
My blue sky profile.
0:52:7.230 --> 0:52:8.430
James Brundage
Go ahead and see my own profile.
0:52:9.670 --> 0:52:10.920
James Brundage
Follow words.
0:52:11.840 --> 0:52:12.950
James Brundage
I need more followers.
0:52:13.0 --> 0:52:14.850
James Brundage
I also need to build a formatter for followers.
0:52:14.890 --> 0:52:17.390
James Brundage
It's not all perfect yet follows.
0:52:20.260 --> 0:52:39.240
James Brundage
So what I'm doing here is basically taking their object model and turning it into PowerShell commandlets and decorating those PowerShell commandlets so that we can format them appropriately and making nice formatting for a social media network and also getting to the point where I can say send blue sky here.
0:52:43.410 --> 0:52:48.490
James Brundage
Showing off he SA at the PowerShell community.
0:52:49.510 --> 0:52:53.840
James Brundage
Call and let's go for, you know, some bonus points.
0:52:53.850 --> 0:52:55.680
James Brundage
And let's grab a lengthy discussions.
0:52:58.600 --> 0:52:59.610
James Brundage
Sorry for a second.
0:52:59.620 --> 0:53:1.930
James Brundage
I gotta get the right tab open.
0:53:5.700 --> 0:53:7.360
James Brundage
Uh, you know what?
0:53:9.690 --> 0:53:11.110
James Brundage
I'll just link to PowerShell dot.
0:53:13.520 --> 0:53:23.60
James Brundage
You know, check out the best Get hub repository or check out the best or the PowerShell source.
0:53:26.250 --> 0:53:26.450
James Brundage
Here.
0:53:30.160 --> 0:53:33.140
James Brundage
Come PowerShell that works shell.
0:53:34.440 --> 0:53:43.950
James Brundage
Sorry I didn't write demo file for this one yet, but I did just go ahead and send that out and I can now go ahead to my profile on Blue sky.
0:53:46.450 --> 0:53:46.990
James Brundage
And.
0:53:49.100 --> 0:53:50.620
James Brundage
Go ahead and there we go.
0:53:52.920 --> 0:54:2.290
James Brundage
Now, because you know, one goes for bonus points, I'm gonna point out two things here, and the first is this is a GitHub action.
0:54:3.660 --> 0:54:7.410
James Brundage
You can go ahead and use it on the marketplace now.
0:54:7.420 --> 0:54:9.720
James Brundage
Actually I I revised 3 things.
0:54:10.300 --> 0:54:19.520
James Brundage
Second thing is that I can actually repeat this approach and I'm just gonna show you the files that I don't have checked in right now, which include commands for all of Mastodon too.
0:54:22.100 --> 0:54:27.60
James Brundage
So if you want to start automating social networks again with PowerShell, I got you.
0:54:29.280 --> 0:54:30.230
James Brundage
That's fun.
0:54:30.660 --> 0:54:34.110
James Brundage
The other little big fun thing that's especially a good community.
0:54:34.120 --> 0:54:37.190
James Brundage
Call you know, point out is an approach.
0:54:37.200 --> 0:54:49.680
James Brundage
Here we often write a lot of modules where we need the module to configure itself post load OK like connecting to a resource with a particular credential being a you know a common one.
0:54:51.450 --> 0:55:0.510
James Brundage
As far as a good, sustainable, cheap approach to this, what I've ended up doing here is creating the concept of a module profile.
0:55:1.70 --> 0:55:5.590
James Brundage
So I'm going to go in profile, split path, get child item.
0:55:7.540 --> 0:55:14.320
James Brundage
Filter PS A star module profiles really have simple concept.
0:55:14.330 --> 0:55:27.410
James Brundage
Instead of having your normal host prefix your profile dot PS1, you have whatever module name dot profile dot PS1 and any module could choose to have a respect for a module profile.
0:55:27.820 --> 0:55:33.580
James Brundage
So in this case I'm choosing to have respect for a module profile, and if I go look at that file.
0:55:37.220 --> 0:55:41.760
James Brundage
It's just going off and getting the secret of the credential used to connect and connecting.
0:55:44.430 --> 0:56:12.760
James Brundage
So not only do we have a nice way to talk to blue sky and coming down the Pike and nice way to talk to Mastodon and you know whatever other social networks we end up eating up over time, but we also have accidentally kind of discovered a good convention that we can all use to make our modules configurable with basically a single line of code cause all we need to do is basically split path or profile test path to see if there's a profile for our thing and then run it.
0:56:14.620 --> 0:56:18.130
James Brundage
So that's the PSA on PSA.
0:56:18.190 --> 0:56:22.810
James Brundage
And I think that was a long enough demo, so I'm gonna stop sharing and get off the hot seat.
0:56:23.800 --> 0:56:25.660
James Brundage
If you want me back on it later, we can come back to it.
0:56:27.460 --> 0:56:27.920
James Brundage
Thoughts.
0:56:30.980 --> 0:56:31.480
Darren Kattan
Like it?
0:56:34.160 --> 0:56:35.710
James Brundage
Asylon said they liked it.
0:56:36.390 --> 0:56:38.220
James Brundage
I think. Umm.
0:56:41.460 --> 0:56:56.930
James Brundage
But yeah, if anybody is interested in helping with that over the long term, there are plenty of social networks to eat up and having a good generic function that we can use to talk to all of them and shout out stuff might be beneficial to the community at large.
0:56:58.50 --> 0:57:2.450
James Brundage
You know, as long as it doesn't hack another election, knocking on wood.
0:57:7.620 --> 0:57:10.230
James Brundage
So yeah, that's that's my main thing for the day.
0:57:10.240 --> 0:57:10.870
James Brundage
I'm shutting up now.
0:57:13.310 --> 0:57:13.740
Team Room 40/3125 (8)
Thanks, James.
0:57:14.750 --> 0:57:15.130
Team Room 40/3125 (8)
Pretty cool.
0:57:16.450 --> 0:57:16.800
Team Room 40/3125 (8)
See there's.
0:57:18.930 --> 0:57:20.380
Team Room 40/3125 (8)
Some stuff going on in the chat.
0:57:22.120 --> 0:57:24.970
Team Room 40/3125 (8)
I'll pause for a minute if anyone else has any other kind of questions.
0:57:25.60 --> 0:57:27.620
Team Room 40/3125 (8)
Otherwise, we're we're getting close to the end here, so.
0:57:35.100 --> 0:57:38.420
Team Room 40/3125 (8)
Don't forget to wear your Halloween costumes to the October call.
0:57:39.510 --> 0:57:41.770
Team Room 40/3125 (8)
We'll be having a costume contest so.
0:57:43.420 --> 0:57:45.110
James Brundage
Well, at least you're warning us.
0:57:45.120 --> 0:57:45.970
James Brundage
Thank you.
0:57:45.840 --> 0:57:46.30
Team Room 40/3125 (8)
Yes.
0:57:46.40 --> 0:57:50.390
James Brundage
Ah, now I'll know why they're random zombies in the call next month.
0:57:51.20 --> 0:57:51.630
James Brundage
Uh.
0:57:56.970 --> 0:57:57.580
Team Room 40/3125 (8)
Just kidding.
0:57:51.980 --> 0:57:57.810
James Brundage
Please presuming you know, but yeah, I don't know if I'll do it.
0:57:57.590 --> 0:57:59.10
Team Room 40/3125 (8)
Nobody has to wear costumes.
0:58:0.710 --> 0:58:2.270
Steve Lee (POWERSHELL HE/HIM)
I think we'll see only Sydney.
0:58:2.280 --> 0:58:2.990
Steve Lee (POWERSHELL HE/HIM)
We're in a costume.
0:58:5.170 --> 0:58:11.700
James Brundage
Yeah, I keep meaning to actually, like, you know, get my embroidery machine set up and start to rock it with PowerShell.
0:58:13.160 --> 0:58:13.650
Team Room 40/3125 (8)
That'll be all.
0:58:11.910 --> 0:58:15.60
James Brundage
But hey, you know, next time.
0:58:18.270 --> 0:58:32.790
James Brundage
Thank you guys for continuing to set this up and having, you know, such a open community and getting so damn much done in the past month, in fact, so damn much done every month that you can easily fill 45 minutes of meeting time.
0:58:32.800 --> 0:58:33.260
James Brundage
Just saying.
0:58:33.270 --> 0:58:34.330
James Brundage
Hey, here's what we've been up to.
0:58:35.210 --> 0:58:36.180
James Brundage
Yeah, you guys are awesome.
0:58:36.250 --> 0:58:36.590
James Brundage
Thank you.
0:58:39.980 --> 0:58:40.540
Team Room 40/3125 (8)
Thanks.
0:58:41.720 --> 0:58:42.490
Team Room 40/3125 (8)
Thank you.
0:58:42.600 --> 0:58:45.60
Team Room 40/3125 (8)
Thank you all for being awesome. Exactly.
0:58:48.570 --> 0:58:49.160
Team Room 40/3125 (8)
Cool.
0:58:50.230 --> 0:58:51.720
Team Room 40/3125 (8)
Well, thanks everyone for joining.
0:58:52.90 --> 0:58:54.980
Team Room 40/3125 (8)
We can end off there and thanks everyone for coming.
0:58:54.990 --> 0:58:56.960
Team Room 40/3125 (8)
We'll see you in October.

