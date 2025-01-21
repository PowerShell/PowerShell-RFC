# PowerShell_OpenSSH Community Call-20250116

January 16, 2025

 
Steven Bucher   2:32
Hello everyone.
We'll get started in a minute or two. I posted the agenda and.
Topics in the chat there, so feel free to add any additional questions, comments, demos, whatever you want to that issue. We'll be using that primarily to to guide this this meeting so.
Yeah. Welcome all. Happy New Years and.
We'll give it a minute or two before we get started.
I still see the number of participants.
Growing. So we'll we'll wait a little bit.
Cool, I think.
I think we can probably get started here.
We got a fairly big agenda for the New Year, so welcome all as as as John put it happy you know insert here it's good morning.
Good evening, good afternoon.
Wherever you are in the world, thank you for joining us at the first PowerShell Open SSH Community Call of 2025. We got a great agenda today.
I will post the link again in the chat here to follow along.
And actually well, since since I'm the first one, I guess I'll share my screen and.
You know we can.
We can kinda get started here.
Yeah. So I guess it's been a while since we've last had the community call and Alana has kinda happened since.
I think it was.
October was our last kind of more official one, the N1, I think landed on kind of AUS holiday and so a lot of us were out and it was also the week of Ignite.
But yeah, so we'll get started here.
So the first thing I'm actually going to be showing off is we had a new release of a new CLI tool that we've been developing for a while now called AI Shell.
This is a CLI tool designed to help connect your PowerShell and Shell instance to a variety of AI assistants.
This is something that we've been working on for a while to to help kind of bring assistance. We we've learned, you know a lot that using the CLI can be very daunting the you know, the number of commands and parameters alone are in a couple you know th.
Of different kinds of permutations, different commands.
It's hard to know everything.
It's hard to know exactly how to get the job done, especially for for.
Beginners to the CLI as well as folks trying to.
Kind of scale out their their businesses.
And automate their task.
It can be challenging and so.
We recognize that and we've been working to try to.
Figure out ways that we can make the experience a lot smoother and easier.
So we've created a simple cmdlet here called start AI shell that will bring up a side pane here to kind of be your assistant.
We really like the idea of someone at your work kind of helping you along the way in the CLI, and so we've developed this, this.
User experience here, but there's an alternate way that I'll talk about as well. We have a number of AI agents that we we call them, which are different forms of AI assistance, you know, in the CI you can run, of course, thousands and thousands of PowerShell commands you.
Can run native commands. You can run all sorts of different things and so it's not really super feasible to create a singular. You know AI model to to rule them all you know, so to speak.
And so we wanted to make this platform extensible.
So that you can build agents for modules that you build out. You can build an AI assistant based specifically on data that your company has built out for things and different kind of segments of areas.
And so there's two agents that we've developed, the open AI GPT agent and then the Azure agent, which connects you to the copilot in Azure.
I'll first start with the open AI1.
This is a very basic.
Agent that has no specific kind of training or anything.
All you have to do is configure.
With a Azure open AI instance or a public open AI key, I'll drop a lots of links to kind of walk you through how to do that in a moment here. But this is a very similar experience that you'd get from just opening up chat jpt in the.
Same window.
It's generic.
It definitely has.
It may have some flaws, but it can get you know the job done for a lot of more widespread commands and things like that. So.
How do I?
I can ask questions like what are the top 10?
Most CPU intensive processes.
So this will generate.
Oh well, it thinks I'm running a VM on my Mac, so it thinks I am on Mac.
How do I do this on Windows?
There we go.
So I'm able to get more PowerShell commands here and stuff so but a key difference between this that we're really focused on is the integration with with PowerShell here. So I can run these chat commands to help me interact with the code here and so I can run.
A command called Post and it will post the commands here directly into my my PowerShell window and so I can easily run them. I don't have to copy and paste.
I don't have to do a lot of different things that way, so it's a lot easier and stuff, but a big experience is taking the output and kind of acting upon it, doing some more testing and stuff with it.
So see here I have this olama app process that's running here.
I don't really need that right now, so I'm gonna up arrow key.
I'm gonna say hey invoke invoke AI shall help me stop the O Lama app process.
It's going to send this data now to the sidecar here and walk me through how to do that and you can see here it grab the correct ID here and so it knows OK.
Yeah, I can do it this way. I can go back code post.
I can also run a hotkey key command DD to just simply do it that way and.
I can stop the process.
I can then go up and check and Yep, looks like it's gone.
So that's kind of some of the flow that.
We are working on for AI shell to make it a lot more integrated with your PowerShell session.
One last thing that I'll call out too is how we we handle errors. And so I'll just run.
A.
I'll just totally mess up a, you know, a path here.
Something like this, where it's not gonna run, obviously and so.
It doesn't recognize that.
And then UCI have a course in error in the in the path here. So I we have there's another cmdlet that this module has which is called resolve dash error.
This will send the last error that you just had to the sidecar, which for whichever agent you have and so it can ingest the error object that you are working.
That was generated from the air and then help you resolve it this way.
So this is a pretty simple example, but it can be, you know, helpful to at least try to debug and see things. If the error message is not super descriptive.
So yeah, that's kind of some of the the integration pieces. The other big piece of this, this tool is the Azure agent.
So if you're familiar with the copilot in Azure, this is the same copilot that we're using here to help you generate.
The Azure CLI and Azure PowerShell commands right here in your CLI.
So what I can do is I can actually switch over and say hey at Azure I'm going to.
Switch over my agent here and say OK.
Yeah, it looks like I'm. I'm registered.
All you have to do is be logged into Azure with Azure CLI.
We have Azure PowerShell coming and.
Have access to this copilot. Which?
Should be should be on for most folks.
And so now I can just ask questions of how do I create a resource group and?
An app insights instance.
So this this copilot is a lot more scoped to Azure commands. Azure CLI, Azure PowerShell and also general Azure knowledge.
So you can ask more broad questions about Azure Resources. You can ask more broad questions about what end to end Azure scenarios and but we're really trying to focus on Azure CLI, Azure PowerShell command generation.
So here's a great example.
Let's create a resource group.
Let's create an applications insights instance.
And so this is a multi step process here. It kind of says hey well look there's a bunch of placeholders that I have going that I have to fill out resource group name, location, app name.
Another location here.
One unique thing about this experience is we recognize these placeholder values and connect help walk you through how to fix them.
So I'm gonna run replace here and it will walk me through each of these parameters to help me get to a state where I can actually run these commands much more easily so you can see OK resource group name and say test RG.
Community.
Call.
And then here is another great thing that we've we've implemented which four particular parameters like location or SKUs or existing research groups. We try to populate or we will populate the the available values here.
So for location I can just kind of use. You know this is the same predictive Intellisense that you may be familiar with PowerShell.
And so we can kind of go through here and.
And I can say OK.
Well, let's see what kind of Europe ones I wanna do.
So let's do North Europe, for example.
We also try to be smart where we recognize what pattern you're kind of using previously and so we can say, OK, well, maybe you want to do test application Insights community for the name of the app name. So and then there we go. We now have more Exp.
You know, commands and stuff to to to run here I can use the same experience that I was showing off previously where I can go.
Control DD I'll post this one.
Step one is is put in here.
I'll hit enter here.
It will create me a resource group and I have not logged in there, but you can imagine this will run.

