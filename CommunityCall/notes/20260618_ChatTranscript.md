PowerShellOpenSSH Community Call-20260618_092840-Meeting Recording
June 18, 2026, 4:28PM
50m 42s

Jason Helmick started transcription

Jason Helmick   1:34
Hi folks, if you're just joining, happy June, happy summer, happy Thursday. We'll get started here in about one to two minutes. Give a chance for everybody to sign up.
Well, let's go ahead and get started. Happy June, everybody. Happy summer. So hello, good morning, good afternoon, and good evening. Welcome everyone to the June 2026 PowerShell Community Call. Let's call it the summer edition since we're in summer. Everyone is welcome to join and discuss the PowerShell project and its development.
So we have our agenda posted in the meeting chat and a link to the discussion. Please feel free to post questions in that discussion issue or since you're in the meeting right now, go ahead and post them right here in the chat for the meeting. And please check out and follow our code of conduct. Once again, for the last five months, I've been able to do this. This is month number six.
Happy 20 years to PowerShell this year. I get to say that for six more months, although I think Aditya corrected me. We don't do a call in November and December. Well, I'll work it out. Anyways, happy 20 years to PowerShell this year, and we've got an action-packed agenda today. We've got things like conversations around a recap of PS Conf EU, some team items.
And then we have some demos. Hopefully everybody showed up to do the demos. That would be awesome. So for the first step, let's go ahead and have a PS Conf EU recap. From what I've heard and the feedback that I've received is it went excellent, but let's hear from some folks that were actually there. So Stephen, Tess, Andy, Sean, if you're around.
Well, I see Stephen first, so y'all can just kind of join in and tell us a little bit about how your experience at PS Conf EU was. Take it away, Stephen, since I see you.

Steven Bucher   4:17
Ha ha.
Sure, yeah, so, so this was my fifth year at PSConf EU, and it was just as awesome as all the previous years. It was held in Germany this year, my first time in Germany, so really enjoyed getting to know that country and the people there. We presented the state of the shell, which we were excited to share, you know, some of the...
the amazing work Jason's been doing about inboxing. We also shared a little bit about some of the latest changes and latest things that we've kind of publicized in the blog and the state of the shell. We talked a little bit about how kind of now more than ever your skill set with tools.
With building reliable, repeatable tools is now more important than ever in this age of AI, and so I think that was a pretty interesting topic. I also had a session on Azure Machine Management, where I showed off some of the latest and greatest with machine configuration and...
Some of the work I've been doing in the Azure space, we had lots of great conversations. I think we had some really awesome lightning demos, lightning talks, or I guess they called them community demos. And we did have way too many pretzels. So that was a big thing.

Jason Helmick   5:32
****.

Sean Wheeler   5:46
Yeah.

Steven Bucher   5:50
But yeah, Sean, Tess, Andy, anything you guys want to add? I had a great time. I personally thought it was a fantastic time and was very happy to have gone again.

Sean Wheeler   5:59
Yeah, this was my first time going to that conference and it was a great time. I got to meet several docs contributors that have made a big impact on our docs. Got to meet them face to face for the first time, so that was great.
and a lot of great sessions. And yeah, it was a fun conference.

Jason Helmick   6:28
Well, that's awesome. Tess, Andy, do any of you want to join in?

Tess Gauthier   6:34
Yeah, I mean, I'll echo what Stephen and Sean said. It was really great to see folks in person, some familiar faces and some new faces. It was one of the things that I really enjoyed was Jeffrey's keynote on navigating the AI revolution and some of the small group conversations during Ask the Hive Mind at the end of the day.

Steven Bucher   6:47
Yep.

Tess Gauthier   6:55
So yeah, really great time and.
Thanks again for having us.

Jason Helmick   7:02
That's awesome. That's awesome.
I want to give opportunity. Anyone else want to say anything that was there?
Cool. Well, thanks very much. Yeah, the feedback I've gotten about that show is just, it was really incredible and I really appreciate it. Hope everybody had a really wonderful time at PS Conf EU. Looking forward to next year already. So let's go ahead and continue with our agenda. So
My friend, Aditya, we have some servicing updates, I think, for 7.4, 7.5, and 7.6.

Aditya Patwardhan   7:35
Yes, so we released some updates to 7475 and 76 on Tuesday. The packages are on their way out, if not already available. And they are mostly the changes over there were.NET framework updates, as in.NET SDK updates with security fixes. So it's.
we would highly recommend update your portion installations to the latest ones. And a friendly note and gentle reminder about 7.4. 7.4 is slated to go end of life in November. So if you have not thought of migrating to 7.6 yet, I highly recommend doing that sooner. And if you see any issues with that,
Feel free to message any or file issues.

