# PowerShell_OpenSSH Community Call-20240620_092938-Meeting Recording

June 20, 2024
32m 56s

Sydney Smith (SHE/HER)   0:41
Hello everyone.
Welcome to our June community call.
We will get started in just a minute here.
If folks another minute to join.
Alrighty ohh I see that we we've got quite a few folks in here now.
So we can go ahead and get started.
Thank you so much to Jason for posting some of our reminders that we have at the beginning of every community call.
First of all, this meeting is recorded.
If you do not want to participate in the recording, please go ahead and leave the call now.
The recording will be posted to our community.
Call YouTube Channel which is linked.
Here we try and post these calls as soon as we can, but we kind of guarantee you within a two week time span usually happens much sooner than that.
Please also remember that we take our code of conduct very seriously.
If you aren't familiar with the code of conduct, it's linked there.
But really, the key point is, you know, treat each other with kindness and all those sorts of things.
We also do have a discussion linked.
It's in the PowerShell PowerShell repo and the discussions portion we posted discussion before each community call with the agenda and this is a place where you can post an advance of the call and you topics you want to discuss.
Any demos you might want to have any questions you have anything like that?
Umm.
And also during the call you can continue to post there as well if you'd like to kind of interactively participate in the call that it's totally welcome.
Please feel free to use the chat or use the raise hand feature if you have any questions, comments, feedback throughout.
With that, we can go ahead and get started with the agenda for today.
First up, we have Damien.
Who's gonna be talking about PS come for you, which is happening next week?
 
Damien Caro   2:54
Hey, yes.
Sorry, I'm just joining.
 
Sydney Smith (SHE/HER)   2:58
Yeah, no worries.
 
Damien Caro   3:00
So, yeah, next week I'll be in invert in Belgium.
Lovely Belgium with few Forsman team.
I'll get to that in a minute to talk about.
I'm a few things that we've been working on actually pretty much everything we've been working on in the last year, if you haven't looked at the agenda and if you're coming to PS Configu, which I would very welcome talking with each of you have some spending some time chatting about different areas.
But we have seven sessions if I count correctly around AI in the shell, we're going to talk about container in galleries.
We're going to talk about the CV3 machine config winget Azure client tool, Azure CLI, Azure PowerShell, and we're also going to talk about Azure Arc and PowerShell for remote connectivity.
I'm part of the crew is going to be there.
We have Steve. Amber.
I don't know if you're in the call.
Uh, with me while I, Demetrius, Arnov and myself we are May have one of my team members from Shanghai.
He's waiting on his visa, so fingers crossed, but other than that, I think that's the whole crew that's going to be in invert next week.
So whoever is there looking forward to seeing you?
 
Sydney Smith (SHE/HER)   4:30
Awesome.
Yeah, it's going to be a really great conference.
And certainly if you are there in Antwerpen next week, please reach out to any of the Microsoft folks who are in attendance.
They're really there to speak with you and talk with you, so go up and say hi and Next up we have.
Michael is gonna talk a little bit about collaboration that around DSC and Winget.
 
Michael Greene (POWERSHELL)   5:00
Yeah, this has been awesome.
And there's gonna be a session on it at PS.
Comfy.
You so to tie those that last topic to this one, Demetrius from Wingate is coming over and it's gonna be talking about this scenario.
But I just dropped the link in the chat.
There was a awesome session.
Uh at the build conference?
Not too long ago that highlighted a whole new set of scenarios for DSC.
So if you've used DSC in the past, you've seen probably in Windows where local configuration manager was running as local system and it was really focused on servers and the idea was this is an agent type scenario.
It's built into windows and not really an agent, but with machine config.
We're operating as an extension.
Might want to call that one an agent, I guess, but the whole premise was it's running as local system and it's focusing on operating system settings and turning on features configuring IIS.
One of the things people have talked about for a long, long time now for DSC is could you run it as a user to configure your user environment?
And it's been kind of like we sort of ho hum around.
We don't know how many people want to do that.
You know how many people want to do configuration as code for user settings?
Things like that.
Well, it turns out that this dev box team that works in at Microsoft in Azure Umm is really interested in the scenario where if you're gonna provision a bunch of virtual desktops for a developers to align to a project, it's really appetizing to be able to make all those machines look the same.
So the person in charge of the project, the idea is they would just define.
I want them to have VS code.
I want these extensions in VS code.
I want these settings in Windows, want them to all have the same desktop background.
You know, like all this kind of stuff, and to be able to define that with your project and then as you spin up a developer environment, actually configured the user environment, not just the operating system environment.
So you'll see that if you go watch that video, it's the DSC section starts at 2125 and I'll just sort of say this is one of those scenarios where they call it out as DSC.
I think we're maturing to the point where it's like, do we really need them to even say the words DSC, right?
Like the functionality they gets delivered here.
Seems like magic to an end user and it's really exciting.
And actually, there's some really cool accessibility scenarios here where we can just say, you know, you don't have the ability to see a screen.
So how do you turn on?
You know, narrator, that kind of thinking.
Uh, turns out those scenarios work really well for these, for for for DSC running as a user because you could sort of like profile what you need your machine to look like.
This is a whole new area of exploration, but we're super excited about it and I hope you go take a look at that video cuz I got really, really excited whenever I watched it. Thanks.
 
