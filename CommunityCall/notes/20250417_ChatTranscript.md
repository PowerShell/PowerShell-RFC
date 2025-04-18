# PowerShell_OpenSSH Community Call-20250320

Transcript
April 17, 2025, 4:29PM

Sydney Smith (She/Her) started transcription

Sarah Hakim   0:59
Alright, it is time to start.
So welcome everyone to our April 2025 PowerShell Community call.
I'm Sarah and I'll be hosting today.
To start, I want to give a reminder that this meeting is recorded and recordings are uploaded to our YouTube channel after the call.
So if you do not want to be included in the recording, feel free to drop now.
Please also do remember throughout the meeting, our code of conduct, which should be LinkedIn the chat.
And we ask everyone to be respectful and mindful.
Let's go ahead and get started and I'll hand it off to Sydney for some updates.

Sydney Smith (She/Her)   1:43
Hello, my name is Sydney and I'm one of the PM's on the PowerShell team and I have a few updates I wanted to share today.
So the first one all pasted in the chat, but I wanted to make sure folks were aware that we have been shipping previews of seven, six. We recently shipped preview four with a bunch of community PRS in it.
I want to remind folks, too that.
76 is an LTS release and the earlier in the cycle you can test things out.
And find potential issues for us, the better we plan to GA this come usually it's around November. So the sooner you can tell us what we need to fix is certainly the better.
So check out.
PowerShell 76 Preview 4.
My second update I saw there was a question about this in the GitHub issue and I know there's been a lot of confusion, so I wanted to make sure folks were aware of some changes that are happening with our Docker image.
Is let me just.
Grab the links.
To the GitHub announcement in the docs about this.
The gist of this change is that after years of the PowerShell team maintaining PowerShell Docker images for testing and development moving forward, the Docker images will be produced bythe.net team. We decided to make this change to enable faster image availability. The net team releases on.
A cadence faster than we have been able to keep up with over the past couple of years.
And this will also allow for more consistency across the ecosystem. Sincethe.net team produced the same platform distributions that we have in the past and they even produce a few more than we did previously.
So if you have any questions about this, definitely check out the docs and the GitHub announcement, and if you're having any issues, just go to the GitHub announcement and.
Tag Jason and a comment as he's helping with this experience.
And making it as smooth as possible for all of you.
Great.
My third announcement is the maybe the least fun one, but I did want to address.
I think we've maybe addressed this in past Community calls, but there is an unfortunate issue that we've been seeing intermittently on the PowerShell gallery.
With search results not returning, unfortunately, we have had a difficult time resolving this issue.
It's not a simple fix.
We have a lot more logging in place now.
Now in monitors set up to notify us when this is happening and we are still debugging how we can best fix this. If you've read Steve's yearly investments blog or maybe we've talked about this on previous calls, the PowerShell Gallery is also undergoing some big infrastructure changes.
In order to move on to ask.
So we're hoping along with these changes, we'll be able to resolve some of these search performance issues going on with the gallery as well. But we hope that the infrastructure change is a smooth change behind the scenes and you shouldn't have to worry too much about this, I.
Also see a question in the chat. Any ETLA on namespaces for gallery this.
Is not.
Namespaces are not a committed feature for us right now, so we don't have any.
Planned work on name spaces at the moment we've been focusing more on enabling trusted galleries through container registries.
So with that, I will pass things back to seraf.

Sarah Hakim   5:39
Thanks, Sidney.
Exciting updates.
Let's go ahead and shift gears to some doc updates with Sean.

Sean Wheeler   5:49
Hi.
Not much to talk about this month.
Did do an update to the documentation for the.
76 preview releases and.
The other releases we had most of the docs work has been happening over in Dsci know Mikey's put in a lot of work there and did some rearranging of the content.
Mikey, can you give us an update about the DSC content and where you're headed with it?

