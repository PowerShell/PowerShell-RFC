# PowerShell_OpenSSH Community Call-20231130_092605-Chat Transcript


Michael Greene 0:36 

Hello everyone. We've got about 5 minutes or so before we get started, but I
just wanted to thank you for joining the meeting is being recorded as Jason just put in the chat,
you can find previous recordings at the YouTube channel and the link it for that is also in the chat
as well as our code of conduct and our contribution guide.


So if you're not familiar with those before the meeting starts, now, it's a great time to take advantage and go through those things.


And then finally, the agenda for today's call.
It comes from a discussion channel in GitHub and that is the last link that you will find there in that chat.
So we look forward to talking this morning.
We've got a lot of topics to cover and we'll get started here in about 3 or 4 minutes.

All right, it is 931.
I would say let's go ahead and get started.

Uh, let me just restate what I was covering before in case you weren't joining.

And in case you hadn't joined yet, so this is the November 2023 PowerShell community call.

This is an open community.
Everyone is welcome and for anybody who's joining us for the first time today, thank you for joining us and we look forward to a really great discussion.
This meeting is recorded and all the recordings of the past series of meetings are available on the YouTube channel which is linked there in the chat.
Also, Please remember our code of conduct and our contract and the guidance provided in our contribution guide a contributing document.
So if you haven't before, now is a good time to go take a quick look at our code of conduct and how to be an effective contributor to the community.

And finally, the very last link that you'll see there is the GitHub discussion which contains our agenda for today, which I happen to have open in front of me.

So let's go ahead and get right to it, because as you can see on the agenda, we have a lot to cover.

We wanted to kick things off talking about the GitHub bot discussion and for anybody who might be joining the call without context for that, I wanted to explain what it is and then open the floor for feedback.

So essentially we over the past couple of months we've been looking at the PowerShell open source project and trying to look at ways that we can improve and we collectively decided that the issue list has grown to a point where it's not practical to manage or just generally unmanageable.

And we wanted to think about the issue list more as a stream of things that are incoming and outgoing in real time and not just a list to that's continuing to grow.


So we started looking at what essentially one I'll explain on that one bit more, which is when an issue comes in, we wanna be able to act on it.
It should get triaged usually by a working group and then it should either we should agree to act on it or we should close it with comments out of respect for the author who put their time into submitting the issue.


We shouldn't just leave it open for for long, long periods of time, and many of the issues that were in the list were actually more than a year old.
Some of them were quite old, in fact, which is not being a good steward of the project.
And so and that actually offers an opportunity where if an issue gets closed, the author can go add additional context to the issue and submit it to be retreaded.
That's kind of a concept that we wanna do.
Encourage is let's not just have these things hanging out there for a long time.


If for some reason it's not being worked on then maybe we need to get more information and make a new decision, and so we went out and observed a bunch of other projects across GitHub, including projects like ASCII and AZ PowerShell, and concluded that using automation, which obviously is a core tenant of the way we look at the world anyway, it has been a common approach for a lot of these large large scale projects.

Brown, Derek joined the meeting

Michael Greene   8:09
We did kind of acknowledge that setting a rolling time scale will impact older issues, which means as we're thinking about this as OK, the bot basically works on comments and labels and provides automation based on those patterns.

And we were thinking about it as in issue really should be handled within six months and handled can mean that we decide to do something about it and fix it or or take on a new feature or it can mean that we just our transparent and provide feedback that this we have to have the courage to say no, we're not gonna be able to work on that.

So we should respectfully communicate that back to the author, but that rolling time scale means, well, what do we do with the issues that are already in the list that are older than six months and as a consequence of that ruling time scale, you have to go close them.
It's not really what we wanted, but it does allow the authors of those issues to then go add new content, reopen them and get them retreaded, and then for things that no longer apply, they can remain closed is the way we were thinking about it.

We've had a lot of feedback or the past week about the results.
Love that early bulk closure that gets us into that rolling time scale as a result of that, the changes we've made so far, the bot originally was whenever it first published onto an issue and was saying here's a seven day notification that if there's no further activity, this issue will be closed.

It was posting that message twice, which meant a lot of people were getting too email notifications when that seven day window started.

BRANDWOOD Jamie joined the meeting

Michael Greene   9:48
So that has been fixed and going forward only one comment will go in about the seven day notification as it approaches, this is the end of the six month window and then also we've revised the text.
This change hasn't gone in place yet, but just last night in the committee meeting we went through and created a new text both for the seven day notification and for the closure notification that we felt provided a little bit more context about the intention of why that activity is happening.
And then exactly what to do, either to keep it open or after it has been closed.
If you would like to reopen it, or if you're not the author, but you feel strongly that it should be reopened, how you can go about that?
So with that, let's open the floor for discussion.
I will ask because we do a lot on the agenda that if the discussion is gonna go past about 9:45, then we should schedule a dedicated meeting in January to focus specifically on this topic.

But let's first open this floor and have about a 10 minute conversation.
And just think in depth about the use of automation to manage the issue list.
You're welcome to provide critical feedback as well, like critical feedback, meaning that if you feel strongly that this isn't the right approach, I'm happy to hear that.
Or if you feel like we should make changes to improve upon this, we're certainly happy to hear that as well.

Steve Lee (POWERSHELL HE/HIM)   11:10
They, Michael.
Kenneth, just add a a few things first.

Michael Greene   11:12
Certainly, yeah.

Steve Lee (POWERSHELL HE/HIM)   11:13
So so one of the things I think there's two points of feedback that we're important and one of the things that we did talk about as a committee that we couldn't enact is really to leverage reactions, right.
The basically the bot that we're leveraging has no way to query reaction reactions, meaning like thumbs up, thumbs down, stuff like that.
So I think for now what we decided is we're gonna have like a manual process that we can empower the working groups such that we have a new label simply called.
Keep hoping, and if it working group notices that an issue has a lot of activity and that activity but a lot of reactions which doesn't trigger or reset the bot, then they can put that label on their along with other labels like the particular working group that probably should be part of that discussion, or at least review what to do with it.
And in that case, the bot won't close those, right?
So anything to keep open, we'll just be open indefinitely until someone takes action.
So that was something new that we're gonna do until we can get actual bot or automated tools around that.
The other thing is we recognize that one mistake we made is that, you know, if an issue was never triaged, we we probably should not have just auto close this too late now to do that.
So I think going forward, we also fixing the bot so that if it's never been true, I mean no person has made some call on that issue then that would also be left open, right?
So at least with those two minor things, hopefully things will be better going forward.
Uh, yes, Justin, if you want to author that, then by all means, uh, I'll spend time reviewing it, referring to get of action.
I know that the the graph QL query that getup supports supports getting stuff like not only reactions, but also people who may have umm, what does it not subscribe?
But I've like, what's the term I'm looking for?

Damien Caro joined the meeting

Michael Greene   12:59
Yep, so that you can sign up for.
You can subscribe to notifications, meaning you want to get email whenever any activity happens in the issue, but you can also submit a reaction which let you choose from a list of emojis.
And I think we should probably just have a dedicated issue to collect feedback on how to approach this and kind of see like which which do people prefer, do they want us to actually summarize both and we can figure that out later, not on the call implementation.

