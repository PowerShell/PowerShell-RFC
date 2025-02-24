# PowerShell_OpenSSH Community Call-20250220
February 20, 2025
38m 23s

Jason Helmick started transcription

Michael Greene (POWERSHELL)   1:52
Welcome to everybody who's joining.
We'll get things started in about 3 minutes.
All right. According to my my PowerShell watch.
Blurry. It just has a background that says get date and that's an attempt to be PowerShell funny and like a dad joke. Sort of a way.
So welcome everyone to the February PowerShell community call I'll be hosting today.
I just wanted to make sure it's clear to everyone that we are recording the call and this will be posted on the PowerShell Community YouTube channel. Typically about a week.
Following the meeting, please do remember throughout the meeting our code of Conduct which is linked here in the chat, but we we ask everyone to be respectful and you know to really try to improve everybody else's lives.
You can also go through our contribution guide and take a look at that on GitHub and understand how to contribute to PowerShell and I can tell you one of the things that we really really appreciate right now is reaching out and helping with things like docs and.
Helping.
Create content even outside of these repos, so that people who are new to PowerShell or who or who are growing in their career can learn from your experience. So the agenda is posted in the discussion channel. It just happens to be discussion item 25,000 so that.
A nice round Number and we're going to kick off our agenda with a note from the Jason Helmick about an upcoming or sorry, a A blog that.
Is going live for the 7.5 announcement.

Jason Helmick   6:20
Hey, thanks, Michael. As a matter of fact, let's kind of continue with Michael's thing on on this. First of all, we got 75 out the door.
Show some love.
I'm going to show you the blog.
A couple of months ago, Sean Wheeler and I stopped by towards at the end of the year and did some demos from the upcoming release of 7575 is now out.
It's our stable release.
Let me share my screen and I kind of want to continue the theme that Michael started here about how much we appreciate your contributions.
This is a great example of that. If I can get the right screen to share, here we go.
We'll try that.
So this is the blog and you can go out to the PowerShell team blog site.
And take a look at the blog for 75. An important note here. This is now based on dot net nine and also an important Note 7. Two support has ended, so now's a good time to be shifting to a newer version in this release and this.
Is where I want to continue on with kind of Michael's theme here.
You're going to notice something in this release, and Sean and I had pointed this out a couple of months ago and it really is incredible to us.
You will also notice this.
We have a link in here to go to the what's new page for 75.
You really see this also on this. What's new page? Look at all of the names of the folks, not from Microsoft that contributed these features into the project.
This is really incredible.
We've had a huge community outpouring in a lot of different areas like Intellisense and the engine, stuff like that, and a lot of really useful improvements to the project have been made by you, the Community.
So along with contributing to things like docs and.
And helping us with content areas, you have been helping us in the project at building out a great project.
So here's what we'd like.
This is our stable release with 75.
Please use this in production and give us some feedback on it.
Use it in production if you're not bound to an LTS.
Speaking of an LTS, we also are currently working on the folks on this call have probably already seen us start to ship.
We're working on 7/6 and we're shipping out the previews.
Now 76 is going to be our next LTS release, so I know it sounds like we're always concerned about stability, but as we approach the LTS we'd really like it. If you can try to. It's a preview, so you're not going to really use it in product.
But if you could put this into a test environment and throw some workloads at it, help us ensure that the stability is reaching the levels that we want for LTS and the plan for the release.
Of seven, six is pretty much the similar plan that we always have.
I I'm thinking that towards the end of the year, we'll we'll do our 76 release. However, keep in mind we always release based on quality. So that may change a little bit giving a month or two in there. But in the meantime go out and enjoy 7.
Five, take a look at all the contributions that folks have made and some of the really useful things that have been added to the project and we look forward to your feedback.
So as always, try it out and let us know what you think.
Thanks Michael.