Jay Adams   14:55
Oh, yeah, yeah, it's fine.
You just have to turn it.

Steven Bucher   15:03
I think.
And then.
But one thing you'll notice is that we have the next command in your predictive Intellisense.
So it will.
Kind of walk you through step one, Step 2 and so on and so forth.
I just didn't log in correctly here, so that's OK though.
But yeah, that's that's a little bit about AI shell.
We have lots of documentation and blogs posted.
Of course, we're completely open source.
So you can take a look at the source code. We have documentation on how to create.
Agents and we recently just published documentation on how to deploy an Azure Open AI instance extremely easily with a bicep file that we've developed.
So you can deploy your own models and play around with that, but yeah, I'll drop all these links in the chat and yeah, excited for everyone to try it we.
Very early, you know, we're still in a public preview, so we're definitely changing a lot of things, but we'd love.
We love your your guys opinions on this tool, how we can make it better for you and and how we can improve it in the future here. So yeah.
Wonderful.
Thank you.
Let me post all the links in the chat and then I will.
Pass it over to Anum to talk about the PS resource.
Get release on him.
Are you Are you ready to to say about it?

Anam Navied   16:36
Yeah, yeah.
All righty, let me share my screen.
Hey everyone.
So we recently Gath PS resource get version 110 and this release included some bug fixes for repositories like artifactory and.
Azure DevOps where you may have PS galleries and upstream feed.
So we just wanted to ensure that that end to end experience is as seamless as possible and that also.
The error messages PS resource get gives is clear about whether an error is happening in the upstream feed or in your main feed.
This release also included some bug fixes for container registry repositories, so there's increased publishing support, and we've also made sure that any or when you get a package from the container registry repository, the metadata and the dependencies information you get is consistent with what you'd get if that.
Package was coming from.
PS gallery.
Yeah. And then one of our goals for container registry repositories is that first party teams like AZ can publish their modules, which are big, and they have a lot of dependencies and a lot of folks rely on them so they can publish those as modules to Microsoft art.
Registry or Mar and Mar is a public repository and it's a trusted source.
Of Microsoft published modules.
So you can come on here and you can find the easy module because the easy team has started onboarding to mar.
So I've already kind of searched in here and I can see that the AZ compute module is here and then I can use PS resource get to.
Find.
That package from Mar.
So I found easy compute. I can see that there is dependency on AZ accounts and then I can also install from the MAR Registry repository.
And this will also go and get.
So I'll get AZ compute and then we should also get AZ accounts, which is the dependency. If I can spell out.
So we've got AZ accounts as well.
So this basically integration with PS Resource get allows people to get AZ and other modules coming soon. An efficient in a reliable way from a Microsoft trusted source.
Yeah.
And that is my demo.
Oh, one second.
Let me go back. I'm sorry.
I think I did.
You were right.
I did do that. I'm sorry.
Let me download it from.
What I get for just doing a command that was already loaded in, let me get it from Mar.
Thank you for catching that.
It's already installed at this point.
Let me kill that install.
Sorry, let's take in a second.
Take me to setting.
OK.
There we go. So we can install it from Mar.
And then.
You can see that it installed it and then it should also have installed the dependency.
Yep, for easy accounts, yeah.
And that's my demo.