Mikey Lombardi   6:30
Yeah, absolutely.
So we've.
I did a overhaul of the TOC so things should be a lot easier to find now.
And updated all of the content for the V3 Georgia.
Started expanding on the reference content. Adding in conceptual documentation.
And upcoming is schema and schema docs updates.
Those should be landing.
Soon, probably next week.
And then expanding the reference content for the built in resources and updating the tutorials, there are plans we have an existing tutorial for writing a resource and go there are currently plans for doing it in C# rust and then eventually Python And I'm going to adapt the.
Existing PowerShell.
Example to the same sort of structure and paradigm and all that'll live in the DSC samples repo and site.
And just a lot of upcoming small tweaks and fixes and expansions to the docs as we go. And I think there's probably on the order of about 30 new docs.
With more coming.

Sean Wheeler   7:46
All right.
Thanks, Mikey, and I'll hand it back to you, Sarah.

Sarah Hakim   7:51
Thanks Sean and Mikey.
Exciting to hear how many new dogs will be added and now let's hear about the PowerShell Summit, which just wrapped up in Bellevue last week from James.
Alright. James, are you here by any chance?

Sydney Smith (She/Her)   8:26
I'm just looking at the participants list and I'm not seeing James, so he may not have been able to join the call.
I'm happy to to give a quick recap of summit and then Sarah, maybe you can share some of your experiences too, because I know you were able to attend.

Sarah Hakim   8:36
No worries.
Do.

Sydney Smith (She/Her)   8:46
So, as many of you are probably aware, we had PowerShell Summit last week in Bellevue and many folks from the PowerShell team were very lucky to be able to attend, including myself.
As always, it was great to see so many of you connect with the community here about the things that you have all been working on and the successes and issues you're hitting along the way.
We also had the bicep team and the Terraform team.
Come for the first time to summit, so that was pretty cool to see just the embrace that you all gave those new product areas. I think that's all I have. But Sarah, do you want to share a little bit about your experience?

Sarah Hakim   9:31
Definitely this was my first PowerShell summit, so it was definitely awesome meeting everyone and hearing everyone's scenarios for PowerShell and also sitting into the presentations where we're not only learning a lot but also having a hands on experience.
So definitely great experience and excited for the ones that are coming up.
So I'll go ahead and pass it on to Steven to talk about what.
The upcoming conferences are and what's coming up.

Steven Bucher   10:06
Yeah, I just was thrown in some fun pictures. If anyone has any fun pictures or anything from summit they wanna share. Like please do. So I I feel like we love. We love seeing all the.
It was.
It was a lot of fun last week.
Summit, I'll share my recap and then I'll talk a little bit about what upcoming conferences that you can find us at.
But it was partial. Summit last week was awesome as usual.
It was a great time to reconnect with community members in person in Bellevue here.
There's lots of great sessions like Cindy mentioned, we can't talked about.
Our team, Microsoft, brought in folks from the Bicep Terraform. Of course, the PowerShell team was there too, and so we had a great time sharing what we have been built out and also had just great discussions. A lot of our, a lot of the stuff we like to.
Show at these conferences are kind of future looking things as well. Looking at things like that.
We're trying to, you know, figure out over this next year that we love to have kind of more in depth discussions on so.
Definitely check out the recordings on YouTube once they're published.
I don't know when they're published.
I just thought there was a question in the chat about that.
I think typically they're like two to three weeks after the conference.
I don't.
You know, don't quote me on it, but I know they'll probably be out sooner, but they're there surely be out.
Relatively relatively soon so.
Keep an eye out for them. I have a whole list of sessions that I missed because there was other sessions that conflicted on that. I've got to go back and.
And watch.
So definitely check it out.
Once they're published.
Let me think.
Yeah, I think that was kind of a bit on on summit.
Geoffrey Snover was there.
Jim Ture, who's also on the call here I see. Was there talking about, you know, PowerShell and some of the?
Awesome impact that you can make as an individual contributor.
So that's definitely a session you wanna check out.
I was there talking about AI Shell and AI agents. I joined forces with.
Mike Nelson in the community and we talked about AI agents and kinda what the heck they are.
We can use them, so definitely check out those sessions once we get them published, but always a great time as usual. I think that's kind of all I wanted to share so.
Yeah, as far as new conferences coming up, there's a couple new. There's a couple conferences that we'll have a presence at, the first one coming up is MMS, which is the Midwest management summit.
This one we haven't previously gone to in the past, but I will be attending the MMS conference in a few weeks here.
I'll be helping share some details on on Azure Ark and PowerShell.
Of course, I don't have a session on PowerShell, but I will be attending a bunch of other there's a load of PowerShell sessions there, so definitely check out the.
Schedule to find out anything if you are happen to be attending that conference.
We'll also have some minor presence at Microsoft build this year.
We'll be talking mainly about our copilot experiences helping that we've been developing.
For Azure PowerShell and Azure CLI and so I'll be there as well there. And then of course PS Confu is coming up in June.
I think there's still tickets available.
I should double check that before I say that, but PowerShell conference EU is the sister conference to the PowerShell Summit that just happened last week.
However, it's held in Europe, it's gonna be in Malmo, Sweden this year.
Definitely check it out.
And we'll be sharing some of the same content that we shared at Summit there, but maybe a little tweak since it's a month or two later.
But of course there's fantastic PowerShell DSC and 365. I mean, I'm just reading off like a bunch of the the titles I see.
We'll be there talking about SSH stuff too, which we're very excited about.
Fabric automated labs excuse me?
All sorts of really cool stuff.
So definitely be sure to check that out.
And yeah, I think that's.
All I have on conferences, I don't think I'm missing any.
But yeah, it we're we're busy with conferences this season, I guess is the is the main take away. So be sure you know if you see us at any of these conferences feel free to to say hi and share your thoughts.
Oh yeah, I'll pass it back to you, Sarah.