Michael Greene (POWERSHELL)   9:42
I was just talking to myself on mute.
Thank you, Jason.
That is awesome.
Next on the agenda, I see Andy, but I don't know if he's on the call yet.
He might be in a room with somebody.
If they're sharing a conference room.
Why don't we skip ahead? Sydney, if you're available now, why don't we go ahead and talk about the gallery? And if Andy joins later, we'll we'll come back.

Sydney Smith (She/Her)   10:08
Sure, sounds good.
This announcement will be pretty quick, but we have discovered that there are some ongoing issues with our e-mail relay system for the PowerShell gallery. So we are asking folks that if you are looking to contact gallery support rather than using the UI in the gallery, just directly send.
Us an e-mail, so I'll put the e-mail in chat, but it's PSG like PowerShell Gallery admin.
At microsoft.com.
So you can just send an e-mail directly there and we'll be able to service any issues you may have seen. The other issue related to the gallery that I was also gonna mention and make folks aware of is that we have been seeing intermittent issues with the search.
Index on the PowerShell gallery so.
If this is happening and you go to the gallery, it's gonna tell you that there are zero packages available.
We are invested heavily in trying to resolve this issue.
It's a bit of a complex 1 so I can't give an immediate ETA, but we are aware of it and taking it very seriously and doing our best to find a resolution.
Think that's it for me?
I'll pass it back to you, Michael.

Michael Greene (POWERSHELL)   11:22
Thanks Sydney.
Next up, we've got Mr. Sean Wheeler.
I did ping Andy by the way, to see if he can join.
But Sean, if you'd like to take it away and give us an update on what's happening in the world of docs, that sounds awesome.

Sean Wheeler   11:35
Yeah. So let me drop some links here in the chat and share my screen. Of course with the new releases, we have updated release notes for seven, six and 475 and.
We've propagated all of the commandlet reference 476 as well.
76 is currently built on dot net nine, just like 75.
But.
Once it becomes available.
Net 10 will be the the next LTS release of .10.
So the the plan is to build 76 on that as well.
So there's that.
I also want to talk about what are their new content we have and call out some more Community folks.
We have a new article about comments that was contributed by.
GitHub users surfing old elephant really appreciate that.
Surfing an elephant contributed 4 pull requests in this last month and open five issues has been contributing some good content both through the issues and through.
Updates NPR. Si also want to call out Ari Hein. He's been a monster.
Submitting multiple P Rs and continues to do so.
Doing a ton of clean up work.
Just.
Fixing spelling errors.
And formatting issues and really tightening things up.
Not a lot of not. Not real changes to the content, but certainly improving the quality of the docs, making things easier for us to maintain going forward.
Really appreciate help like this from the community.
The the comments article let me go over there.
And show you what.
That's all about this was an article he proposed to.
Document how PowerShell does comments.
We talk about it in a lot of different places, but there was never one location that summed everything up and I think the important part here is most people understand the the the various comments, but there are some special use cases like comment based help, the # requ.
Directive. It's used for signature blocks for signed content and so on.
Also, it's worth noting that you can put comments in.
Your regular expressions.
So that's a way to document your regular expressions that can be so hard to understand.
As well as Jason.
So nice contribution there.
While I've got it, I wanna call on Mike Robbins.
He has some updates about.
Azure PowerShell Docs, Mike, are you there?

Mike Robbins (Azure PowerShell)   14:53
Yes.
I'm gonna go ahead and share my screen.
So one of the new documents we published a few days ago is the impact of MFA on Azure PowerShell on automation scenarios.
The key here is if you're using a Microsoft intro user identity for automation, whether it's with Azure PowerShell, Azure CLI, or really any automation, you need to take action. You really should be using a managed identity service principal or a Federated identity, and this article will give you.
More information.
I'm gonna drop some links in the chat.
So then also.
Forget Easy access token and it's it's the bottom portion of this article that we've added a section.
So the default output for get easy access token will be changing from string to secure string.
And then of course.
We release a new new release each month.
It's Azure PowerShell now with all the GAM preview modules.
It's like 188 modules, so it's almost 8000 commands. I would recommend taking taking.
A look at the release notes and I dropped the link.
There's an AKA dot Ms. Link where you can always access the latest Azure PowerShell documentation.
And that's all had.

