# PowerShell_OpenSSH Community Call-20240215_092609-Meeting Recording

February 15, 2024

Steven Bucher   7:05
OK, I think we got a pretty good quorum.
So we can we can probably get started here.
So hi everyone.
Welcome to the February PowerShell Open SSH Community code.

Danny Maertens joined the meeting

Steven Bucher   7:15
It's great to see everyone here and saying hi in the chat.

Chris K. (@_that_chris) joined the meeting

Steven Bucher   7:19
So the this meeting as usual will be recorded and uploaded.
Probably the the next week.
So you can find all of our pest community calls there and of course a reminder of our code of conduct.

Colin Webster joined the meeting

Steven Bucher   7:32
The link should have been sent to the chat further up and some of our some other additional links to the contributing guide are sent.
And of course, the agenda has been sent.
We got a pretty packed agenda.
We're gonna be talking a little bit about the recent uh 2024 investment blog that Steve posted and we'll have we have a bunch of other agenda agenda items to go through.
First, we'll probably start off actually with an update from the Azure Automation team is Nikita, are you here?

Nikita Bajaj   8:09
Hi, Steven.
Yeah, I'm here.
Can you hear me?

Steven Bucher   8:13
Yep, I can hear you.

Nikita Bajaj   8:16
Alright.
Hello everyone.
Let me just quickly share my screen and.

Matthew Dowst joined the meeting

Greg Malleus joined the meeting

Nikita Bajaj   8:29
Can anybody confirm if you can see the slides?

Steven Bucher   8:37
Yes, I can see it.

Nikita Bajaj   8:41
Alright.
Ohh so today I have with me Sanjeev who's from the engineering team and I'm Nikita from the PM team and we'll talk about the new feature that went public preview last month.

Aleksandar Nikolic joined the meeting

John Shacklock (ITC) left the meeting

Joel Stidley joined the meeting

Nikita Bajaj   8:55
That's the automation runtime environment, and along with this feature we have also introduced support for.

Yon Dale GCR FIII121 joined the meeting

Ben Kahn joined the meeting

Nikita Bajaj   9:05
Support for Azure CLI commands in the runbooks so of.

Robin Greenback joined the meeting

Nikita Bajaj   9:11
Sorry, I think my laptop is giving you some problem just a second.

Trent Blackburn joined the meeting

Nate Kasco joined the meeting

Sanjeev Kumar   9:37
Yeah.
And they could.
Do you want me to share that?

Nikita Bajaj   9:38
OK.
You still see?
Ohh yeah, is it still visible?
Sanjeev, can you just confirm?

Sanjeev Kumar   9:44
Umm, not yeah.
Yeah, it's visible now.

Steven Bucher   9:46
But there we go.
Yeah, I can see it as well.

Nikita Bajaj   9:47
OK.

Jack Davis left the meeting

Nikita Bajaj   9:47
Alright, fine.
Thank you.
So let me just talk about the motivation behind releasing this feature.
Uh, we heard from a lot of customers that it was difficult to upgrade the RUNBOOKS to the latest versions or customers who wanted to keep the names of the runbook saying they had to create multiple automation accounts or if there were dependencies between different modules because of which there were breaking changes within the runbooks, then also customers had to create very multiple accounts.

French, Marty joined the meeting

Nikita Bajaj   10:19
So we have tried to resolve all those concerns through through this feature.

Shenoy, Nithin joined the meeting

Nikita Bajaj   10:25
That's the runtime environment and.

Amir Terani joined the meeting

Nikita Bajaj   10:30
OK, so the partial 7.2 run book, which was well, yeah, the components of the partial seven or two runbook are actually crypt.
That's the script code the the language it's partial, the runtime version.
That's 7.2 and the packages that were required during the job execution.
Now this was the old experience which was required.
Uh, which was the hampering the the the upgrade of Runbooks to the latest version.

Bryan Gritton left the meeting

Nikita Bajaj   11:03
So now what we have done is we have decoupled the script code with the rest of the components and the runtime environment.
Now includes the language that is power.
That could be partial or Python And runtime version could be for example 7.2 for partial or 7.4 and the packages could be the default ones which are easy partial in Azure CLI as well as the custom packages which you can import from the gallery.
Ohh so there is a slight change in the experience when you create a runbook.
However, all the existing runbooks in your automation account would be available in the new experience and these are linked to these system created runtime environments.

Bruno joined the meeting

Nikita Bajaj   11:52
So which are created by uh by the service itself, so that you don't have to go through the hassle of upgrading, moving all the existing runbooks to the new experience.
So let me just quickly cover what have been the key benefits of this feature.

Patrick Wahlmueller joined the meeting

Nikita Bajaj   12:08
Now you have complete control over the job execution environment, because now you can define the language, the runtime version as well as all the modules that are required at runtime.
Ohh as I mentioned earlier, you're all existing RUNBOOKS would continue to show in the exist in the new experience the upgrade becomes very simple to the latest runtime version.

Brosent, Dominik joined the meeting

Nikita Bajaj   12:33
There is no change in the RUNBOOK required.
You can continue to manage it within the same automation account.
Also, if you find that there are some that the result of the upgrade was not as expected, you can roll back to the previous versions also.
Umm.
And because now you can segregate different modules across different runtime environments, you can organize your code within the same account.
Very efficiently.
Uh, and the Azure CLI command support, which is one of the key highlights of this feature, and it was, uh, asked by a lot of our customers.