Jason Helmick   8:21
That's awesome. Thanks, Aditya. And yeah, what a great important reminder that we are approaching the end of life for 7.4 here in November. 7.6 is out right now, and 7.6 will become the LTS when we get to November. So that's a great thing to be thinking about. Alex, Alex is stopping by to give us a little bit of information about AZ and a few other things. Alex, do you want to come up and...

Alex Wang   8:43
Yes, thanks. Yeah, maybe I can share my screen. OK.

Jason Helmick   8:44
Tell us what your points are.
Sure.

Alex Wang   8:54
Okay, hello everyone. I'd like to share 2 updates from the Azure CRI and Azure PowerShell. The first one is an improvement for the easy logging. So this is for the improved performance and the usabilities. The second one is just a reminder about the Azure AD graph.
retirement and the users need to do next. So this is the about platform migration and the service community. Yeah. So okay, we can go to the first improvement for the like easy login. Currently, this is this improvement is just a very real problem for the enterprise administrators.
So, when someone manage the hundreds of substations, so all work across different many tenants and the traditional logging flow can be slow because the either CRI will list all the substation and also load the full substation list after authentication. So, in the small.
In the smaller environments, so this may be not significant, but in the enterprise skill, it can be a significant delay for every logging. So, for fix this issue, we introduce a...
Dash, skip, dash, and also discovery these parameters, so users can pass the pass bypass subscription discovery and reduce the time complete the logging, so when the target tenant and the subscription is already known.
So this approach enables a faster and more targeted authentication experience.
We can see if the previously behavior is if you log in, you can see a very long or huge list. Maybe I can show a small demo in here. Okay, is it log in? You can see it's...
Ohh.

Jason Helmick   11:03
Oops.

Alex Wang   11:04
Kate.

Jason Helmick   11:06
Oh, AZ, yeah.

Alex Wang   11:08
In.
Yeah, because my list is very huge. You can see it's very slow.
Okay, come on.
Okay, so this is a very huge list, so it's very slow. So if we use the new parameters, we can see the, you can know the tenant ID and the target substitution, and you can also use this keep substitution discovery. It's very quick.
Can see.
And directly authenticate.
Yes, you can directly go to this subscription and the tenant. So this is improved performance.

Jason Helmick   12:07
Yeah.

Alex Wang   12:08
Okay, this is the behavior we improved.
Kate.
Yeah, so this one, I think currently for the Azure PowerShell, we also have the same parameters, so you can use. And the second topic is for the Azure AD graph retirement reminder. So this one is, I think currently it's already impacted the kind of customers.
HRAD graph was first announced as a deprecate on the, I think, at June 30, 2023, and it become full unavailable in the September 2025, with no further maintain. But however, we still see a significant amount of traffic.
using the CRI and the PowerShell IP ID, so which could lead to unexpected disruptions in the future. So I suggest any application script or any automation flow still depend on the Azure AD graph. So I think it's a.
Face the service disruptions so late, so the key message is very simple: if migration not happen, so something we need to do, make a we need to attention now, so anyone still use the CRI early, then the version two dot.
38 or 8 parts are earlier than the version 12.0, so please operate the latest available versions as soon as possible. Yeah, we already provide the guidance in here. I can I can post that the blog, these two blogs in here.
Yeah.
Thank you, thank you, Jason.