Sean Wheeler   16:29
All right.
Thanks, Mike. And I'll hand it back to you, Michael.

Michael Greene (POWERSHELL)   16:29
Awesome.
No thanks, Sean.
You both of you are are are always moving the ball so far for every single one of these calls there's so much docs content to talk about.
So that's always absolutely awesome.
Andy is going to be joining us.
He is sorting some things out with his puppy right now and the PowerShell community is absolutely puppy friendly.
So we're going to give him some time and he'll be joining us.
They'll be joining us here in a little bit, but why don't go ahead with Steve Lee. If you want to talk.
There was some.
Discussion there was a couple of PRS posted in the discussion for this call.
Might want to take a quick look at and maybe talk about.

Steve Lee (POWERSHELL)   17:11
Yep.
So rather than talking about specific PRS here, let me talk a bit about how the Community can really help the partial team out right.
Let me just like right now there's 139 pull requests and 984 issues.
And by the way, I don't want to thank everyone who already is participating in reviewing pull requests and also responding to issues, right.
That's been a great help.
Our team has not grown in size, but the work that we have to do has grown.
Right. So our team especially focused on the PowerShell side has been focusing a lot on compliance, securing our supply chain and things of that sort.
So these may not be things you see directly, but it is helping ensure that our product is more secure and it's better for you.
What I think we really what would really help other than having people maybe help review some of the pull requests to see if there's any kind of obvious design or coding issues, but also just.
You know if if there's a poll request that does impact you or you really think is valuable, I think if everyone can just give like a little thumbs up on the PR, that would help us identify how to prioritize our efforts to review these things, right, like we.
Do appreciate people contributing, but it's kind of gone beyond our ability to review them and having more support requests at this time is actually gonna make things worse. So I would say.
Not that I don't want you to submit a port request, but I think.
Rather than just trying to submit a report, I don't just be in a backlog that's gonna get like a year old and get complaints later.
I think it might be more helpful just at least help us identify.
Help us work through the backlog.
Let's get the PRS down to a more reasonable number.
I don't what that is right now.
And help us prioritize by, you know, identifying which P Rs.
So for example, if there's a regression, I think that's probably the top priority other than security, security handled separately anyways via MSRC.
But if there's like a regression or a blocker that's being fixed via PR, please identify those. If there's like a nice half feature that you think would really move the product forward again, give those a thumbs up and then we'll we'll try to evaluate those. Again, we're trying.
To dedicate some time per week, which is not limited to that day, but Monday's is where there's a lot of efforts across the different projects like open SSH to look at the repo.
PowerShell course, DSC and stuff like that.
So James, you have a hand up?

James Brundage   19:44
I mean.
Yes, if I can get off mute, I'm off mute.

Steve Lee (POWERSHELL)   19:50
Yes.

James Brundage   19:51
Cool.
So yes, there is currently a regression that is a little bit deeply frustrating.

Steve Lee (POWERSHELL)   19:57
Are you talking about the one that you have in chat?

James Brundage   20:00
Yes, I am talking about the one that I have in chat.

Steve Lee (POWERSHELL)   20:02
All right.
I don't want to take the call here to talk about that.

James Brundage   20:04
This.

Steve Lee (POWERSHELL)   20:05
Why don't we have a discussion or issue open?

James Brundage   20:06
Yeah.

Steve Lee (POWERSHELL)   20:09
Because I think there's gonna be a difference of opinion. And I think the language working group is the ones that need to make a decision on what is the thing we wanna do going forward for that.

James Brundage   20:19
It actually brings up another thing. The language working group has had outstanding applicants for a very long time.

Steve Lee (POWERSHELL)   20:21
Sure.

