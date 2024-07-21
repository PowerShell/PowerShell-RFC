# PowerShell_OpenSSH Community Call-20240718

Transcript
July 18, 2024,
 

Jason Helmick   3:34
Good morning, good afternoon and good evening.
Welcome to the July 2024 PowerShell Community Call where everyone is welcome to join and discuss the PowerShell project and its development.
Hey, we've got our agenda posted in the meeting chat and a link to the meeting discussion.
Please feel free to post your questions in the discussion issue or hey right here in the meeting chat.
We'll we'll get them either way and please follow the code of conduct now before we get started this morning.
I've got 3 announcements, 3 summer anouncements.
First of all, happy summer.
I'm starting today and for the next three months I'd say through September, we're going to go to summer hours on the community comma, which means the call will be about 30 minutes in length.
You know, folks are in and out on vacation.
You're in and out on vacation, enjoying all this sunshine.
So we I think we did this similarly last year as well, but hey no panicking here, we still have plenty of things to talk about and we're trying to make sure that at the end of the call, we've got enough.
We've got about 5-10 minutes or so so that we can still do community demos, making of which on the community demos.
That's really my second kind of announcement.
We are so very appreciative.
Do we everyone that has done demos that you've been bringing to the community called we really, really enjoy that a lot.
So we'd like to encourage everybody to continue to do that and bring fun new things for everybody to check out, and it can be anything.
It can be anything from your day job that you're working on, or that you did, you know, a neat solution that you came up with.
It doesn't have to be something stellar and spectacular.
Well, it is stellar and spectacular.
It's a solution you came up with, so that's why we encourage everybody to go ahead and bring a demo to the call.
The one thing that we do ask though, to help us plan for the call is please add your name and the demo that you'd like to do to the discussions before the day of the call.
We'll go through and give you a thumbs up if we've got you scheduled for the call and you'll see yourself up here in the agenda.
So thank you so much for that.
My third and last announcement and then we'll get to the actual agenda is I don't know if you folks are planning on attending techmentor next month.
It's in a couple of weeks here in Redmond on the Microsoft campus, but if you are looking at attending TECHMENTOR, which is a great IT focus Conference, well, we'll be there too.
So we look forward to seeing you in a couple of weeks and joining us for what we expect to have is a great time for several sessions from the PowerShell team plus so many other great sessions at Tech Mentor.
So that's we look forward to seeing you if you're going to be there.
And with that, I think we're all set to start our actual agenda and I see Sydney already here.
So Sydney, would you like to chat about the community PR update?
 
   6:48
I would love to.
 
Sydney Smith   6:49
However, my good friend Steven is actually the one who went ahead and pulled the numbers on how we've been doing for PR's.
 
Jason Helmick   6:54
Ah.
 
Sydney Smith   6:56
So I'm gonna pass it over to him and he's gonna give us a little update on how we've been doing in regards to merging community PR, closing them, that's sort of thing umm yeah.
 
Steven Bucher   7:06
Thanks Danny.
So we I wasn't at the last couple Community calls by her.
 
Speaker 3   7:11
There was some interest in kind of understanding how the communities are going and if you're aren't familiar, we on the partial Team love to spend every Monday focused on community work as this means reviewing issues, reviewing PR.
 
Steven Bucher   7:26
This is kind of, you know, our time in the week to really hone in and focus in that area.
And so we prioritize PR as a lot since people have put in their own time to to make these PR's, we wanna show appreciation for that.
And so we have been making good progress on kind of, you know, PR.
I don't wanna say burned down, but PR reviews we it's it's really tricky to you know say burn down because some PR's can take much, much longer to review because they because of the different implications because of the different potentially breaking changes just because of the kind of impacts and.
I guess you know impact that it can and adverse you know review can make?
So we we really like to take our time and make sure that the PR's that the working groups review and our team reviews are up to a really high, high standard.
So it's kind of hard to say, you know one number another, let me pull it up.
But since the beginning of 2024, there has been approximately 113 Community PPRS created since January of this year, and we've closed 72 of them.
Uh, so, you know, do the math percentage.
But yeah, I'd say that's a pretty, pretty, pretty good number.
 