Jason Helmick   14:03
Thanks, Alex. Oh, I love the performance improvement. Congratulations on that. That's awesome. Oh, folks, we have some demos today. I'm really excited about these demos. Before we kick into the demos and get the first one going, I have a non-agenda item. Folks have been pinging me and I just wanted to
to say a couple of quick words and give you a link as well. We do not have anything to announce today in regarding to the inboxing project. However, my hope is, is that before the next community call, we'll be able to provide you with much more details in an announcement that we can discuss. That's not a promise, but it's my hope.
So, we'll see how that work goes. However, as someone pointed out, I did promise something in a previous, I think the previous, yeah, yeah, the last community call, and that's that I was creating a master tracking issue for the MSIX issues and work.
that is occurring around MSIX. Again, as a reminder, if you haven't seen the announcement, I'm going to drop the link into the chat. If you haven't seen the announcement, MSI is being deprecated starting with 7.7. 7.6 will have MSI support. In that time period over the next couple of years, we are working on
shifting everything to MSIX. There are several scenarios that are problematic. We are aware of them, and we keep asking you for advice on scenarios that we may not be aware of. This tracking issue tracks the work both that our team is doing, the PowerShell team, and you'll notice that there are several other teams involved as well.
Now, just to let you know, that issue is not going to, I will update that issue relatively frequently, but don't expect a lot of changes really frequently as that work has been aligned and scoped and is being prepared to be done. However, some of that work won't even start till the beginning of next year. So just to have a little bit of.
level setting on that. One of the other things I promised you that will be coming in the near future is also a tracking issue list for what I call 527. And that's the things to migrate from 51 to 7 and the things to think about, plus all of the issues. I'm also starting an inboxing working group.
Now, the initial start of that will be with both internal and external employees, employees, internal and external folks. It'll be a very small group as we get the processes built and the initial issues discovered. But we will be making the results from that public, and we will be expanding that
working group for other folks that may want to join it. That expansion probably won't happen until next year, but we're going to get started on it now. So I'll let you know more about that as we move on into the future. So with that, let's go ahead and see if we can get started with demos. Now, the first demo, and Hamza, are you here?
I think I saw you were here. I'm really excited to see this Windows Intelligent Tour. Oh, awesome. I think you have presenter rights, so go ahead and take it away.

Hamza Usmani   17:05
I am here, yes. Yeah.

Grote, Justin   17:07
Yeah.

Hamza Usmani   17:13
Awesome.
Let me go ahead and throw up the screen and someone give me a thumbs up when y'all are looking at it.

Jason Helmick   17:21
Yeah, we can see it.

Hamza Usmani   17:23
Okay, fantastic. Okay, so I am from the team that has recently just shipped our first version of what we've called Intelligent Terminal. So for those of you that haven't heard of it, I know a bunch of you have already reached out and said you have heard of it, which is sweet. But for those that haven't, this is a fork of Windows Terminal that you know and love. So it has
everything that Windows Terminal has. However, it also has what we call native agent integration. So this is, you know, how far can we take the terminal in terms of improving and enhancing the user experience with how developers use agents today. So some of the stuff that's really come from how we breathe and work with agents today, we've brought into this.
This is just our v0.1. I am going to talk about some concepts and ideation after I show you a very quick demo. And I'd love for you all to jump into our GitHub, give us feedback, let us know if you like this project and where you would want us to take it even further. Okay, so let's jump into it. I'm in Intelligent Terminal. I'm in this small little Electron app that I've created for the sake of this demo.
One of the core concepts in Intelligent Terminal, and I will toggle this off, if you don't care about agentic features, they're not there. And if you do, one of the core concepts is what we call the agent pane. So if I hit Control Shift period, I can toggle on my agent pane, and this is where my agent of choice lives. So for me, I have Copilot living right down here.
And this is kind of like my pair programmer, but in the shell, right? And so I'll show you an example of what that might look like. So let's say I try to run and build my Electron app. It is a small app, so hopefully no issues. But oh, I actually am running into issues now. And there we go. That took a while, but I am running into issues.
You can see that in my agent pane at the bottom in our agent status bar, there's this little thing here that says, hey, we detected an issue, something failed. Press Control Alt Period to send this to your agent pane. Or I could just click it. So I click it and now this error has been sent to my agent of choice. So Copilot, I have this currently set to manually prompt me when you find an error, but there is a setting to just have it.
auto send your errors and issues to your agent. And then it says, oh, you know what? I actually looked through your project. I figured out exactly what that error is. Run that command. So we'll run that command and we'll run build again and see what happens this time after our agent in just a few seconds found out what was wrong. And now if we try to run and build and start our Electron app.
Fingers crossed the demo gods are with me, and there we go. We have our little tiny life system monitor that I was working on for the sake of this demo. So you could see, you know, this is a very simple example just to keep it down to a few seconds, but most of us daily drive on our team intelligent terminal, and when you're actually working with real code or...
doing real things in the shell, this actually is super, super handy. That is an example of, you know, building something, but what about, and that was PowerShell, but you know, we know we have some Linux enthusiasts and lovers, and it works the same way in Linux. A lot of the times what happens to me, because I'm A PowerShell user, I forget how, you know, WSL commands work. So
What if I didn't have an error? I could just say, you know, like, list all files, including hidden ones. Maybe I forgot that it's like, you know, LS dash LA. And so again, this, you know, my agent would take that and it would tell me exactly how to list those files. It would give me a command. I'll run the command and there we go. There's all my batch files. So like I said, it really kind of is your pair programmer and your intelligence in the shell. It's always there. This little icon here will show you.
When it detects an error, this is to toggle on and off your agent pane, and then if you were wondering, "Hey, Hamza, what is this button over here?" This is your agent sessions, so if I hit Control Shift slash instead of period this time, you will see all of my Copilot sessions that I've run on this PC. So, if I want to quickly jump...
into, you know, one really quickly, like I was trying to set up my Figma MCP for some design stuff earlier. This was a native co-pilot session. This will just launch that session directly into the session itself and launch it into a new tab. So I can do that very quickly and then continue back doing whatever I was doing and that session is there.
for terminal users that know and love and breathe the command palette. This is the command palette here. You are already used to all of the options that command palette gives you. We've added a new option. If I hit command palette, and of course it is freezing, which is just fantastic for me.
Give me a sec, folks. My entire PC just froze, which is great.
Okay, here we go.
Come on, come on.
Okay, so if I hit Alt Shift slash, this will bring up command palette in what we call prompt mode. So, you know, very similar to how you might send something into your agent pane. A lot of the times you want to send something to a background tab or a background task, right? So let's say I'm working on this Electron app and then I'm like, you know what, actually explain this repo to me.
Again, this will take your agent of choice, kick off a background tab, give it the context it needs, and then it will go off in the background and start working on with Copilot to explain this repo, this little PS demo. It'll show me exactly what's going on over here, right? So just another quick way for you to invoke. It's not really the main interaction way that we imagine most users.
use agents in their shell. The agent pane really is that for us, but it's just another quick way to get you there. Okay, sweet. The last thing I want to quickly touch on is extensibility. So we wouldn't make this thing if we didn't want to make it as extensible and as easy as possible for you to use. So I'm using GitHub Copilot, but you're not tied down to using GitHub Copilot, right? Any ACP compatible, which is...
Agent plan protocol compatible agent will be detected automatically on first run. So I only have Copilot on this machine. This is a fresh machine, but on my main machine I have Codex and Cloud and Gemini and Copilot and so.