Steven Bucher   21:04
Awesome. Thank you so much, Anand.
I'm trying to answer every question I got in the chat but.
Feel free to re ask questions if I didn't answer any about the AI shell experience or, you know, feel free to put questions or even raise your hands during the during the.
During the call, let's see what is next.
OK PS gallery updates.
I know this is also a question that Justin brought up to Aditya and Amber.
Are you both here to to maybe address this or DT?
I think I see you.

Aditya Patwardhan   21:46
Yes. So I'd like to talk about some other updates specifically about the CDN. So as most of you might know.
Partial value used to use HIV as the CDN provider and recently they declared bankruptcy and they started going offline since yesterday.
So we moved to a different provider and over the last week we did the work of migrating it to a different provider.
And there are some hiccups while migrating over the weekend and hence there are some issues that were found, but I would like to confirm that we have completed the migration to the new provider and all of the traffic is serviced by the new provider, though the URLs are.
Still the same, so I do not expect any updates to any firewall rules or anything to allow any new URLs through but.
The way we did the migration was we started sending 20% of the traffic to the new server on Friday.
That's why the issue that were found were intermittent, because Hio was still servicing 80% of the requests and once we found the issues, we fix them and then eventually we rolled out 200% to the new provider and now everything is being serviced to it we.
Don't see any issues on our end, but if you find any issues.
For getting packages please open an issue on the gallery depot.
That's it.

Steve Lee (POWERSHELL)   23:14
Hey I did it before you finished.

Aditya Patwardhan   23:15
Yes.

Steve Lee (POWERSHELL)   23:16
Do you wanna quickly talk about the seven 6 preview?

Aditya Patwardhan   23:19
Yes. So the Seven 6 preview.
Happened today or yesterday.
So 76 preview two came out and the reason it's called Preview 2 is we tried to release a preview one in December and we had some unrequited issues for publishing the packages so but we had already pushed A tag to upstream for preview one and to keep.
The history clean we decided to move on to preview two and release the Preview 2 yesterday.
And that is still based on dot net nine, the latest version of it, and has bunch of changes that were not included since the last two years, 75.
So we are expecting A750 release soon.
I don't have a date yet, but it it should come out soon.

Steven Bucher   24:14
Awesome. OK.
Thank yeah.
Thanks for detail for the for the updates on the gallery and the release.
Let's see.
What do we have next?
Or actually on the preview two, I know Justin was working on some stuff.
Did you did you wanna add anything, Justin?
To what was what's been going on?
Or I know you've been working hard on the releases so.

Justin Chung   24:47
Oh yeah, there is a small mistake during the release.
The metadata dot Jason was for the release tag was.
What is supposed to point to the latest stable release? But I mistakenly changed it to point to the the current release I was doing, which is to preview .2 and that PR is out right now to fix that.
So if anyone's trying to install the latest stable.
My runitation is there.