Darren Kattan   8:45
Quick closed or merged.
 
Steven Bucher   8:49
Uh, I we don't have that.
I I'm pretty sure they're merge, most of them are merged so.
 
Darren Kattan   8:56
Not mine.
 
Steven Bucher   8:59
OK.
Well, we can.
We can take a look at that offline.
Feel free to message which one you're looking at.
I mean it.
Like I said, we we do like to take you know.
Umm, a good look at at the PR.
So Preta show that.
OK, cool.
Anyways, so this is just for the the PowerShell repo but.
Yeah, wanted to wanted to kind of share that, but anywho, I think that's how we got on that agenda.
 
Jason Helmick   9:35
Also.
Well, awesome. Well.
 
Sydney Smith   9:40
Yeah.
And I think I just want to emphasize one thing that Steven was was kind of saying alongside all of this and that is that PowerShell are are really or emphasis with these community or with with reviews and everything is really the stability of PowerShell.
We've shown telemetry we're numbers before and with PowerShell seven we see a large number.
I think we've had over 2 billion, you know, monthly sessions with just the latest version of PowerShell.
And so we really want quality when it comes to these PR reviews.
I'm not saying, you know, we don't want to get through as many as we can, but taking our time with these reviews is so important.
So we do appreciate all the Community contributions and feedback and all that sort of thing.
But yeah, it can take time to get through PR reviews as well.
That's all.
Thanks Jason.
 
Jason Helmick   10:33
Well, that's awesome.
Well, to kind of switch topics a little bit and I'm gonna go right back to you.
Steven is so several of the PM and engineers were at psconf.eu and I didn't get a chance to attend this year, but I heard that it was like spectacular.
And so I'm dying to hear more about psconfeu.
So, Steven, you wanna chat with us about the on the goings on around that?
 
Steven Bucher   11:01
Yeah, absolutely.
So we were very fortunate to to attend PS Conf EU this year.
Uh, it's a it was held, I guess beginning or sorry, middle of June we had myself, Damien Caro, Steve Lee, Amber Erickson, Aurnov Mutemwa.
Alex, we had a lot of folks from the, from the from go through all the names and I but we we had four people from the core PowerShell team as well as some folks from other partner teams at Microsoft including Demitrius from Winget and Aurnov and multimode from ARC and machine configuration respectively.
And Alex Wang from the Azure PowerShell team was able to join us as well, which is really exciting.
And so we had a great time chatting with folks, meeting new folks, seeing familiar faces in the PowerShell community, all the presentations and stuff are actually already uploaded to YouTube.
They were extremely fast on the turn around time for the sessions recordings and then publishing them on YouTube.
So I highly encourage people to check out the psconfeu you YouTube channel to check out all the sessions that we presented on as well as all the amazing sessions the community is presented on.
This some really really awesome ones.
No out there.
And so we should just want to call that out and try to think if anyone else was at PSC come to you and wants to add anything, feel free to interrupt me here, but.
Ohh can post.
 
   12:39
On the stuff.
 
Steven Bucher   12:42
The other call out is that the psconf.eu team is doing a mini Conf, a virtual conference.
They do this, they've done this the last couple of years, which is kind of a virtual, you know, gather online formatted 1.
This is in October of of this year's OHH.
Thank you, Alexander.
Yeah, you were you for posting the the link to the recordings, but yeah, there's a lot of amazing content out there.
So I highly check out.
Highly recommend checking out some of the stuff we shared some stuff around some of the AI and shell investments that we've done.
Some of the updates to the Azure client tools, some updates to the machine configuration Steve presented at a good session on DSCP 3 and Amber as well on PS resource get.
So definitely check those out.
If any of those subjects are of interest to you.
 
   13:39
And.
 