James Brundage   20:27
There has been no touchback on that.
There's been no kind of touch back on any of the PRS already submitted.
It's great to kind of ask for the help with regressions in the triage.
The problem ends up being in terms of end to end communication and involvement every time. So far the community has been trying to help.
It has been not effective.
Because it has not actually been able to move across the finish line.
So it would be really great if a we can try to fix the process and get some more community involvement.
B.
We can try to do some more proactive outreach.
C We can start including some community modules in your test passes and frameworks so that breaks like this don't happen.
And D.
The PowerShell team should be more actually willing to accept people's help, not just say they want people's help, because that distinction makes sense.

Steve Lee (POWERSHELL)   21:27
So James, you have to recognize that there is a saying too many cooks in the kitchen, right? So at some point, I'm I'm not saying about you specifically, right?
I want in general like there has to be some limit on the number of people, otherwise decisions aren't gonna get made. I do.
See Patricks on a call.
I'm not gonna call him out, but I think he I he's on the language working group. So I I want him to take an action item to review.
Any submissions for joining and they should make those. They should have those discussions and make those decisions and hopefully.
We can at least clear that backlog out if there is a backlog in that sense, I do see Jordan.
Have a question in the comments about how do you bring attention so again.
Easy wins.
Little bit harder.
I mean like, well, here's what I'll say.
Like sometimes A1 line or three line change seems like an easy win.
But we all anyone who's worked in software for enough time to know like that can also have other side effects that are not apparent until it's shipped right.
So not every not everything that is by lines of code is by default an easy one.
I would still say for now.
Let's see.
So so use the thumbs up if PR is important to you, whether it is something you care about or something that's impacting you.
I think for easy or small stuff.
I'm gonna have to talk to the team, and I'm hoping some of the PMS could step up and help us identify for engineering which ones those might be.
So my goal has always been for the Community day, for the partial team is to say, hey, we're not gonna just look at, you know, really hard ones because that takes forever.
We're also not gonna look at easy ones because then the hard ones never get attention.
So we wanna have a balance between those two buckets, but we need to kind of get some.
Assistance on how to identify which ones we should focus on, so I'll take it as an action item.

James Brundage   23:21
Yeah.

Steve Lee (POWERSHELL)   23:22
Don't have an answer right now other than my suggestion.

James Brundage   23:26
Well, that's again the thing.
It's like you're asking for us to help you determine this triage.
It's hard to get anybody's traction on issues.
I'm not sure how we should help you determine this triage.

Steve Lee (POWERSHELL)   23:38
No, but I mentioned one of the benefits of GitHub is that we can query against reactions.
Right, so I what I like, what is not gonna help is having everyone comment and put a plus one because that doesn't inform me unless I spend the time to go into every issue.

James Brundage   23:46
Sure.

Steve Lee (POWERSHELL)   23:55
And again, that's not gonna happen, right?
That's not gonna scale.
I think for now.

James Brundage   24:00
Yeah. So was neither is likes because.

Steve Lee (POWERSHELL)   24:03
For now, let's just try to use the reactions 'cause we can query against that and we can use that as a way just to identify, hey, we're gonna try to look at these first.
I'm not saying that we're not gonna try to look at everything, but everything means it could take a long time.

James Brundage   24:16
OK.
I have one other suggestion.
Could we possibly make the Community review time open so that people who are actually waiting on their PR could advocate for their PR in the community during time?

Steve Lee (POWERSHELL)   24:20
Go ahead.
My hesitation is now. There may be a queue of 10 to 20 people and that doesn't necessarily help.
I think what may be interesting though is having let me think about this, OK. But I'm just thinking, I'm just brainstorming here because I'm proposing a thing yet.
But my one thing that may help again, it's gonna be a challenge for different time zones across the world, right?
Which is, hey, if there's a PR that we identified and the author that PR is on a call that will actually speed things up because we can talk through.
In real time, the issues and get it worked on right. Rather than have this back and forth that takes a week, a month, a year, whatever the case may be.
So that's something that might be interesting.
We could look into.

