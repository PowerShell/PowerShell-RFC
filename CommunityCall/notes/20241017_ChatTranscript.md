# PowerShell_OpenSSH Community Call-20240919

September 19, 2024
31m 8s
 
Sydney Smith (She/Her)   1:19
We'll get started in just a couple minutes.
Feel free to talk amongst yourselves in the meantime, but you are on recording.
Hello everyone.
Welcome to our October community call.
As it was posted in the chat, this call is recorded. If you do not wish to be a part of this recording, please go ahead and leave the call. This call will be posted to YouTube later on.
We guarantee sort of within the next two weeks, but usually it comes quite a bit sooner than that.
So yeah, thank you all for being here.
We always appreciate you being on our call.
As always, you can find our code of conduct linked into the chat, but we take it very seriously.
But most of all, we just be kind to each other.
The agenda is also posted in the chat.
We always post our agenda in the discussions section of the PowerShell PowerShell Repo, and there you can post topics, questions and demos ahead of time.
That's where we will facilitate demos going forward.
For calls, so we do have one demo lined up for today and then a number of topics. But throughout the call we do want this to be interactive.
We welcome all sorts of questions and topics, so if you would like to participate, go ahead and use the chat or the raise hand feature.
You can also continue the discussion in the discussion on GitHub so.
With that, we can go ahead and get started with our agenda.
So the first topic is actually just kind of our schedule for the rest of the year for this call, so.
Per usual, we are not planning on having a December call since we do this call on the 3rd Thursday of the month. The way it falls is kind of right on the holiday season for most of us.
And so we, we typically just skip the December call and we're planning on doing that this year, which means that next month our November call will actually be our final call of the year.
Year and it also falls during Ignite this year, which is pretty fun and we're planning on just having it be a community really celebration call. So it'll be a lot more demo driven, hopefully a lot more community participation, less of the PowerShell team, more of the community that.
A typical call. I already went ahead and created the discussion for it in GitHub, so I'm just going to go ahead and post it there.
So just go ahead and start posting ideas for.
Things you might want to show off and celebrate.
It's just really a way for us to celebrate all the different things you guys have done with PowerShell over the last year.
We might have a short section of a few minutes just of the PowerShell team recapping some releases from the past year, and maybe celebrating PowerShell 75, but other than that, we really want it to be focused on things that the Community has done this year. So go.
Ahead and post in that discussion what you would like.
To show there is no.
Demo that is too too simple or too basic or too advanced.
So we welcome the full spectrum.
It's always appreciated.
I think that's all that I have for that update.
Next up, we do want to remind that PowerShell 7 two is quickly approaching end of life.
November 8th is coming up 472 end of life.
A common question that comes up is yes, we are working very closely with.
Partners in Microsoft to remind them and support them through this update.
I know in particular Azure automation is planning to get out support for seven four very, very soon here.
And so just a reminder that if you haven't gone ahead and started your testing on 7/4, which is our current LTS, now would be the time to as again 7 two end of life coming up in the next couple weeks.
Yeah, I did see in the chat here asking if anybody from the team will be at Ignite.
Even is gonna be there representing the team. So definitely look out for him and connect with him.
He'll be so excited to talk to you and talk partial.
Let me go back to the agenda, all right.
Great. That's enough of hearing me talk.
I'll pass things over to Tess. Who's gonna talk a little bit about open SSH.

Tess Gauthier   7:25
Thanks, Cindy.
Hi folks. A quick update on open SSH. Some of you may have already seen it, but we released 9.8 on GitHub last week.
Focus of this release was splitting the SSH server into a listener binary.
And a session binary.
But there shouldn't be any major differences to the end user from this change.
There were also a few security fixes included in this release that were also patched in Windows via version 9.5 also last week.
These fixes relate to changes in SCP and SFTP as well as the SSH agent, and you can check out our release notes For more information on the specifics.
I also wanted to note that we're tracking an issue with SSH server.
Not starting after the Windows patch due to the permissions checks on the program data, SSH and program data, SSH logs folders that was added in version 9.4.
So if you're seeing any trouble starting SSH server after updating your Windows machine, the issue on our GitHub repo explains the expected folder permissions and how to fix that.
As always, thanks for your feedback and please open any issues on GitHub if you encounter any problems with the release. Thanks.
Oh, Danny.