Watkins, Jeremy [USA] joined the meeting

Nikita Bajaj   13:11
So partial as well as Azure CLI command now proved to be a very partial tool.
A powerful tool to to manage your Azure resources and last but not the least, we would be able to reduce the time gap between the partial releases as well as their support in runbooks in Azure automation.
So a bunch of key benefits with this feature and let me quickly cover the road map towards gas.
So we would be supporting all kind of run books as well as we'll introduce support for hybrid jobs also very soon.

Matthew Sanders left the meeting

Nikita Bajaj   13:50
Uh, you would be able to configure runtime environment through the automation extension for VS code.
Umm, there would be capability to clone the runtime environment.
If you have already have an existing runtime environment, you would be able to copy it and make just incremental changes to it and link it to a new runbook to to just simply upgrade.

Hunter, Brandon joined the meeting

Nikita Bajaj   14:10
So that would be a very simplified experience.

Meelis Nigols joined the meeting

Nikita Bajaj   14:13
Again, the support for bicep ARM template commandlet coming up and we would also have our back permissions for the runtime environment.
So I would like to hear from any of you anybody has tried D this feature and would like to share your feedback.

Vivian Thiebaut joined the meeting

Nikita Bajaj   14:33
Would like to come.
Would you like to unmute yourself or you can just post your feedback on the chat.
Uh, we would be glad to know how are we faring till now?
Uh.
Also, if not right now, you can just drop an email to ask azure.automation@microsoft.com and reach out to us whenever you try out this feature.

Gilbert joined the meeting

François-Xavier Cat joined the meeting

Nikita Bajaj   15:02
Anyone willing to share any feedback for this feature?
So I'll just monitor, just chat for a few seconds.

Sanjeev Kumar   15:12
Period.
There is one ask for Azure CLI support on Python, so we are currently supporting for PowerShell as of now, Python is on the way so it will end soon.

Justin Ekpa joined the meeting

Steven Bucher   15:30
Nikita, could you, could you send the the where folks can contact you in the chat just so those have a record of that in case they they come up with something different or come up with something later.

Nikita Bajaj   15:37
Yeah, sure.
I can do that.

Steven Bucher   15:43
Looks like there's some stuff in the chat as well.
I think I saw something in the Q&A tab of the teams.
Ohm let you take a look at that.

Strong, Jeremiah left the meeting

Nikita Bajaj   16:01
Thank you folks.
I do see a lot of good feedback here on the chat and uh, for those who have not tried it, I would encourage you to please use it and share feedback with us.

FloH96 joined the meeting

Shenoy, Nithin left the meeting

Tyrone Roberson joined the meeting

Nikita Bajaj   16:15
Good as well as bad, both are well.
OK.
Thank you very much for your time.
Thanks Steven for giving us the opportunity.

Steven Bucher   16:27
Awesome.
Thank you so much.
Nikita and Sanjeev looks like there's some activity in the chat.

Vin Latus (ISD) joined the meeting

Steven Bucher   16:33
So that we can keep the conversation going there.
So as far as the next agenda item, let me just get my screen together.

Shenoy, Nithin joined the meeting

Steven Bucher   16:42
So we're going to spend some time kind of walking through the dev 2024 team investment blog.
If you haven't had a chance to see it, there's a link to the blog that Steve posted.
We'll kind of be going through one by one just to talk briefly about each of these items.
Well, First off, we'll first start off with a detail on PowerShell 75AD tech community.
Uh, you're gonna give brief athletes around that?

Aditya Patwardhan   17:11
Yep.
Hello everybody I am Aditya so as most of you might have already known PowerShell document released a .net 9 preview.

Strong, Jeremiah joined the meeting

Aditya Patwardhan   17:21
One uh earlier this week.
So and on based on that, we'll be releasing a new PowerShell preview coming up very shortly.
And we are going to attempt to release a preview of PowerShell based on a newer version of net previews almost every month.
Uh, so and the the ship cycle for power PowerShell 75 is going to be similar to previous years where we follow on the releases of previews every month and then have arc and we'll be targeting AG later in the year after .net releases the GA for net nine uh.

Brooks, Ron joined the meeting

Aditya Patwardhan   18:01
So that's more.
That's all about the updates for 7/5 for now.

Steven Bucher   18:10
There's a detour.
Patrick, you on do you wanna talk a little bit about Pseudoterminal support?

Patrick Meinecke   18:15
Yeah.
So one of the things we're working on is utilizing pseudoterminals for native executables.
So the the reason that we're looking into that is because in order to read standard out or standard error from an executable, you need to redirect it.

FloH96 left the meeting

Patrick Meinecke   18:33
Typically, but when you do that, it signals to the application that it should act differently, like it can't do anything interactive.
It can't.
It usually doesn't write colors.
That kind of thing.
So if you want to.
Examine the output, but not necessarily consume it.
There isn't typically a way to do that.
But we're going to be using pseudoterminals to give us a way to do that without telling the app that it's redirected.

Nikita Bajaj left the meeting