Steve Lee (POWERSHELL HE/HIM)   13:28
Yeah, I I think the the important thing here right is like when when there's an issue someone had opened.
And yes, we recognize this important to that person, but we don't see any activity.
We don't see any reactions.
It's hard to really figure out like what is really the priority of that amongst all the other thousands of issues that I've been open, right?

Michael Bass joined the meeting

Steve Lee (POWERSHELL HE/HIM)   13:45
So we have to make a call based on what we see.

Andrei Henrique Possamai left the meeting

Steve Lee (POWERSHELL HE/HIM)   13:47
Unfortunately, that means a whole bunch of them.
Kind of just went away for now.

Michael Greene   14:01
I'll leave the floor open and and just kind of enjoy the silence for a couple of minutes here.
If anybody would like to chime in with thoughts.

Ryan Yates   14:08
I was just going to say I I noticed that Steven mentioned that in one of the comments and one of the issues recently there, there's a possibility of moving away from the current bot that you currently use to probably the one that's part of the GitHub platform itself, and which might have more, easier to use and ways forward.

Factor, Aaron joined the meeting

Ryan Yates   14:30
With this I I think we still need the bot.

Michael Greene   14:32
Yeah.

Ryan Yates   14:33
I added my comments into a matter to you yesterday.
It took me a couple of days to write that that comment, but I do think it's needed going forward.
I think it's just been a an interesting experience with this.
We wouldn't be the way I'd done it, but it it's done.
Now we just gotta move forward, haven't we?

Steve Lee (POWERSHELL HE/HIM)   14:56
The definitely learning experience for everyone.
It's gonna take me some time.
Uh Ryan to read your comments out, I'll do that later.

Ryan Yates   15:07
Yeah.

Steve Lee (POWERSHELL HE/HIM)   15:07
I appreciate the detail right up is rather long for me to take time right now.

Michael Greene   15:11
Yeah.

Steve Lee (POWERSHELL HE/HIM)   15:15
Again, I I would just point out like this is an evolving thing, right?
Like we, we had to do something because the thousands of issues they're not being looked at.
And so that was not doing the right thing for those set of folks.
So we're hoping that by having a rolling set and having empowering the working groups more that we can actually surface what's really critical for both the community and the team to really focus on addressing clipboard.

Michael Greene   15:26
Exactly.

Christopher Nguyen joined the meeting

Michael Greene   15:42
Yeah, the intention that I, I really, I I have empathy for the responses.
I have seen that said, you know, I put my personal time into uncovering this issue and submitting it, and now it's been closed and I have a lot of empathy for that situation.
The intention is that that issue was important and how do we provide a path where we can bring it into focus and if it's in a pool of such a large number of issues, it's gonna be difficult to sort of draw those most impacting issues out.
And so this provides an Ave where we can say, OK, let's close that the large number and then for those issues that are really impacting, people can click the button that says reopen, they'll get retrial aged, maybe they were mislabeled, maybe things have changed etcetera, etcetera, etcetera.
And we can sort of like shine a new light on those things.
So I don't know, we probably could have done a blog post or something like that to provide more thinking, but it probably wouldn't have changed the end result, which is it's tough if you spend your time creating an issue and it's been closed.
It's tough to absorb that.
You need to go in and reopen it, especially if you have a lot of them and so going forward that will be something that we kind of keep in mind.
Says, making sure that everything has been appropriately triaged and that our messaging in the closure is that it doesn't mean your issue wasn't important.
It might just need more information and need to be reevaluated and so forth, and so that was a learning experience.

Steve Lee (POWERSHELL HE/HIM)   17:24
And let me actually remind her I remember now one of the feedback or one of the statements someone made in the meta issue, I think.
So I wanna be clear.
Like there is no key API that was driving this decision right?
Like.
No one from Microsoft telling us, hey, you can't have issues this or whether it's more like we got feedback from people.

Michael Greene   17:35
Right.

Steve Lee (POWERSHELL HE/HIM)   17:40
Hey, some stuff is like years old not being looked at and you know we don't have a big team here.
So we're trying to do the best that we can and we really need to kind of surface what's most important for the community to spend time on.
And this is one of the things we decided is we gotta just reduce the number of issues.

Guest left the meeting

Steve Lee (POWERSHELL HE/HIM)   17:58
700 still large, the set issues, by the way.

Michael Greene   18:00
It is, yeah.
Had comments.
And so you now have 700 plus issues, some posted on 2016 with no progress issues like issues like this that were closed will be reopened by the Community.

Phil Bossman joined the meeting

Michael Greene   18:16
But I think it's similar ones with a + 1 from 5 months ago are opened.
Will it be reprocessed?
Yeah, I uh, after it's been closed.
The what we're thinking, and the bot may need some maintenance to get us to exactly this behavior.
But we wanna do this short term, meaning we're implementing these changes now that after it's closed, if it gets reopened, it should be retrial aged.

Grote, Justin left the meeting

Michael Greene   18:42
The idea here is that the author is basically saying I need you to take a fresh look and the hope would be that they are adding more information at that point.
Like, OK, you looked at it.
Maybe you didn't understand what I was trying to tell you.
Let me add some more information or maybe you know 15 people have subscribed to the issue which tells us this is not coming from a single person.
This is impacting a lot of people, that kind of stuff is the way we're thinking about interpreting the reopen and then having it go through the triage process.
Again, if that makes sense, then that that six month clock would start at the point where it has been triaged, not at the point where it's been opened.
And then there was a question, will it be tracking how long it takes us from the point where the issue is open to when it has been triaged?
And yes, that is something that we're gonna 100% I believe is important to view as a metric about our effectiveness at reaching triage.

Ryan Yates   19:49
Yeah, just add on that.
It's not just about the original and commenter that says this is an issue, but it'll be anybody that has commented within that issue that can replicate it as well.

Mathias R. Jessen (IISResetMe)   20:01
Yeah.

Ryan Yates   20:02
And because the problem that you have with anything like this is somebody who's raised an issue and then has gone away because then they don't, they no longer care about the project or they maybe they don't actually use use it anymore.

Januszko, Missy joined the meeting

Ryan Yates   20:17
And so we have to be.
Mindful that there's no point having multiple issues raised with the same thing when we've got an actual issue already for it.

Michael Greene   20:30
Yeah.

Mathias R. Jessen (IISResetMe)   20:30
Yeah.

Michael Greene   20:33
Agreed.

Mathias R. Jessen (IISResetMe)   20:33
I think from my perspective, as someone who is both opened, some of these issues in the past, that has now been closed and it's now sitting sort of on the other side of the table having to try some of them and working groups.
My immediate consideration is that.
Some things would get lost if you ask the original author of an issue.
To summarize, a discussion amongst 16 people, right.
So if I opened an issue in 2018 and I had good feedback, right, good traction, a bunch of people chimed in.

Steve Lee (POWERSHELL HE/HIM)   20:55
Yeah.

Mathias R. Jessen (IISResetMe)   21:02
Some people misunderstood it.
I came up with some better.
Whatever, right, so this conversation goes on amongst a bunch of people.
And then you're asking me now, five years later, to go back and try to sort of reinterpret what a bunch of other people misunderstood.
When I first said it right, I also fear that in some cases you're probably right, Michael, right.
We'll get more clarity, but in some cases we'll also lose something, right?
There's a bit of sort of a game of telephone involved in trying to recreate the original request as a new one.