Steve Lee (POWERSHELL HE/HIM)   7:57
A sitting let me add on something real quick related DSC, so I had just published a DC V3 Preview 8 yesterday I believe and I'm in process of trying to get it published to the Microsoft Store to make it a little bit easier to install.
 
Sydney Smith (SHE/HER)   7:57
I don't.
Yeah. Yeah.
Go for it.
 
Steve Lee (POWERSHELL HE/HIM)   8:09
So look forward to that and that's it.
 
Sydney Smith (SHE/HER)   8:14
Great.
James, did you have something you want to say around that?
 
James Brundage   8:18
Yeah.
Sorry, I've just had some crab Andreas with Robles.
So for DSC it would be great to have it run in the user space.
Accessibility would possibly be very legally huge, because if you have somebody who has accessibility requirements like vision requirements, this can become like, well, a lawsuit waiting to happen.
If you're not giving them a box that's already provisioned with that in mind, so those two are critically important.
The other one is a Michael said.
Hey, do we even need to say DSC anymore?
Yes.
Yes, please say DSC because it's one of the ways that PowerShell will kind of worm its way back into a lot of other ecosystems.
So I think that it's really critically important that especially as PowerShell underlies how you configure Windows and PowerShell underlies how you configure Docker that we keep mentioning PowerShell on that.
We keep mentioning DSC, so that's that's my messaging two cents.
 
Sydney Smith (SHE/HER)   9:21
OK.
Yeah.
Thank you.
Thanks, James.
And you can see the comment Demetrius would love to connect with you around the accessibility compile compliance.
I don't want that you were saying.
Umm.
Go.
Wait.
Moving along with our agenda, our next topic was a build new cap.
I was lucky enough to get to attend build that I got to see James Fair.
Actually, I was actually there helping staff the Linux.
What is the total learning experience for me?
But I really got to think about how PowerShell fits into the Linux ecosystem and also connect with the number of folks around PowerShell scenarios.
That the liberty of bringing the wonderful PowerShell signs that some of you have given us off in 3D printed for us and put them up on the Linux good.
So that was a lot of fun to get to connect with different PowerShell users, build and hear about what you all are doing with PowerShell these days.
I think a really huge theme of build was around copilot and AI type scenarios.
So if you have any, if anyone has any feedback around build or you know things that they might have heard there or seen on the got to watch sessions with love to hear more about that we've also able to attend the MVP event there which was a really great way to connect with more PowerShell folks as well.
We are really looking forward to figuring out plans around Ignite and so if you have topics you'd love to see us cover are really interested in around Ignite plans too.
Would love to hear that as well.
And with that, I will pass on to Aditya who is going to talk about the PowerShell.
Something releases.
 
Aditya Patwardhan   11:12
Hello everyone I have a quick update.
We had a security update releases for 7/2 and seven four earlier this week, so give them a try and give us feedback if you find anything.
I would like to remind everyone that 72 would be going end of life later this year in November, so I would highly recommend everyone to move on to 7 four and start planning for it.
That's it.
 
Sydney Smith (SHE/HER)   11:39
Awesome.
Thank you for that.
And just a reminder that we did do A-75 preview release last month in May, we covered that one in the May, Community Hall.
So if you had questions about that, still happy to answer them and you know last month called when we covered and little bit more.
Yeah.
Thanks for getting out.
The seven, two and seven four releases as well.
Let me go back to the agenda and I have another agenda item.
Yes.
Umm, so you may see in some of the repos that there are two PR's going up on getting merged around code of conduct updates.
So I just wanted to call this out in case folks are seeing this kind of wondering what's going on here.
We are still following the Microsoft Open source code of conduct, so a not a lot of the language around that Apple Code of conduct has really changed.
The big change here is really around some of the resources that are being provided to Microsoft employees around support and reporting for code of conduct violations.
Umm answer.
That is the change.
That's kind of being put in.
I think it's always a good time to sort of provide folks of the code of conduct and to just refresh yourself with it if you have it read it before.
I know you are all wonderful people and are great at following our code of conduct, but I'll just paste it in the chat again if you just need a refracture or want to read it, and if you were having any issues with it, do report it and please let us know.
Next up, I believe we have docs update from drugs.
 