Patrick Meinecke   19:06
So when it first comes first makes it to a a release like a preview release, it probably won't look all that different, but over time it enables a lot of different things.
Like we could have feedback providers look at a command's output without actually consuming it.
We can.
Uh, like scan the output for markers that say it has JSON content?
So maybe we could, and this is a.
Stuff that we're going to try to do, but it's just to give you an idea.
We could scan it, see that it's JSON and then convert it to a formatted object automatically.

Ian Martin left the meeting

Joel Bennett joined the meeting

Patrick Meinecke   19:54
Another thing that it helps with is we have to completely reimplement the.
System that diagnostic, that process API.
In order to do this so some of the stuff that would previously be deemed to too much work to do, like hooking up the pipes directly for output, that becomes at least feasible to do.

at joined the meeting

at left the meeting

Steven Bucher   20:28
Cool.

Ian Martin joined the meeting

Steven Bucher   20:30
Thanks Patrick.
Jason, would you like to talk about some platform support stuff?

Jason Helmick   20:36
Why, sure, why not?
So hey folks, think about or go to and take a look at the PowerShell release page on GitHub and if you go down to the assets where you've all been, you see this giant list of Windows, Mac, and a variety of flavors of Linux.

Ben Lange joined the meeting

Jason Helmick   20:53
It's kind of like a candy store.
Everything you could possibly want these are this is our image distributions.
Well, this process of building testing all of these distributions has gotten really complex because you'll notice that with every version there's a whole bunch of architectures we need to keep all of this updated.
This is kind of gotten ahead of us.
So what we're doing this year is we're reworking this entire process.
And so you'll notice for a few months we might be a little bit laggy, but our intention is to rework this process and to improve our tests, our documentation and the infrastructure to which we build on.
See we not Microsoft, but we the PowerShell team build all of this ourselves, including all of the infrastructure that goes with it.
So it's pretty complicated now.
One of the things that we're going to be doing, and I hope that you'll be able to see this publicly in the next few months that we're also going to be creating a backlog of distributions along a prioritized backlog along with some estimates.
So you'll be able to see publicly when we plan to be able to work on a specific distro and to get it out.
Along with that, we're improving our testing processes and our automation processes so we can spend more time dialing into issues where we need to rather than fighting issues on our pipelines.
So we expect to see some changes and expect to see some improvements over this year as we go through this process.
Thanks Steven.

Steven Bucher   22:19
Cool.
Thank you, Jason.
Sydney, you wanna talk about some bug fixes and community PR's?

Sydney Smith (SHE/HER)   22:26
Yeah, absolutely.
So alongside the PowerShell setting 5 milestone, one thing we've been thinking about talking about and sort of a goal of ours is for PowerShell 75 to really be a community focused release.
So one thing about PowerShell 75 is that it's not an LTS release, so we kind of alternate.
So PowerShell 74, the most recent release was LTTS.
PowerShell 76 will be LTTS.
PowerShell 75 is not, and so while the quality bar is still extremely high with PowerShell 75, this does introduce a little bit more flexibility to maybe get in more changes.
More feature requests, those sorts of things, and we really want to in this cycle, prioritize the existing PR's from the community over trying to get in.
Sort of.
Maybe new tiers from the team, so that is a goal of ours.
I will say that this is a list of things we're planning to invest in.
This isn't a list of things that we're patting ourselves on the back on the saying like we're doing a really great job, one at the moment, totally recognize that the the open PR's in the PowerShell repo are quite art right now and some folks have been waiting quite a long time to get their PR reviewed.

Segura, David joined the meeting

Sydney Smith (SHE/HER)   23:44
So this is something we're hoping to focus on in the next milestone, not necessarily saying that this is something we've been doing a great job on.
One other question that came up around this that I just wanted to kind of directly address was like, hey, so one way our team does this is we do something called Community Days on Mondays.
And so the way community days work, it's basically everyone on the team, works in open source and one way or the other.
And so we like to spend Mondays really focusing on triaging issues and reviewing PR's and whatever repose we spend the most time in.
So for a lot of folks on the team that's in the PowerShell repo, but for some other folks on the team that may involve working in other repos as well, there was a question that came up that was like, hey, are you still doing community Mondays?
Like I haven't been seeing as much action, sleep, and totally fair question.
The short answer is like yes, absolutely.
We are still doing Community Mondays.
Our calendars are still blocked off, and that's still the priority on Mondays.
The longer answer is that they're like there might be some reasons why you've seen less action, so to speak, over the past couple months that are just like various things that like you as in Community member like hopefully shouldn't have to worry about things like the holy day.
Is there like people needing to take personal time or just various like resources getting shifted to other repos or projects?

Segura, David left the meeting

Sydney Smith (SHE/HER)   25:15
Hopefully you'll see that get sorted out and kind of stabilize shortly.
Here, we're kind of reaching more of a normal point after some fluctuations around the holidays, but we can kind of keep pulls on that and keep checking in during community calls.
So feel free to keep the questions coming on that, but the short answer is like, yeah, we're still doing community days and it is a focus for seven, five.
So that's my long answer on community.
I'll pass it over to Aditya.
Have to talk about gallery fundamentals.