Michael Greene   21:23
Umm.
I see.

Ryan Yates   21:32
Yeah.
And also we will slapped a lot since then.
So you know things things have definitely changed as well.
The you know the landscape has changed quite a bit.
There's probably quite a lot of these issues where we still haven't tracked that they've actually been fixed by certain changes that were either caused by changes in .net or actually changes within hour repo ourselves.

Michael Greene   21:50
Yep.

Antenor, Daniel joined the meeting

Ryan Yates   21:55
So we we need to at some point try and map those up together so that we can go back through the change logs in future and say actually a lot of these things that we've got had open actually worth fixed via the the releases that we've had in, in the past.

Michael Greene   22:13
Hmm.
That's a good point, yeah.

Mathias R. Jessen (IISResetMe)   22:20
One last thing, I think Justin also mentioned this in the chat the IT would be a huge pain if we have to like fight with the platform, right?

Michael Greene   22:21
Sure.

Mathias R. Jessen (IISResetMe)   22:29
So like if you go back and and say like ohh I have a 5 year old issue here it was close.

Nieto,Thomas joined the meeting

Mathias R. Jessen (IISResetMe)   22:33
I wanna reopen it because it's still an issue and I think it's still relevant, so I reopened it or I tried to reopen and then I have to like fight with a GitHub bot that wants me to either reopen new, whatever, right? Yeah.

Steve Lee (POWERSHELL HE/HIM)   22:46
Well, Martin is I think in that case I think the the, the hope is that if it's important that the one of the working groups that should be looking at that issue will put the label on it, right, like cable open or have the discussion and decide what to do with it.

Michael Greene   22:46
I I'm yeah.

Steve Lee (POWERSHELL HE/HIM)   23:00
So that individuals don't have to keep gaming the system if you will, to just keep it open.

Mathias R. Jessen (IISResetMe)   23:01
Right.

Michael Greene   23:04
Yeah.

Mathias R. Jessen (IISResetMe)   23:04
Yeah, exactly.

Michael Greene   23:05
Yeah, this this is actually a really important thing.

Mathias R. Jessen (IISResetMe)   23:05
Exactly, yeah.

Michael Greene   23:08
And then we'll have to move on to the next topic here pretty shortly.
But uh, as working group members were going to have to take on new courage to be willing to close issues, and that is not as easy as it sounds, because we all have empathy for people who have submitted them, and we, we might even recognize like this is a real issue and people are having to work around it.
But we have to have the courage to say, but if we're not going to fix it, we're not really doing anyone a service by just leaving it open.
So we have to be had the courage to do something about it, including including just being transparent and saying we're not gonna be able to work on this right now.
So we're gonna close it and.
You know, if it comes back in a new form in the in the future and it includes more information than we'll take a fresh look, that is not as easy as it sounds.
Uh, so that's just something we're gonna have to learn to do.
OK.
The next one I'm going to open the floor to Steve Lee to talk about 7.4 GA and if there is any additional need for discussion of the bot, it's something that we can keep going in our monthly community call and we also could even have a dedicated discussion just on that point.

Steve Lee (POWERSHELL HE/HIM)   24:27
Well, we we have the meta issue, we can probably just keep you know, as long as there's activity that one won't close, right, so.

Michael Greene   24:27
Uh.
With that submission, great.
Right.
Great.
Perfect.

Steve Lee (POWERSHELL HE/HIM)   24:36
And that the committee will probably continuously review that issue and see what comes out.

Michael Greene   24:36
That sounds good.

Ryan Yates   24:39
We see we should probably convert that matter issue into a discussion because it's a discussion not really an issue.

Michael Greene   24:40
Uh, yeah.

Steve Lee (POWERSHELL HE/HIM)   24:45
Yes.

Michael Greene   24:45
Yeah, yeah, yeah.

Ryan Yates   24:47
I'll.

Michael Greene   24:47
Issues are not great for chat.

Ryan Yates   24:47
I'll.
I'll go.
I'll go and do that now while someone.

Steve Lee (POWERSHELL HE/HIM)   24:50
OK, that's a great point, Ryan.

Michael Greene   24:51
Thank you, Ryan.

Steve Lee (POWERSHELL HE/HIM)   24:51
Thank you.

Michael Greene   24:52
Very good point.

Steve Lee (POWERSHELL HE/HIM)   24:53
OK, so why don't we shift to we shift to a 7.4 GPA.

Michael Greene   24:54
Yes, it's issues are not great for chat.

Steve Lee (POWERSHELL HE/HIM)   24:58
So we did a GA meaning general availability of 7.4 earlier this month, which means that it is our current LTTS.
Now 7.2 is still under support for another year, so there's a one year overlap between LTC meet and long term supported versions and LTS versions have a three year support lifecycle.
So we encourage anyone who is on 7/2 to consider a validating 7 four and shifting on the seven four you have a year to do that. Umm.
And then we're gonna start.
I don't know when we'll have our first preview release because we do have a number of engineers out now already towards the end of this year.

Thomas Marantz joined the meeting

Steve Lee (POWERSHELL HE/HIM)   25:35
So if we don't get a preview out of seven, five, which I would still be on net 8 next month, then it'll probably happen in January with the caveat that we again we have a small team.
So whenever we have servicing releases that have security fixes, typically right now, that would mean that we would have a 72 update, A73 update and A74 update.
So that's three releases in a month, so the general guidance I've given the team as when we do that, we're going to postpone a 753 because it's having one more forward leases in a month is just means we can't do anything else, right.
And I'd rather do other stuff as well.
So we may skip a month if we have a concurrent servicing releases have to go out, but hopefully we'll get a 75 preview early because there has been a lot of changes in the master branch since the first release candidate, right?
Since that time we've branched off.
And so none of the changes have been happening in Master branch have flowed into 74.
Umm.
Hopefully if you have not seen already, I do have the blog post on the partial blog about some of the major things have to happen in support.

James Brundage joined the meeting

Steve Lee (POWERSHELL HE/HIM)   26:45
Not all the changes, I mean, there's been a lot of fixes in small changes from the community, which we highly appreciate, but some of the major things and.
Not really.
Talk about 7-5 yet, so look forward to my blog hopefully.
Late Jan or early February when I do have a the customary team investments blog about not just PowerShell but also like open SSH gallery and stuff like that because our team does more than just PowerShell right?
So hopefully they'll come up very early in 2024.

Phil Bossman left the meeting

Michael Greene   27:18
No shortage of ideas and and passion for 7/5.

Steve Lee (POWERSHELL HE/HIM)   27:18
If there's any questions, yeah.
Again, some 5 is not an LTS, so that will be a what's called an STS or was it standard term support or short term?
I don't know.
I don't know the .net term is for that, but it's one year one year support so.

Michael Greene   27:38
For the next agenda item, I'd like to flip items three and five.
I'd like to invite Vikram to go next.
If you're ready and talk about Azure automation, because I know it's quite late for you, but I appreciate you joining the call and your team as well.
If you're still on.

Vikram Medari   28:00
Ohh yeah we are here.

Michael Greene   28:02
Right. Wonderful.

Vikram Medari   28:02
Thanks Michael.
Nick.
Nikita, who is the PM for Azure automation?
And then I have Sanju engineering manager, so they would be talking on behalf of Azure Automation.