Grote, Justin   23:16
My God, Spock.

Hamza Usmani   23:18
And so they would all show up here. Sorry, did someone ask a question real quick?

Grote, Justin   23:18
Sorry.
Now it's just me yelling at my dog, continue.

Hamza Usmani   23:25
Okay, no worries. And then of course, if you wanted a custom agent, so you know, if I have like Hamza's agent on my PC, I could just add that here too. Of course, you know, you could choose your model and then you could do interesting things. Like I know some people have already reached out to me and told me like, hey, why isn't right the default for the agent pane?
I only like having my agent paint to the red. I don't like it on the bottom. So, you know, we have tons of customization here. Like I said, auto error detection I have on, but auto error suggestion I don't have on. But if I did do that and I saved it, you know, just as you would expect, if I did something really silly and I did see the documents and I misspelled it,
That would be an auto error that it automatically sends to my agent, and my agent would, I imagine, very quickly figure out I meant to go to documents and not documents, right? And actually that failed too because it's not a relative path. So again, my agent will figure all that out for me. I don't have to worry about it. So these are some of the settings we've added to this.
So this is intelligent terminal and in a quick, you know, 2 minutes, here are some of the main features we've added. So this agent pane, you know, the ability to work across multiple shells, not just PowerShell, the ability to quickly jump back into different and prior agent sessions, as well as track different agent sessions, right? Like right here, you can track which ones are
idle or currently running as well. And so that is sort of what we brought with our v0.1. This is the first release of this project. And so we are thinking a lot crazier and a lot further based off of community feedback that we're already getting in. I want to share some of that with you all very quickly.
and then give you a link to our GitHub so you can actually let us know in our feature request if any of those stuff that I'm showing you right now really resonates with you. And if it doesn't, tell me why. So here are some of the concepts that are on some of our further out roadmap as well as some of our immediate roadmap as well. So one of the things we're thinking about is, hey, we have this concept of an agent pane, but a lot of users have told us
I don't want a dedicated pane. Some users love it, but some don't. Some want, you know, one pane and they want to be able to quickly toggle between shell mode and agent mode. So in this, and this is a design mock-up I'm showing you, you know, we're not showing off real code yet for some of this stuff, but in this mock-up what you're seeing is I have a shift tab to switch between modes like you're used to in your CLIs, but in the...
In the terminal itself, and so I can switch between put and shell commands and then go into my agent and asking it something in natural language. So, this is what we're calling the unified pane settings, so it's a setting that merges your agent pane directly into your shell as well. This next concept is this concept of making output in your shell actionable.
This is what we're calling actionable output. So what you're seeing on the screen here is a quick snippet of me hovering over an output, and I have some commands available to me. Some of these commands might be copy or expand or collapse, especially when you get a really long shell output. Some of these commands might be fork in a new tab if you want to quickly take that output and say, hey, I want to take it, save to file.
These are some of the output actions that we're thinking about to add intelligence to this terminal.
Next, this is a sort of a crazier idea that some folks have been asking for, but we absolutely want to validate with the community, is this concept of a content pane. So this is just another pane in the terminal where we could show rich content like markdown and things like code diffs. This is a very rough, you know, sort of UX mockup you're looking at here, but
It is more for exploratory and helping you understand what we're showing off here. But you can see your shell and your agent painter on the left, and then on the right, we are showing off some diff, some code diff that you might be working on. You can imagine this is markdown. I want to quickly read a read me or quickly draft a markdown in a repo I'm in, or even work with my agent to create one. And so it's right there.
And then finally, something that, you know, I think the terminal folks have been asking for for a while is, can we have rich tabs and can we have vertical tabs as well? So what you're seeing here is both of them in really sort of 1 design. You know, you can imagine rich tabs, but horizontal, but we're showing off both. So this is rich tabs that understand what GitHub repo you're in, what branch you're in,
Whether you're doing something, whether it's a shell task or an agent conversation, and what the status of that might be with these quick little colored dots, as well as of course showing, you know, vertical tabs, which give you a lot more space when you're dealing with potentially a ton more tabs and complex setups. So, like I said, in a minute, these are some of the concepts we're thinking about on our roadmap, but I have a ton.
on our roadmap that is already there, that is being validated by the community. We already have lots of passionate users in our GitHub that are giving us future requests, basically every single day, and bug requests. Don't get me wrong, this is a new product. There are a few rough edges that we're fixing. We actually just shipped our first servicing release yesterday, fixing a ton of bugs.
So here's what's on a roadmap. Some of the stuff I already showed you. We want to make local model support really, really good. So if you work with Olama and local models and you don't want to go to the cloud for every agent pane interaction, you shouldn't have to do that. I myself love local models and I want to see that in the product. We want to add built-in repo and get awareness to the terminal itself. I showed you some of that in the rich tabs.
You know, the terminal was aware of the repo and branch you were on. We want to bring sort of in shell a sense, so you know, sort of like the IntelliSense, but for the shell directly and build it into this terminal. Output actions I showed you, content pane, and then we also want to make this terminal as eternal as possible and as robust. So how can we help you restore not just your agent sessions, but...
also your shell sessions. And so that is a concept. I'm sure it will open up a can of worms for a lot of folks. I've already had a ton of questions on that, so we're fleshing that out as we speak. But this is what is on our high level roadmap. And if you're interested.
please check us out. Of course, the last slide is not opening, but check us out on our GitHub. I will throw it up on the screen right here. Microsoft.com slash intelligent terminal. You know, this just released about two weeks ago. And if you want to check out our blog as well, which will show off some of the munis and
Fun stuff. We brought some of the stuff that I showed you. But yeah, go ahead and check it out. Give us a feature request. Give us a bug. We already have tons in there. Thank you so much. I tried to keep that as concise as possible. Thank you, folks.