Aditya Patwardhan   25:47
Hello so for the next period of time will be focusing on improving the reliability of gallery, uh and making improvements in the back end to make it more reliable, more performant.
There might be some features which would come in on at the later part of the year, but we don't have any details to share right now, but we might share some details as as we are closer to that time period.
But the focus for gather is going to be making improvements on its reliability.
That's all I have.

Steven Bucher   26:25
Well, I think on them you're next to chat about ohh it's PS resource gets stuff.

Anam Navied   26:32
Yes.
Uh, thank you Steven.
Umm, so our goal I get this.
Our goal for PS Research guide for the upcoming semester and year is to add support for container registries, both public and private.
This is gonna allow enterprise customers to have a repository for trusted and untrusted content.

Mike O'Neill joined the meeting

Anam Navied   26:54
Since PS Gallery by default is untrusted, so they'll be able to have or, we'll be able to have a Microsoft trusted repository, while PS Gallery is still the untrusted repository by default and so in addition to having a Microsoft transferred repository, it will also give enterprise customers the options to have a private gallery and that will allow them to basically determine their package availability needs for their environments.

Amber Erickson joined the meeting

Anam Navied   27:24
So an example for this would be like air gap solutions or other situations based on like what is needed for the team for their galleries.
So yeah, look forward to that change in an upcoming preview release.

Steven Bucher   27:41
Something's on.
We'll pass it over to Amber to talk a little bit about concurrent installs and local caching.
To Umberto here.

Amber Erickson   27:53
Hi sorry I've been having some computer issues today.
So can you hear me OK?

Steven Bucher   27:59
Yep.

Amber Erickson   28:00
OK.
Yeah.
Awesome.
So we have been trying to work on improving the experience of installs in general, especially with modules like AZ that have a ton of dependencies, a few dozen dependencies.
So basically I specifically have been looking through ways to optimize specifically the find and install experience.
Especially because find initially or install initially calls find to look for the dependencies.
That is something I don't know specifically like what else I want to talk about with concurrency, but it's improvements that we are trying to make for an upcoming release kind of separate from the container works container work that we're doing, but we should have some vast improvements on that within the next couple of releases.
That's and that's all.

Steven Bucher   29:07
Could do you wanna share a little bit about the local caching as well or the?

Amber Erickson   29:14
Local caching?
Umm sure.
Yeah, I can talk about that a little bit.

Patrick Hoban joined the meeting

Amber Erickson   29:17
We have not started work on that yet, but essentially what we want to do in the near future is to set up local caches so that it can also improve the experience of finding and installing a bit faster and without having to make frequent network connections, which can be a bottleneck, especially with the gallery at times.

Steven Bucher   29:42
Yeah.
Another thing I'll I'll just add to it to the another thing that this enables is a potential for a feedback provider to actually utilize the gallery as a source.
So if you type command commandlets that aren't there, we can utilize this cache to suggest to install the particular modules that the commandlets are coming from.

Amber Erickson   29:52
Yeah, good point.

Segura, David joined the meeting

Steven Bucher   30:03
So that's a long term goal.
We have of that area, but like Amber said, it's something that we're looking at in the work has not started yet, but it's something we want to get to this year.

Yon Dale GCR FIII121 left the meeting

Steven Bucher   30:12
Most of the stuff like we're talking about here are are forward thinking and and I guess the next item is mine, which is intelligence in the shell.
So the we're we're still committed to bringing a deeper level of intelligence into the shell through improvements to PS, Reline as well as other additional improvements we can think about that is not just bringing natural language chat experiences to the shell, not too much to share there.
But I just wanted to call out that we are still very interested in investing in that area.
From there, I'll pass it over to Michael to talk about configuration.
Where you you hear Michael?

Michael Greene   30:55
Like I'm here.
Yeah, things are going well.
I just want to encourage everybody to go take a look at the repo.
It's github.com/powershell, DSC.
Uh.
And it just kind of go through the issues list and look for scenarios that are important to you and give us feedback.
And that's really all we're looking for at this point in time.
We're hoping to have another release within the next couple of weeks that will address some common issues we've been running into and validation, but at this point I think the best way to stay involved in that project is in the issues list.
There are quite a few active discussions happening over there, from everything related to grouping of Resources, Secrets management, orchestration of imperative code combined with declarative code, and you can you can imagine all the different scenarios.

Greg Malleus left the meeting

Michael Greene   31:49
But look forward to hearing from anybody who's interested.
I think is probably it.

Steven Bucher   32:00
Cool.

Aditya Patwardhan left the meeting

Steven Bucher   32:00
I think Danny is up next to talk a little bit about some open SSH stuff.

Danny Maertens   32:05
I am so open SSH.
We have some a few exciting things on the way that we're not allowed to talk about yet, but stay tuned.
Probably over the next few weeks, but besides that it's it's just kind of renewing a let's take the commitment we have with openness stage, that being their preferred remoting protocol for PowerShell remoting going forward and even for really all of the Windows Server workloads you should be moving towards say command line centric remoting, not using RDP really looking at using black or opening stage.
Uh, and then kind of building on the the partnerships that we already have kind of within Microsoft, whether that be with VS code.
Andy just released a 0.01 of using VS cover modding to connect to an arc machine.
So give that a try and and so I was like our partners like Bash, Azure, Bastion and how do we can be improve their SH and remoting experience there.
I know Tess is up after me and has more to share on a stage.
The only other thing that I'll call out is that there was a question added to our agenda about supporting states remoting for PowerShell with a password.
I think there is a lot of really good answers on the actual issue, but just so the high level passwords over SSH are really only intended to be used interactively and so when you create that stage section, you get prompted by SSH for that password and say there's a lot of concerns from the open SSH itself and open BSD foundation and really just security in general around, I'd say noninteractive passwords for us, SSH and how those are stored and the security implications of a password versus the key.
So I'm happy to people wanna ask more questions on the chat, but I'll keep it, move it along to test.