Nikita Bajaj   28:15
Yeah.
Thanks, Michael.
Thanks, Vikram.
I will just quickly share my screen.
Can someone confirm if you can see?

Steve Lee (POWERSHELL HE/HIM)   28:29
Yep, I can see it.

Nikita Bajaj   28:31
OK.
Thanks.
So I believe most of you would already know about Azure Automation, but just for the sake of reiterating, automation provides you a serverless platform to execute your partial scripts as well as Python scripts in case some of your Python users here.
And in case you don't want to go completely serverless and execute it on a machine of your choice, that's arc enabled, then you can just install the hybrid worker extension and and run a an automation job there also so that that's the flexibility that automation provides.

Vivian Thiebaut left the meeting

Nikita Bajaj   29:10
And when you just compare it to inshell executions execution, you can execute complex orchestration scenarios here where there are a lot of these parent child runbooks where one runbook depends on the other or also for state management where automation is being used in addition to very regular mundane scenarios that are related to you know weekly or monthly maintenance tasks.

Grote, Justin joined the meeting

Nikita Bajaj   29:43
So just quickly, I'll talk about the releases that we have done in the last few months.
Once we recently announced support for partial 7.2 runbooks that went GA General generally available this week, so I'm sure most of you would be wondering that Steve said they announced partial 7.4 GA and Azure automation is still on PowerShell 7.2, but just rest assured we are.
Actively working on reducing the gap at which partial releases it.
The newer versions and automation supports them in their runbooks, so you would see faster release cadence in the coming months as well as we have newer versions of PowerShell.
Also, about two months back we announced umm the automation extension for VS code that when generally available and it it is powered with GitHub copilot.
So it just improves your your script authoring experience.
I think most of you might have already used it and for off Azure Automation as I already mentioned, there is the.
You can configure any machine as a hybrid worker and run the script there.
Since it's based on the VM Extension framework it is.
It is very simple.
You can just install the extension and run the script there.
So let me just pause in case anybody has any questions or comments.

Andrey Vernigora joined the meeting

Nikita Bajaj   31:30
OK, so in case the you want to reach out to us anytime, please the send us an email and ask azure.automation@microsoft.com or if you have any feature requests just effort on the ideas portal or add any feature requests there or you can reach out to us also on Twitter at uh the Twitter handle is at Azure Automation.
So, hoping to hear from most of you here.
Thank you.
Thank you for taking out time.

Michael Greene   32:10
And I put in the chat.
If there's anybody who is actively using Azure automation to to execute their scripts and to manage orchestration at scale if you'd like to provide feedback, or if you would just like to say hello to the team or thank you to the team, feel free to use the chat window or I'll leave a couple of moments of silence here in case anybody would like to come off of mute.

Vikram Medari   32:34
I think there is a cushion from Levis like that.
Do you wanna take it?
Will there be an update to the available AZ modules? Yeah.

Nikita Bajaj   32:41
Ohh yeah Bill due to the.
Yeah.
So we are coming up with a new feature called runtime Environment where we give you the complete control to configure the job execution environment.
You can pick up the modules that you want at the time of execution instead of loading all the modules that are present in your automation account.
So I think another two weeks from now we should be ready to announce public preview for that feature.
And in there you would be able to get the latest easy modules.
I hope that answered your question.
OK. Thanks.

Vikram Medari   33:23
Yeah.
And that includes the HCLG as well, right?
Like in in the coming.

Nikita Bajaj   33:26
Yeah.

Meelis Nigols joined the meeting

Nikita Bajaj   33:30
Yeah.
So uh, so the support for Azure CLI commands in PowerShell 7.2 Runbooks is also coming along with the runtime environment feature.

Marc A Goff left the meeting

Nikita Bajaj   33:42
I think that has been an ask from a lot of you.
Uh, and we are almost there.
One was that one ounce.

Mutemwa Masheke joined the meeting

Michael Greene   34:01
I see a couple of comments in the chat.
Just people saying they love Azure automation and they're using hybrid runbook workers and they're working very well.

Nikita Bajaj   34:03
Yeah.

Britton Johnston left the meeting

Michael Greene   34:08
So that's great to hear and thank you.

Nikita Bajaj   34:11
Yeah.
Thank you.
Helps us, though.
Be assured that yes, we are going in the right direction.

Michael Greene   34:20
One, thank you very much for the Azure automation team for joining.
I know it's about 11:30 PM, I think where you're at.
So I appreciate you staying up late to join the call and I added to the chat.
We are.
We would like in 2024 to have at least a couple of instances of the Community call that are more time zone friendly for geographies around the globe.
So if anybody has any opinions on that or would like to provide feedback, that would also be great to hear.

Bridget Tueffers (SHE/HER) left the meeting

Nikita Bajaj   34:51
Thank you everyone.

Michael Greene   34:55
Great.
Keep the comments coming as we transition over to the next topic.
Mr Sean Wheeler, would you be interested in providing us with an update on documentation?

Sean Wheeler   35:05
Certainly.
So with the release of the GA releases, have not four.
Of course, the we had to update the release notes.
These have all been updated with the the latest information now.

Phil Bossman joined the meeting

Sean Wheeler   35:20
And.
It no longer says preview for seven four also recently.
I archived a bunch of content for the Windows PowerShell information and added this article.

Andrey Vernigora left the meeting

Sean Wheeler   35:41
I realized that we hadn't.
We didn't have like one place that explained the difference between Windows PowerShell and PowerShell, and it's still a question we get umm from a lot of folks, the.

Amber Erickson joined the meeting

Sean Wheeler   35:56
In November, the oldest versions of Windows finally dropped out of support.

Luke Murray joined the meeting

Sean Wheeler   36:05
Umm, so there was a lot of content here about how to install Windows PowerShell.
Umm, that's no longer relevant because the only supported operating systems there are currently it's preinstalled.

Phil Bossman left the meeting

Sean Wheeler   36:21
If for some reason you're looking for that old content, you can go to the previous versions link here in the version picker dropdown and all that content is available here.
Umm, some other things.
Ohh, drop a bunch of links in the chat here.
Updated the get what's new module.
Uh, with the the 7.4 release notes.
Uh.
Published that to the gallery.
Updated the SDK content.
So with the latest 7.4 release.
So that's now GA content and then.
Mikey Lombardi has done a a huge uplift of the class and enum documentation, including adding a bunch of new articles here.
So umm it it's it's almost a total rewrite of classes and lots of added information.
Uh, that was kind of spread out all over or wasn't easy to find.
Umm, so, uh.
And it was just too much to put into one.
So he's split it up here into articles about constructors.
Expand the table contents.
Here you can see the types of things in the article talks about inheritance with examples.
An article about class methods.
And.
Working with derived class methods and.
Properties as well, so lots of great content here.
Here.
Umm.
And that's my dog's update.

Thomas Marantz left the meeting

Michael Greene   38:33
Cool.
Thank you, Sean.
That we we always appreciate the docs updates because that gives us a source of information as we're going through and using PowerShell and automation in our daily lives.
So definitely appreciate and have a lot of gratitude for the work that your team does.
The next item on the agenda is at Ignite recap and I'm not sure who from my team planned on speaking up here.
Steven, were you interested or is Sydney picking this one up?