James Brundage   25:14
Yeah, I think even having 10 or 20 people show up for the community might be its own sign of support. And seeing above amongst those tenor.

Steve Lee (POWERSHELL)   25:20
It's a fan support, but it's gonna get stuff done.

James Brundage   25:23
Among those 10 or 20, you'd also get some stack ranking of the ones that are there, so you'd get a feedback cycle on somebody's PR from the other people that showed up.

Steve Lee (POWERSHELL)   25:34
So James I.
I like your enthusiasm and ideas, but having multiple layers of bureaucracy is probably gonna slow things down, and I I do see Ryan has his hand up, so let's give him a little bit of time here.

Michael Greene (POWERSHELL)   25:46
Yeah. Thank you.

Ryan Yates   25:47
So I I had a thought on this last year when we were looking at how many issues were open.

James Brundage   25:47
OK.

Ryan Yates   25:57
And how many were constantly getting a + 1?
Or can we come back to this?
About looking at doing maybe a regular call from what would be not just the wider team, but a community lead call for issue triaging.
So that would be say once a week or once a month.
Where we can pull in a group of us. It doesn't have to involve all the team, but it can involve people that can have the access to be able to push things into the different working groups. This would allow for, say, something that's been stuck for a year.
In APR, to get a little bit more traction and then grow up the chain doesn't add a lot more.
Effort I don't think, but it would be probably be an area that would help.

Steve Lee (POWERSHELL)   26:56
Again, I like all the different ideas.
I think maybe rather than using up the entirety of this call here for this.
I'll, I'll let me create an A discussion on the repo and I will paste a link into the chat in a second after I'm done here and let's let's just discuss it that way. You know I want to kind of at least go broad now for ideas and.
We'll we'll try a few things and we'll see what works.
I'm I'm open to new ideas.
Yes.

James Brundage   27:21
I have one quick riff on Ryan's idea which I like quite a bit. If the user groups that are already operating, yeah. If the user groups that are already operating would dictate 10 minutes of their time that is normally hangout to helping review and triage PowerShell issues.

Steve Lee (POWERSHELL)   27:26
5 seconds.

James Brundage   27:36
That might go away.

Ryan Yates   27:38
Yeah.

Steve Lee (POWERSHELL)   27:39
That I like that.

James Brundage   27:39
So I'm going to start trying to do that in my user group and I would encourage others to do the same.

Michael Greene (POWERSHELL)   27:39
That is a cruel idea.

Ryan Yates   27:41
Just just adding, just adding very quickly on that one with the groups kind of being in a little bit more of a hibernation stage state over the last couple of years, I'm expecting this year we will have more groups kick off.

Michael Greene (POWERSHELL)   27:44
That is a cool idea.

Ryan Yates   28:00
I'm looking to try and do more UK time zone centric stuff which will work quite well for the EU.

Michael Greene (POWERSHELL)   28:05
Yes.

Ryan Yates   28:10
Community and some of the.
What's the APAC region?

Michael Greene (POWERSHELL)   28:18
Yeah.
And Ryan, as as the as the meet ups hopefully start happening again.
I want to make sure we renew our offer that we're always open to virtually joining.
You know, and helping out with discussion or doing some demos or whatever might be to help, especially as we sort of like start reinvigorating those meet ups. If if having sort of like.
In the agenda so and so's going to join and talk about a project they're working on, if that would help with attendance.

Ryan Yates   28:51
Yeah.

Michael Greene (POWERSHELL)   28:51
Then that would be possibly something we can help you contribute.

Ryan Yates   28:55
That leads on to very interesting point that I put in about. Should we just create all of the monthly agenda items for this community call as discussions already. So that way they they're already pre loaded. We can then put out things like Reddit posts, tweets, Skeets, LinkedIn to.

Michael Greene (POWERSHELL)   29:11
Hmm.

Ryan Yates   29:20
Be able to get more people coming to this call, or at least having visibility of this call.
And putting in things for the the community to discuss.

