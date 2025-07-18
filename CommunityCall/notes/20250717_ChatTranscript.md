# PowerShell_OpenSSH Community Call-20250717

Transcript
July 17, 2025, 4:27PM


Alex Wang   3:37

Let's get started. Hello everyone. Good morning, good afternoon, good evening. Today's contents I think are also there for. We will also record it at same time for the review. I'm Alex, currently working as a PM for the Azure CLI and the Azure PowerShell.
I will be hosted this committee call, so we'll come to the July 2015 partial committee call. Everyone is welcome to join and discuss the partial projects and its development. We have our agenda post in the meeting chat and a link to the meeting discussion.
So please feel free to look at. You can ask a post the questions in the discussion issue or post in the chat group and also please follow our code of conduct. So we really care about the for the announcements.
And also, happy summer. I know many people are also taking a summer holiday, so we'll come back. If you don't have a holiday yet, you can take a short break and recharge. Yeah, that would be wonderful. OK.
Let's go ahead and dive in with our agenda for the today. First, Jason and sorry. First, Jason and Xin, do you want to talk about Palette PS?

Jason Helmick   5:41
Thanks, Alex. Well, I I love your background. It's very summery. I just love that. Hey folks, before I get in, before I get started with Plat APS. So Sean, give me a second here. A friend of mine sent me a message this morning. Matt Gill sent me a message and it turns out his message was a work item. So thanks, Matt.

Alex Wang   5:45
Mhm.

Sean Wheeler   5:52
Sure.

Jason Helmick   6:01
I'm going to see if I can clear that work item before we talk about Platty PS. Later in this session, Stephen Bucher is going to be by to talk about his just amazing experience at PowerShell Conference Europe. And I just want to say the feedback that I've gotten from that conference. Wow, it sounded like it went off. It was just a fantastic.
Conference. So congratulations to everyone. But before that, just to keep your summit minds going during this summer July, I almost said holiday. It's not holiday for me. I wanted to let you know that the PowerShell Summit North American has opened up their CFP or their call for.
Presentations. As a matter of fact, let me see if I can drop a link in here. And so if you would like to speak at PowerShell Summit North America, now is your time to get your submission in and start to select those.
And folks, I think a lot of you, if you've been to this, the North American summit before, you know that what a great opportunity for folks to just present things about what they do and some of the solutions that they come up with, so.
If you're thinking about presenting, if it's something that you've been wanting to do for some time, now's a great time to get your submission in and all that kind of good stuff. So, OK, let's talk about Platy PS. Now I'm going to let Sean talk about mostly about Platy PS, but I just wanted to steal some of the highlights.
I'm so sorry about that, Sean. First of all, thank you for a lot of folks have been asking us about this and I really appreciate the interest. So Platty PS is a tool that basically allows you to author help files in Markdown and then converts them to the necessary formats for updatable help so it.
It converts it to YAML, it converts it to MAML for get help. All of this kind of stuff. It's one of the primary. It is the primary tool that the content teams here at Microsoft use to provide you with that PowerShell documentation that you see on the Learn platform and all that kind of thing.
Well, we've been working on this project for quite some time to improve that content pipeline. In fact, we've been working on it for years. And yesterday, yesterday afternoon, we finally RCD or release candidate to Plat EPS Microsoft dot PowerShell dot Plat EPS version 1.0. dot 0.
I just want to give a huge shout out to Aditya and Jim Truer who really pushed the engineering on this and and especially to the content team and Sean for all of their hard work on getting us to this point. Now let me just give you some information around.
And then I'll let Sean kind of dive in to talk a little bit more about what we've done. Now we're going to GA here here in the next few weeks as soon as our internal partner kind of signs off with us. They've been testing with us all along and in fact.
One of the interesting things is I know for a fact that Sean has had to do some content work and that he has actually touched and updated over 60,000 documents for the the that uses this new pipeline and I'm just, I'm really impressed by all of that.
So like I said, we are going to be gaing this in a few weeks when we GA, because I know some of you have been really interested in this. We'll put a new readme U on the repo. So we're going to open this U for contributions. Now look, you're going to have to go through Sean's rigorous process.
Of going through discussions to issues to PR. Same kind of thing we like to do in PowerShell, but this will be much more rigorous because we need to be a little bit careful that we don't accept changes that that are detrimental to our internal partner because we don't want to like blow docs out of the water. But we are very excited about that.

Sean Wheeler   9:45
Yeah.