Tess Gauthier   34:13
Thanks, Danny.
Yeah.
So I'm just going to briefly talk about SSHD config, which is going to be a DSC resource that allows you to manage the SSHD config file.
For those that don't know, the SSHD config file is just a text file that controls the settings for the open SSH server.
So if you're just managing a single open message server, you might modify that file directly, but there are a lot of directives and a lot of arguments that are within that file.

Ben Kahn left the meeting

Tess Gauthier   34:44
So if you're managing multiple open SSH servers at scale, that can become a cumbersome process.

Patrick Wahlmueller left the meeting

Tess Gauthier   34:52
So we're trying to hopefully simplify that the first scenario is within DSC terminology.
More of a get scenario to just look at what the existing configuration is for the open SSH server and then ultimately the end state is a full get set and test scenario to be able to manage that.
So uh config file.
This is more of a backlog item for us at the moment.
Just full transparency.
So I think it might have even been mentioned in the team investments last year, but we do hope to get to this later in the year.

Reiswig, Darwin E. left the meeting

Reiswig, Darwin E. joined the meeting

Steven Bucher   35:33
Awesome.
Vicky tests Jason.
Do you wanna talk a little bit about help system?
And I think that sort of wraps up the the rest of the the blog public, so.

Jason Helmick   35:43
What?
What we're getting close, aren't we?
Yeah, sure.
So, folks, you you may have heard of this great little tool that we have called Platy PS and this tool lets you do the same thing that we do, which is create updatable help for your PowerShell commandlet stuff like that.
I don't know if you know this or not, but we also use Platt EPS internally at Microsoft.
It actually produces all the documentation for our reference pipeline.
Basically everything you see from Microsoft on the commandlets is made with Platt EPS.

Jonas Sommer Nielsen joined the meeting

Jason Helmick   36:11
Well, recently we started a new initiative with plant apps.
We're working with our internal partner on the dots team to tied more details and more information in your PowerShell documentation, which is a great thing.

Izzy left the meeting

Jason Helmick   36:23
So we're redoing Platy PS from the ground up a new schema, a new architecture.
We've got great engineering on this from a detia Jim and Jean Wheeler's providing a lot of the, the, the, the little finite details that we need to make sure that we get right while we're doing this for the internal partner, which is our our internal reference pipeline, we're going to help them start to convert some of those pipelines like the PowerShell one and some of the other teams pipelines just to make sure we've got it right.
So after we're done dogfooding it, I'm going to start writing up scenarios for a public release that will do later in the year of Platte.
Has which I'm really excited about.
Umm, with this new architecture.
So look forward For more information as we start producing things around new scenarios.
It'll do more than just make up datable.
Help.
You might find it to do all kinds of neat conversions for you very quickly, from markdown to YAML to mammal and back and forth, and validate it.
It's pretty cool and you'll find a lot of neat uses for it, so we're looking forward to this over this year as we work on this within our internal partner and then bringing me bringing it to you publicly when we've gotten it all nailed down.

Steve Mutungi (HE/HIM) left the meeting

Jason Helmick   37:34
So thanks a lot.
Thanks Steven.

Steven Bucher   37:37
Cool.
Thank you, Jason.
The last thing we'll just kind of call out from the blog is that we're still investing in maintaining a bunch of our kind of tangential projects outside of just the core PowerShell engine.

Patrick Wahlmueller joined the meeting

Steven Bucher   37:53
And this includes stuff like the VS code extension, Pscript Analyzer console, GUI tools module, text utility module, PS READLINE module and secret management module.
We're still very much.
Uh.

Thomas Marantz joined the meeting

Steven Bucher   38:08
Excuse me, we are still very much spending time maintaining these these these repositories having a decisional releases and monitoring issues and PR's in those repos.
Of course, that's not limited to that list.
Those are just some of the key ones that we we we look at but yeah.

Ben Kahn joined the meeting

Steven Bucher   38:30
Cool.
I think that covers everything to do with the blog, so if anyone has any phone questions with you know anything that we talked about, you feel free to add in the chat.
So next item on our list, let me just get the agenda back up is to welcome a new member of our team.
Justin Justin Chung is in intern joining us.
Just are you here?
Do you want to give a brief introduction?

Justin Chung   39:01
Yes, hello everyone.
I'm Justin Chung.
I'm the New Leap intern over at PowerShell and Open SSH Ohm.
I'm working with Dongbo and Steve and I'm really excited to be here and add value to these projects and these these teams and this community look forward to working with y'all and yeah, I I guess that's it.

Steven Bucher   39:30
Cool.
Thank you, Justin.
I mean we're we're we're super excited to have.

Justin Chung   39:32
You know.

