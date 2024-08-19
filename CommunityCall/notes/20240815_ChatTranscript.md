# PowerShell_OpenSSH Community Call-20240815

August 15, 2024
31m 8s
 
Sydney Smith (SHE/HER)   0:48
Alrighty.
Well, it is 930 here, which means it is time to start the call.
So welcome everyone, and thank you for being here.
As a reminder, this call is being recorded, so if you do not wish to participate in the recording, please leave the call now and the call will be uploaded to our YouTube.
We also will upload the nodes as well to GitHub.
Umm this is a reminder to also remember our code of conduct which is linked here in the chat as well and we also post an agenda to the discussions tab of the PowerShell repo each month where we build out the agenda for the call.
This is a place where you can put any questions you might have in advance any topics you want to be discussed.
And also we really encourage Community demos.
So if you have anything you would like to demo at the call, we facilitate demos before the call so you can post there if you have anything you'd like to demo.
And I think we have two awesome community demos coming today.
One file reminder is that we are participating in what we're calling summer hours for this call.
As we did last fall, was just means that we're going to try and keep this call to around 30 minutes and then it come fall we'll be back to our normal hour long calls.
OK.
And uh, if you'd like to participate throughout the cool, feel free to definitely use the chat to bring any comments, questions, anything like that if you'd like to come off mute, feel free to use the raise hand feature and we can just saltate that as well.
With that, we will get started with the agenda.
So I have the first agenda item which is just a reminder about some PowerShell releases.
 
Carpenter, Marvin   2:37
That is the best choices.
Yeah.
Yeah.
No Sunday.
Hey, what's?
 
Sydney Smith (SHE/HER)   2:43
Ohm one second.
I think we got some.
 
Carpenter, Marvin   2:47
Ohh yeah, we're gonna meet him.
 
Sydney Smith (SHE/HER)   2:49
OK, great.
Thank you.
Last month, we did have two powers.
Alright, leases both A74 release as well as A72 release.
I'll post a link to those in the chat and this is just a reminder that those are security patches, so if you aren't updating to be sure and go ahead and update to those and in August, we are expecting to have a 75 preview release.
So look forward to that and I and one other reminder is that PowerShell 7/2 is coming close to its end of life.
It's end of life will be at the beginning of November.
I believe it's November 8th, so do you make your plans to upgrade to PowerShell 74 which is our other long term support release for this currently out we are working internally with all of our different partners who use PowerShell as well to make sure that they're also updating with that cadence as well.
So I think those are the reminders and then I'll pass things over to Jim, who's gonna talk a little bit about a change that's coming in the next preview.
 
Steve Lee (POWERSHELL HE/HIM)   3:54
I should before I get to gym.
 
Jim Truher   3:54
Right. Oh.
 
Steve Lee (POWERSHELL HE/HIM)   3:55
Uh.
Send you want talk about the seven two lifecycle.
 
Sydney Smith (SHE/HER)   3:56
Yeah.
Yeah.
I just mentioned that no, you're good.
 
Steve Lee (POWERSHELL HE/HIM)   4:00
I'm sorry.
 
Sydney Smith (SHE/HER)   4:02
I almost forgot it too.
 
Jim Truher   4:06
So I'm gonna start sharing something here just to make sure, because I'm gonna do a very, very brief demo.
I if coming up with our next preview for is a.
Change that, a breaking change.
That was that happened because of a change, a breaking change in .net and .net Preview 6.
The way environment variables are removed has changed the .net frame.
The.net core used to not be able to have an empty environment variable that changed in Preview 6 and that had a trickle down effect into us into our product.
So the way you used to use used to remove an environment variable was either remove item or you could set the environment variable to a null or you could set the environment variable to an empty string with net six you can now create environment variables with empty strings and they and the environment variable is not removed.
So in this easiest way to to go through this is to essentially just show you what's going on here.
Off I have an environment variable like this.
This is current.
This is current PowerShell.
Let's set it to string first.
And we can test the path of edmx and we'll see.
That that, that path now exists in the in the Mt exists.
If I set this to as to an empty string and do the same thing.
We'll see that that that variable is now missing with the new, with the latest release.
I'll just started up here.
We can create.
We can now have a an environment variable that exists but has no value, and we can just simply say that we'll do what we did before.
We can see, obviously that that this guy exists, and now if I go and remove and if I had used previous behavior to remove that variable by setting it to an empty string.
Uh, we'll see that it still exists.
However, PowerShell can a remove this simply by setting it to null and so if I set this to null, the environment variable is gone.
We actually found this uh, we actually had some of our validation fail because of this change.
So our tests sort of told us that something was up and I had been tracking this for a while, so I was all ready with the with the fix.
We didn't actually have to change anything in the product to make this work, which is really nice.
So this is something that may affect you in your scripts and usage.
So you want to be careful about it.
We're going to certainly document it, but the we think this is generally a good thing having an empty.
A centrally variables environment variables used as flags as a very useful behavior, so we're very happy that this happened as an old Unix guy, I was always dismayed by the previous behavior, so I'm pretty happy to see this.
If you have any questions, please put them in the chat, I'll be happy to answer anything that comes in.
 