Steven Bucher   25:24
Thank you, Justin.
All right, I'm gonna pass it over to Tess to talk about some open SSH folder permission issues that we've been working to to fix.
Do you wanna test are you?
Are you here?

Tess Gauthier   25:40
Talk about it.

Steven Bucher   25:41
Cool. Take it away.

Tess Gauthier   25:44
All righty. So for the folder permissions issue that first started happening after the October Windows Update. We've been working to address that. The changes are slated to be available in a future Windows Update starting at the end of February. For most OS versions, and some will be in.
A mid March update for some older Windows Server versions.
So we've changed the strict permissions check on the program data SSH folder.
That was causing the SSH server service to not start in version 9.4 and that's replaced with a logging message that will include the recommended permissions for the folder, but will not actually prohibit the start up and that message will be logged in event viewer.
Another issue that I did want to highlight from the October Windows Update.
As well relate specifically to Windows Server 2019, including Azure Marketplace images for Windows Server 2019.
These images come preinstalled with the open SSH client updated to the most recent version version 9.5, and then if the open SSH server is installed afterwards, it's the older version 7.7. And because there's some overlap between the open SSH client.
And open SSH server components.
The open SSH server won't start, so check for any pending updates on the machine after the open SSH server is installed, and once you apply the pending Windows updates, the open SSH server should be updated to the most recent version and the service will be able to start.
It's only applicable on Windows Server 2019 due to some underlying Windows Update mechanisms for features on demand like open SSH.
But just wanted to highlight that in case anyone's run into that.
Yeah. So that's the updates from the open SSH side. Thanks.

Steven Bucher   27:49
Cool. Thank you, Tess.

Tess Gauthier   27:52
Oh, I just saw in chat a question on opensh.
Feel free to put it in the chat if you and I can try to answer it async as well.

Steven Bucher   28:02
Yeah, we'll we'll definitely have time to. We'll, we'll, I have questions in the chat. We will have opportunities at the end to to kind of.
To ask questions verbally too, but since we still are got a big agenda, we'll just kinda keep going and then go through it that way.
Who's next?
Andy, are you here to maybe give a bit of a a trailer?
Comment on the bicep DSC work that you've been doing.

Andy Jordan (they/them)   28:34
I can give you one better actually.
In fact, I'm gonna cover 2 quick topics.
The first is the short one.
We are working on a PowerShell extension release. A prerelease should be out really shortly followed by a stable release within the next week or so.
Honestly, we're most excited just because this will actually have the fix for Sierra log causing the assembly conflicts for who knows how many people, especially using PowerShell 5.1 we removed an entire dependency just to get rid of conflicts, so that was nice.
Big shout out to Justin for.
His help with that? So that'll be out, but.
To some fun stuff.
I mean, that's fun, but to even more fun stuff. We've been working on like a prototype to actually use bicep to write your DSC configurations.
And I'm gonna go ahead and share my screen and show just a very short little trailer of what that kind of looks like, which is exciting because it's it is really quite simple.
So let me see share.
Oh no. Requesting allow. OK.
We're gonna hope that this works.
I think I had to reinstall teams.
S.
Is the monitor with a big black terminal sharing great? OK.

Steven Bucher   29:38
Yeah. Yeah, I can see it.

Steve Lee (POWERSHELL)   29:38
Yes.