Danny Maertens   8:43
The only thing to add to that is the 9.5 patch was applied to all versions of Windows where open SSH exists.
So even if you are on server 19 and had open SSH77, if you're up to date on your security patches, you will now have open SH 9.5.

Sydney Smith (She/Her)   9:09
Hey, awesome.
Thank you both.
Next up on the agenda I have Andy to give some updates on script analyzer and vs code.

Andy Jordan (they/them)   9:23
I just getting a screen shared here.
One second loss of permissions to allow from a Mac.
Here we go.
So first up we got version 1.23 PowerShell script analyzer out.
This was a fairly small release.
It mainly allowed us to test our brand new like release pipeline that we've, you know probably heard.
We're all working on internally.
Lots of security stuff.
So it is a nice freshly, properly signed, newly tested release with a whole bunch of like under the covers, fixes to it.
System. It does have a great Community contribution for handling the redirect operators when they're not in stream order, so that is in there.
And then I had wanted to get to share that there was a pre release of VS code out. We are still working on it, but it is up and coming pretty hot here.
It does include that new PowerShell script analyzer. It has a great new feature from a Community member that now lets you switch between using the call operator like ampersand and dot source like you know, literally a period when.
Your scripts in the debugger.
That was a long requested feature.
I'd love to see that contribution come through and get in.
We are also pending some updates from our amazing contributor Justin Guroti that will fix up the debugger in quite a few different ways.
Some of those will be in the pre release, some will not.
And over on the VS code side itself, if I can click underneath the bar, you guys can't see. Here we go.
Very similarly.
But when this release comes out, it will exercise.
Our much like completely up to date pipeline, I'm still working on the back end to get that pipeline. The requirements for it like work like allowed. So I can get this out.
That's sort of what the delay is there. However, it is going to come with a great fix for.
And I'm spacing on it PowerShell Shell integration.
So upstream vs code kind of move where the script was. If you are currently using the extension and you're like, wait, why did the Shell integration script like stop working? That's because.
The path to it just sort of needed to be adjusted.
That fix is in here as well as some fix for some expected environment variables for that shell integration script. We continue to work with the team over NVS code on getting all that stuff to work better and the good news on that front is since we don't have.
To support Azure Data Studio as a thing anymore.
In terms of our PowerShell extension on the marketplace, like we get to update Rvs code engine finally.
So we used to be stuck on like older releases.
We now get to support just straight up to date 1.94 I think was the latest release of VS code in our package, which means we get to take in newer features from VS code as they become available rather than us go.
We can't do that yet.
Our engine is out of date, so yeah, please look forward to the pre release. It will be out momentarily within a week I'd say.

Sydney Smith (She/Her)   12:17
Awesome. Thank you so much Andy for all the hard work there.
That's been really great to see and I would say for both of these projects, really appreciate how community driven they are and all the community P, Rs and triage and work that goes into that next step. I have a docs update from Sean.

Sean Wheeler   12:38
Buddy folks, let me drop some links here for you.
Got a lot of stuff to talk about.
Mostly it's updates.
To four new releases.
I've added release notes for PS Resource Git to cover the release history and talk about the changes in each version. We're now up to 1.1 Preview 2, which has a couple of new features. New commandlet compress PS resource that allows you to.
Create an upke without publishing it so that you can then sign it before publishing it.
And then there's to support that.
Publish PS Resource has a new parameter nupkeg path where you can point it at the nupkeg that you've built and publish it to your repositories.
And as Andy said, there's been updates for script analyzers.
So there's updated documentation.
There's a couple new rules.
I forget what they are, but have to look at the the release notes again.
And then there was a couple of rules that became configurable in this version.
So check out the new documentation there.
Of course we had another release of PowerShell 75 preview.
We're up to Preview 5 now and I just want to call out if you scroll through and see all of the changes here.
There's a ton of contributions here from the community, and we certainly thank all of the Community input that we're getting for these releases.
And with that, there's a couple of highlights.
If you haven't seen the performance improvement that Jordan Borean added for Array Edition, that's really cool. And two new commandlets convert from XMLCLI and convert to XMLCLI.
And you may be thinking, well, what are these for? We already had export and import CLI well export and import CLI are.
File driven. They export rights to a file, import reads from a file.
The convert to and convert from is about doing this in the pipeline.
So now you can.
Do things like this where you.
Convert objects to to CLIXML in the pipeline.
And convert them back.
Also.
Added.
New release notes article for PS.
Readline this is not new information.
Just split it out to its own article.
So now you can see the release history of PS Readline, what version shipped in what version of PowerShell, and what the changes are.
Summary of the changes and last but not least, we have one new article.
Here in.
The PowerShell documentation about strategies for improving the accessibility of your output from PowerShell.
If.
If you're a user that uses a screen reader, you're probably.
Very familiar with the pain of.
The screen reader trying to read the terminal screen and so this article talks about ways that you can reduce the output on the terminal screen.
Move the output to a more accessible view like out grid view or export it to other formats that can be opened in applications that are more accessible like CSV format in Excel or HTML in your web browser.
And so on. So.
That's it for the docs updates. It looks like James. You had a question.