Steven Bucher   39:04
Yeah, I can.

Michael Greene   39:04
Like Steven.

Steven Bucher   39:04
I can.
I can chat to it, just wanted to kind of give a a brief recap on Ignite.
Ignite was two weeks ago.
There was a lot of pretty awesome Microsoft announcements there.
They PowerShell team had a great presence there.
We were boothing there.
We talked a lot with customers.
I wonder if there's even some customers that we saw at Ignite in the call today.
I know, I know.
We chatted with a few who are like, Oh yeah, but I'm always.
I always come to the community call and stuff, so it's really great to to meet folks in person.
We connected a lot with other Microsoft folks as well.
We had two sessions there.
The first one was Sydney did, which was about all about secret management, which was a big hit.
Unfortunately, that one is not recorded.
That was just a live demo, but we do have a recording of our panel discussion which I will share in the chat.
I just can't do two things at once.
Ohm.
There we go.
So we had a panel discussion with our whole team and a few other Members at Microsoft to have a discussion around kind of what's new with PowerShell, how we handle the open source community as well as all the new kind of releases and different sort of areas that we cover.
So definitely check out the recording in case you missed it.
Oh, there.
Yeah, there's Chris.
I was like, I was wondering where Chris was we, Chris was there at Ignite and yeah, Sydney session was super popular.
I don't know if anyone else in the PM team wants to add in anything there.

Michael Bass left the meeting

Steven Bucher   40:40
Pause for a second.

James Brundage   40:42
I'm just sorry I missed you guys.

Sydney Smith (SHE/HER)   40:45
I'll just say it was really fun to be there and get to see all of you.
Who are we were able to and we're really looking forward to seeing more of you at conferences in the new year.
We're kind of in the midst of session planning for a number of conferences in the spring.
So if there's any topics that you would love to see us talk about, that's always helpful for us to know or you're gonna be at conferences in the coming year that we can kind of join you at.
That'd be great for us to know as well, but it was so much fun to get to connect with you at Ignite.

Michael Greene   41:20
Yes, 100%.
Please let us know if there are conferences that are important to you and you're not seeing the PowerShell team there, because we we should be thinking about that as we're going into our next year's planning.

James Brundage   41:34
Love to see everybody at PowerShell Summit.

Steve Lee (POWERSHELL HE/HIM)   41:38
Well, I mean the the obvious ones will will have some representation.

Michael Greene   41:41
Right.

James Brundage   41:46
I'll just, you know, making sure right.

Steve Lee (POWERSHELL HE/HIM)   41:52
I think right now I just say that there's at least a 50% chance, albeit PS coffee U in 2024.
I have to get approval.

Michael Greene   42:02
Same.
Awesome.
Why don't we move on to the working group updates?
This was a big thing that we wanted to drive for the November call because it highlights at the end of the year.
UM, you know, kind of what each working group has been doing over the course of the year.
Obviously, we're not expecting, you know, a full 12 month report, but just some sentiment about highlights for the working group and what's important to them.
I wanted to 1st highlight so that we are thinking about the groups in the agenda.
We've got the interactive security engine, commandlet module and language working groups.
We discussed this last night and there are two other groups.
One is the quality test group and the other is Dev X or developer experience.
Neither of those have really had much activity during 2023, so it's likely like we can leave them sort of open, but we're not gonna ask for report outs and whether or not those remain active working groups can kind of be up to those groups to decide.
And if they, if they've served their purpose and it's now appropriate for them to just sort of close out, then that's fine.
That's an OK thing to have happen, and there may be new working groups in the future.
So why don't we start off with the interactive working group?
Mr JSON Helmick, I see that you are on the call.
Would you like to share?

Jason Helmick   43:28
Well, sure.
Hey, I'm Jason and welcome to the interactive working group.
So PowerShell is this amazing product with two sides, an interactive site where you can explore, discover, learn and troubleshoot, and then an automation scripting side where you can codify your intentions in the interactive group.

Christopher Moore left the meeting

Jason Helmick   43:47
We focus on the user experiencing issues that are exclusive to interactive, so things like formatting issues, display issues, problems in the console terminal, the help system, tab completion, all that kind of stuff is where we focus on the Members well, myself and Jim.

Erde, Sam left the meeting

Brown, Derek left the meeting

Jason Helmick   44:05
Truer dongbo.
Patrick Meinecke Aditya Sean Wheeler.

Damien Caro joined the meeting

Jason Helmick   44:10
Sean's there not just for documentation cause, but also because he's a walking encyclopedia of PowerShell.
Occasionally, Steven Boucher stops buying in.
Our Community member is Ryan Yates and I really want to give a big shout out to Ryan for his hard work and his level setting in the meetings.
I'm just as a look inside of how we conduct our meetings and what we like to do.
Most of us are working on issues all throughout the week, so any Member can bring an issue that they've been working on.
Or maybe that's caught their eye.
But what I like to do is stack the deck at the beginning of the week.
So on Monday I posted a list of five issues.
Now what we've done is those have been two issues that I select from our oldest list and issue from the newest and issue from the most commented and one that is catches my attention.
We discuss debate and find the next step for those issues because of the approach that we've taken, we've been able to burn down a couple of years and matter of fact, we started with the 2016 and now we're into the twenty 20s.
We've also been able to burn down some of the top issues going forward.
We're going to focus more on new and most common IT issues and things that we can triage quickly.
What's the most challenging issue we talked about this in the working group of maybe me bringing a couple of issues to show you what some of the most challenging or but I sat back last night and I was reviewing the list over the past year of the challenging issues that were in front of us.

Vikram Medari left the meeting

Jason Helmick   45:38
And I sat back and it kind of hit me.
We discussed and debated each and every single one of these.

Damien Caro   45:43
Yep.

Damien Caro joined the meeting

Jason Helmick   45:45
Sure, some of them got resolved quicker because of consensus, but every single one of these was just as important as the last.
They were all hard.
They are all hard issues and we really respect the issues and the conversations that everyone has had in them, so in it's difficult, but we keep focused and we keep moving forward on those issues.
But in closing, I just want to thank all of the working group members for all of their dedication and hard work.
But really, I'll be honest, the things goes out to all of you for discovering, filing and discussing those issues that you run across to help improve the product.
So from the working group of interactive.
Thank you all very much for your help and support.

Michael Greene   46:30
Thank you very much, Jason.
And I see that Sydney Smith has started her camera that takes us to the next working group focused on security.

Sydney Smith (SHE/HER)   46:42
Cool.
Yeah.

Damien Caro   46:42
Yeah.

Sydney Smith (SHE/HER)   46:43
So I am in the security working group alongside Travis and Anna and we meet on Mondays during PowerShell Community Day and sort of what we do in our working group is we get tagged in any sort of issue that may have security implications.
And so as a team, we review whether or not this issue may have security implications.
What those might be and sort of analyze any sort of changes to PowerShell no from a security perspective and provide our guidance there.
And then alongside that, we sort of help drive any sort of patch releases that might happen through PowerShell.
So that's kind of what our working group is up to.
I'll keep it pretty short there and pass it on.

Michael Greene   47:32
Ooh, thank you, Sidney.
The next one we've got the engine working group and I believe Matteus is going to help us out with information there.
But you still on mute.