Sydney Smith (SHE/HER)   7:38
Awesome.
Thanks so much, Jen.
Next up, we have food.
 
Steve Lee (POWERSHELL HE/HIM)   7:44
All right, I think I got the next two items.
So I'll kind of go over this.
Look at me.
Too long, so an issue that was raised to us a while back and it took a while to fix was that if you install a partial 7 on Windows using the MSI installer and you uncheck the box to not send telemetry and let me be 100% clear like we've reviewed gone through Privacy review on our telemetry and there's no personal personally identifiable information being sent.
All right.
But even with that, some people prefer not to send any data back to Microsoft, which is fine.
But if you uncheck that box and if you either opt into Microsoft update to get updates or using wouldn't get to do updates, or you're reinstalling MSI, that installer did not preserve that option.
So you have to uncheck it every time, so if you're doing a non interactive update then it would revert back to the default, which is the sense telemetry.
So this is a problem.
So we did fix this with the help of the some community people.
UM to preserve the choices you made.
Specifically, MSI installer.
Uh, that way if you run it again, it will pick that up and use it the next time.
So one of the side effects of this, if it's not clear, is that you need the fixed version of the installer rechoose whatever options you want.
That would not get preserved in the registry to be used next time, but it only takes effect the next time you do an update, so it takes like 2 updates for this to be preserved going forward.
Hopefully that makes sense, but this should be addressed now for more point of view.
Umm.
And I'll look in the chat in the actually let me take a look in chat real quick.
Any question regarding this?
Uh, OK.
So the other thing I want to cover real quick is what's happened in DC V3.
So I am trying very hard to get a release out this month.
There's been a lot of back and work across the team regarding release pipelines and stuff like that and the release pipeline that I had for DCP 3 broke.
So it's been a challenge to get it working again, but I'm down to 1 issue and I'm hoping I can get that resolved today and get a release out today.
Otherwise, if I can't get out today, it will be next week.
There's been some a good changes coming in.
We are still targeting hitting like a GA around the end of this year and I think we're on a good path.
The only major blocking point right now is getting a compliant pipeline to ship out releases, so hopefully we'll get that out soon.
And with this one, we should be able to, well, I should be able to publish to the Microsoft Store, so I'll be I'll publicize that once it's working, there's a technical thing I had to fix due to how the store validates command line tools, so it was not accepted previously, but I think that's been addressed again if any questions go just put in chat. Thanks.
 
Sydney Smith (SHE/HER)   10:39
Thanks to you, I am just calling back up.
Amber, are you on the call to talk about PowerShell gallery statistics?
 
Amber Erickson   10:47
Yep, I'm here.
 
Sydney Smith (SHE/HER)   10:48
OK, great.
 
Amber Erickson   10:49
Umm, so I just wanted to give a quick update on the gallery stats.
Our individual stats pipeline was temporarily down.
We were in the process of migrating, updating to new Infrastructure.
The staff pipeline is back up, so all of the individual package statistics are up to date.
We didn't have any data loss at all during that transition.
It just wasn't running, so it wasn't updating the metrics that were coming in.
Uh, we do actually now have alerts in place to monitor us, to monitor the pipelines.
If there's anything that happens if the pipelines not working, if something strange occurs, so I know in the past sometimes the SADS pipeline has gone down and maybe it's been a week or at least a few days before anyone notices.
So now we should be alerted within a few minutes, half hour at the very latest to be notified that there's something going wrong.
So, uh, everything is OK with the sauce pipeline now and going forward, hopefully we'll be a lot more proactive with any issues that arise.
That's all.
 