James Brundage   16:56
More, thank you and a quick comment.
It's great to see convert to and convert from clicks now and also sorry to be the you know proverbial mouse with the cookie.
Could we get an import and export Jason?
Because having completeness for any of the conversion scenarios is pretty important.
Anything that we can invert to and convert from, we should also be able to import and export basically.
It would make life so much easier.

Sean Wheeler   17:27
That sounds like a good issue to add to the repo and we look forward to P Rs.

James Brundage   17:34
Cool.

Sean Wheeler   17:34
So.
And that's the end of my bit. Sidney, back to you.

Sydney Smith (She/Her)   17:42
Great. Thank you so much, Shawn.
And yeah, totally agree. If there's already an open issue in the repo, I would say go for it.
With that, I think I'm passing things over to Michael to talk about Azure MFA.

Michael Greene (POWERSHELL)   17:55
I will put the link for the context of the discussion in the chat.
This is especially for anybody on the call or listing the recording who's an Azure customer.
Damien's not available today, but he asked if I could call this out in the community call Azure is beginning the roll out of enforcing multi factor authentication and that docs page provides all the details what you need to know for right now is it will start for.
The.
And there is a process to ask for an extension, but that will happen within this calendar year and the dates are all there next calendar year in the first half of 2025, we'll see that get start getting enforced for Azure PowerShell.
The reason you might care for your release pipelines that you're using to deploy infrastructure as code if they're using a user account.
To authenticate then multi factor authentication would start getting enforced and that would cause your release pipeline.
To have a problem.
So then the answer is to move to an app ID or to managed identity depending on the platform.
But there are some cases and this is the kind of if you if the answer is you don't know the thing to be aware of is for some organizations that have either home grown or sort of, you know quote UN quote third party identity management solutions, some of those.
Are.
Or have been implemented to use user accounts, kind of like service accounts instead of using an an app registration and that kind of thing and that kind of thing so.
If you happen to be aware of a release pipeline within your organization that's using a user account to authenticate, you know this gives you a good six months to or I don't know, the dates. It's not published yet, but this gives you some heads up time to go.
Take a look at that and see if that's going to be a problem and try to get ahead of it and that's it.
Thank you.