Mathias R. Jessen (IISResetMe)   47:46
Then you, Michael.
That's right.
I am one of two community members of the engine work group.
We probably have the most convoluted scope of all the working groups, because what we're interested in is all the complexities inside system dot management dot automation.
So sort of the core API PowerShell, there's a lot of Dragons in there.
Partially product has been around for a long time and as of working group, I think we probably try to and we maybe also come across as a little conservative somewhere sometimes because we try to sort of keep PowerShell, PowerShell basically.
So I saw a bunch of people in the chat already complaining that we tend to reject a lot of the suggestions or feature requests that come in, and often it's because we sort of have a philosophical difference of understanding in what the request for asks us.
But usually what we do is we'll get, we'll get two types of issues and either box or sort of breaking changes or suggested features that would count as breaking changes and then we sort of tried to work through them and try to figure out sort of what are the implications of these things and where both in terms of performance, fragmentation of the API itself, are we breaking anything or are we sort of going too far away from what PowerShell is at its core?
Umm.
In every meeting, we try to sort of cut a balance between triaging uh pull requests as we've been doing for the last year to try to sort of a keep that going right.

Ryan W. joined the meeting

Mathias R. Jessen (IISResetMe)   49:21
Some community members have had this experience that you open a pull request with a complicated change or a change.
Sort of deep, deep down into the API and then nothing happens because no one is reviewing it or nobody's asking the questions that were sort of drive you forward in closing the PR.
And so we tried to pick up some of these complicated low level engine peers and try to sort of move them forward by giving feedback or giving recommendations to refactoring changes.
Other than that, we also tried of course new feature requests and box.
The way we work, I should probably mention we have six active members in the Working Group 2 Community members, myself and Keith Hill, and then for people from Microsoft Rain, Patrick, Jim and Dongo from the PowerShell team.
And so we meet every two weeks with a rotating meeting chair.
So for the last two meetings I've had the.
What are the privilege of picking up which issues we had to review?
Uh to one of the points earlier Steve closing.
Closing all of these issues from the past has actually helped us in prioritizing our work a lot, because previously we'd have to look through maybe 1200 issues that we're all sort of relevant for the work we do, some of them five years old, some of them five weeks old.

Nikita Bajaj left the meeting

Mathias R. Jessen (IISResetMe)   50:38
And so in order to try to prioritize that based on feedback and so on.
So having this cleanup has been has been massively helpful in picking up some of the slack in the internal working group.
Yeah, I think that's.

Michael Greene   50:56
Awesome.
Thank you very much.
I will cover the next one which if I find the window would be commandlets and modules.
So this is actually a really awesome conversation we meet.
Uh, I believe weekly.
It seems like weekly it might be every other week and just go through issues and PR's that have been submitted on anything related to a commandlet or any of the modules that are built in or that ship with PowerShell, or regularly use with PowerShell.

Ryan W. left the meeting

Michael Greene   51:27
A lot of really deep technical conversation around changes to modules, things that were broken unintentionally through change and can be revisited and that kind of thing.
But you can imagine debating exactly how the code under the hood works for anything as as common as like copy item that you are using everyday or get child item and that kind of where you start process and things like that and really interesting to see how some of these tools changed as they went from PowerShell five one through PowerShell 6 and PowerShell 7 and where they're going in the future.

Victor Ordu joined the meeting

Michael Greene   52:04
I wanted to say thank you for everyone who joins that call from outside of Microsoft, especially Mr Doctor DNS.
Uh, uh, Thomas Lee, thank you for joining our calls.
Certainly Jeff Hicks for joining and providing feedback and then Doctor Tobias Weltner, thank you very much for joining the calls and providing feedback as well.
Steve Lee any or Jim Schur.
Thank you for your help on the call as well, but anybody that uh or anything else that you wanted to mention in regards to the COMMANDLET or module working group?

Steve Lee (POWERSHELL HE/HIM)   52:39
I guess I think I mentioned is like a lot of issues fall under this area as well.
I mean engine gets a lot as well.
Umm.
And and you know, one of the things that we always think about it like, does it really need to be something in PowerShell 7 itself or does it make sense to do it on a gallery and get feedback that way?
And then we can decide whether or not there's sufficient usage that we need to revisit the discussion of being it being part of the core package, right?
So I think for anyone opening a new issue, especially if our new commandlet thing about that first, it's like really can it be on the gallery, we really wanna promote that as a way to get things and also decouple updates of those modules versus PowerShell itself.
So again, I also appreciate you know, the community members of that working group as well as the members of the working group.

Grote, Justin joined the meeting

Michael Greene   53:27
Outstanding.
The last one is the working group focused on language and we've got Jim Chu.
I wasn't sure if if Jim wanted to talk about language or if, Steve, if you wanted to take that one as well, it's up to you up to Jan.

Steve Lee (POWERSHELL HE/HIM)   53:41
Have Jim talk about because I'm actually not in that group.

Michael Greene   53:44
Ohh got it.

Jim Truher   53:45
So the language group is the smallest working group.
Really trying to make sure that the PowerShell language is in the syntactical elements are are looked after.
Very carefully, and in fact we have decided as a as a working group to follow or to try to follow the C sharp model for language enhancements and changes, not not bugs but just enhancements.
So there's a PR out currently to change our governance to discuss what that process should be and we've got some commentary in it.
There is A and and we have been meeting.
Somewhat regularly but but we have determined that that that this working group should probably behave a little bit different than differently than some of the other working groups, which is why we've come up with this PR to change our governance for our process.
Uh, yeah, there is a link.
Let me see.
I'll I'll track it down and put it in.
Put it in the in the chat.
It's very I cribbed it almost well.
I didn't crisp it entirely from the C sharp language, but I did.
I did a use that as our model since we're so tightly related to C and in fact in some cases it's, you know, some of CC sharp syntax is our A it's elements of C sharp syntax are found in PowerShell, maybe twisted just a tiny bit to match the interactive the the shell aspects of it.

Harms, Greg J joined the meeting

Jim Truher   55:32
But yeah, I'll put the I'll find the link and post that in this chat.

Nieto,Thomas left the meeting

Michael Greene   55:37
Awesome.

Jim Truher   55:38
Yeah, there it is.
Patrick did it.
Thank you so much, Patrick.

Michael Greene   55:44
Very good.
And then our last area is the committee in general.
And Steve, if you're interested, you can take that one on and I'll, I'll, uh, supplement or anybody from the committee can supplement as needed for more on request as I say.

Steve Lee (POWERSHELL HE/HIM)   55:51
Yeah. Ohh.
OK.
I'll I'll cover this briefly and I just checked and it looks like the membership list on the repo is out of completely out of date.
I think the regular participant members participating members, as Jim Michael, myself, Damien Caro from Azure PowerShell, Azure CLI side, we have a few folks who are not formally out of the committee, although they haven't since leaving.
Microsoft hasn't been as active, so I think we need to probably update that and we may have to revisit the discussion we started a long time back of adding someone from the community to the committee, but in general, I mean the committee these days doesn't have as much work as we used to.

Michael Greene   56:29
Yeah.

Steve Lee (POWERSHELL HE/HIM)   56:40
Now that we have the working groups taking care of a lot of the load and only when something isn't resolved at that point, or if someone in community doesn't agree with that decision, it gets appealed up to the committee.