Andy Jordan (they/them)   29:40
So I think most people in this call are probably familiar with DSC itself.
Specifically, I'm talking about DSC V3, the new project written in Rust that has a PowerShell adapter and like is, you know, the next project of DSC and right now you can write YAML and you can write Jason and under the covers it essentially transforms those and underst.
Them as arm. And so we have this thought of like, well, bicep is a whole language meant for allowing you to write.
Arm resources in this much simpler declarative language designed for it called bicep, that then also a mixed arm. So we tried it out.
Actually, we were like, could we write our DSC resources as bicep code and then have the arm that bicep emits actually work in DSC? And the answer was mostly yes. In fact with a developer build a DSC.
That Steve just got a change into for me.
I can actually use bicep with like one small little change to their code to emit the correct schema, which that is a whole. This is not like something we can ship that is AI.
Got it working my hard coding. Change the schema over here.
We have to work out like, oh, what's this gonna look like as a file?
What's what? Where? What are we actually gonna do to productize this? But for testing and demo purposes, check this out. Over on this screen here, I have a very simple bicep file that might look really familiar if you've written.
Xaml for DSC.
I'm saying hey, use the PowerShell adapter resource called Microsoft dot Windows Windows PowerShell right now. Bicep is actually saying by the way, I don't know about that resource, but I'll let you use it.
It doesn't know 'cause it's adsc resource.
Not an arm resource. One of the things is like, you know, in the future when we make this a product. But yeah, it's just let's run this PS script.
With a few properties on it, I run this through the bicep CLI. In my case I'm just using VS code to click their CLI button, but it also that command looks like.
This with really terrible text.
I'm gonna try to highlight it.
Maybe that's a little bit better. My developer build a bicep, run the build command.
Pass it the bicep file that I wrote and once it does that, I actually get a Jason file spat out which is an ARM template that DSC understands right now.
So I got to write that bicep. I get this Jason. And this was so great that with this dev build I've got working. So let me run.
The get command from that, so I'm going to run my dev build of DSC.
I'm so sorry about these colors. Dev build of DSC.
With debug logging.
Config git.
Here's the file that I gave you, which is the Jason from the bicep.
So like right now it's, you know, build it over here.
Send it over here.
We're talking about like, what's that user experience actually like?
Will DSC just understand a bicep file?
We don't know.
We actually we want people who are using this to tell us, but if I run this.
Minus the fact that my terminal's a little wonky, and so you got some extra ASCII input over there.
Yeah, it actually works if I scroll up a little bit you can see that it went ahead and started the Windows PowerShell command invoked.
That command ran my little script.
That little embedded script from the file.
Pass it all the way through. Sure enough.
Got my output right here which was wanted to run the script with result Git.
Which if we go way back, even though it's ugly, the get script if you run it is going to return the result, get there's and new lines and whatnot that ignore the new lines and all of the quoting issues and stuff.
But yeah, as a trailer like this, this works.
We're really excited to be working on this with people.
We're excited to get feedback. We want to actually turn this into something usable soonish, but that's. Yeah, that's I had to share.

Steven Bucher   33:31
Cool. Thank you, Andy.
Keep losing the agenda. Where what's the next thing?
It is docked update I don't.
I know Mikey.
Are you here?
To talk about this. Otherwise, I think Mike Robbbins is here.

Mike Robbins (Azure PowerShell)   33:48
So.
I'm actually yes.

Steven Bucher   33:52
Cool. You want. You want to take it away, Mike?

Mike Robbins (Azure PowerShell)   33:56
Yeah, let me share my screen real quick.
Hey, we've got a few docs updates for for PowerShell.
One is we have a new article for about type conversion and this tells you how and when PowerShell performs type conversions.
We've also got a new article on about comments, and to me this one's really good and this was actually so both of these were community contributions.
There was a lot of work done on the documentation for the PS script 10 lineser rules.
And if you wanna know about the specific updates, you can take a look at what's new in the PowerShell docs, this article, and I'll tell you what I'll do.
So I've just posted links in the chat for everything that I'm talking about.
So for Azure and that was looks like that was it for the PowerShell docs updates for Azure PowerShell. We published a new version.
So the corresponding documentation for Azure PowerShell 13.1 we also finally archived all the Azure RM documentation.
So if you're looking for that documentation, you can go to the Arizona PowerShell module page.
You can go to previous versions.
And you'll end up on our previous versions and then just go to Azure PowerShell.
Documentation is still there, just in case you need it.
I also wanted to mention about about the upcoming MFA requirements and I don't know that Damian Carroll is going to speak about this here in a second, but I also put the link to this document in the chat just to be aware of it.
And that's it for the documentation update.

Steven Bucher   35:47
Awesome. Thank you so much, Mike. And I think that's a good segue to the MFA, Demi.
Do you wanna talk a little bit about some of the MFA enforcements coming up?

Damien Caro   35:58
Yeah. I just wanted to remind everyone that MSA enforcement is in place in the portal. You may have already seen it.
You are required to use MFA unless you've asked for an extension.
We are preparing for applying the same requirement for the command line tools, so Azure CLI, Azure PowerShell and other Sdks.
We know some of the flows and you.
Probably want to be aware of that some of the flows are not going to be functional when MFA is enforced.
The first one we have that I can mention is ropc.
So if you're using a user name and password authentication with Azure CLI or Azure PowerShell, those commands will not work.
You may want. You may use them because you have an account in Active Directory that is synchronized to Azure ad and you're using that identity in Azure AD or enter it now.
To be authenticating against Azure.
The recommendation is to move towards manage identity service principles, those kinds of identities geared for automation purposes, and I'm going to drop in the chat one of the article that describe all those different flows and how to prepare properly for that.
But we wanted to remind everyone that this is coming and if you have any of your flows using one of those user accounts.
You want to really think how to migrate to service principles or manage identities.
That's it.
That's it from that front.