Sydney Smith (SHE/HER)   12:05
Great.
Thank you, Amber, and thanks for all your work there.
I'm just monitoring the chat right now and I think most of the questions about telemetry are getting answered, but I did see a question raised about the PowerShell container images which have not been released since April.
The reason for this is that they we had to go under a major infrastructure change around the PowerShell build and release process.
The most complicated of this was the Docker children release process, so there have been unfortunately many delays in getting those releases back out.
We do expect those releases to go out in August.
Umm, but unfortunately we haven't been able to do a release since April because of those infrastructure changes.
I see some more questions, but I would go on to the next item and then we can come back to some questions in the chat.
Uh, so the next kind of section of items is around some different conferences that have been going on and are planned in the future.
So just last week we were able to attend Tech mentor, which was on Microsoft campus in Redmond, uh.
The team was able to participate in an three sessions, one of which Jason was able to collaborate with James Petty.
One session kind of like overviewing the history of PowerShell.
Then the team was able to do 2 sessions.
One was kind of a state of the shell, kind of like what's new in PowerShell, bit more, a little more learning and demo happy.
And then we also did a panel Q&A type session.
I know there was other members of the PowerShell community that were there, so it was really great to see you all and connect with you and as always, don't hesitate to reach out if you are gonna be at conferences and would like us to attend or collaborate or anything like that.
It's always very welcome.
And on that note, just wanted to Flag 2 conferences that I know are coming up this fall related to PowerShell or sort of events.
One is PowerShell Saturday, which is being put on by the Research Triangle group.
Phil, I see you're on the call.
Would you have any interest in speaking to this?
If not, that's all good.
 
Michael Kanakos   14:35
Hey, Sydney, it's Mike.
How are you, Mike connects.
 
Sydney Smith (SHE/HER)   14:37
My you're also on the call.
 
Speaker 1   14:38
Yeah, I was gonna defer to Mike, so that's great too.
 
Sydney Smith (SHE/HER)   14:41
Come on.
I'm just gonna put the link in the chat.
 
Michael Kanakos   14:44
So it's nice to see everybody.
It's so hard for me to make the call because my team decided to schedule a team meeting at the same time as a community call.
Uh, but.
 
Speaker 1   14:55
No.
 
Sydney Smith (SHE/HER)   14:56
I.
 
Michael Kanakos   14:56
Uh, what?
I wanted to say was we are having a PowerShell Saturday on October 5th and 6th and we would love for people to come and join us.
It's in the Raleigh area.
If you are in the local area or nearby, you can go to the website link that Phil posted in the chat.
We have a pretty great event plan that's actually a two day event.
We have a number of wonderful speakers from the Community that are going to be there.
We have shown Wheeler, who's going to be there, Steven Judd.
Adam Driscoll.
Jay Adams, Stevie Koster, Frank Lesniak and a bunch of others.
We're going to do a deep dive Sunday with Steve.
It's Steven, John and Sean Wheel are going into some deeper topics than we can do in the sessions.
We're gonna feed people.
We'll have lots of swag.
We'll have lots of stuff to do and so we would love if you guys could support us on this event and maybe come and join us.
And if you can't come and join us, I would ask that you can help us in another way.
So just a quick plug here.
You know, our community has never really recovered from pandemic times.
You know, attendance for local events is still pretty down.
You know, we're down to what I know of is 4 user groups.
James Brunder just trying to do something out in the northwest, you know, Andrew, Doug and our group and attendance has been tough.
And so if you can't come and attend our event, we totally understand.
But we're gonna have plenty of social media plugs if you can just maybe forward a social media tweet, or if you could say something about the speakers that are attending.
It would help get the word out to others who maybe have never been to a PowerShell Saturday.
So myself and Phil are in the chat.
If you have questions or you can go to the website, it would be happy to answer any questions, but it's gonna be a fantastic event.
Raleigh is a great place to come and hang out for a couple of days, so if you feel like coming and hanging out with us, we would love to have you there.
 
Sydney Smith (SHE/HER)   16:47
Awesome.
Thank you so much for that.
 
Speaker 1   16:49
Thank you.
 
Sydney Smith (SHE/HER)   16:50
Looks to be a great, great event.
Another event that is going to be virtual this fall that I know of.
Really powerful.
It's PS called to you put on their mini calling event which is coming up in October.
Don't think I see anyone on the call from that event that I'm going to post the link in the chat from where?
Information on that that is posted on gather so it's a fully virtual platform and a great place to connect with folks in the PowerShell community.
That's all I am.
Someone other comment question in the chat here that got quite a few votes relating to PowerShell storing modules in OneDrive and it starts being any progress made on that issue.
So I would say uh, we don't have any immediate plan to change where modules are being installed to their shows like immediate planned work and that.
Realm, but it's definitely an ongoing issue that we we talk about I think in many ways I could see it going more of like a configurable path and in PowerShell seven, we did add an API to determine where the installed module location would be at the stack cords RESERVING this issue.
And I'm more configurable way.
One note is that if you are having the issue with like your modules like syncing to OneDrive and that causing like latency issues or other types of problem, you can't configure OneDrive to not sync PowerShell files.
So you can there is like a setting in OneDrive to not sync like any files and then like.
Yes.
So anything like that, so that would be one thing to explore if you as sort of like an immediate fix, if you are having those types of issues.
But yeah, I think it's gonna be a really complex issue once you start to dig into it, sort of the different effect with so many folks out there using PowerShell and different versions of PowerShell to really solve this problem.
But it is something, I guess on her mind it does anybody else wanna say anything about that?
OK.
Umm.
Then we will move on to community demos.
I think the first one we had up here.
Umm, we're gonna be that DSC.
I will give you.
Are you here for the demo?
 