Steven Bucher   39:33
Justin.
Justin just started with us this week, and so I just wanted to bring him on and show them everything to do with the PowerShell community so.

Justin Chung   39:36
Umm.

Steven Bucher   39:46
Cool.
Uh, Next up.
Is Sean with some docs updates?

Sean Wheeler   39:54
Yeah.
So umm.

James Petty joined the meeting

Sean Wheeler   39:59
Let me share my screen.
Not a whole lot new to talk about in the past month we did with with the release of PowerShell 75 Preview One we have documentation for that now.

Patrick Wahlmueller left the meeting

Sean Wheeler   40:15
So if you go to the version selector here, you'll see it at the top 7, not 5 preview.
So we'll continue to keep that up to date with each release.
And there's been a ton of updates in the DSC documentation for DS V3.
Check out the change log article there for what's new.
Mikey Lombardi has been working really hard on that documentation and.
There's, I think, a new alpha scheduled pretty soon, so they'll be a lot more changes.

Rhett Hartman left the meeting

Sean Wheeler   40:54
He's done some reorganization of the documentation as well to help it.
Flow better, so hopefully you can take a look at that and then in other news, umm uh, I'm not responsible for the Windows PowerShell module documentation, but I wanted to point out that they have.

Jan Egil Ring joined the meeting

Sean Wheeler   41:21
Released the documentation for server 2025 in preview here and I've been working with the folks that that maintain this and working with the the Windows team a little bit to, umm, clean up some of the documentation here and fix some of the problems with update help for some of these modules.
We're not going to be able to fix all of them.
It's still a work in progress, but it's getting better with each iteration, and like I said, this is not.
I don't have responsibility for this content, but I'm trying to help out where I can and so we should see some improvements there and I'll hand it back to you, Stephen.

Steven Bucher   42:10
Cool.
Thanks so much, Sean.
Jason, do you wanna talk about some updates to be working group application process?

Jason Helmick   42:21
Why?
Why, sure, it's me again.
So the the working groups help both the community and the PowerShell team triage and move issues along, and so we we really appreciate when Community folks can invest time and help out these working groups.
We've been asking for new members to work in groups and it gets hard for us to track it in a particular issue.
I've noticed that we've kind of missed a few folks and been kind of late on some things.
So what we're improving that process as well.
Essentially, we're creating a new form, a new form for that you can fill out to get membership into a work group, and I'm going to show you that form and how to use it in just a second.
It'll be on GitHub.
It'll be pretty easy.
It'll be a new issue for them to fill out.
The idea is fill out the form and and that'll go to the working groups when the working groups meet, they'll go through, see if they have any membership forms, and then they'll discuss and respond accordingly.
So I guess kind of what I'm saying is, is that even when the form goes up, don't expect an immediate response because the working group has to meet some working groups, meet on different schedules.
I I know that the interactive working group meets almost weekly, but some working groups meet once a month, once every two months, so you got to give him some time to get the issue in and then respond.
But we really look forward to folks joining.

Harms, Greg J joined the meeting

Jason Helmick   43:50
So let me show you that form really quick.
Let's see if I can figure out how to share my screen and get the right screen up.
Uh, well, let's try this button.
This one seems to work.
Ohh hey that's it.
I think that's it.
Yeah, that's it.
OK, cool.
So this PR hasn't closed yet.
It will close in the next couple of days.
When it does, this form will become available.

Shahan Karim joined the meeting

Jason Helmick   44:09
I'll show you where to get it in just a second, but let me just show you the form I'm going to go over to preview.
So excuse my little preview mess, but essentially when the form comes up, we really appreciate you coming in.
Just wanted to give you a note that not all of the working groups at this moment are accepting new members, but the list of the ones that are are listed here.
So you can select what I know, it's kind of grayed out.
You can't see things very well right now.
Umm, we will change this list as as working groups have their membership open up.
We have a question for you.
Can you provide at least an hour of work a week to that working group now down here?
And I've given you a warning.
This is a public forum.
This will be on GitHub, so we have some questions for you.

JH (Guest) joined the meeting

Jason Helmick   44:49
Give us some information.
Keep in mind that it's gonna be public.
Why do you wanna join the group?
What skills do you bring to the group and can you give us some references to links, articles, code issues that you've worked on, that kind of thing?
Fill this out and when you send this, the working groups will pick it up.
How do you fill this out and how do you send it?
Well, you go to issues and when you're in issues, if you go to new issue you will see it, although you don't see it today, you will see it on this list as the working group membership for.

Eric M left the meeting

Jason Helmick   45:21
So let me kill that.
So there you go.
We look forward to having new members, umm, a file and join and we look forward to talking with you.
So thanks Steven.

Aric Bernard joined the meeting