Sarah Hakim   14:47
Thanks Steven.
Exciting to hear what's coming up and also seeing all these pictures in the chat.
It's exciting to see that everyone had a great time, so thanks for sharing.
And yeah, go ahead.

Speaker 1   14:57
Can I add something, Sarah?
Well, I thank you to everybody who showed up for the PowerShell Summit and I think one of the engagements for PowerShell Summit certainly.
Compu is the community engagement and one of the best parts of PowerShell Summit is when you go to another conference.
Is that the Lightning demos and really get engage with the community and as shown in the pictures and all of that is that that is one of those unique conferences that you know you can engage with the Community, you can participate in no matter what your your.
Level is you can participate, and that's what this is, you know, even the community call.
We certainly want to engage the Community and do demos here, but that's also a great place to do and thank you to the community.
So that's I wanted to say.

Sarah Hakim   15:48
Thanks for sharing, Phil.
And let's finish up with a demo from Ryan. If you're still up to share.

Ryan Yates   15:58
Yep, just give me a second and I'll get it sorted.
As usual, team just slows down.
But there we go.
Alright. Can you see my screen and is it OK for you right now?

Sarah Hakim   16:15
Yep, we can see it.

Ryan Yates   16:17
Right. So to begin with, I'm gonna talk about something that isn't to do with the project I want to share, which is about my PowerShell repo around my PS project. That is the.
Profile and how I've changed it over the years, but around something that's useful for this. This sort of group and things that we do with software development and that's actually not just the software development thing, but a general psychology thing.
In that we create initial interaction notes. As a mental note.
So I've just published a couple of repositories and let me see if I can just share the links directly in the chat to you that you can go and have a read through some of these.
My profile project isn't published just yet, but it will be later on this evening.
So let me just jump back.
So this is a whole set of different notes.
This is something I'm using for.
My research and there's a lot in this repository, so I won't spend a lot of time going through what the initial interaction notes are useful for, but go and have a look through this because this is going to be something that I'm going to be using as I.
Say in future for any PowerShell projectsany.net projects. Anything that I work for work through as a as a consultant, that sort of stuff.
And for other research purposes, I've got a public repository which I'm actually in the process of doing for.
Go in and detailing stuff like, well, what have I seen recently like the Scott and Mark learn to series where I'm going to publish again these initial interaction notes to that.
That's the second link that I posted in the chat, and as that's enough to speak about those areas because you can go and read all of that to your heart's content.
Let's go and have a look at the PS profile. So as you can see, I've got quite a lot of different notes, different documents in here.
I want to talk a little bit about architecture decision records which.
They they're not just called architecture decision records anymore. They've been renamed to be in any decision record and these are used predominantly within software architectural development for the process of what are you going to be using within the architecture of your solution, whether that be I'll be using.
Net as a language I'll be using go as a language. I'll be using Azure as a platform.
Are we going to have a particular cicd tool that we're going to be using?
And this is where you enable people coming to your project to be able to see why.
Have you made changes in those over the years, which is very, very useful for getting people started up on a new project? So as you can see, I've gone and added a couple in the in here.
That's an interesting.
OK.
Well, you can go and read why have I picked all sorts of stuff in this?
It is a lot more docs, but it's quite useful if you can manage to build a template to do this and find automation to do it based off of conversations that you've had in meetings, which is where I think it will be beneficial.
And then let's have a look at actually what the the whole layout of my repository is.
So within the PS profile repository I've got a source folder which contains all sorts of different files.
My original profile was a singular massive PS1 file that I had since 2013.
So it's had a lot of iterations.
It was getting a bit painful to manage, particularly as I had different version settings, different host settings, different machine settings, and all sorts of stuff like that. So I've split it out a little bit.
Mainly because I make use of a core profile.
Hello.
And various things I bring into it. As you can see, lots of different areas with this.
And then I have as part of the testing that I've been doing for many, many years with the PowerShell repository, I spin up a new instance with a very minimal profile.
So if you have a look here, this is what it looks like if I start up a full environment with a full profile, I've got lots of stuff.
Let's just do it the other way.
As you can see, if I use Windows Terminal, I can either set. Do I want a no profile?
Do I want a minimal profile or do I want to just start a new version?
So if I start a minimal profile, this is what I feel is the things that make my experience really really nice and useful very quickly. As you can see, that's a nice quick.
Coming into the session, but if I was to do a full session, this one's a bit more takes a bit more time to do so.
So this is as I say, just something I wanted to share and I'll get that repository published in the next couple of minutes.
So as you can see, that's far too long a profile to take in.
So that's everything I'm up to show.
Any questions at all?