Friedrich Weinmann (CSA, POWERSHELL) left the meeting

Steve Lee (POWERSHELL HE/HIM)   56:52
So some of the stuff that we talked about more now is stuff like the bot, right?
So how do we tackle some higher level problems?
So if you guys hated them, the auto closure, you can blame the committee for that discussion and decision.
But these are the things that we talk about is like, alright, what are some long term strategies?
Where do we wanna go with this product with the not just the engineering side, but also the community and stuff like that.
So these are type of discussions we have.
My granting went ahead.

Michael Greene   57:21
No, I don't think so.
Yeah, the the committee is, in my opinion, serving the purpose of uh, morally strategic decision making.
And unless there's an escalation, fewer of the like which which issues to take on which PR to take on, things like that, it's more high level than that.

Amber Erickson left the meeting

Michael Greene   57:43
But yeah, it would be great to have some additional representation from outside of Microsoft.
Uh, at some point in the future so.

Amber Erickson joined the meeting

Steve Lee (POWERSHELL HE/HIM)   57:51
Yeah.
I guess one thing I'll just add because this came up a few months ago is like any reports of abuse gets reviewed by the committee and we try to take that on very timely adhering to the Code of Conduct so.

Michael Greene   57:57
Umm yeah.

Steve Lee (POWERSHELL HE/HIM)   58:04
And and I'll just say in general, I think people are pretty good on the repo.

Michael Greene   58:08
Yeah, yeah.
All right.
That keeps us right on schedule and the last item on the agenda is Community demos.
I think a couple of people had mentioned they might be interested, so I will leave the floor open and if anybody is interested in a community demo then we'll help you share your screen and you can take the floor.

Mutemwa Masheke left the meeting

Josh Corrick left the meeting

Silva, Rodrigo N left the meeting

Steve Lee (POWERSHELL HE/HIM)   58:31
Justin.

Michael Greene   58:33
Yeah.

Steve Lee (POWERSHELL HE/HIM)   58:35
Are you going to be quick, James?

Grote, Justin   58:36
No, you can go first, James.
That's fine.

James Brundage   58:39
All right.
Well, let's be quick then let me.
Let's see, here was my screen showing up in camera, yet no, not yet.

Steve Lee (POWERSHELL HE/HIM)   58:51
Now I need to I need to find you.
There you are, OK.

James Brundage   58:54
OK.

Steve Lee (POWERSHELL HE/HIM)   58:56
I'll get you should be able to present now.

Mathias R. Jessen (IISResetMe) left the meeting

James Brundage   59:00
Alright, so I'm gonna start sharing.
And let me know if everybody can see it.

Steve Lee (POWERSHELL HE/HIM)   59:09
It's working.

James Brundage   59:10
Cool.
Umm, so this is demonstrating upcoming build of pipe script.
I will be also talking about this the specific PowerShell user group in a couple of weeks.
A very exciting feature.
There was a language overhaul on terms of how PowerShell or pipe script works with other languages, and one of the best side effects of this is an inline interpretation.

Sydney Smith (SHE/HER) left the meeting

James Brundage   59:33
So normally if you ran this in PowerShell, you would actually get this file open for you.
But no, it's actually running.
It's running this file.
And I can get its results back as variable and I can do the same thing for JavaScript too.
How is this working?
Well, there's a pre command look up.
This becomes an an alias to invoke interpreter invoke interpreter has a mapping of which interpreters exist for which languages.
In this case, it's literally just calling Python or Java scriptsorno.exe for each of the different file types, but it's nice, open ended and extensible, and now we can actually seamlessly work with any other language.
Inside of PowerShell, so I'm really excited about this one.
There's a lot of other really fun stuff coming in the next bill, but this one in particular is how do you one say it really awesome sauce?
So that's my quick demo.
Uh, hope it was quick enough.
You want to see more come by in a couple weeks.
Fun not fun.
Interesting.
Uh, Michael, I believe you're muted.

Steve Lee (POWERSHELL HE/HIM)   1:01:13
You're getting.
You're getting comments in the chat.
James, Justin, Are you ready?
Since we're getting close to out of time here.

Grote, Justin   1:01:21
Yeah, I can.
Uh, do a couple here real quick.
So let me just share here.
So I just have two kind of quick things I want to show.
One is a little thing I made called power serve, so this is made for some very specific situations where you have like a monitoring program or something that needs to call PowerShell all the time like from the command line.
Like it doesn't have any way to integrate PowerShell or do any of that kind of thing.
Monitoring programs are notorious for this, like they have like a custom script that they need to run like once a minute or whatever, and so they're calling up PowerShell to run it and then close it down.

Amber Erickson left the meeting

Grote, Justin   1:01:52
So teams done a good job of really making PowerShell work pretty quick on the startup and shutdown.
There's still a lot of overhead that happens there.
So for instance, in this example, if I bring up my task manager here real quick, if I go ahead and try to do this like the run this one script 100 times just to say hello and run it in parallel 30 times, you know my my computer starts going nuts with the terminal 100%.
You know, loading all of this memory for all these different PowerShell instances to run it.

Vin Latus (ISD) left the meeting

Grote, Justin   1:02:20
So I'm just gonna cancel that, which may take a second, and that may have impacted the video.
Sorry if it did so.
What power serve does is power serve is a an AOT ahead compiled client that uses the that utilizes the alt function in the newer net versions to make a really fast, powerful kind of like how like in like go and set.
You can compile these really nice, small, fast executables.
You can do that in net now with net eight.
And So what it does is that it ends up spinning up a PowerShell instance in the background and then uses a very simple sort of client server protocol to it, where it basically just sends commands to the server.
The server executes the script in a run space and then returns it back via Jsom.
That all just happens in a PowerShell process because it's not standing up whole PowerShell processes every time, and it's using run spaces within a single process.
It's actually able to run much faster, so that same command which took up all that memory and got stuck to run the same thing there.
It took only 600 milliseconds to get those results and you saw there was almost no CPU usage because it's able to go in.
Use those run spaces and resolve.
So we've done this with some tests for some of our internal monitoring where we use a lot of custom scripts and it it's reduced like our the workload on our servers just from the PowerShell overhead of starting up PowerShell and stopping it for these hundreds of scripts that have to go out and check information.
All these systems is reduce those workloads by like 80% on the memory usage and like 50% of the CPU usage and so it's a really handy little thing.
There's on my GitHub is for power client for that and then one other thing that I have just here.
On the other side is feedback providers or a new feature in PowerShell 7.4 and I made two different feedback providers.
Recently I made one for.
Whenever you run certain graph commands, it can detect whether you're, whether it needs, like the advanced syntax in terms of like the eventual consistency and such, and give you feedback on that.
And I also made a script feedback provider that lets you write feedback providers just as PowerShell script.

Nieto,Thomas joined the meeting

Grote, Justin   1:04:22
So basically, once you install this module, you can just run a register and provide the parameter and then there's some simplified syntax or in this one just simply says if the last error, if it had a recommended action because there's this property, a lot of people don't know about in error details about a recommended action.

Austin Hollett - Network left the meeting

