PowerShellOpenSSH Community Call-20260716_092905-Meeting Recording
July 16, 2026, 4:29PM
18m 16s

Jason Helmick started transcription


Jason Helmick   1:49
All right, folks. Well, good morning, good afternoon. Hello, good evening. Welcome to the July 2026 PowerShell Community Call. This is one of those summer edition ones, so it'll be a little bit of a shorter call, but
Everyone is always welcome to join and discuss the PowerShell project and its development. So, we have an agenda posted in the meeting chat. Please feel free to post questions in this chat or in the discussion issue. And please follow the code of conduct. As usual this year, I want to start off with a...
a celebration of the happy 20 years to PowerShell since its formal shipping started. It's been 20 years. It's been great to PowerShell. But I also want to add another note. And this is kind of based from what Andreas was saying in the chat. First of all, happy MVP renewals and MVP first timers that have gotten MVP for PowerShell data center
or whatever your award category was. A lot of folks doing a lot of amazing work. Not everybody has been recognized as an MVP that's doing that amazing work, but we all appreciate it from here at Microsoft. The things that you guys do, it's very helpful and very, very important. On that note,
I just happen to want to, I just happen to have this on my talking agenda before we get started. I also want to celebrate another 20 year milestone today, which is Jeff Hicks has now officially been a PowerShell MVP for 20 years. I want to just thank Jeff for all of the learnings
the work that we've done together. Thanks for being a great mentor and a representative of the MVP community. Jeff is not alone. We have some long time MVPs. Some are in this call, like Alexander, for instance. Thank you all for your long, long, long support on this project.
And with that, let's go ahead and get started with our agenda. So our agenda today begins with servicing updates. Servicing updates. My friend Aditya, do you have something you want to talk about with the servicing updates?

Aditya Patwardhan   5:05
Yes, we have servicing updates coming for 74, 75, and 76 shortly. And we'll also be having our 770 preview coming up with the latest.NET 11 included. So look out for those. And I would like to remind everyone again that November is when 74 and 75.

Jason Helmick   5:19
Boo!

Aditya Patwardhan   5:25
Become end of life.
Seal.

Jason Helmick   5:28
Yes, and let's say that together, folks. In November, 7-4 and 7-5 become end of life. But my friend Aditya, don't go too far because I think there's also something kind of special for you to talk about with Sean.

Aditya Patwardhan   5:39
Yep.
Yes, so we released an update to Platypus last week. It was a couple of bug fixes from me and like 7 bug fixes from the community, which is highly appreciated. So please give it a try. If you find any issues in your pipelines or something, let us know.

Jason Helmick   5:43
Take it away.

Aditya Patwardhan   6:03
try to get to them.

Sean Wheeler   6:04
Yes, I'm super pleased that we got this one out. This fixes all of the structural issues with MAML, so we won't have any of the errors with MAML, like the extra nodes.
the things that broke get help and the invalid information that it was returning before.
with get help. But I want to point out that the help, the current updatable help is still broken. We have to get this version into our build pipeline, and we have to work with a different engineering team internally here at Microsoft to get it deployed.
Once that's deployed, then.
Your update will help. We'll have good information again. But this fixes the structural problems, so you can begin using it for your own documentation. There's some format things that we want to address in the MAML conversion, but it doesn't affect
the accuracy and the technical structure of the MAML files. So.
Super happy to get this out.

Jason Helmick   7:28
Yeah, and a couple of things that Sean, I think that you said there, you and Aditya pointed out are worth highlighting is
And I just lost my brain. But the important ones is that first of all, Sean assures me that the technical accuracy is there and that the structure has been fixed. We may have a few more formatting issues. But the two things I think worth highlighting is what Aditya said about, Aditya did some work here, but a lot of the work came from you folks and is greatly appreciated.
It really, really helped us out a lot. And one of the reasons why this update is now out is because of thanks to you. The second thing I think is worth pointing out here is the point that Sean mentioned that while Cloud EPS 1.02 is out, we still have to work with a partner team to get it into the build system for the documentation and for help.
I don't off the top of my head have a timeline for when that's going to happen, but we'll bring this topic back at the next community call to make sure we button it up. So that's great. Thanks, Sean and Aditya. That was fantastic getting that work out. The next topic that we have for today, and I'll ask Steve to show up for this one, is
around the DSC version 3.3 possible GIA and when that might be happening.

Steve Lee (POWERSHELL)   8:48
Yeah, so I'll talk about the plan. Before I get to that, I do want to make an announcement that Mikey Lombardi has joined my team as a software engineer, focusing on DSC. He still owns the doc content, but now he can actually quote un quote legitimately do more of the coding, which is great. I believe he's not here because he's been having some technical hardware issues, so hopefully they'll get resolved soon.

Jason Helmick   8:50
Yeah.

Steve Lee (POWERSHELL)   9:10
All right, regarding the DC33 release, we made a ton of progress. I don't have anything in the demo today, but the current plan of record is I'm trying to get a 33 preview 4 out later today, a 323 servicing release out also later today. The hope is if it's successful, the 33 preview 4 will also be published to packages at Microsoft.com for the.
Dev and RPM packages. And then the target is to have a release candidate probably early August and then GIA sometime before end of August. So that's our target. And then we'll immediately roll in two, three, 4.

Jason Helmick   9:51
And that sounds exciting. And yes, a huge shout out for two things. One, DSC33 is going to ship GIA here shortly. And 2 for Mikey Lombardi. If you didn't know, and Steve just kind of announced that to you, yes, Mikey is now formerly an engineer on our team. So he will be doing the great work that he's been doing now as a