Jason Helmick   10:00
So we will open up the repo and that kind of thing. And one last thing before I hand it over to Sean to talk about Platty PS is not in this call, but if you get an opportunity like at a conference or something like that or maybe a user group call.
You want to have some fun? Ask Sean how the last and one of the most serious bugs got resolved as we were walking into RC. That even today is just making me chuckle. So with that, Sean, take it away. Talk about Pilate PS. Good luck.

Sean Wheeler   10:34
Yeah, so let me drop some stuff in the chat here that we're gonna talk about.

Jason Helmick   10:34
But.

Sean Wheeler   10:42
So as Jason said, we've released RC1. It's actually posted to the gallery, so you can get it off the gallery. The we have more work to do in the repository here, reorganizing the content, but.
The now the main branch we renamed master domain and actually renamed master to V1 and what was the V2 branch is now main.
Excuse me and.
So all of the new content, all of the new code I should say for Microsoft dot PowerShell dot plat EPS module is in the main branch. We are not doing any more development work on the old version.
That's in the V1 branch. It's there for historical reference and I have a couple of open PRS that need to be merged to. I don't know PRS to.
Rearrange some of the content of the branches. Also need to spend some time updating the documentation and we're going to have a whole contributor guide that will be publishing.
We need to go through and clean up all of these old issues in PRS that are labeled for the previous version. As I said, we're not, we're not accepting those anymore. We're moving forward, but I am.
Super excited to talk about the new version. The new version is much faster. The the workflow is much better. It's more expandable, unlike the old version of Plat EPS.
The commandlets were designed to work on files, so it read files and it wrote files. You couldn't really pipeline things together. The new commands are pipelineable, so and and there's a whole.
Command help object model so you can create objects and populate them with information before you export them out to the various formats.
So you can now build tools and build your content programmatically so.
The the documentation you see here is mostly accurate, needs to be updated a little bit for the some of the last minute changes, but what I wanted to show you along with this.
Release of Plat EPS. We've updated our publishing pipeline and you may notice some changes in our reference content now. So some things that we're missing that are now here we have.
Parameter set names and this is very important when we talk about the parameters. I'll show you below, but also they've updated the right hand navigation here. It's expandable.
And it floats with you so you can jump down to an example and as you scroll it stays available here so you can jump back and forth to the top and the bottom and when we get into parameters.
You can see that there's there's parameter properties. Now these are the the properties that are common.
To the parameter for all parameter sets, and then where it says parameter sets here, this is collapsible. These are the properties of the parameter that can be different per parameter set. Let's take a look at.
Are my favorite where object. It's the most complex because there are 32 the maximum number of parameter sets.
And if we go down and look at the property arameter, you can see.
The property parameter applies to all of these property sets, and you can see the values of these different parameter properties.
For a particular parameter set. So things like position could be different. It could be named in one parameter set and positional in another. It can take pipeline input in one and not another, and so on.
So much higher quality documentation with the new version.
Um.
And while we're talking about docs, let's talk about some other things that are new in docs. You may have noticed some changes to the documentation site. We used to have a few other items.
In this area here like the edit button and the thumbs U thumbs down, that's now in the menu so you can get to that here.
And um.
With the new version of Plat EPS, one of the things that we've enabled, I haven't updated the content yet, but this search, it's it's not a search.
It's it's a filter. This filter here in the table of contents allows you to quickly find a command and go to it, but we've added the the ability to search.
By alias. So like I said, I haven't turned this on yet. For this content, we have to edit the content, but you'll be able to put it in the alias here and it'll find it.
So lots of good work going on behind the scenes for this stuff.
And I've been working so hard on Plat EPS. There hasn't been much time for any other new content. The one, yes, piece of new content that I wanted to point out is we've published the Monad Manifesto, sort of so.

Jason Helmick   17:46
Yay. Thanks for this, Sean.

Sean Wheeler   17:56
Uh.
I I I didn't want to. The modern manifesto is not going to change, so I wasn't going to publish it as markdown, but we've linked here to Jeffrey Snover's blog post that.
Kind of gives the back story of the Moned manifesto and then this link here links to the PDF you can download of his latest copy copy. He graciously gave us permission to publish this.
It it kind of seems like an oversight that this hasn't been here all along, but now it is.

Jason Helmick   18:38
Yeah, I just want to point out that as Sean says, it seems like an oversight, but it's actually very difficult to put this particular doc on the docs platform for varieties of reasons. But finally, it's here. So finally in official docs, the Monad Manifesto and Jeffrey's explanation. So there you go. Thanks, Sean. This was a big deal.
At least for me, so thank you.

Sean Wheeler   19:03
All right.
Back to you, Alex.

Alex Wang   19:10
OK. Thank you. Appreciate Jason's team and your work, your team and hard work exciting to hear the updates for the plat PS. So we are looking forward to the new release. Yeah, I see.
Do you finish your documentation update besides Clive PS?