Jason Helmick   29:50
Wow. Mind blown. Thanks. So that was awesome. Thank you so much. Do you have a few minutes to maybe hang out and answer maybe a few questions in the chat?

Hamza Usmani   30:02
Absolutely, I have like 10 minutes after this call, so I'll hang out here.

Jason Helmick   30:06
Oh, thank you so much. That was a great demonstration. So folks, if you have questions for Hamza and about the intelligent terminal, please go ahead and ask him. And yeah, that was fantastic. But not to be necessarily outdone, but trying. James, are you, I think I just gave you presenter rights if you want to come up and talk about Plaster and the new version, Plaster 2.0.

James Petty   30:31
Yes, let's see if you clicked the right button.

James Brundage   30:31
So.

Jason Helmick   30:35
I think I hit the right button.

James Petty   30:38
There we go. Thank you. I'm assuming you can see my screen now. All right. So after four or five years of dormancy, so PowerShell.org, with the help of Jeff Hicks and Gilbert, who's hanging out in the chat somewhere, we have revived the Plaster project. So we took this over from the PowerShell team.

Jason Helmick   30:38
Hey, there you go.
Yes, we can.

James Petty   30:58
Well, four, maybe even five years ago now. So what is Plaster? So essentially, Plaster, let's, there's Gilbert, let's use kind of, it's built for scaffolding. So anytime we want to make new modules, new scripts, new functions, new whatever, we don't want to have to kind of code those from hand every time. Yes, we could let the AI agent do it for us, but what's the fun in that?
So we have here the Plaster, a nice little PowerShell module version 2.0. And what it's going to allow us to do is take templates that we have, or that you create, create them however you want those to look, and it will reuse these same templates over and over and over again to create your modules for you. Now, if you're familiar with Plaster, before it used XML,
Which?
is mind bending to kind of to put it nicely. So here is a very simplistic version of what that XML file would have looked like. And now everything is backwards compatible. So the XML files are still just as they were, but we can take the same information and we can put it here into a nice human readable JSON format.
instead of this mess. So I just have two kind of quick demos here. So I have first one here, very simple. This one's going, if I can find the right tab. This is my template. As you can see, it has absolutely nothing in it, but that's perfectly fine because it does absolutely nothing. And it's just going to create a very generic review.
marked on file, readme.md. Now, this will create anything that you have a template for. It can be a readme file, the PS1, it can create the PSM1, the PSD1, it can create your build.ps1, it can create your license file, whatever, however you want it to, it will copy all these templates for you.
I'm just going to make sure I'm in the right.
Ohh.
path here, otherwise it's not going to work. And we're just going to run. You know what? I made the demo, God's mad.
I forgot to install the module. I'm on a different computer than I was planning to be on, and I don't have...
the correct one. So let's see if I can actually do any of this. Oh, there it went. OK. Here we go.
Very simple here, nice little, so thanks to Jake for the nice little logo here. Now I have this one prompted to give me a name, just so we can actually see the feedback.
Bye.
So this one has just prompted me for the author's name and the name of, in this case, I'm making a module. You could have scripted, you could have put this in, we'll see that in a minute. But here, I put it here in this file, in this folder here, and it created, it just copied the exact, these two files, just kind of copied the templates that I had. And we'll see here next.
I had this one here, this guy right here, so this is the JSON manifest that we're going to going to look at.
Not that one. It's this one. OK.
I can't type and talk at the same time, so let's just do this.
I'm just going to do demo 3 here. It's just the one I'm going to run. So these are, so invoke cluster is a command that will actually kind of create here. If we see in, I showed you the wrong JSON file. Here's the one. So these are all the parameters that it's going, that it could pull from. You'll see that we have some switches in here.
We have the module name, put the author, you can choose your license file. We already have those saved off in the repository. Somewhere deep down in the bowels of this are copies of all these. So it's already pulling those out. And then we have just some different choices we can have. Do we want to build it? And we're going to do a built-in pester test.
Do we want to build up PS1? Do we want to get dot ignore from default? And that, so we'll just go ahead and run this.
So it ran all that and then it put it here in my output here folder. So we can see it already created the folder exactly how I wanted it. It already gave me my first pester test. Again, this is a template that I already had. It created my git.ignore on its own. Again, another template that I have.
that had put everything it wants to put in there, created my manifest, my PSM1 file, generated a readme for me, and this is all driven based off of all of that information that was here inside of this JSON file.
That's all I got.