Steven Bucher   37:42
Cool. Thank you, Damien.
I think that brings us to the conclusion of our core agenda.
I believe we addressed the looking at the GitHub issue now. I believe we addressed all the gallery.
Things feel free if we if there was still, you know, open questions around the gallery stuff, feel free to post in the chat or or raise a hand.
Otherwise, we'll try about web sockets and PowerShell.
Let's before we get the demos. I just want to address the questions.
I'd love feedback from Steve and Shawn on this ex post. OK, I've been taking a look at this yet.
Yeah, maybe.
Steve, do you see the ex post in the hub issue? I haven't.
I'm I'm reading it now but.

Steve Lee (POWERSHELL)   38:48
Sorry, can you?
Not sure what you're referring to.

Steven Bucher   38:51
So the someone in the GitHub hang on, let me in the GitHub page there was a discussion in link there, so maybe I'll give you a time to take a look at that and we can address that.

Steve Lee (POWERSHELL)   38:55
Oh, let me go there.
You know the the discussion for the agenda.

Steven Bucher   39:06
Yeah, yeah, I'll throw it in the chat again.

Steve Lee (POWERSHELL)   39:07
All right.
No, no, I I got it.
I I forgot to look there.

Steven Bucher   39:15
So yeah, we'll.

Steve Lee (POWERSHELL)   39:21
I mean, I'm opening the.

Steven Bucher   39:26
Well, you can take a look at it and then we we can address address it there and if there's any more comments we can do it after I know.
So James, did you wanna talk?
Do a quick you know 5 minute demo or something on the web. Sockets in PowerShell. Are you here?

James Brundage   39:43
I could do a quick song and dance.
I luckily had my other meeting move back, so if you want to give me some screen sharing rights and I will kick off some fun shows.

Steven Bucher   39:55
Sure. Why?
Let me figure out the screen sharing thing again.

James Brundage   40:00
No worries.
I can also do the virtual camera thing, but I've generally had complaints that that's not the easiest thing to read for text.

Steven Bucher   40:09
Make a presenter.
OK.
You should.

James Brundage   40:19
Yeah.

Steven Bucher   40:19
Then it's not feel free.

James Brundage   40:21
We're starting the clock. OK, so quick, 1000 foot version. Web sockets are part of A are part of the web standards. They are a way to send either binary data or Jason rapidly back and forth over HTP, they're great.

Steven Bucher   40:23
No, go for it.