Reijn, GWJ (Gijs)   19:26
Uh, Sydney?
 
Sydney Smith (SHE/HER)   19:28
Hey, I mean give you presenter rights.
 
Reijn, GWJ (Gijs)   19:29
Yes.
 
Sydney Smith (SHE/HER)   19:30
So you can share and everything.
 
Reijn, GWJ (Gijs)   19:32
Thanks a lot.
 
Sydney Smith (SHE/HER)   19:44
There we go.
Got you.
Already you should be good to go, yeah.
 
Reijn, GWJ (Gijs)   19:50
Good, thanks. Alright.
Yeah, very short introduction.
I'll I'll keep this demo short.
I was booking a little bit DC version three and and when I actually was working with with DC, I quickly found myself working more with PowerShell objects.
So I thought, well, why not create a module on it and probably a little bit ambitious with with the naming, but I it's it's something.
And what I mainly wanted to focus is more on the authority altering experience for people and also have the ability to convert your version one and two documents instantly to version three, which I created actually 2 commands.
Also, if you can see here just a very basic very basic example on the configuration which is in first one or two and when I run this command.
Uh, you will see that it creates a Jason that can instantly be fed to DC itself. So.
If I didn't screw up the demo.
So as you can see then it's converted to a configuration document in a format for version three.
So if you run it now with the C itself.
You can see that it instantly works well that I'm working behind the scenes with also the adapter.
So I messed up the demo a little bit, but anyway.
That brought me to the next step and creative wrapper around the do you see resource command itself?
And I was also playing a little bit around with the argument, complete this on it.
So if you now say for example, do you see comments?
Umm.
And watch it. Umm.
You can see the argument completely sure already there.
I'm still a little bit new to it, but it doesn't bode for the PowerShell adapters but also the sources that are available.
Currently yeah, this demo broke a little bit because I'm again working on the adapter, so I screwed up a little bit so I'm that was it actually. Thanks.
 
Sydney Smith (SHE/HER)   22:49
Thank you. Uh.
 
James Brundage   22:50
Thanks.
 
Sydney Smith (SHE/HER)   22:53
Ohh many yes.
Always playing the demo gods.
Ohh Thomas, you also had a demo for today.
You saw it throwing it.
 
Thomas Nieto   23:04
Yep, I'm still ready.
 
Sydney Smith (SHE/HER)   23:06
Alrighty, I will Grandview rights you presenter UM.
 
Thomas Nieto   23:06
You got me.
Sounds good.
 
Sydney Smith (SHE/HER)   23:17
Sorry, you always just takes a second because I have to scroll through everyone.
You OK?
You should be good to go.
 
Thomas Nieto   23:36
Alright.
Can you see the screen?
 
Sydney Smith (SHE/HER)   23:39
Yeah, looks good.
 
Thomas Nieto   23:40
Alright.
So, uh, last year I demoed back in April of 2023, my module called any package which is the one git or package Management replacement.
Basically what it is, it's an extensible, unified package management interface.
So let's say you have your scoop Wingate.
Chocolaty.
Whatever you can manage it all from PowerShell, all with the standardized commandlets, so it will do the translation for you and do all that fun stuff.
And So what I've been doing in the last month is with the full command completer predictors and feedback providers, I went ahead and created interface for those package providers to allow it to hook into the command not found peace.
And so right now.
UH-2 package managers installed and imported, so I have the PS resource get and SCOOP and with that they went ahead.
And now if I wanna go ahead and say, let's say I don't have been installed, it will go ahead, find the command not found query, scoop query whatever imported ones they are, find those commands from the packages and then return the results to the user.
So now I can go ahead and select that.
Also has a a tool tip down here, so if you were to run it later and you were to forget what, you know what the recommendations were for, it will show you which command it has.
So the one that I really enjoy is let's say I don't have the start Log command command that installed.
It's gonna query PS resource, go on the gallery and the internal repos.
Whatever caches all those results builds a Dictionary of it and quickly returns the results.
So you can see start log is contained in those.
I don't know 10 or 15 ish packages, so I can go ahead and install one.
If I flip desired.
So that's basically the demo.
It's just a nice easy way to, you know, run a command.
It's not there query any package manager that supports this functionality.
Right now I have it in SCOOP and PS resource get, so that's all.
Thanks for listening.
 