Sarah Hakim   22:10
We have a question from Phil.

Ryan Yates   22:13
Yep.

Sarah Hakim   22:14
Or.

Speaker 1   22:15
No, sorry. I was trying to clap and I was raising my hand.

Ryan Yates   22:16
Hi.

Speaker 1   22:18
But great job, Ryan. And it's always great to see, you know, people's profiles and actually, you know, when actually at somebody had mentioned, hey, what if we even do an entire session come and share your profile.
That'd be an interesting Lightning demo session kind of world of everybody.
Just show it.
Show what you got.
And so that's kind of been interesting. Fun exercise.

Ryan Yates   22:38
Yeah, but one of the reasons for doing this and having the minimal profile as opposed to just doing no profile is there's a lot of things that we can change going forward within stuff like PS style to make the experience a bit nicer. But that would be a.
Configurable item that you would want to do as an individual with your session as opposed to something that we would want to do within the interactive user Experience Group or the engine groups to make it.
A.
This is the way to go.
Forward, if that makes sense.
Particularly as a lot of people are making use of, Oh my posh for their prompt, whereas I don't. I particularly make use of as much and as little as possible. Is just on a box as opposed to stuff that's in 3rd party installations just in case I have.
To go to one of those environments where they say, hey, hey, no, you can't use this.

Speaker 1   23:36
Yeah. It's certainly interesting to understand, you know, the optimization that you can provide and you know what?
What's it give and take is what you want for performance in that point, so it's good to see. Thanks again.

Ryan Yates   23:48
Yeah. And I I set a lot of things like PS, PS drives and what not. And that over the years, especially if it's in a a big enterprise environment with lots of active directories and slow networks, that will slow your system down.
You just do not want that, but that repo once again it published.
Go and have a read through it because there's a lot of things to think about.
I that was inspired and where I've learned things over the years.
Including Steve's optimizer profile, blog post.
Which hasn't been updated in a while, but I think it's pretty much still as he as he runs it.
And yes, Mike, just as you've asked it is private.
I'm going to go and click the buttons to make it public in just a minute.

Sarah Hakim   24:37
All right. Seems like we've covered the questions.
Thank you Ryan for sharing your demo.
And I think it's time for us to wrap up.
So thank you everyone for attending and continuing to contribute. And just to remind everyone, we have these community calls the 3rd Thursday of every month.
So we hope to see you in May. Thank you everyone.

Sydney Smith (She/Her)   25:07
Thanks Sarah.

Sydney Smith (She/Her) stopped transcription