Jason Helmick   35:33
Oh, that's awesome, James. I just love plaster. Oh, can you do me a favor, James? Repeat, who is the primary maintainer doing the work on that?

James Petty   35:43
That would mostly be Gilbert and myself, but definitely, definitely Gilbert first.

Jason Helmick   35:47
That's.
Yeah, and that's what I want to do is a huge shout out to Gilbert Sanchez and to James Petty for doing this work. Really appreciate it, Gilbert. So that's awesome.

James Petty   35:57
Yeah, and you can, and you can find everything online at powershell.org/plaster. Goodness, that has all the information on it, the updated change log, and all the good things. There are some open issues out there already, so if you go out there, start using it, go ahead and open up some more, open up some more issues.

Jason Helmick   36:18
That's awesome. Thanks so much, James, for showing that, and thank you, Gilbert, for the work on that. And for our last demo. Hey, James, you there?

James Petty   36:21
Okay.

James Brundage   36:28
Do we have the right James now?

Jason Helmick   36:30
Yep, yep, yep, yep. Good point, good point. I had thought that Mr. Brundage, you, the stage is ready for you.

James Brundage   36:38
All right, I did take some time to type up a message about this, so I'm going to put that in the chat first, and I'm going to hit share. And we're going to briefly show pugs before we end up taking over a code window. Everybody see my code window now?
Okay. All right. So let's have some fun. About 2 weeks ago, I noticed a really fun change inside of VS Code. Tell me if you can spot the difference here. So I'm just starting up a server.