Sydney Smith (SHE/HER)   25:53
Well, it looks like you have maybe a question from James.
 
James Brundage   25:56
Yeah.
Question, comment and bit of quick feedback a congratulations on finding one of the coolest tricky areas in PowerShell because command not found is overpowered and broken as you have definitely already figured out.
Umm B.
Just in terms of being polite to everybody, if you are overriding command not found, please please create a PowerShell event called PowerShell DOT Command not found and broadcast it just in case anybody else is also wanting to override command not found.
Otherwise, we're kind of setting ourselves up for a prompt redux and see to the PowerShell team.
Uh, this is something that the kind of come up as we had talked about, you know, the argument completers and the prompting that you've been doing.
This approach is really fantastic and a little bit easier to use than require in the extra installs and it would be in a good example case of where all of these built-in delegates should actually be events so that they can be more commonly used and commonly overwritten and more securely used and securely overwritten.
So does that make sense to all parties?
And again underlining the compliment, congratulations on finding one of the coolest tricks in PowerShell.
 
Thomas Nieto   27:23
Yeah.
And James, this piece is using the feedback provider.
There is a feedback type of command not found, so this is what it's hooking into that.
So basically the feedback provider on the command not found is in triggering the.
Any feedback provider that subscribes to that basically and we'll call that.
 
Sydney Smith (SHE/HER)   27:41
Yeah, very cool.
Really great demo, Thomas.
Very cool to see you.
This is something that is always asked for when we group feedback providers, Ryan.
I also see your hand up.
Ohh videos accident.
 
Ryan.Wakefield   27:57
It helps with my headset, actually.
You know unmutes itself when I click it.
 
Sydney Smith (SHE/HER)   28:00
Ohh yeah, you got it.
 
Ryan.Wakefield   28:03
No, I I was.
I was curious from Thomas.
You know, you mentioned obviously either you know PS resource guy and the other one are you looking to add like a local repo especially for orgs that have their own modules and everything internal only and whether you can query those.
And I guess I'm also going to be curious to see how you're querying for those commands.
 
Thomas Nieto   28:26
Yeah.
So this will work off of any repo as long as it's registered with PS resource get.
So basically what it's doing is when the module is imported, it creates a run space to cache those results and it just runs fine.
PS Resource Git on or find PS resource on all of the results and then build a dictionary off of that.
What would be nice?
I think Sydney has a an issue out there for caching PS Gallery and creating like a a local cache for that.
So I don't know if in the future I could go ahead and create a have action to, you know, do that once for everybody and then just download the cache file.
But right now it's just building that cache locally, but yes, it does work.
 
Ryan.Wakefield   29:04
So it's building it from the complete, you know, PowerShell Gallery itself, I assume.
 
Thomas Nieto   29:10
Right.
It's just it's just calling fine PS resource on everything.
 
Ryan.Wakefield   29:11
Ohh.
 
Thomas Nieto   29:13
So if you have any internal repos, it's gonna return all those results as well.
 
Ryan.Wakefield   29:17
OK, cool.
Thank you.
And I agree, and I will also say Congrats on finding an amazing way to use commandlet not found.
That's a big bane of a lot of our existences, I'm sure.
 
Thomas Nieto   29:29
Yep.
And then for the last thing I would say is I have package providers out there.
So I have a about 10:00 or so right now, just in grody is module not found or sorry module fast.
I went ahead and created .net tool in the last year and there's some other ones like Chocolatey MSU's program.
So Windows programs we talked about PS resource, Git, scoop winning, Git, PowerShell and then on the Linux side we have App, Home Brew and a couple others.
But anyone's welcome to creative if you'd like to collaborate, or if you'd like to have any other command not found integration, that'd be great to to do, and I'll go ahead and link the repos as well in the chat.
Thank you.
 
Sydney Smith (SHE/HER)   30:11
I'm fine.
Thank you so much.
Thank you for all the participation today and all the demos and we really appreciate it.
And looking forward to seeing everyone next month, I think we basically are perfectly right on time unless anybody questions got missed today, I'll pause for a second and just make sure we didn't miss anything.
Umm, it's like looking for a link to working group sign up form.
But yeah, again, thanks for all the participation today and looking forward to seeing you all again next time.
Thanks everyone.
 
Jason Helmick stopped transcription
