# PowerShell_OpenSSH Community 20240418_092857-Meeting Recording

April 18, 2024,

Jason Helmick   2:41
Well, ladies and gentlemen, let's go ahead and get started.
Hello.
Good morning, good afternoon and good evening.
Wherever you're located and welcome to the April 20, 2024 (PowerShell community call.
Work.
Everyone is welcome to join and discuss the (PowerShell project and it's Development.
Hey, we have an action packed agenda today and I posted that agenda into the chat session along with some additional notes.
Please feel free to post questions in the discussion issue or here in this meeting chat.
We'll do our best to follow those and if you haven't, please take a look at our code of conduct.
That's the one thing we take seriously.
So with that, let's go ahead and dive in with our agenda.
So several folks from the PowerShell team attended the (PowerShell Dev OPS summit last week, and we wanted to take this opportunity to kind of share the experience and have some folks that were at the summit share this experience as well.
It was a very unique and amazing experience and it gets me all excited for PS Config you.
So this morning what I'd like to do if we could, is start with a couple of the organizers from (PowerShell Summit, James Petty and Missy Janosko.
Now I don't know who's going to go first or if you guys want to just kind of do this in tandem, but without spoiling what you you have to chat about, I just want to say first of all, thank you so much for a great conference and I have to say it was probably one of the best summits that I've ever been to.
So really enjoyed it, so let me go ahead and open up the floor to James Petty and Missy Janosko and they can talk a little bit about the summon as well you guys there.

James Petty   4:39
Alright, I'll let Missy go first.

Januszko, Missy   4:39
OK, great.
Ohh, all right.
Uh, first of all, I just want to say a great big thank you to a couple of folks.
Thank you to our sponsors for the event.
Without them, we could not put the quality of event on that.
We do thank you to our speakers for bringing your ideas to life on the stage.
Thank you to the PowerShell team for being there and checking in with all your fellow attendees.
Thank you to our volunteers who helped us do some couple of things like set up and tear down and manning the booth with questions.
And lastly, thank you to our attendees for bringing all of your ideas together and having those great technical discussions deep into the wee hours of the morning.

Jason Helmick   5:39
Hey, Missy, were those sessions recorded?

Januszko, Missy   5:42
Yes, they were.
They will be available on YouTube on the (powershell.org YouTube channel, probably within about, I'd say.

Jason Helmick   5:44
Ohh.

Januszko, Missy   5:54
30 to 60 days.

Jason Helmick   5:57
Outstanding. Outstanding.

James Petty   6:01
Yeah, I know a lot of our attendees for the first time in a really long time had session anxiety, like they could not decide which session to go to you.
You can only be in one place at once, and then like, don't worry, they're they're recorded.

Januszko, Missy   6:15
I had that same session anxiety.

James Petty   6:20
Or even the times when you wanted to watch all four sessions at once, which happened quite a few times.
So I've been attending summit since 2016 and this is, I think, what the best one that I've been to in a really long time.

Jason Helmick   6:33
Well, James, what I mean for you, how did the show overall turn out do you think however attendees and and and all of that kind of stuff?

James Petty   6:42
Yeah.
Ohh.
The attendees are great.
Every expectation that we had was completely blown blown out of the water.
Just this, the PowerShell community is just in in amazing community unlike any other, like Missy said, littered till I want name names but some people said up till 456 o'clock in the morning sometimes just chatting and having a great time we closed down the bar every night at 1:00 o'clock and it's not just a few people.
There were dozens of people that they had to say go home.
Every, every night, just and the and the same people that were up until one 2:00 o'clock in the morning were there at 7:00, o'clock having breakfast and continuing the conversations that they have they had from a couple hours ago.
It was just fantastic and also the speakers who ran out of time during Q&A during their sessions.
They just keep bringing that session out into the hallway or down into our open spaces room that we had in it.
And the conversation is just just kept going.
They never stopped.

Jason Helmick   7:42
It's awesome.
Well, you 2 stay around and stick around.
Feel free to chime in and join in with additional thoughts and all of that kind of stuff.
One of the things we also wanted to do was bring on one of the attendees to the session and Matthew, are you there?

Matthew Gill   7:58
I am right here.

Jason Helmick   8:00
But Matthew wants you.
Won't you go ahead and?
And once you go ahead and say a few words and all that.

Matthew Gill   8:05
I I don't know if you pick the right attendee, there are people who know me and Steven, I think said.
Yeah, yeah, there's no time constraint.
And that was a mistake that you'll only make once, I think.
Hi everyone.
I'm at Matthew Gill.
This is my second summit.
I was at 2023 and I was at 2024 and this is a James, I think you said Ohm.
You said that this is an unconference.
That was how you kind of touted as an unconference or a nonconference or a non France.

James Petty   8:34
OK.

Matthew Gill   8:36
I just kept making things up as I went along there.
Umm, because it is.
It's completely unlike any other conference that I've ever been to.
I've been to the big neon extravaganza vendor wine and dine, sort of, you know?
Oh, it's in Vegas.
Next to the sphere, and it's really cool and you won't remember that night and.
Yeah, but what I'm seeing at that conference at those conferences, it's not real.
It's it's.
Hey, here's the thing that we're gonna try to sell you.
We're gonna wine and dine.
Try to get behind that and you know get some momentum there and and then it's never gonna work in your environment.
The way it works in our perfect little demo world at Summit, you're rubbing elbows with like the greats at Microsoft.
The greats in the community you're rubbing elbows with, people that have written books that are writing books that are coauthoring.
A lot of these things you might not even realize because you we live in a world of avatars, right?
You never see the actual face.
I think Corey, did you, Corey?
I think said it it's a good chance to put like the little icon that you see to the actual person, which is totally true because no one's actually a cartoon.
And you know, you're talking with these people and you finally have that moment.
I know that moment with I don't know if Fred's on the call, but I had that moment where I'm talking to Fred and I said yeah, you know, we're actually gonna move to basic dependency framework for these things.
And he goes oh, thank you.
And in my head I go.
That was weird.
Why would you say thank you to that?
That doesn't make any sense.
And then as we continue to discuss it, I go, I look at his name tag and I go, uh, got it?
Umm, this is truly a an amazing experience.
It's more than a conference or an an unconference.
It's an experience.
It is a.
It is a a retreat.
It is a wholesome, just sort of community event.
I think, James, it may have been, it's like a big user group meeting and that is totally true.
It it will get you out of your shell.
No pun intended right out of your shell.
If if if you are very shy.
Jay, Jason, this is this is me, man.
This is how it works.
It will get you out of your shell if you're.
If you're shy, you know no color, no matter the color of the wristband that you were wearing it.
It's going to excite you.
It's gonna confuse you.
It's gonna challenge you and all the best ways, and it gives you the opportunity.
Ohm it gives you the opportunity to really showcase what you've done.
There was a Phil Bossman.
Does the Lightning demo around and everybody says it?
If you're not presenting and you don't think that you've got, you know you've got something to show you.
Do everyone's got something to show?
It doesn't matter.
Big, small, practical or impractical.
Funny or not, you've got something to show.
So whether or not you gave a session or you gave a lightning demo, or you just rotate it around, sort of revolve it around at breakfast and lunch and at the social events and talk to different people the entire time.
Uh, you.
You drink from the fire hose this past week.
It was an exhausting week. I think.
James, you have it on the site, Max out your brain, you Max out your brain at summit.
You go full.
You're a tea totaller at on (PowerShell when it comes to that, so summit is only getting better.
Again, the there you go.
It's only getting better.
Uh, I don't know how it used to be, but in in my time it's only gotten better.
I routinely shown back up to work more invigorated, more excited, more rejuvenated.
It's like I go to a (PowerShell spa.
It is amazing.
So if you didn't get to go to summit, figure it out.
Figure it out next year.
There's some other things out there, right that are that aren't that aren't the same, but they're similar, right?
Like there's there's SQL Saturdays.
There's like James and Chattanooga there's on the river or whatever there is.
I mean, there's other things out there that approximate towards it that trend toward it, but some really the place to be.
And if you can get to coffee you, that'd be amazing.
I don't know that I'm gonna make it, but it it would be.
It'd be really great.
Umm, it's just the best.

Jason Helmick   12:53
He, Matthew was it.
Was it your first time presenting this year?

Matthew Gill   12:57
I presented last year.
I presented the diary system that's gotten 0% love since then because you just get caught up.
I I've got 4 kids.
I was talking about.
I got 4 kids, so keeping my brain on track is you think I'm scattered, you know?
Now, because I'm just me.
No, no, no.
4 kids, right?
So I built this thing and I had showed it off last year.
Just kind of as an aside, to keep track of things at work.
Because who wants to use OneNote right?
And I haven't done that again, but that was the first thing I presented.
And then I did submit Ohh 33 sessions for this one.
But I was also on the content team and all of the sessions that were submitted that got accepted were so much better.
It was.
It was sort of like the IT was the best way to not get selected because I knew it was coming and I got to.
I got to see that like, yes, like it's the session.
Anxiety is super real.
I would say to that the sessions are gonna be recorded.
Go to the ones that you absolutely have to see right now, and then watch the recorded ones.
But the hallway track is invaluable.
We don't record the hallway track, so being able to go to the social lounges at the hallway track, talk to the vendors who you know what they're doing, right, they we didn't get the people that just sell the product to show up at Summit, right.
We got, we've got the, the, the people that actually know what they're doing, they actually are, are programming or coding or you know, you could flip your laptop around and sort of showcase what you're doing.
They comment on that and they're fun to be around.
So umm, I apologize, missing the they're the hallway.
Track was great.
So I mean I I I went to probably 8 sessions or so because I mean you're always missing ones.
You wanna see?
So no, it wasn't my first time presenting a Lightning demo.
I hope I had expanded.
I'm not saying I had material for 45 minutes because I didn't rehearse anything anyway, but probably could have gone for 45 minutes on that on that Lightning demo. So.

Januszko, Missy   14:54
I'm in need of your Lightning demo so.

Jason Helmick   14:54
Of that.

Matthew Gill   14:57
Do you?

Jason Helmick   14:58
Yeah.

Matthew Gill   14:58
Do you?

Jason Helmick   14:58
And I just want to say, Matthew, that.

Matthew Gill   14:58
Do you?
Do you need nursing wear or do you just want the Lightning demo again?

Januszko, Missy   15:04
I I to belong to a clothing cult that does like drops in the middle of the night.
So I Shopify API would be told.

Matthew Gill   15:09
Yeah.
Awesome.
I'm sorry, Jason.
You were sad.

Jason Helmick   15:17
No, I was just saying that the and missing was referring to it and the Lightning demo that you did at this year's summit was outstanding.
So thank you so much.

Matthew Gill   15:26
Appreciate.

Jason Helmick   15:27
And your your enthusiasm is contagious.
And and I just want to shout out Ryan.
Yeah.
Support this out.
Oh yeah, we're already been talking about.
This is giving me all excited for PS comfy use, so this is awesome.
Thank you so much Matthew for sharing.

Matthew Gill   15:41
Awesome.

Jason Helmick   15:42
Please feel free to stick around and jump in also and I I haven't.
Steven, maybe you could help me.
Is Aspen here today?
I'm folks I wanted to also if we could bring someone on from the the (PowerShell summit.
North America also has something called an on ramp, and this is for folks that are earlier in their careers and are getting started with (PowerShell.
And a lot of other things.
I mean, remember the first day you were introduced to get or Mark down and stuff like that.
It can be kind of overwhelming at 1st and so, umm, we we definitely appreciate the fact that Summit runs this on rap session for folks.
And I think we might have someone here from that on RIM that might be able to say a few words.
Aspen.
Are you here today?

Aspen Clark   16:32
Aspen is here.

Jason Helmick   16:33
Awesome.
How you doing?

Aspen Clark   16:36
Good.
How are you?

Jason Helmick   16:38
Awesome. Awesome.

Aspen Clark   16:40
Umm so hi, my name's Aspen.
I work for Western Washington University on our help desk and rose encouraged to attend, honoring up as someone who I does a lot of poking (PowerShell and then a lot of banging my head against a wall and asking people questions until I find out that this thing that I thought that I was really trying to find a complicated answer for already has a built in answer to some module or another.
And so yeah, on ramp was a really great experience, umm, for getting started, it was.
Like drinking from a fire hose, for sure.
And was being hit with a lot of information all at once, but there was all very helpful, particularly for tools that are out there, and even if it was something that I didn't fully grasp onto in the session, knowing it exists, knowing there are people who support it, so things like pester.
Yeah, I also like I hadn't set up a GitHub account.
So it made me do that.
Umm.
And that's already been really helpful.
Learning more about Markdown, there was a session on that.
Umm.
And we are asked like, has anybody heard of or like used markdown?
And I was like, I hadn't.
No.
And as we got into talking about, I was like ohh yes, all the time.
I just did not know that's what it was called in this in this particular setting.
And I also really enjoyed getting to have some sessions on things that are not directly applicable to the work that I do, like Ansible and Docker and containers and just the overall concept of what is DevOps as those are things that surround the world's that I am in but are not things that I often work on directly.
So that context was definitely very helpful.
Umm.
And yeah, just being able to be at a (PowerShell) conference and getting to participate in those hallway chats was very helpful.
Also, I think that the on ramp schedule was the same as the main conference in terms of the timing of breaks and lunches and everything was really amazing to be able to connect with people and find people who work on the things I'm interested in or just listen and absorb as much as I possibly could.
Umm.
And yeah, I also wanted to give a shout out to at that on ramp does scholarships for a handful of folks.
I was a scholarship recipient and would not have been able to go to this conference about that, so that was amazing to just have things taken care of and be able to attend and take a huge step in my career.
Umm, just by spending a week learning and engaging with people with PowerShell.

Jason Helmick   20:00
That's awesome.
Aspen and I admire your courage for going to the on ramp and and and that's a big that's a a lot to take on all at once and what amazing courage coming on and talking about it.
Thank you so much.
I really appreciate it and glad you had a great event for that folks.
I've also noticed so many other friends that got to spend some time with last week, like my buddy Jamie Lewis and and John Janelle, that kind of stuff.
One of the things though that I wanna point out is that we as the PowerShell team really, really were impressed and overwhelmed with feedback.
That's exactly the experience we want.
Is we want you to be able to talk to us and tell us about this pain, this problem.
I love this.
I don't love that.
That was awesome and we got amazing feedback from you and to kind of talk about our perspective on the show.
Steven Bucher is going to say a few words.
Hey, Steven.

Steven Bucher   21:01
Hey Jason, I don't know how I can follow those.
Those those last presenters there because I think they they summed up the set the stomach pretty well.
But I I just wanted to kind of share a another, you know, heartfelt thank you from from the PowerShell team to the community there and the community who is here today, the community engagement is invaluable to us as as the product Drivers and and the team figuring out how to uh make this product better for you.
And and it's something that we equally get excited about our our, you know seeing people using seeing the excitement they have and the impact that (PowerShell has in their own jobs.
And so it is such an amazing honor to be able to spend time with the people who are in the front lines, so to speak, with (PowerShell and DevOps community.
And it's been just a wonderful, wonderful time.
We did a number of presentations on on the stuff that we working on.
So we'll those will be available on the recordings and stuff.
But yeah, my to keep it short and simple, but just wanted to big big.
Thank you.
And and big thank you to the community.
That's all I'm trying to say I guess.

Jason Helmick   22:15
So with that, please feel free to.

Steven Bucher   22:17
Ohh and shout out to Matthew's 3D printed thing.

Jason Helmick   22:20
Ah yeah.

Steven Bucher   22:21
I forgot I had this next to me, but there there's some 3D printed.
There's so many 3D printed fun little arts, 3D printed (PowerShell logo stuff that was going around this summit, so hopefully the the the STL's will get you know populated in the community channels so.

Matthew Gill   22:38
I was talking with James about that yesterday about how we're just terrible at uploading things that we remix and we build.
So that's it's happening in a promise.

Jason Helmick   22:47
Oh, that's awesome.
Pineal.
I saw you.
I saw you with your hand.
Did you?
Do you have something you wanted to say real quick?

Neal, Kenneth   22:54
Oh no, I was failing to show off my (PowerShell logo.

Jason Helmick   22:58
Oh, awesome. That's what.
That's great.
Well, please feel free to to to have more comments and conversation about getting together in the community like we did last week and I just wanna remind everybody that soon and soon to be coming up is the (PowerShell) conference in Europe and the EU.
So very excited about that as well.
And what I'd just like to do for one quick second is all of the folks here that have spoken today and all of the organizers and the hard work that they've done, just a round of applause to y'all because deep appreciation for everything that you've done this time.
So that's awesome.
Now back to business.
So a couple of more items for this community call so ohh this is gonna be a good one.
Steve, you wanna talk about a DSC V3 update?

Steve Lee (POWERSHELL HE/HIM)   23:51
Absolutely.
I'm not doing a demo today, but kind of give a preannouncement.
This will probably happen next week.
We're planning on getting our next release of DSC V3 out, so the big news here is we're shifting from an alpha phase to a beta phase.
So it will be Preview 7.
Umm.
And some of you who've been following may be wondering, hey, we had A5, we have now a previous seven coming out.
What happened to six?
So the short answer is that I've been.
I was testing publishing an MSI X package of DCV 3 to the Microsoft Store, and because I did that as a version 6, the store requires you to increment the version.
So I'm kind of used up that version number, so we're skipping the six and gonna 7.
So that also means that one of the things that you'll hopefully see next week is that you'll get DCB 3 for Windows in the Microsoft Store.
You can install it that way, and we'll also have preview and stable channels.
Obviously there won't be a stable release until we get to GA of 3.0, but we'll have previews coming out, hopefully around every month or so.
And after that I do have a whole session on DSC V3 at psconf.eu for those who will be there.
I'll probably since that's in June, I might also start having some other content, maybe by next community call on DCV 3 itself.
There's been a lot of changes on both the configuration side and also for resource authoring, so there's a lot of content to cover.
And I know Mike, he's been shifting also from reference documentation to more like tutorials.
Hopefully all that will come out to run the same time and hopefully we can see more any Members participating, giving us feedback before we really lock down the design.
That's pretty much it.

Jason Helmick   25:31
That's a that's that's outstanding and and and for folks that weren't there, there was a super amount of interest last week in DSC.
This is a great time to follow along what's going on on the repo and to start getting into the conversations, because this is the time when it matters the most.
So as we're making that shift to beta, this is awesome.
Talk to Steve.
Talk to the team.
Talk to Mikey.
This is a great opportunity for configuration management.
Thank you, Steve.
And with that, we have in something about ACR Support II at the moment.
Is it Azure container registry?
There are so many TLA's floating around the day that I'm getting a little bit slow, but Amber, maybe you can tell us what this is about.

Amber Erickson   26:13
Yeah.
Yes it is.
It is Azure container registry.
Uh, so hi everyone.
I'm Amber of for those of you who don't know, I'm on the PowerShell team.
I work on PS resource get and I'm actually just gonna do a quick demo today on our latest release of PS Resource get.
It's 110 preview one and this is a fun release because we do now have support for Private ACR instances, and that's kind of fun because you can make your own private gallery in ACR.
We have this here.
This is just a test repository for PS Resource.
Get down here in repositories.
This is kind of an overloaded term, so in the ACR sense, repositories is actually going to be where our modules are located.
I can see here this is just a random AZ module.
We have our module names, our module versions here, and I'm going to do a quick demo on how PS Resource get now interacts with this.
Umm, so I have my ACR repo registered already.
We have this you RI, which is actually just in the overview panel here.
It's the login server URL.
So that's actually we're going to be using for the Uri.
You can see here.
Now we have this new API version, container registry.
You don't need to worry about that.
PS Resource get just automatically picks that up and registers that, but just something to know.
So now we can just publish.
This is just a really simple test module we're going to publish this to ACR.
If that's finished, still working on it.
Take it little bit of time.
So if we search for this package might need to refresh.
There we go.
So we have our community called Demo.
We've got this version, nothing there before, but you can see that.
Here it's 957 just got published.
This is what the manifesto looks like.
So we're actually using OCI artifacts, but publishing the nut keg, so that's actually going to be what's stored.
Our metadata is right here, so not very human readable, but it is PS resource get readable and that's kind of all that matters and this is again just the manifest, all the information there.
If we go back to resource get, we can do a find, uh, PS resource.
We'll do community call demo on just to make this go a little faster, I'm going to specify the repository.
And then we've got our module there.
We can do a quick install as well.
Yeah, that if we do a get and stalled.
Yes, resource.
We've got that there and then actually.
Just print out the entire object as to see everything so properly getting all of the metadata that's stored in the manifest, parsing it out into the object and effectively acts the same way as it would with any other repository like the PSGallery any ADO repositories?
This.
So that is that.
One other thing that I wanted to mention I think is kind of cool.
There's this cache section here so you can actually create rules to pull from upstream, like external repositories, which is kind of neat, and that's pretty much it for the private ACR registries.
We are also working in PS resource get to provide support for public ACR registries.
Another big one is mayor, which is Microsoft Artifact Registry, which is effectively Microsoft Public ACR instance.
Further down the road, we're also planning on parting, providing support for external container registries like Amazons Elastic Container registry.
So be on the lookout for that as well.
If this interests you at all, please check it out.
Let us know how it's working for you.
If there's anything that you want to see Support for, if there's anything you don't like.
Just let us know we are not necessarily dumping all of our efforts into the container registry work.
We're still working on other bugs and issues and PS resource gap, but we are really excited about this work, so we'd love to get any feedback and just love to hear from you guys about it.
So that's about it.
I will pass it back to you, Jason.

Jason Helmick   31:49
Things, Albert, that is awesome.
That's great.
Congratulations.
That's amazing.

Amber Erickson   31:55
Yeah.

Jason Helmick   31:57
It and so, you know, almost every compute.
Well, no.
Every community call we start to round things out with an update from my friend Sean Wheeler on Doc.
So, Sean, don't you jump in and say hello, by the way, thanks for last week.
Telling me it was great to have you around.

Sean Wheeler   32:13
It was a wonderful time.
I had a blast and made a lot of new friends and I see a lot of names in this meeting that I recognized from Summit last week, so great to see everyone.
So just to pile on to what's already been talked about, lots of updates coming in DSC V3, Mike, he's been cranking away on the documentation.
As always, check the change log here.
You get to this from the (PowerShell docs.
You can go to the DSC page here and switch to version three and you'll see all the documentation for resource kit.
Uh, we've updated the article here about supported repositories to talk about the Azure Container Registry and under how to here we have more detailed information about working with container registries.
Umm over in Azure PowerShell there's a lot going on.
Check out the what's new?
Want to point out a a few things?
There's some breaking changes coming.
Umm, one of them is security related.
Umm, there's a feature for a security warning where Azure PowerShell and Azure CLI.
Well, display warning message if.
There's a secret being disclosed in the output.
That's going to be turned on by default.
It with the 12.0 release of Azure PowerShell, but that is configurable.
You can turn that off.
There is an article here about protecting your secrets and that goes into this.
Functionality and tells you how to turn that off.
Uh.
And then the other new article is something that.
There's been a lot of requests for doing an offline installation of Azure PowerShell.
On the various platforms, so step by step how to download the package, extract it and get it installed.
When your target machine is not, uh Internet connected.
So take a look at that if you need to and continue with the Security theme.
I've reorganized our Security content in the PowerShell documentation so that it's all in one place now.
So we had these these articles about the security features and using Application Control in one area and then preventing scripting, script injection attacks and restricted remoting sessions.
We're in another place and then all of our PowerShell remoting stuff is very Security united.
So now you can find it all in one place for easier access, and we'll be adding to this content.
As things go on and that's my docs update.

Jason Helmick   35:41
Outstanding.
Thank you, Sir.
That's amazing.
Yeah.
And it's Sean's kind of pointing out not that you need something else on your list, but now is a really good time to focus on your Security.
Oh, I see.
Steven has popped up.
What's up bro?

Steven Bucher   35:55
Hey, I just wanted to before we get into the demos and stuff for the community, I wanted to do 2 things first.
Not to put Ryan on the spot, but Ryan, did.
You wanna give a little bit of a shout out to Pierce?
Come for you to here.

Ryan Yates   36:13
Yeah, it's a piece called for.
You obviously has been running now since 2016.
The the previously was organized by the same team that run the (PowerShell and DevOps I met.
I'm Jason.
I think you remember those years.
Many, many years ago, and they were overtaken, overtaken with the bias weltner and and now GAIL Colas.

Jason Helmick   36:29
Yep, there you do.

Ryan Yates   36:34
Alexander Nicolich and Rob Sewell.
And and various others from the community.
You know, a lot of people behind the scenes and sponsors and speakers from all over the client and it's it's yourope premier event.
It's it's one that I'm very proud to have helped out in in the past, but I'm very, very proud that it's still going very strong.
I I think it's gonna be a lot of fun for those that can attend, and I'm I would like to be able to attend this year, but I don't think that's gonna be possible unfortunately.
So best of luck to everybody going and I know you will have fun because it's a wild, organized event, just like the US summit is as well.

Jason Helmick   37:13
Yeah.
And I've had a chance to talk with many of the organizers.
For the European summit, GAIL Collins and Ryan Yates and a lot of folks that help to build that, it is an amazing conference.
So definitely huge, shout out to them and their hard work and looking forward to it in a big way.
So you said you had two things.
What else did you have present it.

Steven Bucher   37:36
Yeah, and no.
I I do have another thing, but I'll say also to the psconf.eu piece before we move completely move on for that.
We're very excited to be going myself, Damien, Steve, Amber and then we have representation from the machine config and the Azure arc team coming as well as winget.
And then I might be missing one more.
I'm probably missing one more, but that's the general representation.
We will have a PS confuse who are very excited to be having presentations and bringing participate.
Participating in that conference again this year.
So the second thing I kind of want to talk about just cause I it's been awesome seeing the the chat going on in the all the buzz going on in the chat with kind of stuff around converting and asking all sorts of these fun questions around different modules and stuff.
One thing I talked about at Summit to folks is that these Community calls we really like to have them be a very open space for demoing and basically an equivalent to kind of the Lightning demos that you had at Summit.
So if you went to summit, you're familiar with the lighting demos.
But basically we wanna use this platform here for anyone to share anything that they're working on with (PowerShell.
If it's just something little that that's really helped out there work, or if it's something they've been working on for a long time.
Uh, anyone is welcome to to share, you know, the little scripts that they're making.
I mean it's it's it was really the Lightning Devils are widely successful at the summit conferences and we understand it's daunting to present to a large group.
But you know, I was gonna say 9 times out of 10, but I feel like it's almost 10 times out of 10 every session, every lightning demo.
There's someone else who's like?
Ohh yeah, that's that's something I need.
Or that's something that can be useful, right?
So I just kind of wanted to talk a little bit briefly about that.
So we can.
You're welcome to prepopulate the agenda around that, or if it's something that's coming up in the chat right now, you're more than welcome to just kind of hop on and keep it brief.
5 minute demo.
But yeah, I think that's kind of all I had there.

Jason Helmick   40:02
That's awesome.

Steven Bucher   40:03
I let me check the I think right now I know James uh, you want to maybe take take 5 minutes to talk about your some Docker and (PowerShell stuff do you want?

James Brundage   40:16
Yeah.
If we got the five minutes, it also seems kind of on topic with where we've just been, you know, showing stuff.
So do we want to?

Jason Helmick   40:25
Go for it.

James Brundage   40:26
All right.
Well, uh ohh.

Jason Helmick   40:27
We'll wrap it up with you, buddy.

James Brundage   40:30
I don't know about that pressure.
Let's see if I can actually screen share.
Looks like not quite yet.

Steven Bucher   40:35
Ohh, hang on, give us a second.

James Brundage   40:36
That's OK you can all see it with blurry text, right?
If I turn on my camera.
So I gotta say, like the best one liner is still on the screen and I'm not gonna get rid of it now.
Uh, yesterday I was getting ready for my condo meeting and I realized I'd had a little bit too much fun with Docker cause my box was at 90% memory.
I got 64 gigs of memory, so 90% memory.

Steven Bucher   41:05
James.

James Brundage   41:05
I'm really hurting it.
Go ahead is already now.

Steven Bucher   41:09
Sorry, I'm just like I we can't see it.
I'm still trying to find your name so I can.

James Brundage   41:14
I got it.
I got it.

Steven Bucher   41:15
Ohh you got it.

James Brundage   41:15
I think maybe.

Steven Bucher   41:17
OK, cool.
Thank you.
Whomever else helped with that?

James Brundage   41:18
Alright, it's it's working.

Steve Lee (POWERSHELL HE/HIM)   41:19
It's working now.

Steven Bucher   41:19
There we go.
That's a lot better.

James Brundage   41:22
Cool.
We can all see it.
OK, so yeah, yesterday I realized it was absolutely killing my box with Docker and I had a meeting starting in a minute and, you know, luckily I've got this whole PowerShell module that let me run this evil one liner.
Docker container LS pipe to Docker container stop now.
Right now I'm not running any Docker containers, so let's go Docker run interactive.
OK, I'm not gonna not going to pick that one.
That one's too cool.
Let's go ahead and go for you.
Get I don't need privileged either.
Yeah, let's do it.
G HCRP way, because that ones on you get.
We're on.
You HDR can't find an image locally, it's gonna go grab it.
And if you're not already familiar with Docker, Docker is a container azatian platform.
It's basically letting you run code within a box.
This is really great for (PowerShell because we get to get out of the code injection fear because if you're running it in a box with nothing else on it, what's there to pone?
So here I am inside of this box here and I can get module you get is already loaded and I'm just gonna go ahead and get clone.
Yethub.com, PowerShell, PowerShell.
I'm going to show off a little bit.
You get functionality in the second.
I'm actually say Git clone nothing and tag completion should be my friend eventually.
That did a really fast clone of sparse checkout.
So if I go into (PowerShell here, it actually still has nothing.
I could do more, get demonstrations, but we're on rocker here.
Not get.
Let's go out of our instance here for a second.
So one thing that can be annoying about Docker is well, just some of how you pass PowerShell commands to it.
So one of the things that rocker does is it allows you to pass a script block.
I might as well show my loaded modules in there too.
So here is a box running nothing but basically core modules and you get fantastic love it.
Uh, it gets a little bit more interesting if we go ahead and push it to a branch build.
So let's be really brave for a second here and let me go grab another link.
What I've been doing is I've been bringing every bill that I'm doing into Docker, so if you actually look at any of the projects that have this, they will already have packages, whether they're fully out the door or not.
We're just publishing a branch build.
I'm doing a lot of Webby related Dockery related features in the next to pipe script, so let's go ahead and try to demonstrate that and a cool rocker feature at the same time before we get too far ahead of ourselves though, let's do some basics tab completion.
Totally a thing.
As I go pre read all of the Docker help so that you can tap through it.
So no more.
What the hell is that? Switch again?
Use (PowerShell.
Use docker.
Use rocker.
It's great, but we get really fun when we start to do something like, well, if I just ran this, I wouldn't have its web server available to me because it's not published.
If I wanted to do this old school, I could do something like P 1180 or publish 1180 or I can use some (PowerShell.
OK, so I'm gonna try to publish this container on port 1001.
OK, it's running my job.
I'm going to go ahead and local host 1001.
It yelled at me because I used HTTPS one second and pop that over.
And control C.
Hey now I'm in my instance.
Look still running.
And exit my instance.
And it's gone now.
And it gets even better because I can just go ahead and really easily.
Just Gen tab completion for the win.
Detach not to tach keys.
Just attach.
Alright, and giving it another SEC and we should be back up page isn't working right now because it's taking a little second to come up.
Sorry there, but I got one more for the road for you.
One of the other things that's a big pain in the **** in Docker is a mounting directories and one of the things we have also been talking about on this call is well working with 3D objects.
OK.
So we're gonna go ahead and go to one more branch, build here, and we're going to check out PS3 D.
And it's going to go and say, hey, I need to give that update two places.
There we go.
Copy that.
So we're going to go run interactive.
If you Y and.
We're also going to go.
Say current directory equals mount.
Everybody have a pretty good idea of what this is gonna do, right?
Easier than remembering the Docker or command line format, right?
Alright, so if I der into mount.
I should have probably done this in the directory for PS3D for maximum effect, but we'll go there in a second.
Let's go ahead and say.
Ohh.
Invoke open scad test dot scad.
Now notice I'm not having to install Openscad.
That's already on the Docker image, so I've got a Docker image here with open scad (PowerShell slicer and the nice library.
Or two completely isolated box turns (PowerShell into 3D.
Another completely isolated box turns (PowerShell into a little you get box another completely isolated box turned into a MIDI web server.
Are we all getting where Containers could be really awesome for (PowerShell?
Cool.
So I am really excited overall about getting my feet even more wet with Docker.
I'm also excited about this little you get for Docker project Rocker, which is what I've been demonstrating today.
Go ahead and install an import.
There are some examples help make this module a little bit better.
It is Young that, umm, I'm also really really really really looking forward to what will happen once we fully containerize every single module that exists, because that means we can put any module that exists safely in its own isolated web service, and it could play with all the other ones completely securely.
Because again, if you happen to manage to code, inject.
Congratulations, you've code injected into a box.
You have code injected into a cage.
You are not escaping from that, so I am absolutely loving Docker and (PowerShell.
I think that it rocks.
I think the future here is bright and I haven't had as productive of a week after summit that I can remember because it's just been like it's finally clicked.
I'm getting deeper into Docker and I'm dockerizing everything and I want you to all join me.
Make your modules running containers.
Make them little mini services and let's see if we can take over the freaking world.

Jason Helmick   50:19
That's awesome, James.
Thank you so much.

James Brundage   50:20
Mike, drop.

Jason Helmick   50:21
And and I have to say I I I'm a huge container fan of specially of of creating work Containers for secured workspaces.
I do this all day long.
It very interesting direction you have go in there.
So thank you so much.
Umm.
And I think I've been checking to see if we had any questions or additional things.
So we are gonna end this show just a tiny bit early this month.
I want to thank everyone for allowing us to talk about our experiences at the summit.
What a great experience it was to be together and to share information and from our perspective, what a great time for us to get feedback.
So if you're asking yourself the question.
Does my idea matter and do I have anything to share?
The answer is yes and yes.
So please come to the repo, come to the community, call, come to a conference, talk to us.
Email U.S.
Chat with us.
Whatever you'd like to do in order to convey your ideas.
We wanna hear them.
So thanks everyone.
It's an honor to be in the community with you and I hope you have a wonderful month and we'll see you next month for more (PowerShell goodness.
Cheers, everybody.

James Brundage   51:32
Thank you.

Matthew Gill   51:33
Thanks everyone.

Januszko, Missy   51:35
Thanks.

Jason Helmick stopped transcription