Steven Bucher   13:40
And the the one other thing about Psconfeu is every year we hold an AMA or an ask ask me anything kind of session with with the PowerShell team. And we took questions online and kind of in person there.
And so there were so many questions that we couldn't get to all of them at the conference.
And so we wanted to take a little time today to cover some of the the bigger ones, bigger questions we got from that conference.
So hopefully that if you were at the at the conference and didn't get your question answered, we can answer it here today as well as these are just general some FAQs that we got from the conference.
So give me a minute to pull up the sheet.
But 111 great question.
I think this is actually quite relevant given the MVP renewals.
Was someone asked this?
Kind of.
What's a good path forward to become a PowerShell MVP?
And this is something we don't talk about too often in the community calls.
So I figured it's a good time to highlight that a little bit.
Sydney actually is the one working primarily on the MVP program, so maybe you can share some thoughts with us.
 
Sydney Smith   14:45
Yeah.
So if you are familiar with the MVP award, it's Microsoft's value professional program.
Uh, and it's an award that Microsoft gives out to folks in the community who have high technical and displayed high technical ability and also sort of like community outreach and share with other folks in the community.
So there's a number of different types of contributions that you can have that really can come together to put together and MVP profile that you'll submit.
You'll get nominated and then submit and then it will be reviewed by a number of people to kind of get the MVP award.
Given that PowerShell is a community and it's open source, a number of different ways you can sort of start to build that community around you.
That makes you really great for an MVP submission.
So I would say like if you're looking to sort of build towards MVP, it's really great to get involved in the PowerShell community.
So some ways you could start doing that would be I'm getting involved in open source, whether that's the teams open source.
I would say that would probably be the best thing to do because it builds familiarity with folks at Microsoft, so that could be not only opening issues, but also getting in our repos and responding to other folks issues.
Getting involved in conversations and discussions on GitHub, of course.
Always following our code of conduct that's gonna be really important in, you know, representing yourself well.
But yeah, getting involved in open source, whether it's PR's, but also reviewing other people's PR's and responding to other people's issues and that sort of thing, building out content around PowerShell for other folks.
So that could be presenting at conferences, running conferences, presenting at our community call, creating content like video content on YouTube or maybe blogging whatever content looks like to you, whatever medium you really enjoy just building out content, teaching people about PowerShell, teaching people about what you're passionate about, whatever technology wise that is relating to what we're working on and what we're releasing is really what's gonna build you that reputation.
As a MVP, yeah.
So speaking app conferences, especially PowerShell Summit and psconf.eu, and if you're yeah, if you're looking to like build towards that you want more experience speaking great ways to get started with that would be yeah user groups again demoing our community call and just practicing speaking at other conferences that sort of thing.
There's a lot of different ways to get started, and if you are like, hey, I really want to get more involved in the community, but I don't know how I would recommend starting with the community discord or reaching out to anyone in the community or really anyone on the PowerShell team, and we can probably find a place for you to start working.
But yeah, user groups are great.
The other thing I would mention too is, but if you're kind of doing these things, if you're getting active on GitHub, you're finding a good space with responding to issues, feeling building comfort there, getting involved and working groups would be another great step towards the MVP award.
So those are just some ideas to kind of get involved.
But I think the biggest thing is just showing that you have competence in PowerShell, that you're comfortable speaking about PowerShell, whether that's to the community, to the team and that you're comfortable giving feedback.
Umm to folks in a really respectful way would be all things that we look for in the VP award.
So Yep, awesome.
 
Jason Helmick   18:34
The only thing I would add to that is I just want to take a moment and I there are several MVP's and former MVP that are online on this call and I just want to thank everyone for your, for your amazing contributions over the years.
And this extends beyond the MVP community.
This extends to all of you, your contributions and the fact that you're helping folks is what matters the most.
So thank you very much for your contributions.
Steven, what else did you have?
 
Steven Bucher   18:59
Yeah.
So I'm trying to group some of these questions kind of together because a lot of them kind of are in the same same vein.
We had some a couple questions around PS resource get around you know how long will the competitive, how long will the compatibility layer between PowerShell get V2 and PS resource get be available for it as well as does it work on Windows PowerShell 5-1 and that's kind of the main question that know if you can take them real quick, Sydney.
 