Michael Greene (POWERSHELL)   29:31
Yeah, I like that idea.
I don't.
I don't.
You know, there doesn't seem to be a big reason not to create that earlier, or possibly even stage the next few.

Ryan Yates   29:40
Because I really love the fact it was 25,000 was the the number for the discussion and it would be, it would have been fantastic if we could have like 25, one for January 25, two February, so on and so forth.

Michael Greene (POWERSHELL)   29:41
Which is.
Exactly.

Ryan Yates   29:56
'Cause, we like stability.

Michael Greene (POWERSHELL)   29:58
It just happened to work out. Sometimes you get lucky, right?

Ryan Yates   30:00
Yeah.

Michael Greene (POWERSHELL)   30:03
Yeah, I like that idea, which brings up a point we don't have any agenda.
I March is likely to be the time zone call where we do one a bit later so that it's more friendly to people in the APAC region.
I don't know if we've committed to that yet, but we should, like start the discussion and maybe get some feedback on, you know, if if that's still of interest to people having having the meeting at a time that's friendly, you know, to the Pacific, to the Asia PAC.
Time zone.
So yeah, we should get that going.
I think that's a really good idea.
Why don't we ask Sydney to jump back in and talk a little bit about the PowerShell Summit North America, which is not too far off. And then I see Andy has joined us.
They can take the floor following Sydney and then we'll call it a day.

Sydney Smith (She/Her)   30:55
Great. Yeah. I appreciate all the energy around wanting to make the PowerShell community function better, so I appreciate all the feedback there, but wanted to let you know that a bunch of us are gonna be at PowerShell Summit North America, which is coming up in about 6.
Weeks or so I think at the beginning of April, it's in Bellevue. So it's in our neck of the woods.
So we always love to join and spend as much of the week there as we can.
I believe the schedule is already up for it.
So you can see what sessions our team will be providing and then as well as amazing community members.
So yeah, thanks Steven for linking and we look forward to seeing you there.

Michael Greene (POWERSHELL)   31:40
Awesome. And Steven is also planning on being at PowerShell Conference Europe. So we should have good representation across all these upcoming events and definitely looking forward to all of them.
Andy, how's your puppy?

Speaker 1   31:58
He's hanging in there, but there's a lot of early morning emergencies now as he's aging. Unfortunately, yeah.
So sorry, I was dealing with that this morning, but let me get my screen shared over here.

Michael Greene (POWERSHELL)   32:06
No problem.

Speaker 1   32:09
We may have talked about this at the last meeting a little bit that it was coming.
Requesting a bypass system privately.
Yes, allow.
Hopefully I can see that it can see my screen.
So does this.
Didn't share.

Ryan Yates   32:25
Yes indeed.