Sean Wheeler   19:35
Yes, I'm done.

Alex Wang   19:37
OK, OK.
OK, the next one for the PS resource cat. So we also have some updates. So maybe Amber will bring us some release related content of PS resource cat so we can see some update.

Jason Helmick   19:56
Yeah.

Alex Wang   19:57
Amber, yeah.

Amber Erickson   19:59
Yep. Hey, so I just have a quick update. We released version 120 preview one of PS resource get a few weeks ago and the big thing here is that we now have dependency support for V3 repositories specifically for PS resources.
So these are PowerShell like modules and scripts. This is super, super exciting for us. It's something that we've had in the backlog for quite a while. And this was actually a community PR from Olav Brooklyn. So thank you so much, Olav. I don't know if you're on the call, but we really appreciate that PR.
We have a few other bug fixes in the release as well, mostly relating to container registries, so please feel free to check it, check it out and let us know if there's any bugs or issues there. Thanks.

Sean Wheeler   21:04
You're muted, Alex.

Alex Wang   21:07
Sorry, sorry. OK, very quick update. Thank you Amber. The next one we can go to the the RF, the RFC from the Justin. I think this is the topic with the more PowerShell content out of the OneDrive scenarios.
OK, have you hand over to Justin out here?

Justin Chung   21:30
Yes, thanks Alex and hello community members. I just wanted to bring some attention on this RFC here. I'm gonna drop it in the chat right now.

Alex Wang   21:31
OK.

Justin Chung   21:42
We've had a lot of good discussions on here. I think there might be some more suggestions or feature requests and things of that nature on this RFC.
So this is the last chance. I guess the deadline for this would be by August and after that, yeah, I'm gonna move on to coding it up and getting that feature ready for you guys. So yeah, I just wanted to bring to everyone's attention this RFC.
That's still open and give everyone a last chance to participate in this.

Steven Bucher   22:24
Justin, could you talk a little bit about what the RFC includes?

Justin Chung   22:27
Oh yes, so the RFC is like a design spec to move the PowerShell documents, which is right now in OneDrive to the local app data or or customizable by the user.
So which? And the reason why we're doing this is because the OneDrive documents location it gets back, it gets auto synced into the cloud and it kind of affects partial loading time and it could at least lead to like SFI incidents.
In case there's like some sensitive things with your powershell modules or contents that you have. So we're trying to avoid those S5 incidents and make it customizable for everyone so.
That's what the RFC is about. It's just to lay out the design specs and like get people's input and ideas on how we should move this and what considerations we need to think about. So if if you guys have any any ideas about.
What it might impact or affect, then if we can outline it in the RFC now, then we can patch that sooner rather than later.
Yes.

Alex Wang   23:52
Ooh. Any other question?

Justin Chung   23:54
Yeah. Any other questions about this?

Alex Wang   23:57
Yeah.
OK.

Justin Chung   24:08
All right. Thanks, everyone.

Alex Wang   24:08
What demo?
Yeah, I saw a question from the chat. Maybe just you can look at. OK, last month we are still talking about the partial conference EU, so and it's end in the last month. So Stephen, our best speaker attend.
The conference and share the content about the powershell of DSC and also Azure CLI and powershell. So I can't wait to see what he shared and bring some new feedback from the participant on site.
Stephen, maybe you can go ahead.

Steven Bucher   24:48
Yeah, thanks Alex. Like Alex said, the the powershell conference EU happened, gosh, was it 3 weeks ago now? This happened in Malmo, Sweden. We presented a bunch of.
Really awesome topics around the state of the Shell, SSH, PS resource, get PowerShell security, machine configuration and guest configuration, win, get and a bunch of other awesome stuff. I'll share a little let's see where.
That's my. I'll share a little bit of photos for folks. The videos of these sessions are being processed, excuse me, processed right now. So just to give, you know, some ideas of what it was like there. It was a really great conference.
I think it was about 300 people and attendees. Myself, Tess and Anam from the PowerShell team went and we presented on. We presented the State of the Shell and we presented. Anam presented a session on PowerShell Security, which was a really awesome session.
Definitely look forward to the YouTube once you see it. And then Tess and I present on SSH. We're also joined by Demitrius on the Winget team who presented about what's new in Winget and then Multima on the guest configuration team or the machine configuration team and presented about topics around that. I also shared.
About AI Shell, which I think is here. So yeah, Autumn talked a lot about enhancing trust in supply chain and container container gallery, container registry galleries, excuse me. And then I presented about AI Shell and then we did some presentations on.
Oh, here's us as a team there. And then we did a podcast with Andrew Pla, so that is actually published. You can definitely go take take a listen where we talked about PowerShell at the PowerShell on the PowerShell podcast. Anam was doing a security talk around.
PowerShell security in general. Excuse me, definitely take a look at that. And then just a couple other photos and highlights from PS comp EU. Steve Shell, myself and the team presented and we had a lot of great feedback. It was always great.
Chatting with customers, getting a lot of chatting with community members, new and existing ones alike. We had a lot of great conversations, great feedback sessions, great times for tinkering. There was a lot of times we would just open up our laptops and just work out problems together. And so yeah, it's.
Also great cuz I've now met a lot of the folks I see on this call here. I can recognize some names now, which is always fun. And yeah, I think that was it. I don't know Tess, if you had Tess was also there, if she had any takeaways or anything she'd like to add about kind of a recap, but overall.
A really great experience. We're always, we're always happy to attend this conference.