Sydney Smith (She/Her)   19:56
Great. Thank you, Michael.
The next topic I think comes back to me and this was one that was requested in the GitHub discussion, but I'll put it in the.
So you may have seen that I opened up a discussion topic in the PowerShell repo, which was. Should we ship PowerShell 7 in Windows?
And this sparked a lot of conversation, and I think the meta question for this call was like, hey, why did you guys open this question?
You know, maybe some folks had have heard us say in the past that this was an impossible task and that sort of thing.
So like, is there something happening?
Why now?
And I can.
I can just give you like the the answer.
The very straightforward answer as to like why?
I opened up the discussion topic and something you've probably heard us say many, many times like over the years. If you've been on our calls or around the team, is that at Microsoft?
Sort of. The customer voice is a lot louder than the engineering voice.
So when we go to like make an ask or a request, if we have sort of customers asking for that directly say to the team that's going to carry a lot more weight than if we as the PowerShell team.
You know, say hey, like PowerShell needs to be here.
PowerShell needs to be there.
So that's that's sort of an idea.
You've probably heard us say many, many times.
Whenever we go talk to to you guys at conferences.
Have talked to customers.
It always comes up as one of the top requests to have PowerShell 7 in Inbox in Windows, and so even though you probably feel like there's been no movement on the outside for many, many years, which is completely valid.
Something that's on our sort of, you know scenario list or goal list semester after semester, right?
So we're always kind of turning on this topic.
What can we do to make progress on it? And so as we've been having discussions, one thing we realized is like, hey, even though we've had this conversation maybe hundreds of times over the last few years, we don't have a lot of customer, you know verbatims that we.
Can point to and say like, hey, this is the reason why somebody needs.
PowerShell 7 in Windows.
The simplest way to do that for us was like, hey, the lowest cost. What if we just open up a GitHub discussion and see what people have to say? So that was the reason.
That's why we did.
It was just to start collecting those kind of customer stories around.
What it would mean to have PowerShell 7 in Windows as we continue to have conversations with internal teams.
About making that move.
So it's really as simple as that. I think that the.
What's been really cool is that since we've opened the discussion, there's been.
A lot of sub discussions about different issues that might come up with this different things we might need to consider about, you know, transition plans, editors, all the different types of issues that come up with whether we have them side by side, that sort of thing. So.
Keep keep bringing the hard questions in there.
Keep bringing you know your scenarios one way or the other.
There's no right or wrong answers. Each comment in there pushes us to think through.
Different things as we have these discussions so.
I I see a question there.
What's the best how to make or sorry, how do we make that kind of request in the best way to be heard? So one way is like, yeah, you can, you can tell us. But what we always say to is like if you can go to sort of.
The the partner team and make the request there too.
That can also be really well received.
So if you're going to say Windows and saying, hey, I need PowerShell step in.
That can help us with the conversations as well.
So that they're not just hearing from us, the PowerShell team that we need to be there, but they're also hearing directly from customers.
I'm also seeing some questions about what would happen to Windows PowerShell 5 and like whether or not we would replace them.
And yeah, I think in the short term it's very difficult.
I can say that like it's very difficult.
To take things out of windows and so, at least in the short term, they would probably be side by side, but that's part of the story that we would need to figure out.
And nothing's gonna happen overnight, so.
Yeah, I think most likely short they would be side by side at least in the short term.
Hopefully that answers most of the questions in here. I'm just reading through.
But definitely feel free.
To keep them coming, keep using that discussion. And also if you have connections in Windows or that sort of thing.
Forward and, you know, make your request known.
One other question, I think the main issue being net support lifecycle not matching which Windows requirements that hasn't exactly changed, but we're we're trying to like figure out if we can make negotiations with different teams find work around.
We're all at Microsoft, so it's kind of like, hey, how can we make this work?
I guess so.
Nothing has substantially changed and support lifecycle. It's more like, hey, can we figure something out amongst all of us together?
With that, I am going to go ahead and pass things over to Jim. I believe at this point.

Jim Truher   25:37
Oh, this this is going to be my last call as a Microsoft employee calls in the future will be as a civilian. I am retiring at the end of the month and after 25 years of Microsoft and working for some 50 plus years, I've decided that now.
Is a good time to retire, so I'll still be around, but certainly not as part of the PowerShell team and.
The first thing I'm going to do is sleep for a couple of months and then we'll see what happens in the new year.

Sydney Smith (She/Her)   26:12
Well, Congrats, Jen.

Steve Lee (POWERSHELL)   26:12
Alright.

Sydney Smith (She/Her)   26:13
Oh yeah, go ahead.

Steve Lee (POWERSHELL)   26:14
Yeah, I just wanna add real quick.

Sydney Smith (She/Her)   26:14
This is someone saying.

Steve Lee (POWERSHELL)   26:15
Well, let me just perceive on behalf of myself and the team.
I think sort of think that Jim has done but.
Yes. What Jim had mentioned to me is that he'll take some time off, but I believe your plan, at least in within the working groups and the committee and when you can, yeah, so.

Jim Truher   26:30
Some of them.
Yeah.
You can find me on LinkedIn as well if you wanna send me something, some mail or whatever.
Yeah, just making that announcement.

Sydney Smith (She/Her)   26:46
Well, Congrats on your retirement.
Definitely very, very well deserved. And thank you so much for all you've done for, for PowerShot and the team and the community over the years.
You can check out the chat.
There's also lots of messages for you there.
With that, I believe we are moving on to Community demos right on time and.
Ryan, I believe you are up for a community demo.
You are on the call.

Ryan Yates   27:15
Yeah.

Sydney Smith (She/Her)   27:17
Probably need to give you presenter rights, I'm assuming.
Let me find you.
Here you are.
OK.
I just made you a presenter.