Steven Bucher   45:34
Cool.
Thank you, Jason.
So one last item from us is around the March community call.
So we've had a couple requests from folks who wanna join the community called Live who are in a time zone that this time is not particularly convenient for them.
And so to accommodate those folks in Asia and Australia and India, we are hoping to host the March community call at a different time than usual.
So we're going to try this out.
It will be hosted at 7:00 PM Pacific Standard Time that same Thursday, March 21st.
I believe so for most folks in that area, it will be the next day, probably March 22nd in the morning, but we wanted to just call that out to folks.
Of course, the normal recording process will be in place for those who.
And this made this time is not gonna be convenient for them.
But we're gonna give this a shot.
And if you know folks in any Asian time zones in Australia or India, umm or anywhere where you know the this this time doesn't work, please share the discussion tab with them and we would love for them to join and yeah, so the current idea thinking was do these kind of calls once every six months, so be March and then September, we'll see how this March one goes and we'll kind of keep folks updated but just a call out that the next month's community call will not be hosted at this time.
It will be hosted much later in the day.
So yeah, I think that covers all of our teams agenda items.
I think we've, we've answered some of the questions.
There was one question in the repo around PS Resource Git V3 dependency support.
I don't know, Sidney, if you wanna talk to that or just respond to that issue, ohm.

Domansky, Mark left the meeting

Sydney Smith (SHE/HER)   47:35
Yeah, I can.
I can address it just like quickly a little bit, and then Amber.
And then if you have, if you guys wanna add anything as well.
So I'll say that like I guess I believe that the the issue is essentially so with PS resource get we support a number of different types of repositories and pstree source or in PowerShell get umm like be be too there was only one type of repository supported.
Can you get Me 2 feeds?
And that's what the PowerShell gallery is.
It's a Nugent V2 feet.
That's the type of endpoint it is with PS resource gap.
We support a whole bunch of other types of endpoints, one of them being nugget B3 endpoints.

Oleg joined the meeting

Sydney Smith (SHE/HER)   48:20
With that, there's some challenges.
Uh, in particular, to challenges I believe are called out in this issue.
One is dependency support and another is like find all support.
Uh, the short answer is it would be we would love to support dependencies we plan to.
We hope to we also are accepting PR, so if you are interested in opening a PR to add dependency support, we would be happy to review that and take a look at that.

JH (Guest) left the meeting

Sydney Smith (SHE/HER)   48:51
There are some challenges that make it a little tricky to add both dependency support and find all support.

Jason Walker (AZURE) left the meeting

Sydney Smith (SHE/HER)   49:05
From repositories like Nugget, I don't know if.
Ah, Justin supports it.
And ammer amber.
You wanna speak to some of those challenges?

Anam Navied   49:21
From yeah, I can take a stab at it with regards to dependencies.

Sydney Smith (SHE/HER)   49:24
Sure.

Anam Navied   49:26
The way it's done on Nugget Gallery is a bit different than the way it's done in PowerShell Gallery and so the dependencies are definitely listed in terms of like the frameworks they support or like you were supporting.

McGovern, Sean left the meeting

Anam Navied   49:38
Or sorry if you want to get the package and you want support for net standard 2.0 versus like that 472.
It dependencies can differ and I think on PS resource get side we just haven't gotten around to implementing and kind of navigating that and the scenario would find all is mostly just kind of like traversing the index and getting all the packages.
There wasn't a direct query to doing that, so you would just kind of have to traverse the resources and we haven't done that yet.
But we've also tried to make a note in the docs that we don't support it. Umm.
Uh to I guess then we have other V3 repositories like ADO where they are using like a nugget V3 API support.
But in those cases they have upstream repositories, so it gets a little confusing in terms of like when we're getting all the packages for your specific feed.
For ADO, we want to make sure that we don't also try and go into, reverse the upstream feeds, because then that would be a pretty high workload and maybe kind of slow.
So those are things that we have kind of put for future work that we do hope to get to and I will take also a look at the uh comments in the chat and kind of follow up with that too for how we can kind of look to do that support, yeah.

Sydney Smith (SHE/HER)   51:02
Yeah, we can take a look at just in your module 2 module that and see if we can do anything we pull from there.

Anam Navied   51:05
Yeah.

Sydney Smith (SHE/HER)   51:07
But also I noticed the issue hasn't been triaged and I will apologize for that.
I'm definitely triage it.
Put it in the back box, something we have to support.
Just have a quick gotten to.
Yeah, but yeah, hopefully that gets a little bit more of an understanding.

Anam Navied   51:16
Yeah.

Steven Bucher   51:26
Cool.
Thanks Sandy. Umm.
So we're at the end of our agenda.
If anyone has any questions or demos they'd like to share, feel free to add in the chat.
No, no, Jason, I I saw a couple questions around cloud show.
Anything you wanted to update around that?

Harms, Greg J left the meeting

Steven Bucher   51:57
I think you might be having audio issues I.

Jason Helmick   52:01
Let me try this again.
How's that?

Steven Bucher   52:03
Yep, I can hear you now.

Jason Helmick   52:04
OK.
Thanks.
Sorry, I was typing this in.
Sorry, Alexander II typed so slow.
I was trying to type this in to answer you. Yeah.
So cloud Shell has transitioned to a much larger team now called cloud consoles.
The new PM and I will put this in for you into the chat.
The new PM for that is his name is Mitchell Byfield and he'll be happy to answer questions for you.

Aric Bernard left the meeting

Jason Helmick   52:26
But I can definitely tell you this, they are getting ready to release a ephemeral sessions and the new UX.
I think you may have even seen the documentation published because she and I did get the documentation out and there are I.
I'm not sure what their exact release date is for that, but it's it's really soon now and they have some new things on the horizon as well.