Jason Helmick   36:56
Yep.

James Brundage   37:13
Tell me I'm listening on this like I'm gonna go control click this link.
Oh, it looks like I left my unfun server on this link. So they now open up an integrated tab in VS Code.
for any local loopback server. This is game breaking in a very, very quiet way because it allows us to actually have an app story inside of VS Code, an integrated app story without any extra pain. I'm just controlling, holding Control-R down right now so you guys get a sense of how quickly.
this can be, this is running functions as me in my local run space.
on a loopback server inside of VS Code. You can do this in terminal too, it'll just pop up an extra window, but it's just bloody nice to have it nice and integrated there. So you know, this is fun. And I wanted to write a little thing or two to demonstrate this. I've been working on a bunch of different things to demonstrate how easy it is to make modules. And I ran across this kind of
Well, fun trick of how I can approach making a module. See if you can spot it. So I've got a function slash.
Yes, this is a legal function name in PowerShell. And we're going to go function slash redeclare it. And congratulations, I've just changed my live reloading server in VS Code.
That's already pretty cool, right? Yeah.

Jason Helmick   38:38
Yeah.

James Brundage   38:40
So let's have a little bit more fun. Let's load up the draft of its website here for a second. And I don't need to start it again because I get to live reload. And there is the draft of the site that we're working on and the horrifying stats of, oh, it's only about 900 lines. You know, just.
Just a bunch of PowerShell here. So I wrote a nice little quick server that allows us to declare functions that start with slash that will be live reloaded in your current session.
And that's kind of it. It is really quick, dirty, and easy to make PowerShell servers now. And if we go look at our fun fun, we can actually, you know, see this extend out quite a bit.
So I've got my layout declaration there, I've got my root page, I've got my get command, my state information, a bit about the website, security code of conduct, and a 404 page. So let's go ahead and hit security or.
Well, wait, I have a link at the bottom. I forget about this. Here, contributing, go to conduct.
Security.
Back to home, over to the GitHub.
Hit the back button. So yeah, nice little fun interactive server. And I have a few more tricks up my sleeve in the next build that hopefully will be coming out tomorrow because since I've named the module fun, because it is just a fun functional server, I think I'm going to have to start doing Friday funds of servers and I'd like you all to do the same.
But I'm adding web socket support in the next build, so we're going to go.
Two nice little experimental bits show web sockets. Yes, we got a connection established. This is also running a PowerShell function live in my run space. Now, this can be terrifying. This definitely is not something I would recommend doing, you know, willy-nilly on a live server.
on production for everyone.
But for local uses and for app development, this is basically the ball game. We can really easily redefine these things and export them as we want. And it's just nice and easy. So yeah, I feel like it's almost in sometimes appropriate name of the thing.
So here's my actual web socket. What I'm doing here is I've got the function that clears the web socket or endpoint, takes time.
or checks if it's a WebSocket request. If it is, it spits back two objects and returns. Otherwise, it spits back the web page. Here is our web page. We're going to refresh it one more time. Actually, I'm going to show one more thing that's really cool before we do. We're going to get job. We've got two HTTP servers running and a WebSocket server. Let's go ahead and select the last one received.
Keep.
Let's get rid of our keep here for a second.
Send another JSON message, and there is our WebSocket message. Send a few more.
Congratulations! Now for the extra fun part, we get job, we're just going to refresh this page, which will actually close out the web socket.
give me a brand new WebSocket background job, which I am ready to receive results on. So we've got browser to terminal round trips, nicely done and integrated. And we've got an app story integrated into our editor for free.
basically, for the first time since ISE changed its thread settings to STA all those many, many, many, many, many years ago. So we can now write really, really nice, fun, functional plugins for VS Code with PowerShell.
We can write really, really simple servers with PowerShell. This sort of pattern can be adopted. I'm talking with Badger Roddy from Pode. I'll be talking with Adam Driscoll from PowerShell Universal to try to basically allow us to take this format of writing functions and export them as many places as we can. Just asking for a bit of quick feedback from the kind of tribe here. Is this obvious enough?
Is this this naming format? You know, this makes sense that it's a server. Everybody, you know, see how this can work.
Let's go write a really basic one for a second. Do function slash hello world, back to our original one where we show that hello world and the deep time ticks. Let's go, still the same server.
By the way, now I have completed my WebSocket job.
And.
Again, I'm able to rapidly refresh and live reload. Let's get to a little bit of the extra crazy here though. So I've just set an alias to write progress. And, you know, do not try this at work. Do try this at home.
I'm going to go ahead and provide query parameters here. Activity is foo and status.
Comma is bar and percent.
Sorry, for whatever reason, typing in this title page or address bar is slow.
Comma percent complete.
You can type, I believe in you.
Demo gods, be kind.
Aah.
One second, I will get to you.
is whenever it lets me type 5 letters.
This is definitely not a PowerShell bug here right now. This is the VS Code bug here. But I'm going to say percent complete is 50. Just to prove that we're in an interactive terminal.
Does my right progress up there?
Is actually still my right progress down here.
So this is just broken, beautiful and broken. Please never change.
Please, please never change. This really does open up a world of, again, interactive app development directly inside of VS Code. It also makes it easy to build little local home servers. Like if I wanted to do one of these to control my lights, I can. In case the query string stuff didn't make this clear,
It has integrated form support. You can write a normal HTML form and have it pass it down to your PowerShell on your local box. No fuss, no muss, nothing but fun. So.
A, what you all think? B, how foolish am I to name a module something so cursed?
Because I'm either going to have lots of fun with it or lots of fun with it.