Sydney Smith   19:29
So sorry, those are good questions.
So the first question was they'll take is, does PS resource get work on PowerShell 51?
Answer is yes, absolutely backwards compatible to PowerShell.
Windows PowerShell 51 a question though.
Just answer because we usually get it.
The next question is like so is it gonna ship in Windows?
Umm, the answer is I would love it to shipping things in Windows is hard.
Takes a long time and we're we've started the conversation started down that path, but I don't know when that could happen.
The second question was around the compatibility layer and so if you aren't familiar, there's a compatibility layer that goes alongside PS resource.
Get that allows you to use PowerShell get syntax with the PS resource get sort of back end and and.
The question is, how long will we continue to ship the compatibility layer in PowerShell 7 versions?
And the answer isn't so clear cut because it will be looking at sort of like numbers in telemetry around usage and burned out to see how many folks have moved over to the new, newer versions of he has resource gap and our like off the syntax.
My feeling is that it might take a really long time, or it might never happen that folks move over and that might be OK, but I don't imagine that happening anytime soon that we would get rid of the compatibility layer.
 
   21:06
Umm uh.
 
Steven Bucher   21:10
Another kind of common question we got was ohm around DSC V3.
So maybe Michael or Steve could pipe in about this one, but we got one question around.
Well, DSC V3 become a first class citizen.
Also on Prem or on dark sites and then.
Will there be any tool to to will?
Will there be any creation of a tool on top of DSC similar to that of Chef puppet?
And I know that's not a bit more meta, but maybe you guys can.
 
Steve Lee (POWERSHELL HE/HIM)   21:39
OK, that's fine.
So the goal of the CB3 is to be everywhere.
We're still very kind of early stage.
I didn't mention that psconfeu that we're targeting hitting GA probably towards this fall, late fall maybe so we'll see if we can hit that.
And of course, after that, we'll have 3132, whatever the case may be, it will be ongoing now as far as higher level tooling, you know we are partnering with Azure Machine config as kind of like our preferred higher level orchestration across cloud and on Prem you can use on Prem via Azure Arc.
So that's kind of where that is, but some of the designs that we have in V3 as compared to V2 from learnings should make it easier for anyone like publish.
I've answerable to integrate with V3 versus what they had to do with V2.
So what?
Open those partners also come along.
 
Steven Bucher   22:34
Cool.
 
Steve Lee (POWERSHELL HE/HIM)   22:35
Let it.
 
   22:35
OK.
 
Steven Bucher   22:35
Yeah.
Yeah, I think that was kind of just wanted to cover that.
Let's see what other questions we got.
Umm. Ohh.
So we had a couple of questions around uh, you know people, people are PowerShell.
However, PowerShell could also be quite resource heavy than other languages.
Is there any plans to update the core and reduce the overhead with PowerShell?
 
Steve Lee (POWERSHELL HE/HIM)   23:06
So here's what I'd say.
There's no immediate plan, especially AT-75.
I think Performance both in terms of resource consumption and also just throughput is always on top of mind.
We are starting some conversations with some folks Internal to see if there's some low hanging fruit, if you will or just easy wins.
But again, it's gonna be more of a journey than anything at this moment in time.
 
Steven Bucher   23:32
Umm.
 
Steve Lee (POWERSHELL HE/HIM)   23:32
That or.
OK, so here's one thing I'll mention, because I I think I got some of these questions.
App it's not for you from individuals.
Uh, some people aren't fully aware, you know, because you're built on top of .net or .NET Framework.
We are PowerShell itself is kind of a.
How does the best put this is limited and may not be the right word, but it is kind of bound by the garbage collection rules and net.
So sometimes you may see a high memory usage and that's just the fact of the garbage collector.
If you look online, there are ways within PowerShell to kind of force it to happen.
But you know, this is just the nature of having a managed language, but there are certainly areas with impartial where we can probably try to have less allocations and stuff like that.
And some of those changes are happening through the community overtime.
 