James Petty left the meeting

Jason Helmick   52:46
Since they are now a bigger team working with serial console as well, expect to see some great things from them.
So contact Mitchell and he'll keep you up to date. Thanks.

Steven Bucher   52:57
Cool.
Thank you, Jason.
I see in the chat too.
James posted some stuff about the tickets are still available for the PowerShell and DevOps Global Summit on April 8th through 11th, so feel free to reach out to John Janell for for more details.

Oleg left the meeting

Steven Bucher   53:12
If you are interested.
Umm, cool, let me just double check this.
Yeah, I think we covered everything there.
Ohh.
Sure.
Yeah.
Justin, do you wanna do a quick demo of your update help fast.
I think you're muted.

Silva, Rodrigo N left the meeting

Grote, Justin   53:45
Really likes to move that around on me once I start the presentation.
Sorry.
So yeah, hey, everybody.
So I just have a just kind of a quick demo of just a little script that I made that when updating help, especially across a lot of commandlets, it's currently done kind of serially.

Keith Hill left the meeting

Michael Anstadt left the meeting

Grote, Justin   53:58
So I just basically very simply apparel added parallel to the process.
So this little link just simply links to a GitHub gist that runs it, but when you run this, it's just gonna go through and automatically grab all the different helps and get them.
But do them in a parallel fashion.
So just like that in 6.3 seconds I just updated all the help on my system, so this all brings us the script up.

Michael Anstadt joined the meeting

Grote, Justin   54:22
So you can see where it is, but you can also that just basically links to it, but it's not a terribly complicated script, it's just simply a matter of taking the update.
Help command and running it through some thread jobs and just kind of a real simple example of doing that as well as reporting on progress while doing something like that.
So just a real simple quick demo, but an easy way to kind of get some real quick wins using parallelization.
So thanks.

Steven Bucher   54:50
Awesome.
You just sent.
I see.
Ryan is talking about Ryan.
Do you wanna speak to that or?
Looks like the PowerShell UK user group as a meet up on the 29th and will be announced.

Shenoy, Nithin left the meeting

Steven Bucher   55:35
And the next few days on Twitter.

Hunter, Brandon left the meeting

Steven Bucher   55:39
Thanks for sharing that, Ryan.

Michael Greene left the meeting

Michael Anstadt left the meeting

Steven Bucher   56:12
Yeah, well, yeah.

Nate Kasco left the meeting

Steven Bucher   56:14
If anyone has any more questions or things you'd like to demo, we have a couple more minutes left.

Michael Greene joined the meeting

Steven Bucher   56:20
Otherwise, we can always end a little early.
I'll just pause again in case anyone's type in or.
Yeah, anyone is welcome to come off mute as well.

Watkins, Jeremy [USA] left the meeting

Mullens, Justin left the meeting

Bruno left the meeting

Vin Latus (ISD) left the meeting

Jon Junell   57:03
Hey Stephen, I didn't want to leave you alone.
And I I love cameras.
So hi everybody, I'm John Janelle.
I'm from the chat, but I wanted everybody to encourage them to take a look at the PowerShell Summit schedule.
Monday is a little bit different format wise.
We have two groups of pretty long sections that deep dive into a topic.
So I encourage you to take a look at that.
We're back in the Maiden Bauer Center in Bellevue this year.
I'm the party planner Ish, so I'm kind of working on Axe throwing on the side.
So if you're really interested in that and some PowerShell, uh, encourage you to come out again.
I left my email in the chat, so if you have any questions or about logistics and available via Slack and all the usual ways and folks know how to get ahold of me.
But I encourage you to take a look or if you have someone on your team who's interested in learning about PowerShell for the first time, we do have the on ramp where it's a whole kind of like side track where people can come up to speed on PowerShell and get DevOps and all the fun things and get to meet members of the community and build a little bit of a network.

Shahan Karim left the meeting

Jon Junell   58:07
So anyway, wanted to plug that.
Have a fun time and I'm looking forward to like the 7:00 PM community call cuz that later is definitely a better time.
So anyway, good to see you, Steven.
See you around, folks.

Steven Bucher   58:20
Thanks so much, John.
Yeah, if if if anyone has the opportunity to to go to the PowerShell.

Brooks, Ron left the meeting

Steven Bucher   58:26
And DevOps Summit it's it's it's really great experience.
Lots of well for us on the PowerShell team.
We have a bunch of.
I don't know how many sessions we have, but I know we have a lot there to share some of the details there or details of what we're working on there.
And yeah, it's a great time to connect with the PowerShell community, so definitely very much looking forward to it.
I see there is some talk, some chatting about the Pacific PowerShell user group, the Metaclass name, and they'll also be doing another meeting.
So if you're in the Pacific area, maybe, maybe James or Chris could just spend some information on if, how folks could join or get involved with that user group in the chat.
That would be awesome.
And then kind of, yeah, I think we could probably and off here, ohm.

Patrick Hoban left the meeting

Steven Bucher   59:18
Cool.
Thanks, Chris.
And yeah, so yeah.
Thanks everyone for joining this month's community call.


Steven Bucher   59:26
Last reminder that March's community call will not be hosted at 9:30 AM Pacific Time will be hosted at 7:00 PM Pacific Time.