Jason Helmick   46:00
Hey, James.
That's great. Hey, James, do you have like a link you can drop into chat and folks can ask you some questions and go out there?

James Brundage   46:09
I did, actually. I did right before I started talking. I'm going to throw it. Yeah, I will.

Jason Helmick   46:12
Yeah.
Oh, drop it in again.

James Brundage   46:19
This is.
And I just go to the website, drop it back in. This is over on GitHub at poshweb slash fun. And it was inspired by this thing that I wrote about a year ago, which is a gist of just a server without a route table, which is actually kind of.
worth doing its own quick mini demo of here. I'm just going to go grab a raw file content here, control C this, blah, because I'm uncreative with my file names right now.
And this was the original demo that kind of kicked it off, which should also now run inside of the terminal. Yeah.
So this is again, server without a route table, just functions named slash SVG, slash CSS, slash 3D.
and automatically calling itself every few seconds.
And look at those response times. Oh, 3 milliseconds. Oh, what a terrible, terrible round trip time. I don't know if I can do that. I will say that we do get lower throughput if we're doing an interactive shell. I've seen about 3, 500 requests a minute being the normal.
But if we do take this out of being an interactive shell, on Windows, I've been getting between 40 and 50,000 requests a minute. On Linux, up to 75,000 requests a minute.
So.
PowerShell is a fast enough web server to handle millions of requests an hour.
millions and we can write PowerShell servers now as easily as again.
Function slash.
Cool.
and they're live reloading. Let me do one bit of peeking into the covers and show you all one really cool trick along the way. So the way this is working.

Jason Helmick   48:32
Hey, James, you're about 15 minutes in.

James Brundage   48:34
Ah, really? Okay. The way that this is working is there is a quick way we can enumerate all of the commands. The execution context, session state, invoke command, get commands, yada yada yada wild card. The thing most people don't realize about it is that this is live.

Jason Helmick   48:36
Yeah.

James Brundage   48:56
So if I do function slash foo bar, so yeah, I have my list of functions there. When I re-enumerate this.
There's my foo function.
So you can do live reloading based off of basic PowerShell trickery. We can have an integrated browser window that does whatever we want. We can write functions with one line of PowerShell that serve as endpoints.
And I'm having a lot of fun just continuing to prove how easy servers can be in PowerShell. And before the shepherd crook shows up, I'm going to go ahead and minimize this, restore the bug nest, and stop sharing.

Jason Helmick   49:42
Oh, that's awesome, James, especially the pugness, but I really, really like the server work. So folks, go ahead and chat some questions in and go ahead and check out that repo or Mr. Brundage.

James Brundage   49:50
Yeah, Justin had raised a hand. Justin, what you got?

Jason Helmick   49:56
Yeah, so.
Folks, with that, we have reached the end of our June community call. I hope you have a great summer and look forward to seeing you back for the July community call towards the end of that month and the 250 year, the PowerShell 20 year, all that kind of great stuff. So have a wonderful month. See you next month in July.
Take care.