Ryan Yates   27:48
And.

Sydney Smith (She/Her)   28:09
I don't know if you're just getting set up, but if you're talking, we can't hear you.

Ryan Yates   28:13
No, I was I meeting.
God chap.

Sydney Smith (She/Her)   28:20
Oh yeah, totally. Go for it. Yeah.

Ryan Yates   28:29
I don't know.
Oh yeah.
Let me share this screen.
Alright.
Yeah.

Michael Greene (POWERSHELL)   30:01
Some time when come back.

Sydney Smith (She/Her)   30:12
Brian, have any any technical difficulties or?

Ryan Yates   30:20
Yeah.

Sydney Smith (She/Her)   30:26
You wanna go for it or do you wanna say the demo for next call?
We'll have plenty of time for demos next month.

Ryan Yates   30:34
I'll pop the demo next next moment.

Sydney Smith (She/Her)   30:38
Yeah, that sounds great.
Alrighty well.
I'm just checking the chat.
I'll pause.
We have a oh, I just muted myself for no reason.
We have a few minutes left. If there were any questions that didn't get answered or anything I missed in the chat, so I will pause and if I miss your question, feel free to either raise your hand or post it again in the chat. We can get it.
Answered.
And if not, we will end there for this week or this month.

James Brundage   31:27
I'm typing out a few quick releases in the chat.
I'll do that real soon, fast.

Sydney Smith (She/Her)   31:32
How are you doing?
Yeah, I am seeing some questions in chat so we can get these answered.
I see.
The first question is, are there any updates on publishing PowerShell container images in a timely way?
So I believe that.
The PowerShell we did.
We have published the container images for the last release.
Jason.
You might have a better pulse on this than I do, but I believe that they're they're all out at this point.

Jason Helmick   32:03
Hey, Sidney.
Yeah.
They're not all out yet, but they hopefully soon we.

Sydney Smith (She/Her)   32:10
Oh, they're not OK.

Jason Helmick   32:14
'Re at the point now to where when we do a release like for 746 that these the new container images should go out for the different distros.
So we're hoping during this week.

Sydney Smith (She/Her)   32:26
OK.
Yeah. So we're we're working on releases right now. And so sounds like the pipelines are ready to go for those releases.
With the seven five release coming soon, what are the plans for 7/6?
Yeah. So, as many of you know.
PowerShell has a yearly. Typically has had a yearly release cadence.
So with 75 coming out soon, that means we're a year away from 7, six and seven.
Six will be in LTS release.
I think look forward.
Hopefully Steve does a great job of posting A blog.
With what's coming in seven, six.
But yeah, Steve, do you want to answer this one?

Steve Lee (POWERSHELL)   33:11
Yeah, I mean, that's you kind of basically got to like expect the team investments block to come out early next year, hopefully January this time or did I do that last time or early in the year either January or February, we're still discussing internally what we're committing to.
For in the seven six release. But as Sidney said, it is alts so trying to minimize a lot of churn.
So we're gonna maybe focus a bit more on bug fixes and getting a bunch of the community PRS into the product.

Sydney Smith (She/Her)   33:42
Well, so if you're if there's something you you want.
You feel passionately about 76. You know we're we're in those discussions now.
So make your make your voice heard on GitHub and the issues.
When is the 75RC coming out?
We don't have a specific date, but soon.
Soon. Yeah, this month.
I'm also seeing that the call for papers is open for PowerShell and DevOps Global Summit.
And that reminds me, I was going to make sure to make an announcement at this call too, that we really want to make sure we're supporting user groups.
So if you do ever want to shout out your user group at this call, we always welcome that on the agenda.
We can always promote your user group on this call and also feel free to invite us to your user group.
If you want a representative from the PowerShell team.
I see another question.
Any info slash ETA on PowerShell 74 for Azure automation?
Yeah. So we are working really closely with this team and I know that they're really close to releasing it.
I don't know.
Like I it's not my place, I guess to give you their ETA. But it's very, very soon that they are planning on having 74 available and they are planning on having it available before the seven, two EOL and we're working closely with them on that.
Great.
I think that's all the questions.
And everyone got everything in.
Thank you all so much for being here today.
And we really look forward to having a really great celebration community call next month.
I look forward to some really awesome community demos and just celebrating all the awesome work that you've all done.
So thanks everyone for being here.