Sean Wheeler   13:29
Alright, let me share my screen.
As far as new content goes, we have one new article and I want to talk about.
Uh, tab expansion two.
We have always documented here in the list of built-in functions, but we don't say much about it now.
There is a full documentation here with some examples.
Normally this is isn't a function that you use interactively, but it can be useful in test scenarios like if you want to create pester, test to test that your tab expansion argument completion, things like that is working properly.
So that's what this article covers.
Umm.
Also in support of the recent releases we've updated.
The.
Documentation for.
Uh.
The release notes.
Uh, in the last month we had.
A new preview preview three of PowerShell 75 and you can see what's there.
And as a DTS said, we had updates for 7/2 and seven four.
The one notable change in seven four is now they tell the IT on Windows.
Umm.
The tilde is tab expanded to your home directory.
So if you're using Tilly with native commands like Notepad for example, and you hit tab, it will be expanded so that, uh, it will work properly with the native command.
And also there's, as Steve said, a new version of do you see preview that was released?
Mickey's working on jackups dates for that.
And with that, I'm going to hand over to Mike Robbins.
He's going to talk about some changes.
In the Azure PowerShell space real quick.
 
Mike Robbins (AZURE POWERSHELL)   16:02
So yeah, we had a new version of the AZ PowerShell module release that build version 12.
I'm gonna share my screen real quick for the dock updates.
So the highlights, you can just go out to what's new in Azure PowerShell.
In our documentation, the overview and these first four items are kind of the highlights.
There's a protect sensitive information warning that will be displayed if secrets are displayed in any output and that is enabled by default with a Z12.
It was pretty previously available, but just not enabled by default.
Also, if you're running Windows 10 or Windows Server 2019 or higher, uh, sign in with Wham with Web account manager is enabled by default and we have a new login experience and we have a new support life cycles and now we're shipping STS and LTS versions ohm and this article is actually completely rewritten and it's the combination of two different articles.
So let me go to the interactive log on article and the links in the in the overview of what's new would take you take you to the same place, but this goes through detailed steps of what the new logon experience works works like, and if you'd like to disable then you log on experience.
You can also do that if you don't wanna be prompted every time you log into Azure PowerShell.
You can set a default subscription as well.
Umm, some benefits of Wham, limitations of Wham, and then also how to disable WAM and if you happen to run into any issues with the new features, there's a couple of sections in the troubleshooting dock.
So there's a scenario where there's informational messages that are outputted to the information stream and not the success stream, but if you.
2.
Depending on a third party automated solution you might be using, those messages could end up in the causing issues and you can log in and ignore the information stream as a temporary workaround and these are the documented issues with Web account manager and you can also disable WAM to to work around those.
One thing I would recommend so we have a release notes article.
I'd recommend taking a look at the release notes before updating to Z12, and then also we have a migration guide so it'll give you detailed steps of how things work before and how they worked afterwards, and I think that's it for the DOC updates.
Thank you.
 
Sydney Smith (SHE/HER)   18:52
Awesome.
Thank you so much.
I did see one question I think related to some of this from Missy.
This with AZ, PowerShell.
Uh, what parameters would you recommend to use for connect AV account?
If you need to alt, you need an alternate credential that has MFA enabled.
 
Mike Robbins (AZURE POWERSHELL)   19:13
OK.
And then we've got Damien, one of the PM's for Azure PowerShell on the calls.
So.
So I'll defer that to Damian to get the official answer.
 
Sydney Smith (SHE/HER)   19:22
Sure.
 
Damien Caro   19:24
And I don't know if there is an official answer here, but more if you're using.
 
Sydney Smith (SHE/HER)   19:27
OK.
 
Damien Caro   19:30
If you're using MFA, it likely means you're going to use interactive authentication.
Otherwise, your script and automation script could be broken, so connected to your account without any parameters will route you through the flow that we'll support MSA.
If you're trying to use username password passing the credential like connect this account dash credentials and pass the PS credential object that flow is going to break once you have MFA in the board.
So my recommendation would be to not use any any parameter with connected is your account unless except the subscription ID or tenant ID.
Those would be fine, but anything that forces a certain flow of authentication that is not the default one is likely going to have challenges with MFA.
 
Januszko, Missy   20:22
OK.
Thank you.
We'll give that a try.
 
Damien Caro   20:25
Good.
I know it was a longer explanation.
 