Grote, Justin   1:04:37
Then go ahead and report that and recommended action is needed.
And so I'm sorry, this is probably really small.
Umm, so when you go to write an error if if an error happens.
If you have this feedback provider installed, I don't know.
I got a whole mess going here.
If you have that feedback provider installed, you can.
If you have an error that has a recommended action now, the feedback provider will report it.
So this was just an easier way.
Mostly.

Silva, Rodrigo N joined the meeting

Grote, Justin   1:05:03
Usually the default setup is to have to write feedback providers.
In C, This is a way that you can actually write them just in pure PowerShell by using the script provider module, and you can even see them registered.
Uh, with the gift script feedback provider module, and then unregister them as needed rather than having to do it ITOPS rather than having to need to do it with the the C sharp and the module load unload.
So just do little things that I've been working on there.
Fun little things.
And so just quick demo. Thanks.

James Brundage   1:05:31
Fantastic.

Michael Greene   1:05:33
That is really great.

James Brundage   1:05:33
They both look really useful too.
After she.

Michael Greene   1:05:39
That is really great.

Grote, Justin   1:05:41
I'm sorry if it wasn't super clear.
Here's that feedback action because the this get mg user supplies that recommended action in part of it.
So here's an example of it working with like a real commandlet.

Luke Murray left the meeting

Grote, Justin   1:05:50
So this is an experimental feature in 7.4.
You have to enable it, but all you have to do is install the script feedback provider module, which I haven't published in the gallery yet, but I will soon and then you can write your own feedback providers.
I basically wanted this to work a lot like how like argument completers and stuff already work where you can write them in PowerShell and not have to delve into the C so turned out not to be too difficult to do.
I just basically register a feedback provider for you that ends up calling those script blocks for you in an isolated run space.
Sorry, I'm running over again.

James Brundage   1:06:18
Yeah.
Oh, it's OK.
I it does give me one quick bat on bonus point.
I'm also gonna be adding get the sub module and a sparse checkout features to TypeScript V necks, so you should be able to actually take these things that Justin's demonstrating and just include them from his repos into projects and build your own.
But fantastic work can't wait to leverage it.

Domansky, Mark left the meeting

Michael Greene   1:06:54
Perfect.
I think we're going to finish right on time according to our schedule.

Ryan Yates   1:06:59
Just just.

Michael Greene   1:07:00
Anybody.
Yeah, Brian, go ahead.

Damien Caro left the meeting

Ryan Yates   1:07:02
One last one, I put in a discussion for uh December call because obviously we've we've said this previously that it's nice to have another call at the end of the year that isn't focused by the team and is led by the community.

Michael Greene   1:07:03
Awesome.

Ryan Yates   1:07:16
I've got a discussion that I've put in December 14th.
There's a as a date.
And my thinking about that is because it's not too early in the month.
It's not too late in the month and it doesn't give a need for people in the team to to join if they don't.
Yes, I want to start on holiday.
Ohh, visiting family and and getting ready for for the end of the year shutdown I'm going to put in a link for that called probably around this sort of time as well if anybody wants to join and all that that's a discussion which I'll just add the link into and the chat now.

Nieto,Thomas left the meeting

Ryan Yates   1:07:55
Mind about works.

Michael Greene   1:08:03
Awesome.

Steve Lee (POWERSHELL HE/HIM)   1:08:04
Man.

Michael Greene   1:08:07
Yeah.
For the December call, just to ECHO, arranging with saying, we would love to have the community hosted December call and that can take shape and form.
However, the community decides, but it would give us a chance to kind of step back and let the Community drive for a month.
UM, and you know, potentially bring up topics and agenda items we never would have considered that are of high value, so.
Everyone is fascinated, myself included with the demos.
OK, I think that's it for today.
Is there anything to add from anyone think I know?
It's been a really awesome 2023.

John Kavanagh left the meeting

Silva, Rodrigo N left the meeting

Michael Greene   1:08:47
A lot has happened this year and so thank you everyone who has contributed and participated.
We really have a lot of gratitude for the Community.
Anything anybody else would like to add before we wrap up.

James Brundage   1:09:01
Uh, I go ahead.

Januszko, Missy   1:09:01
Hi, it's Missy.

Michael Greene   1:09:03
You see.

Januszko, Missy   1:09:04
I just want to say thank you to the PowerShell community for all of your extremely high quality submissions for the PowerShell Summit.
Our evaluators are working very hard to read all of the submissions and determine which sessions will get selected, and we have a very like tough set of choices so.
December 15th is our target to have speakers, not speakers, emails out.
So that's all I have.

Michael Greene   1:09:42
That's awesome.

James Brundage   1:09:42
Angers crossed.

Michael Greene   1:09:43
I can't wait.

James Brundage   1:09:45
I was going to say uh little light plug Pacific PowerShell user group.
We will have another meeting this December 8th.
I'll be posting the link soon and you can come by and learn more about, you know, dynamic interpreting about their languages and PowerShell.
Thanks.

Michael Greene   1:10:07
Thank you.
And if anybody else has community events coming up, I would say let's drop them into the chat channel if you'd like to get some more attention to them or invite people who might be on the call or listening to the recording, watching the recording, who who are looking for a local events, they can attend, that is definitely something we want to do more of than 2024, I think.

Sanjeev Kumar left the meeting

Alan Florance joined the meeting

Christopher Nguyen left the meeting

Alan Florance left the meeting

Michael Greene   1:10:32
The past three years were tough for local events, so that's something we are passionate about bringing back.

Ryan Yates   1:10:38
Yeah, definitely.
Uh, the UK groups at some point will return in 2024 and unfortunately, things for me are a little bit up in the air at the moment.
So I'm planning to try and bring them back, but this this is a.
It's a work in progress, shall I say.
And if so, if you follow, follow me on Twitter or or the OR the get PSU G UK Twitter account, that'll be where I I post things or on my blog and and yeah it's a it's A to be determined should we say.

Michael Greene   1:10:58
I have understand.
Fair, very fair, no problem.
And thank you for your work on that ran.
Great.
We can wrap it up.
Thank you everybody for joining.
And I look forward to attending the community call in more of a read only fashion in December and talking to you all again in January.

Harms, Greg J left the meeting

Michael Greene   1:11:39
Happy holidays.

James Brundage   1:11:41
Elements banks.

Ayan Mullick left the meeting

Januszko, Missy   1:11:42
Thanks.

Nithin left the meeting

Greg Malleus left the meeting

Rhett Hartman left the meeting

Brian Kaiser left the meeting

Bartlett, James left the meeting

Keith Hill left the meeting

Tess Gauthier left the meeting

Steve Lee (POWERSHELL HE/HIM) left the meeting

Anam Navied left the meeting

Sean Wheeler left the meeting

Januszko, Missy left the meeting

Jim Truher left the meeting

Lionetti, Chris left the meeting

Andy Jordan (THEY/THEM) left the meeting

Antenor, Daniel left the meeting

Ryan W. left the meeting

Ili Metuky left the meeting

Wojtysiak, Paul left the meeting

Grote, Justin left the meeting

Patrick Meinecke left the meeting

Terry Dow left the meeting

Frode left the meeting

Victor Ordu left the meeting

Hager Constantin left the meeting

Achim Wieser left the meeting

Steffi Ponce left the meeting

James Brundage left the meeting

Jason Helmick stopped transcription