James Brundage   40:38
One wonderful way for PowerShell people to think of them.
Is there an object pipeline that is web friendly? OK.
There is this new social media network called Blue Sky.
They happen to have a web socket of everything going on, which gave me the inspiration to build a little web socket module which you can literally just find on the gallery called Websocket. I'm getting less creative in naming as I get older or more Co can't be you.
Tell me.
So let's go ahead and get help and look at some examples and let's pick out a fun one.
That has all the tags and posts.
This one has all links.
I'm looking for one that just has all the emoji I believe.
If that's it, so we're going to go basically ask the Internet how it's feeling.
Get a real object pipeline back to that.
And.
These are just the emoji real time.
From the Internet, from everybody that is posting.
And I'm gonna control C this for a second 'cause. You know we love Bower, show background jobs. I'm gonna go look at that background job on our CU job deep.
And these are in fact all of the different references that I've gotten.
Think I've got some fun examples going back here, so I've done a bit of ad hoc demos last night.
Yeah. So let's go ahead and change that to four.
So of all the posts that we've been watching, these are the top 10 links. I'm just going to break down this pipeline here.
We're getting our job.
We're receiving job keeping results. We're getting all of the external Uris as a Uri. We're grouping on DNS safe host, and we're sorting on count and picking out the 1st 10.
So lots of memes.
A little bit of YouTube, a little bit of liver or YouTube buffly links to Twitter, ironically.
The Guardian, YouTube and whatever Tver dot JP and.
And I'll go ts dot frr OK again this is real time data.
A real time object pipeline of all of human thought, so that's fun.
Also, it opens up whole new, wonderful territory for us as PowerShell developers to, well, Flex our data scientist muscles start to get into the game of social media automation again and do fun and possibly profitable things.
So that's cool, but I've got one more bonus point for you.
This is up as a gist.
This is not particularly web socket related, but I'm gonna let a picture speak 1000 words here, so I'm gonna put a breakpoint here.
This is an event driven micro service built on top of PowerShell.
We're picking a random port for creating a listener.
We're starting a listen thread job.
Excuse me.
And we're generating an event for every request in and outside of that job, we're just going through our event queue and we're going to be able to actually debug our Web request.
So hit F5, got a new server up.
I click.
Demigods be kind. There you are.
OK, I take out my breakpoint because we wanna actually demonstrate speed as well.
But let's go ahead and click this a few more times and look at those beautifully fast response times. I'm getting back just from serving a random number on top of PowerShell.
And again, this is a debuggable purely event driven web server.
Plus, you know a web socket with all of human thought coursing through it.
Questions, comments. Feedback. Everybody's noses.
Stop bleeding or just start bleeding.
I think I'm gonna go back to the emoji example for a second, just because that's a fun one.
And see if anybody.
Is engaging with it, sorry.
Too many examples here.
That's better.
Oh, it also does cash the job if you reuse the same Uri, which is why we just saw all those emoji fly by 'cause they were cashed.
So yeah, PowerShell Web development is about to get a lot more fun, partially because you know high efficiency servers, but also because Web sockets finally give us this, you know, dare say Holy Grail that we've been looking for in PowerShell and Web for all these years where we.
Can have high throughput communication between a web server and a PowerShell client.
So really exciting times, really exciting discovery, very boring module name.
But easier to remember.
So yeah, that's that's the quick news fit to print on PowerShell Websockets and a little bit on PowerShell Micro services in pure event driven form.
So questions, comments.

Steven Bucher   45:47
Cool.

James Brundage   45:53
Everybody just enjoying the random emoji as they fly by.

Steve Lee (POWERSHELL)   46:03
If you just do any questions in the chat, I think it'll be easier.

James Brundage   46:06
Sounds good.
I'm going to stop sharing in 321. Show's over.
Folks don't have to go home, which can't stay here.
Actually I should have done a porky pig joke there, but all right.

Steven Bucher   46:17
Thanks James.
Very cool.

James Brundage   46:23
You're welcome.

Steven Bucher   46:26
I didn't want to address I I did want to give Ryan.
I know you said you kind of had a big documentation topic. You liked feedback on.
Was there anything you wanted to add?
In this form here or or any particular questions?
Unfortunately, I don't think Shawn is here, but we can.
We can follow up too if if it's a documentation issue.
Hey, you're working on.

Ryan.Wakefield   46:54
Yeah, apologies.
I was trying to find my conversation and mute button.
I'm on my phone and yeah, it was really just more around, you know, David Fowler did an amazing dig into the I think it was the aspire documentation and I I posted a link to my reply on the Twitter slash X thread, but he did an amazing job.
On, you know, just finding gaps in the documentation.
And he was using the the Open AI think it was open AI chat, just chat gpti thing is all he's using.
And so it'd be really interesting to see, you know, really kind of what if there would be any way to kind of tackle that same idea with like the PowerShell documentation and seeing where are gaps, where can we expand more?
Where can we add more details?
Because I'm very much so detail oriented and sometimes I've missed some and so you know, it's just one of those. I'd be curious to see if that could be done with the PowerShell documentation as well.

Steve Lee (POWERSHELL)   47:57
Yeah, let me. So I I just took a look there.
Thanks Ryan for bringing this up.
And just so everyone knows, like I honestly, I'm not on X at all these days for reasons.
But I I'll I'll make sure the next time I sync with the doc folks.
This would include like Shawn, Mike and Mikey.
We'll evaluate this.
I took a look at it and it's like, hey, that's actually pretty cool.
It might not be that hard to maybe get working, so I appreciate you bringing this to our attention.

Ryan.Wakefield   48:26
Yeah, no problem.

James Brundage   48:26
On a related note.

Ryan.Wakefield   48:27
And I'm glad to do so.
Go ahead, Jane. Sorry.

James Brundage   48:30
Oh, just Steven. You mentioned you're not an ex.
I've seen other Stevens show up on blue sky.
Are you happening?
Do you have a blue sky account?

Steve Lee (POWERSHELL)   48:38
If the community tells me that there's a good portion on some other forum or social network where I can give out information, put it in a chat, I'm at this point open to trying something else.