Sydney Smith (SHE/HER)   20:26
Right.
Wait, I I'm watching the chat and I'm seeing a lot of questions getting answered in chat, but if anyone question hasn't gotten answered.
Umm, please feel free to raise your hand or kind of read UM.
OK.
Thomas has the question and the powerful team road map.
With 2024, there was a mention of the teams focus on community PR.
Now that we're halfway through the year, do we have metrics on how many queuing PR's were reviewed?
Slash merge versus last year to see how this initiative is going.
 
Steve Lee (POWERSHELL HE/HIM)   21:05
You want me to take this one so I don't.
 
Sydney Smith (SHE/HER)   21:07
Yeah, if you want to see.
 
Steve Lee (POWERSHELL HE/HIM)   21:08
I don't have any metrics.
I don't know if Dumbo is here.
I don't think he has metrics, but he might be able to collect them.
Maybe we can talk more about that and have data next community call.
Again, one thing I'll emphasize.
Like what we initially set out for 2024 and where we are today, there has been some impacting changes due to a larger focus on the team for some security changes, not security in terms of security issues in PowerShell itself, but really securing our infrastructure and our supply chain.
So these are things that are on the back end in our infrastructure that you don't actually see the code at all.
One of the questions that came up is like, you know this on a daily builds, yeah, that's currently broken because of these changes.
I don't know when that's gonna come back, but that also because of that focus that also has kind of limited unfortunately some of our ability to kind of reduce some of the campers.
I'm hoping that we can refocus our Monday communities to kind of bring some of those in.
I know that we've gotten a number of those versed in, but I don't have those numbers on top of my head right now, so.
Thanks.
By the way, I'll just put a call out like it would also help us if rather than necessarily contributing a new PR to have community folks review existing PR's, that would be very helpful. Thanks.
 
Sydney Smith (SHE/HER)   22:30
Yeah.
Thanks.
Thanks for that answer to you umm.
I'm gonna pause and see if there are any other questions.
I'm also taking a note for the July community call to follow up on that question. Uh.
Which I could get home if you listen.
Well, well, I know it's summertime and also a lot of folks are already at PSU, but we do fairly good attendance today.
Happy to give you all a ton of time back if there are no other questions.
 
Speaker 1   23:31
Can I actually do?
Still a shameless plug for the community overall and.
 
Sydney Smith (SHE/HER)   23:35
Absolutely.
Please do.
 
Speaker 1   23:38
Yeah.
So I posted him to chat about.
PowerShell Saturday that the research Triangle PowerShell user group is going to host call for speakers is currently up and available.
This is an event that we want to lead by the Community.
This is a PowerShell focused event, but primarily community based event, so this will be a Saturday running in October, October 5th and 6th will actually do a second day if we have a good enough request for it and at that point.
So it is currently going to be hosted into the Raleigh Triangle area, North Carolina, and call for speakers is out there.
We want to get the community involved and we want to do this shameless plug for the research triangle of PowerShell user group as well and that we meet twice a month and the.
Involvement there is just been a great one and people do a great job presenting.
I didn't have enough opportunity to do a demo this time, but I want to do one next time of the after hours as we call them and so we have meetings all the time, but quite often it's it's the before meetings and the after meetings where people just open up and have conversation.
And we had a really one, the last one, a good one last time talking about parameter parallelization of commands.
And so I wanna kind of demo that and go through that and and work through the idea of just the community sharing with the community and that's just been what I really offer.
So I just want to say thank you.
Shameless plug for the research triangle PowerShell user group and a shameless plug for the PowerShell Saturday coming up in October. Thanks.
 
Sydney Smith (SHE/HER)   25:16
Yes, thank you so much, Phil.
And anyone who is putting on powertool events, PowerShell user groups, you are always more than welcome to promote them here.
We really love to see that.
I saw that Sean is also posted about SQL Saturday at Baton Rouge, which is coming up in July and has a small PowerShell track.
I I don't know if you answered this.
Uh oh, yeah, looks like he did about some.
Ah, sorry about if you're accepting some remote sessions.
That is great.
I'm just reading the chat.
Well, some people are gonna be in Baton Rouge.
Looks like a few folks are up for some PowerShell GUI stuff.
Can we keep it to like under 10 minutes, James?
 
James Brundage   26:06
Uh, sure.
Maybe, hopefully, probably I will.
 
Sydney Smith (SHE/HER)   26:11
OK, let's.
 
James Brundage   26:11
Darren, do you have something you wanted to talk about?
Cause your your camera's on.
You wanna go first?
 
Darren Kattan   26:16
Ohh man.
Uh, no.
It would be a rehash of the thing I showed last time, still hammering that out, but I'm I'm pretty much the finish line now.
 