Steve Lee (POWERSHELL)   9:51
But.

Jason Helmick   10:11
as an official engineer. He still owns the doc set. And so yes, we're all very, very excited about that. And we congratulate Mikey. It's great. It's great. Well, great. It's DSC. So Anam, I don't even know. Look, I'll be honest. I don't know what this is, what this leucine indexing is.
But I, I, can you help us all understand what this means?

Anam Navied   10:35
Yeah, hi everyone. I wanted to hop on here and give an update for the Lucene Search Issues for PowerShell Gallery. So to provide some context, if you've searched for a package on the website or you've just gone to the packages page on the PowerShell Gallery website, you may have occasionally seen zero results returned along with the message that
The PowerShell Gallery is currently experiencing intermittent issues. And we've discussed in previous community calls that this stems from Lucene indexing, which PowerShell Gallery uses, and some issues with how that scales for the large number of packages that the PowerShell Gallery hosts today. I think as of this morning, we have about like...
16,200 or so packages. So I think Lucene has kind of seen some issues with that, I would say. So you may have been seeing zero packages returned intermittently. Well, I have some great news. The team has been working on a fix for that, and we've deployed it out.
So with this, users should see packages return when they search on the website. And a few things I want to note, if you don't see the full number of packages, then PowerShell Gallery is going to try to remedy that in a 10 to 15 minute window now. With this, also know that if you publish a package, it may take up to...
10 to 15 minutes for it to show up on the Gallery. But from what we've seen on our end and with testing, it's often much quicker than that. We on the team are also focusing on other improvements related to this scenario. So hope to kind of provide some updates about that.
in future community calls. But as the community, if you happen to see any errors about 0 packages being returned or any other issues, please feel free to reach out to us on GitHub. And we also appreciate all the feedback we had been getting about this issue on GitHub. I think it kind of helped.
I understand the viewpoint it was and kind of look into and understand how often and stuff it was appearing. Yeah.

Jason Helmick   12:41
Well, that's great. And thank you so much. Folks, a little bit of information around the Microsoft update, the MOO, the MOO situation. So since the last community call, the MOO updates for 74, is it 17, and for 75, I think it's
Those MUA updates have gone out. For 763, the catalog update is released. All other channels are coming soon. We're being very deliberate about this rollout pace. So that's where we're at at the moment. So we'll keep you posted on the progress, but things have improved greatly and you should start seeing those results. So thank you very much.
And thank you very much for telling us about it and talking with us about it and that kind of thing. Sean, what do you got for docs updates?

Sean Wheeler   13:37
Not a whole lot to talk about. Dropping a link in the chat here. And I'll just share the page. Most of the updates and docs have been focused on.
Uh...
the monthly maintenance releases and responding to.
you know, issues that come in and feedback, other feedback channels that we have. So not a whole lot new. I think we talked about this last month. There's, we did spend a lot of time cleaning up the rules documentation for Script Analyzer. There's still more work to be done in that space.
Um...
Um, but uh...
Things should be getting better. What we're trying to do is provide more context for all of the rules and link back to supporting documentation to provide more value in that rules documentation. And just again, want to shout out to community contributors.
Last month, Ariane submitted a big PR to clean up, 160 articles. We appreciate contributions like that, as well as the issues when you find problems in the documentation.
And when you find problems, if you're willing to submit PRs, even better. So that's my pitch. Back to you, Jason.

Jason Helmick   15:21
Well, that's awesome. Thanks, Sean. Yeah, again, thanks for all the contributions. I remember years ago when Sean says, hey, if we do this, people will actually contribute and help us out with the articles. I was like, nah, nobody's going to help you with docs. And I was so wrong, and I'm glad to be wrong. So thank you for all of that.

Sean Wheeler   15:37
Yep.

Jason Helmick   15:40
Well, folks, that's pretty much the end of the agenda. One of a couple of things that I wanted to add was, you'll see I had posted a chat in this chat with Gilbert Sanchez about inboxing stuff. We have nothing to announce right now, but we are looking forward to
the announcement, which the formal announcement, which we hope we'll have to be able to do soon, working through some things. And then we will enjoy sharing that story and getting down to work with you for that. The other thing that I'd like to bring up, and then we can open up the floor. There was no scheduled demo for this week, so this may be a short summer call.
But one of the other things I just wanted to say is really, really, really appreciate the folks that have been rewarded, awarded their MVPs and so forth. And I was just thinking if there's an MVP that's really helped you out a lot, this is a great time to reach out to them and thank them and tell them
how they were really helpful. But also at the same time, thank you for being involved in this project. And thank you for working with us and helping us get through this. And with that, are there any questions that people would like to chat in or something like that for a couple of minutes? Let's take another like 4 minutes or so.
That kind of thing, and I'll just open the floor for a minute.
I will edit out this pause in the video.
That's pretty good. Okay, well, let's go with this then. We look forward to hearing from you. You know how to get a hold of us. So we'll see you in the next community call. Thank you so much. Have a safe and wonderful summer. See you next time. Cheers.

Steve Lee (POWERSHELL)   17:45
Hey, Jason, there was a hand up. There was there. It disappeared. All right.

Jason Helmick   17:46
Yeah.
Oh, is there a hand up? I'll take a hand up.
Okay, it disappeared. We'll chat it in and we'll catch you next time.

Steve Lee (POWERSHELL)   17:54
You can type in the chat.

Jason Helmick   17:57
Yeah.

Steve Lee (POWERSHELL)   17:59
The.

Jason Helmick   18:00
Cool. See you everyone.

Jason Helmick stopped transcription