Steven Bucher   24:23
And then I'll take there.
There's kind of a there was a couple various questions around different modules.
One was when can we get yellow Support convert to and convert from YAML and I wanna.
I think the answer I think there's a lot of good Community modules out there and I can't think of any off the top of my head I that work with YAML, but I I think those are probably good ones to to check out.
I don't think we have any firm plans right now to to add those commandlets yet, because I think it's covered by the the community there.
So, uh, thank you, Justin.
Yeah, I knew.
I knew there was one by Jordan and then the last one there was a couple of questions around the PowerShell graph module around the names getting bigger and kind of a difficulty understanding what the commands are.
I've tried it with the with the graph team and we're gonna be working with them on that kind of feedback.
So not too much else to answer there.
It's kind of in their their wheelhouse and they have difficulty because they have coverage.
They have to do for all the different capabilities that they have, and so naming that and building out those commandlet sets or can be difficult to cover.
So yeah, that covers a lot of them.
I apologies if we missed any questions.
Still, I I think there is a number of questions we didn't address just because I think we needed maybe a little bit more clarification on what the, what the, what people were asking.
So if we missed something and you want your question asked, please feel free to put it in the chat or in a future community.
Call agenda and we'd be happy to answer it there.
So I just wanted to take a little time to cover some of the keys that we didn't get to at psconfeu.
So, umm, we yeah.
And yeah, I think we can probably move on, Jason to whatever is next on the agenda.
I don't have it in front of me so so.
 
Jason Helmick   26:17
Cool.
Thanks, Steven.
In a huge shout out and thank you to the organizers, the attendees, and all of the community for PS Conf EU for making us and everyone else feel so welcome and wonderful.
So thank you very much.
And with that, let's go ahead and go to a docs update from the infamous Sean the Don Wheeler.
 
Sean Wheeler   26:39
OK.
Thanks Jason.
Not a whole lot to talk about.
This month we our focus has been on maintenance work mostly we've had.
Supporting the monthly releases.
So the 75 Preview 3, the updates the seven, four and so on.
There have been minor updates, the release notes for those, as always, the.
Uh, there's a running update of what's new in docs each month.
The latest edition was this tab expansion two article I'll put links in the chat.
Mikey's Ben continuing to update documentation for DSC V3 through all of the preview releases.
Lots going on there and then I'll toss it over to Mike Robbins.
He's got a couple of items he wants to talk about for Azure PowerShell Mike.
 
Jason Helmick   27:48
Hey mate.
 
Mike Robbins (AZURE POWERSHELL)   27:50
So yeah, I'll go ahead and share my screen.
Just got a couple of things I want to highlight this month.
So we've updated one of our training modules umm for automating Azure tasks with Azure PowerShell.
It was previously used in like it was.
The code wasn't written very well and it was difficult to maintain, so I basically rewrote it the way that I would write it and also it was still used in ISE, so it's it's written for VS code now.
Also this configure Azure PowerShell global settings, it uses the AZ config commandlets and all the settings.
Previously it just kind of showed you a brief overview, but every single setting is detailed now, so if you wanna configure any of those settings you can take a look at this article.
Thank you.
 
Jason Helmick   28:50
Hey, Mike, that's great.
What else you got?
Chin, is that it?
 
Sean Wheeler   28:53
That's it.
 
Jason Helmick   28:55
Well, that's cool.
Thanks, Sean.
Mike and oh, hey, we didn't hear from Mikey Lombardi this time, but thanks to Mikey Lombardi too, who's working his **** off on DSC docs?
Cause that's something else.
So folks, we're we're starting to get towards the end of our call here and we're running out of time for demos.
But let me open up the floor for questions for a couple of minutes before we we end this July community call.
So please feel free to post a question in the chat or if you'd like to come online and ask a question as well.
Lot of stuff going on for the summer, so hope everybody is finding things to keep themselves busy.
 
Sydney Smith   29:33
Hey, Jason, I think there were a couple couple questions in the GitHub issue.
 
Jason Helmick   29:34
Yeah.
Ohh I'm I haven't.
I wasn't even looking at that.
Uh-huh.
Do you just want?
I don't have it up yet.
 
Sydney Smith   29:54
I think yeah, I can just, I can just ask them, I can have enough.
 