James Brundage   26:26
Yeah, mine's more gonna be.
Let's hope the demo gods aren't cruel to me.
Work in progress, so I'm a funny thing happened along the way to the forum.
You guys know I've been messing around with PowerShell and GUI while now I.
Was looking for a thing to go render a 3D shape just because I wanted to and I found a really really nice platform for being able to do lots of stuff in 3D and then I said about wrapping it in PowerShell.
So let's see demo gods, how nice are you?
We got a lot of parameters here, but we're basically just calling show PSVR G, which is the last show SVG, giving it a randomized rose with a bunch of morphs.
Picking a random color palette, picking a random number of copies for 2D and 3D setting in 3D, picking up to 512 random copies and popping it out into a file.
I'm gonna call it T8.
Now I have no control over how good or bad this one might look, but you know could have come out worse.
This is few 100 objects rendering in A3 table in a bubble in a sphere in a Helix in a cube and in just random space.
If you really think that's helpful, any of these are clickable.
Can can contain anything, so to kind of half prove the point here I'm gonna take another quick demo that I ran here, which is just taking some trending images from giffy and throwing them into a 25 by 25 grid.
Do that real quick.
 
Steve Lee (POWERSHELL HE/HIM)   28:06
Interrupt you, but I just made you a presenter cause I on my screen you just showing in a tiny box.
Can you try representing or resharing?
 
James Brundage   28:11
Alright, Yep, there you go.
 
Steve Lee (POWERSHELL HE/HIM)   28:18
OK.
Yeah, now it works.
 
James Brundage   28:20
Cool.
Give me a second to let it kind of catch up.
OK, so we're doing the 25 or they X25 copies of the trending gifties will have some repeats in a 25 by 25 grid, not 3D yet.
We'll make a 3D in a quick second, so let's go ahead and say give you 25.
There we go.
Cool.
Yeah, 2D grids too.
Let's make that in 3D and give it a 3D copy.
Count a.
Anybody wanna pick something?
Let's let's go for 256.
Alright, right at the PowerShell way.
.25 KB OK.
Uh demo.
Gods being cruel here.
Yes, just a little bit that's on me.
There we go.
And now we have all those nice animated GIFs in 2D or 3D space on a randomized color.
Speaking again, anything can be in here, and these are indeed clickable.
So if I just go ahead and I guess we're a day late for that one here.
So let's go click this are we are.
So yeah.
That's gonna be fun and interesting.
The long story made short of all of this is we're about to be able to take everything that we did in PowerShell, including formatters and types, bring them over into the web in 2D and 3D, and it's gonna be awesome.
And I'm done.
Now, less than 10 minutes, right?
 
Sydney Smith (SHE/HER)   30:03
Yep, you got it. Perfect.
Already well.
 
James Brundage   30:08
So one to 10, well, 10 coolness factor just like having just absorbed this as a group because this is I I didn't set out to do this like starting off last month.
I just kind of found myself there, but I'm enjoying where I found myself.
What do you all think of this?
Does this have cool potential?
 
Sydney Smith (SHE/HER)   30:31
Feel free to respond to James in the chat.
Uh.
 
James Brundage   30:37
Uh.
Yes, this would also be great for generating here as the team images or family trees or you know, personal media collections or about anything.
There was an old talk going around Microsoft around the time of Windows 8 talking about familiar versus modern GUI.
The idea being that when modern GUI shows up, people will just magically start using it because it'll be so much better and I think this is a good talking point but didn't exactly fit for win eight.
One of the things that I'm realizing very rapidly in this is that this can change the whole way we interact with everything because the screen density is so much bigger because by being able to take our 1920 by 1080 screens and project them in a third dimension, we can actually have hundreds or thousands of content items on the screen without getting as overwhelmed.
So I think this could actually change quite a lot.
And while I didn't expect to find myself there, I'm really looking forward to where the stakes is.
Uh.
Related side note of understanding a types PS1 XML or a format PS1 XML can indeed live in a web page as metadata and be used.
So it is actually not as bad as it might seem to take PowerShell and Bridge it into web languages with all of our bells and whistles and tact just going to take a little bit more work.
Famous last words, as those always are.
 
Sydney Smith (SHE/HER)   32:17
Awesome.
Umm.
Well, thank you, baby.
Thank you for the demo.
I think I'm gonna cut off the community call here.
There any more questions and give everyone a wonderful 30 minutes back.
And I hope you all have a wonderful rest of your day.
Thank you so much for joining our call today and feel free to put any more questions or things we didn't answer in the chat.
Thank you.
 