James Brundage   48:51
Yeah, there are a number of PowerShell people on blue sky, so you should join.

Steve Lee (POWERSHELL)   48:53
I'm not on blue sky right now, so.
Alright, maybe I'll give it a try.

James Brundage   49:01
There's a great PowerShell starter pack. It's got a few 100 people in it and a PowerShell feed, so.
Big van.

Steven Bucher   49:14
Cool, yeah.
Thanks Ryan for for bringing up that documentation.
Stuff just to wanna take a look.
Two any other topics?
Demos questions from anyone?
I'll. I'll. I'll pause for a moment.
To let things do.
Otherwise we can.
We can end a little early as well.
Marvin, can I see your hand up?

Carpenter, Marvin   49:50
Yeah. So I just was trying to get some clarification on this.
The open SSH stuff.
So obviously we're we're we're installing this and the servers obviously don't have access to the Internet.
So when we originally did this install, we had to do this through the.
I mean, we have to get the binary off of the features on demand ISO and I created like a chocolatey package to do the installation.
But now it sounds like it's coming and it's a part of the OS.
And updateable via.
Windows Update.
So I'm trying to determine what the best route is to actually convert to that.
Because we're on the long term servicing version of of server 2019, so it it no matter what I do, it's never gonna. If I do install optional features, it's not gonna be there.
And then I even notice that when I, when we did some server 2022 stuff.
You you had to have Internet to to get the the binary as well.
So we're kinda in the same boat here.
It looks like if we just install it into the correct.
Directory then Windows Update will start updating.
It is is that is that is that accurate?
Can't imagine that that would work but.

Steven Bucher   51:24
I'll probably ask Tess if you have any insights on the Windows Update.
I'm not too familiar yet on the Windows Update stuff for SSH. We could also follow up offline too if if we're uncertain.
I know these are these are newer, newer scenarios.

Carpenter, Marvin   51:39
Yeah.
All right. Fair enough, I'll fair enough.
I'll I'll consult the docs and see if I missed something or you know, maybe something's got updated since then. Thanks.

Steven Bucher   52:01
Umm.
Yeah, feel free to if if there was stuff to add test. I know I saw you off for a moment there, but.
Think.
OK.

Tess Gauthier   52:14
Yeah, I was just going to say that.
I guess I'm not totally familiar with like the offline.
Updates. So if you're not doing any Windows updates for any other components.
Online, then, there would have to be a different mechanism for you to open SSH updates.

Carpenter, Marvin   52:34
Yeah, I mean essentially patches come through SCCM.
You know what I mean? So.

Steven Bucher   52:39
Hmm.

Tess Gauthier   52:45
I would think it would be the same for open SSH and you'd still have to take the.
Updated server binaries or open SSH binaries and get them onto the machine.

Carpenter, Marvin   52:58
Yeah, I don't think we.
I think we looked.
I don't think we saw, though nothing come down from SCCM.
But we'll we'll, I'll take another look at it. Thanks.

Steven Bucher   53:09
Thank you, Tess. Thank you, Marvin.
There's a couple other still questions in the chat, so I'll let folks answer.
There.
If there's no other questions.
We can wrap up a little early, then I will call out that as far as other community stuff going on, there is the PowerShell Summit North America schedule that got posted I think a week or two ago.
So definitely take a look at that.
And PS Confuu is in June.
I don't think they've announced yet the the exact schedule, but it's a little further out, so I'm sure we'll see it.
Soon.
But yeah, fantastic.
It looks like we're we're.
We're getting folks on Blue sky.
So we can start there.
Thank you, John for posting the schedule of the partial summit and.
Yeah, last thing because I see it on the in the chat.
Whatever happened to the concept of PowerShell pipelines and native apps using pseudo terminals?
I believe we're, we're still working on it.
We had.
There's a couple.
Partner works that we're we're kind of relying on, but Patrick on our team is is working on it.
And so hopefully we can, we can.
Make some progress in that area. So.
No ETA yet on that, but it's still it's still.
Schedule, but yeah, cool.
Well, I think that kind of wraps things, wraps things up for the PowerShell community call.
Thank you everyone so much for coming for the first one of 2025.
We're excited to be back on the normal cadence.
We'll see you next month.
3rd Thursday of the month 9:30 AM.
For February, I don't know what that date is offhand, but you know.
We'll come on the calendar later, so the recording will be up soon.
And yeah, thank you everyone for joining.

J