Jason Helmick   29:54
Sydney, do you just want it?
 
Sydney Smith   29:57
So the first one was just a like why was the.
Archive repository archived.
 
Steve Lee (POWERSHELL HE/HIM)   30:05
OK, you want me to take that one?
 
Sydney Smith   30:07
Yeah.
 
Steve Lee (POWERSHELL HE/HIM)   30:08
Uh, so basically, for folks who aren't aware, you know, there's a full Security focus initiative at Microsoft.
And so one of the things that we looked at is all the repos that haven't had a lot of activity like more than a year and basically we archived them for now, that doesn't mean archive doesn't mean that we're not supporting it, not specifically about the archive module repo being archive, there's no activity happening in that repo.
We don't plan to have anything in the near term.
There are desires to have a new version of an archive module, but that's not gonna happen anytime soon.
So that's the reason why we archived it along with a bunch of other repos that you mean be aware existed under the partial org.
 
Thomas Nieto   30:55
Would it be possible to add a comment in the README stating what the status of the project is?
Because archive could mean many different things to different people.
 
Steve Lee (POWERSHELL HE/HIM)   31:05
Uh, yes.
I don't think that's itself a problem.
That's a good point.
Umm.
If if there's a specific list of repos that people you feel people are concerned about, because it's gonna give I'm more than happy to have that in the read me.
 
Thomas Nieto   31:21
I think anything that shipped in PowerShell would be in a good yeah.
 
Steve Lee (POWERSHELL HE/HIM)   31:21
Just put put, put it in the in the chat, put in the chat.
 
   31:29
OK.
 
Sydney Smith   31:29
And then the second one was from Phil wanting to.
 
Jason Helmick   31:30
Awesome.
 
Sydney Smith   31:37
Promote or discuss the PowerShell Saturday Carolina.
 
Jason Helmick   31:42
He fail, Phil.
 
Speaker 4   31:43
Hey.
 
Jason Helmick   31:44
If you wanna do a big demo or something next month, I'm happy too, but go ahead and talk about it and tell everybody about PowerShell, North Carolina.
 
Speaker 4   31:49
The No, I just wanted to to to remind everybody that PowerShell Saturday is coming and the CFP still open for people.
But you know, last time I I briefly spoke about it.
Didn't really give an explanation of what it was if you weren't familiar with it, so it's gonna be an all day Saturday event and actually all day weekend event we'll have two days.
We'll have 3 tracks of speakers.
We'll have a panel discussion.
We wanna have the discussion be.
Primarily about who can participate and we want topics on cloud.
We want topics on what people are working on today and the the premises that this is a Community event for the people today and so we have the big summit events and PowerShell company you that are fairly large events.
This is the event for the people, effectively, as I call it, and anybody can attend that event.
It will happen in Raleigh, NC at the end of October and so set your calendars.
Mark your calendars.
It's definitely a great event, but also we wanna get engaged with the community and we wanna have those who are ohh feel that they have things to offer and and I think you know everybody does certainly have things to offer and you kind of talked about it earlier about engaging with the community overall and I wanted to be even shout out to user groups as a whole.
 
Chris Chisholm   33:03
And.
 
Speaker 4   33:13
And so this event is organized by the research Triangle PowerShell user group, and that group meets still twice a month, and Doug's doing a great job with his user group.
And there's a European group that's really still meeting.
And so my last little comment is that, hey, there's this other event, but also you know the user groups are still there and they still run effectively.
And if you want to participate in the user groups, reach out to us and we're here for you and you know, we are the community and we wanna be that community.
So thanks for your time.
 
Jason Helmick   33:46
That's awesome, Phil.
Thank you.
And Jean, thank you for dropping in the links for PowerShell Saturday and how to become a speaker.
So if you'd like those links, go ahead and check it out.
And with that, ladies and gentlemen, we are at time for the July community call.
I want to thank everyone for attending and sharing and contributing and we hope you have a wonderful month.
During this summer and we'll see you next month.
Be safe and help someone. Cheers.
 
Jason Helmick stopped transcription