Speaker 1   32:26
Yay, sweet that this was coming and I cross checked Jason.
It wasn't out yet, but since the last community call, we have actually gotten version 2025.0 dot OS in the first stable version of the VS code extension out. We're really proud of this one cuz there's a lot of stability improvements.
I mean, that has been true for a while.
We have really been trying to drill down on quality for the extension and you know Microsoft and PowerShell in general.
But I actually really loved working again with Justin Grodd on this.
Because we have just solved a bunch of problems.
There was a large class of bugs.
Forgive me if I've repeated this, but it was just so annoying as somebody who had to maintain this and had to deal with a lot of these issues always coming in. People would use Windows PowerShell with the extension, which is we supported as best we can liter.
We do supported that as the best effort basis, but we don't have that assembly load context like like.
Ability to separate out dependencies from each other in the extension when using.
In Windows PowerShell and for some reason way back when in the history of this extension, a large dependency had been brought into PowerShell editor services to handle logging called Serialog, and turns out that clash with a lot of other modules that would get loaded up.
So if you were one of those users trying to use the PowerShell extension with the you know specific extension terminal and you're using Windows PowerShell, well you know what based on the number of bug reports I got very often you would wanna load another module that wanted a.
Different version of CL Log and they would clash.
And for a long time, her answer was well, we can't fix that 'cause we don't have assembly load context.
It's not a technology in the desktop version of net. And finally, we said to ourselves, the issue here really is that there's a dependency conflict.
What can we do this without Sierra log?
Can we handle all of our logging without Sierra log?
And I'm like, I think we can and I can't take credit for this.
I kind of just said I think that's a great idea.
We should go for it and gave Justin the thumbs up on, like, hey, I will totally shepherd NPR for this.
We'll get it tested.
We'll get it through preview.
And you know what miracles happen?
Justin did exactly that, PowerShell editor services.
The LSP server underneath doesn't depend on CR log anymore, since getting the stable extension out, anybody's used, the stable extension has been able to report hey yeah, those modules I was having all those loading conflicts with.
Working again and it's not like a temporary fix, it's it's gone as as best we can't seeer log is completely gone.
We don't depend on.
It doesn't get brought in.
We've really minimized that dependency service.
And so I'm super proud of that really.
Thank you to Justin for being such the awesome Community member that he is.
We also had, you know, a few other bugs get fixed, especially like around end of support for older versions of PowerShell as you probably all know PowerShell 7172 and seven, three pretty much met their end of support life cycle and so we've removed like support.
For them in the extension and that happened, I think.
One stable version ago, but what I didn't do was improve the messaging around it.
So we actually have a much better messaging around.
Hey, you loaded this up or you're trying to load it with a version of PowerShell that is not gonna work in. We actually not attack that and tell you. Hey, you need to go update to at least 7 four, which is great. 'cause. That's an LTS you can.
Just keep on using that, but I do just want to show one other amazing thing from Justin.
Maybe he should run this for a little bit. If he's here, I should at least say hi.

Justin Grote   35:55
No, I'm here. You're doing great, Andy.

Speaker 1   35:55
But he implemented extension categories. You wanna?
Well, I was just gonna show that we've got settings for.
Sorry settings categories.
In the extension this was a nice little feature improvement.
You can actually click through.
These are all in different categories around the extension.
You've got your terminal settings, you've got your pastor settings.
You've got your developer settings and your formatting settings.
It was just a nice little quality of life thing that vs code lit up and Justin was like, hey, we can just do this and we got it in.
Great little improvements love seeing this.
And so yeah, I think that's pretty much all I have to share here.
Thanks to people who keep testing the preview when it's out and for using the PowerShell extension as a whole file.
Those bugs we do get to them.
We've got a lot of snippets fixed too lately from people.
So yeah, thank you.

Michael Greene (POWERSHELL)   36:47
Thank you, Andy, and thank you, Justin.
Well, I see we are right at an hour from from when we started.
So thank you very much. I'd say let's go ahead and wrap up.
And thank you everyone for joining.
Great feedback as always.
Definitely, always a spirited discussion and appreciate everybody attending.

Ryan Yates   37:05
Just a very, very quick one. On March 11th, I'm going.

Michael Greene (POWERSHELL)   37:07
Yeah.

Ryan Yates   37:10
I've been saying this to myself for like the last two weeks. I'm going to go into into depth about some of the feature set of GitHub for issue management traction.
And other things that you can get basically for free with the platform.
I will put a link on Twitter and into into the chat.
Once I've got the meeting set up, but it's going to be part of a monthly series on the second Tuesday of the month, which is an interesting day to pick because that's all when everybody goes.
Here you go.
Here's all your vulnerabilities.
So it will be an interesting day every month for the rest of the year is my plan.

Michael Greene (POWERSHELL)   37:55
That's awesome.
Very cool, Ryan. Thank you.
Always awesome for for more content to help everyone benefit each other.

Ryan Yates   38:07
Exactly.

Michael Greene (POWERSHELL)   38:08
Cool. Well, why don't we go ahead and wrap up for the day and we'll see everyone next time.