Tess Gauthier   27:55
Yeah, nothing, nothing to add on my side, Stephen, but it was really great to have the opportunity to meet everyone that was at the conference and just want to echo checking out the recordings if there are any sessions that you didn't have a chance to attend in person.

Steven Bucher   28:12
I'll drop a link to some of the recording the YouTube channel. Um.
So you can take a look at the recordings and I believe they have a mini comp.
Coming up in October, but you can go to their website and learn learn more about it. But yeah, just wanted to share a bit of a recap about the conference, so.
X.

Alex Wang   28:50
Thank you, Stephen. I think on site must be very, very great. So excited to see many familiar face in the conference issue, yeah.
Uh, great share and I so thanks. I think this content almost the present uh.
Yeah. Any other question?
We appreciate your interest and also we are happy. We'll come to discussions. You still continue the discussion in the GitHub discussion, yeah.
OK.
Folks, if no more questions, we complete all the presentation of all the content today. I hope you have a wonderful beginning of your summer and also we will.

Steven Bucher   29:48
It.
Alex, I think there's a hand up.

Alex Wang   29:51
See you next month.
Yeah. Oh, OK, I saw.

Ayan Mullick   29:57
I.
I hope I'm audible. I don't have a question, but maybe a question related to a suggestion. I don't see Damien here or someone from Azure PowerShell, so I'm not sure.
If I can just ask the question or basically the perspective. Oh OK, so basically in the Azure PowerShell modules there is some coverage gaps, meaning for example if you deploy.

Steven Bucher   30:18
Alex works on Azure PowerShell too.

Alex Wang   30:20
Yeah, yeah, yeah, yeah. I will do that first, yeah.

Ayan Mullick   30:31
A data factory, then you know you the test connection. When you create a linked service, you cannot do it with PowerShell or if you deploy a data bricks, you cannot do private network integration using PowerShell and so I was proposing I pasted.

Alex Wang   30:36
Mhm.

Ayan Mullick   30:49
A proposal in the in the in the teams chat where if there could be like a coverage matrix or a coverage table.
That signifies that look we have coverage to do this with this Azure service using PowerShell or not or pointed or there is a community model or something. So if the representative from Azure PowerShell is OK, I was planning to submit.
Like a a poll in this discussions repo to get some feedback from the community if this kind of a coverage table would be useful for the community.

Alex Wang   31:37
Yeah, well to that this is also our goal. We know we have some gap even between CRI and powershell because sometimes maybe the CRI update 1st and then customer use the CRI 1st and powershell update the latest and service team maybe.
The priority or something like that. Currently we are doing some work for the parallel for the CR and PowerShell with different client tools. We want to have a horizontal.
View CRI and powershell is the same level. But yeah, this is a need some time. Currently we have some internal table to show or what's the gap between CRI, powershell or even Terraform. So maybe.
Next the half year we can give some, we can give you some updates for our for our work just to compare the gap between CR and powershell or between the the REST API. Yeah this is a very very good point. We are also want to build this one.

Ayan Mullick   32:48
So are you OK if I submit a discussions a poll request in the discussions?

Alex Wang   32:49
Hmm.
Sure, you can send the discussion in the Azure powershell repo. I think that one is fine. Yeah. Oh, OK, OK, cool, cool.

Ayan Mullick   32:54
OK.
Yes, yes, I pasted that once. OK.
Thank you.

Alex Wang   33:06
OK. Thank you. Very good suggestion. Yeah.
OK. Anything, anything else?
OK. If no questions, thanks everyone. I hope you have a good day. Have a wonderful beginning of your summer holiday. Yeah, see you next month.
I.

Ayan Mullick   33:29
Thank you.

Alex Wang   33:31
Bye.

Jason Helmick stopped transcription
