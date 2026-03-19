PowerShellOpenSSH Community Call-20260319 Meeting Recording
March 19, 2026, 

Jason Helmick started transcription

Jason Helmick   0:03

All right. I think we have quorum. So let's go ahead and kick it off as usual. Good morning and good afternoon and good evening to everybody. And and yes, I am going to say this every single month this year. Happy 20 years of PowerShell, everyone. This is a great year for for our celebration. So welcome to the March 2026 PowerShell.
Community Call. Everybody is welcome to join and discuss the PowerShell project and its development. We have an agenda and what an agenda we have posted in the meeting chat and a link to the discussion. If you'd like to check out the discussion, feel free to post any questions in the discussion or right here in the meeting chat since you're here.
And most important, please read and follow the code of conduct. With that folks, we have an action-packed agenda and I'm not going to wait. We're going to dive right in with an MAR update from Sydney. Hey, Sydney.

Sydney Smith (She/Her)   1:40
Hello, thank you for having me today and thank you to Sean Williams for asking a great question in the GitHub issue. I did want to just kind of like address some of the questions in there because I thought it would be useful for maybe everyone.
Difficult to use or really impossible to use with other enterprise OCI registries and I have some good news about this. So this was I wouldn't call this by design, but this was a known limitation maybe is a better way to put it because when we added OCI support and PS resource get.
There was no.net library for ORAS and this like put us in kind of a difficult position as to how to proceed in the interim while the ACR team was building out that library. The good news is that the library is the.net library is out and available today.
And we've seen some of our partner teams adopt it really successfully. We also have an open PR for us to adopt it. We thought about getting that in for the 1.2 release, but it felt too risky. So we're getting it in with 1.3. So look forward to 1.3 preview with hopefully that full or as support.
So you can use any container registry. You won't be constrained to Azure specific ones. I'm excited in particular to see how folks use the GitHub container registries. I think that'll be really useful way to distribute packages. So that is sort of like 1 component of it.
As you know, we are enabling MAR by default in PS resource get 1.3. Is there a timeline for migrating Microsoft 1P packages to MAR? So as you may or may not know, today the only PowerShell packages in MAR are.
Or the set of AZ modules, which is not a small amount of modules, so I say only with a bit of caveat there and then PS resource get is also available there. We're working on our team adding sort of like the key important modules that we're still developing.
Actively to that, I've heard a number of requests for like the M365 module, so we've been in touch with their team on that. My ask for you all is that.
I'm going to post a discussion and if you could go there and say which modules you would want in MAR and if you have a short blurb as to why, that helps too. And I know you've heard us say this a million times, but like when we go talk to these partner teams and try that to convince them to onboard.
To our solutions, it just makes the conversation a lot smoother when we have some customer verbatims that we can link them to. So that would be great. Let's see what else.
Well, packages migrated to NIR continue to be dual published to the PS gallery for some period of time. The answer to that is yes, because Windows PowerShell doesn't have the latest PS resource get and so it doesn't support OCA, artifacts, etcetera.
And so we will continue to publish to PS resource get there for those customers.
And then for customers who need to mirror the mirror, is the your guidance to do so with MARPS resource get to PowerShell users can still consume packages through the mirror. So what I'm thinking is that.
It might be useful for me to put together sort of a blog, semi road map, but really a current state blog as to like what we see as the like secure supply chain modern way to consume PowerShell packages and I can include some guidance there on how to kind of handle these various.
Package sources, et cetera, and I will use this community call as sort of a deadline to this and.
Get you a blog by our April community call on sort of like what the landscape is around package management today. I see there's a number of questions in the chat, so just give me one second to read through these before I.
Go on to my next topic and make sure I get them answered.
OK, so the.net issue, I see that a number replied to that and she's going to look into it. I don't know specifically what's going on there, but.
Oh, I think I do know maybe what that one is. Are you talking about there's an issue where like one get pulls down, the one get bootstrapper pulls down a newer version of.net or a noncompatible version of.net? Maybe it's an older one.
Um.

Anam Navied   7:07
No, this is the this is the issue where the delete basically like directory delete API that we're using from.net has a bug on Windows PowerShell, yeah.

Sydney Smith (She/Her)   7:15
Oh.

Anam Navied   7:22
But it works in PS Core. But yeah, we can discuss that one.

Sydney Smith (She/Her)   7:22
Yes.
Cool. Well, MARB prioritize before after PS resource get in 1.3. So we had some conversation with like when we add this of like reevaluating how we're handling priorities. I can't remember where we landed on this. I'd have to look at the PRI don't know if Aditya, Amber and I'm you.
You remember what the decision was there, or if we still needed to decide.
OK, let me get back to you on that one then. Um.
Scrolling through.
How cross repo dependencies work? How PS gallery module have a dependency on an MAR module? That is a really good question. Today we don't have any way of.
Doing that with our installation, there is an open PR around platform specific installation and maybe some of the learnings there will be able to apply to this question. But yeah, unfortunately don't have a great answer.
For that at the moment, but it is something we've been thinking about.
I see one to that.
There will be no Windows Update to switch to MAR for five one, so Windows PowerShell will be staying as it is for the time being.

Sean Williams (SIE)   9:02
In that April blog post, could we get like some some timeline details on how long that time being would be expected to last just so that like.

Sydney Smith (She/Her)   9:05
Yeah.
The time. Sorry, I didn't mean to interrupt you.

Sean Williams (SIE)   9:15
No worries. Obviously change is happening right? And for customers that have their own internal mirror of PS Gallery just as a generic Nougat mirror.

Sydney Smith (She/Her)   9:20
Mm.
Yeah.

Sean Williams (SIE)   9:31
We'll have to set up an OCI mirror and it will be easier to prioritize setting up an OCI mirror of hey, the status quo is changing for PowerShell in, I don't know, six months versus hey, the PowerShell status quo is changing without like a a defined timeline because it makes it a little easier to plan.

Sydney Smith (She/Her)   9:48
Yeah.
Totally, totally makes sense. I I'm happy to include that. I think the like party line for today is probably that we have supported NAR for some fairly specific reasons.
Which are that we needed a Microsoft trusted repo for a number of enterprise customers to be able to consume Microsoft packages and that's really like the the bottom line there. The other portion of that is it was the.
Pretty much the only way we could get Powertel modules into air-gapped cloud, which was also a a big need. So if you fall into that bucket of like, hey, I need a trusted package source, the recommendation would be NAR PS Gallery is by default a non-trusted.
Repository, but we will continue to support it for the time being because of legacy workloads and we have a obligation to not break folks who may be depending on it. But yes, we'll give more of an update.
In the blog.
Does that sort of make sense though, or any questions about that?

Sean Williams (SIE)   11:07
I I think it makes sense. Some guidance on setting up an enterprise mirror of MARI think will probably suss out cause like obviously on paper other Nuget feeds are supported, but in practice like with Artifactory we discovered there were a lot of.

Sydney Smith (She/Her)   11:15
Yeah.

Sean Williams (SIE)   11:26
Different provider specific bugs that had to be worked around and.
I I foresee that the same thing will probably happen setting up an OCI feed on Artifactory and trying to use that in PS resource kit. And so getting some guidance on how that's supposed to work so that we can validate that it does in fact work in situ would be really appreciated.

Sydney Smith (She/Her)   11:38
Sure.
Yeah, absolutely. Yeah, I appreciate that. Um.
Just filtering through the comments.
OK, I think we're good there. I see one about assembly load context. PS resource get does use assembly load context, but I'm happy to look more into the issue that you have linked there.
To the issue that you have linked there.
OK. Thank you guys all for being really great with this conversation. I know it's not always an easy conversation and we're not moving as quickly as we would like to, but definitely always appreciate the feedback and.
One other topic.
Is that we are working on a remote Azure Management MCP server and as a personal favor, I could really use some of your help with testing out the authentication experience. This is then part of the experience. It's been extremely.
It's only difficult for us to test against the various customer environments and settings, so if you're willing to help us test, I would really appreciate it. I will note that this is going to be like we're calling it a limited private preview right now.
So the server, the MCP server, right now it supports read requests, generating read requests for a RG. So you could say something like this is kind of a silly example, but like what are all the VMS in my in this subscription, etcetera. That's kind of like the functionality available in the MCP server today. There is a longer road map towards.
Adding more tools, write support, etcetera. But to move forward we have to figure out that authentication works or it doesn't work. The other caveat I will say is that right now we can only enable you at the tenant level. So that's kind of unfortunate. I wish we could do it based on subscription, but.
For this limited private preview, it would be you would have to be comfortable enabling the server, the MCP server at the tenant level. So if you're willing to help me out and sign up, I have a form for you.
It's just going to ask for your tenant ID and then your e-mail. So if we onboard your tenant, I will reach out to you with specific testing instructions in your e-mail and how to use the server for today if you want to start using it and playing with it, but I will put.
The form in chat. I'm also seeing some really great conversation continuing on in chat, but for the sake of time, I know we have a loaded agenda. I will pass things back to Jason and respond in chat.

Jason Helmick   14:58
Thanks, Sydney. That was awesome and I'm really looking forward to the MCP results that you get. Thanks folks for helping out Sydney and and and collecting some of this information. Folks, we have some information on a on our 7-6 release in DSCV 3. So I'll turn it over to Steve to talk about those. Hey, Steve.

Sydney Smith (She/Her)   15:00
Yeah.

Steve Lee (POWERSHELL)   15:20
Sorry, I didn't realize I was talking about 76, but I believe 76 just went out. Maybe one of the maintainers might be better if they're here. I don't. So I see Patrick here.

Jason Helmick   15:26
Yes, Sir.

Aditya Patwardhan   15:31
Yes, it did release.

Jason Helmick   15:31
Hey Sean, can you give us the link to the blog post?

Steve Lee (POWERSHELL)   15:37
Or did you want to do a quick talk about 76 or Sean?

Sean Wheeler   15:38
Yep.

Aditya Patwardhan   15:40
Yes, we have released out 7/6 yesterday and today. Some of the packages might be still coming out, but it is GA and it is available on GitHub as as we speak, but it will show up eventually over the course of the day.
On PMC and packages.microsoft.com and store and et cetera.

Steve Lee (POWERSHELL)   16:03
I guess I'll just for the for the 7.6 topic, you know we know that we came in a little bit late and I know there's some concerns from especially enterprise customers about support. So we we cannot extend 7.4 LTS support or boundedby.net.
But we're looking at, you know, what happened and how we can improve so that we can get better at SIM shipping with the next.net.
All right, let me move on to DSC then, and I'm gonna actually do a quick demo if I can find the right screen here.
All right, so DSC 3 two preview 13 was just released. I want to say maybe half an hour ago. It may or may not probably it's been submitted to the store, so it probably won't show up till later today or tomorrow. All right, but you can get it from the GitHub.
Repo, but I want to just show a couple of new things that may be interested to folks. So let me do a quick here's the list of resources. So I've been using if you look at the my branch name agent skills, I've been playing with copilot and agents and stuff like that, building some resources and.
But learning success, I'll say all right, so I'm trying to get better at it. But for example, what is in preview 13 is a new service resource which I'm going to demo real quick and also optional feature list is in there as well.
Feature on demand is on my machine, but there's a PR open for that one, but that one's not in yet. That should be in for the next preview, which I I'm planning. Ideally, barring any major problems, will be the last preview before I go towards a release candidate towards GA, all right.
So let me just show this real quick. Here's a very simple YAML. So I'm going to play the SN SNMP trap service. Since I don't use it, most people probably don't anymore. So this is how it looks. So this replaces the old PSDC service resource.
Which one of the reasons I was motivated to to write this is that the old resource which is MOF base doesn't work with PowerShell 7, but because the off of DSCV 3 it requires PSDSCV one which has to go through the LCM, it has its own challenges. So it basically wasn't working very well.
With UCV 3, but now if you wanted to for example.
See, let's do a quick test here. So again, if you go to this simple YAML, right, so I'm seeing, I'm checking that this service has stopped. I believe I have manually started the service. You can kind of see here desired state stopped.
Actual state is running and tells me that the different what's different between these two and again it's comparing left to right or desired against actual. So these other information ones don't show up is the status is different. So if I wanted to now stop it, I can just do this you can set.
This thing and because this is all now written in Rust, it shouldn't take very long. So before state it was running, it's now stopped and I can do this to confirm it is actually stopped, right? So that's one.
The other thing I want to demo real quick is both the service and the optional features. This is the Windows optional feature supports get, set and export. So here I'm going to get all the services that are running and I'm also going to get all the installed Windows optional features and this is a list because I imagine.
Managing optional features. You're gonna have multiple, not just one. So let's do the big X port. So if I run this one.
So you can see there's a lot of information being put out 'cause there's a lot of running wasn't looking for running services. But then I can also add other properties as filters so I can do a name. I don't want to do that. Let's do SSH, one of my favorites.
And also name and description support wildcards so you can use asterisks and then for this one I can also this is an array so like OK copilot is telling me I can do Hyper V but I can also do something like this state.
Or rather, let me see what else came up this besides Hyper V.
Uh, let's click on.net.
So instead of state, I could also do a name star dot net star like this, right? And basically any properties within one item is a logical and and then any additional objects is the OR. So it's going to find anything that's Hyper-V that's installed.
And anything that is.net irregardless of the state. So let's save that.
Yeah, let's rerun that real quick. And if I didn't make any live, oh man, oh, it's not name. What did I? OK, so here this is a good that's here so I can do a schema.
And this is where I.

Bohren, Andres   21:10
Missing a dash in the name.

Steve Lee (POWERSHELL)   21:12
Was there? No, no. It's complaining about. It's complaining about the property called name.

Bohren, Andres   21:13
Missing a dash in the name.

Aditya Patwardhan   21:18
Is it feature name?

Bohren, Andres   21:18
Yeah, it in in your YAML file it's missing.

Steve Lee (POWERSHELL)   21:19
Oh, it might be feature name. Hold on no no. So so this is where you can look at the schema. I OK, so this is definitely name. So this is where I also need the error new sync. We have an issue open for that for errors to include the the resource name. But what is it called? Optional feature list.
I I don't think it's name. I think it's actually feature name, yeah.

Bohren, Andres   21:41
In your yml file, I think there is missing a dash.

Steve Lee (POWERSHELL)   21:46
No, no that that part's fine. The the problem is the property is called feature names. It's not called names. Maybe I should change that. I don't know. By the way, as people play with it, whoops, that's not what I want. I want to run this. So so again, these are out in the preview and these are 0.1 versions, so they are experimental, so the.

Bohren, Andres   21:49
OK.
OK.

Steve Lee (POWERSHELL)   22:05
I'm open to feedback to making these changes so you can kind of see here I only have the SSH server running, I don't have the agent running. And then in terms of the optional features you can see all the Hyper V ones. I didn't find the net one.
I know I had this problem before. I don't remember.
Sure.
Actually, I think I remember. Wait, go here real quick.
K.
I think it's because dot oh dot. It's in the display name, not in the feature name. So I can go instead of having this feature name, I could do the display name.
I'll stop it.
All right, let me just run this real quick. All right. But anyways, my ask is please play with these new resources. Again, keep in mind that they're experimental. There's 0.1 versions, so don't use them in production. The design will change based on feedback, so stuff like whether we should call feature name or name.
Things I want to know. Um.
Again, you can see the dot was up here now, so let me know. Open up as issues in the DC repo and you know we'll we'll get those fixed. I believe that's all I got for this subject, Jason.

Jason Helmick   23:28
Great. Thanks, Steve. All right, folks, I've got a couple of topics that I'm going to kind of drill into. The first one is we have a discussion open and I wanted to point your attention to it. As a matter of fact, I'm going to drop it in here.
We're collecting information about this topic right now and it's kind of important to us. This is about the PowerShell update notification and we've gotten a lot of feedback on this and I know folks have given us feedback on how we we could improve this experience and how we might change this experience, but this discussion is very.
Focused on your thoughts. We're interested in your thoughts about an all up either removing it or keeping it. So it's it's a more focused conversation around whether we should keep the PowerShell update notification or remove it in its entirety.
We've had a lot of feedback going in both directions, so we want to bring it to the community and have this discussion open. And I think we're going to talk more about this probably in the next community call after we've had a chance to get some feedback on it. So please, if you get a chance, did I?
Put that into chat. I think I hit enter. Yes, I did. OK, awesome. So yeah, take a look at that and give us some feedback on the update notification. The other thing that I wanted to talk about was Brew. Now for those of you that don't use Mac, maybe this conversation is is not that important, but.
Those of you that do, the BREW conversation is is kind of an interesting one. Just to kind of start things out with, I want to say that we are still working investigating the the BREW. Why did all of a sudden I forget the N word, the notarization?
The the notarization for Brew, I know some of you know that the.net team does this. Things are a little bit more complicated on our side of the fence, so we're still investigating how best to approach that. But I've got good news. The interesting thing is that the community kind of beat us to it in a roundabout way.
In a certain way. As a matter of fact, what I want to do is I I'm going to give you a PR here by Gunny Bush that happened, I think. Sean, you can correct me if I'm wrong, but I think it was last week or the week before.

Sean Wheeler   25:47
It was last week.

Jason Helmick   25:49
OK, and so I just dropped it or here let me hit the enter button. I just dropped it in this PR. What this does is it doesn't use notarization, but it it it is a formula for brew that builds from our source directly.
And this is another way that you can use a community supported way similar to how we do snap with canonical. This is another way that you can get a Brew update for the latest release of PowerShell. Now I think when I looked at the formula and I tried this out myself, it was on 755 which was our our latest stable release at the time.
When this formula was done and I tried it out and I'm going to give you the doc that we have for it. I had a great experience doing it. The only caveat that I experienced, and I'm kind of interested in other folks that run into it, is that.
You do have to remove any prior installed casks that you may have had for PowerShell, but take a look at our docs here. Our docs go through how to use it. It's pretty simple. Brew install PowerShell and if you need to remove and it's got a note in there about removing any prior cask and off to the races.
We haven't had a whole lot of time to experiment with this and and check it out. It is a community release so I would suggest you know if you have any issues or something you you go out to the the Brew community and and submit those there. But I I just want to point out how much we really appreciate the community effort and help.
And the help that Gunny Bush had done specifically in giving us both a heads up and helping us with the docs update. So thank you very much Gunny Bush and thanks to the Brew community for making PowerShell available through it with that.
Let me go on to the next topic that I have, and this is this is from the previous community call Linux distribution update. Specifically, we've had questions about when we're going to have our RPM packages for RHEL 10 and Debian 13.
And with 7-6 just releasing, those are in the pipeline. I'm very happy to say they're in the pipeline. I was also going to be very happy to say that they are out and available, but we ran into a snag last night that we're fixing. We understand the problem and we're fixing it. So give us a couple of days.
And you'll see these distributions come out. We are also and I would, I don't have a timeline for this, but you may have seen me comment in the chat earlier. We are also working on the Deb ARM support. We'll let you know I would don't expect that before the REL 10 in the.
Debian 13 release that'll come sometime after and I'm happy to share more information as we get closer to that. But one of the things I did want, I just love this. So we have some internal tooling to help us.
Kind of understand what operating systems are reaching their end of life as we're managing it with distros and we have some internal tooling that helps us understand what we have published and what's available and what the.net team has published.
And oddly enough, we're sharing it with everyone. So what I'd like to do is have Sean come on and show you the work that he's done, both helping us internally and making those tools available to you externally so that you can also watch what we're doing and where things are and stuff like that. So, Sean, let me just.
Shut up and turn it over to you. Show off your distribution tooling for everybody and where they can get it.

Sean Wheeler   29:33
OK, I'm going to drop some links in chat here about all the things I'm going to be talking about. I created this module called Release Info that I'm going to be demoing today and we'll start with a list of the commands you see here.
I'm not going to cover all of these commands, but most of them and you can get it from the gallery there. The first set of commands I want to talk about are related to this site called endoflife dot date.
Now this is a if you're not familiar with it, this is a crowd sourced information site that they track releases for operating systems and applications and server applications and so on.
And make it easier to find support information about what's supported, what's out of support, and so on. And they also present a very simple REST API. You can get more information about the API from these links at the bottom of the.
The page, but this is just one example for Ubuntu. So the first one I want to show is this get end of life.
Command that I wrote and you give it a category and it'll list all of the things in that category and um.
It end of life. This is there's tab completion, so this will go through all of the categories that they've defined.
And so when you want information about a specific product that they've cataloged, you can just pass it the name. So here's an example of Ubuntu, the whole history of Ubuntu.
And it shows whether it's at end of life, whether it's LTS and so on. So that's the kind of information you can get from it. And then I created this other command called.
Yet OS end of life and this one specifically is for Um.
No, I didn't. I want just the selected.
These are all of the operating systems that PowerShell supports, and so you can see what the supported versions are. Now I'll point out that in Ubuntu here it's listing.
Questing Quokka here and that's 2510, but that is not an LTS release and so in Ubuntu specifically we only support the LTS releases, not the interim releases. You could probably install PowerShell on that, but.
You know your your support is community based at that point, so that that helps you identify the OSS and things. The next one is.
Uh.
A set of functions that are GitHub based and this command gives you a quick view of the PowerShell release history. So running it without parameters it shows you it uses GitHub Graph API to query.
Releases and shows you the current release for each version and whether it's end of support and so on. You can get more detailed information about a single release so you can see all of the.
You know the the releases in between.
And there's a few more options. All of these functions have comment based help so you can get help on them. There's also this get release package again queries releases so you can see what files are available in the release.
And with it there are options where you can download the package you want, so that makes that nice and easy.
The next one is for packages.microsoft.com. I have this command that will search for the packages we've published there, so this goes out to packages.microsoft.com.
Um.
Finds the metadata for our releases, parses through that, and outputs what releases are available, what releases have been published to PMC, and as you can see, we're 76GA has dropped, but we're still working on.
On the pipeline for the PMC releases, so we're not quite there yet, but this is a way that you can kind of monitor as things progress.
And lastly, we have some Docker oriented ones and there's two commands. Oops to look at. Find Docker images and find docker.net info. Find Docker images uses the.
It queries Mar using the REST API to give you a list of all of the Docker images that the PowerShell team published for PowerShell. Those are no longer being supported and updated, so this is kind of a dead command.
I was originally intending to have another one that would query for the.net SDK images since they contain PowerShell, but you don't get any PowerShell information from that query, you only get information about the SDK image.
Which contains PowerShell as well. So this other command find Docker info requires you to have a local clone of the net Docker repo.
But it parses the Docker files in the repo to see what's included in them and gives you this output here. I suppose we could point it at the GitHub repo and grab grab the same files, do the same work, but you get the idea.
These were things I threw together to make my life easier, and I'm sharing them here for you guys. And that's my demo, yeah.

Jason Helmick   36:39
Hey, Sean. Hey, Sean, you you owe me a dollar. I think. I think it took Alexander all of about a second to get. So Alexander, Sean and I were laughing about this earlier. We wanted to know if anybody would catch the fact that he had a plural in one of his nouns and.
And as a matter of fact, Sean, I think you owe me an extra 5 bucks because it was Alex. It was going to be Alexander or J. Cool, one of the two. So thank you Alexander so much for making me a buck.

Sean Wheeler   37:05
Yeah, yeah.
You're you're lucky that I did the work to add comment based help and spruce up the code a bit to to make this public because this was just a hack job in my private repo for the longest time.

Jason Helmick   37:18
That's right.
Well, look at it this way, Alexander. We look forward to the PR, so.

Steve Lee (POWERSHELL)   37:24
What is?

Sean Wheeler   37:25
That would be a breaking change. Just real quick, not much to talk about on the docs update front, although we've updated the docs for the 7.6 release.

Jason Helmick   37:28
Sean, did you have another topic? Oh yeah.

Sean Wheeler   37:44
I've spent some time here recently refactoring the setup documentation to streamline it more, hopefully make it a little simpler for folks, and I'm going to go ahead and show.
This um.
Alternate ways to install PowerShell. This is where the Homebrew information now lives. I moved it out of the Mac OS install article because Homebrew is now a community supported solution.
So this is where you go to to find that information and that's it.

Jason Helmick   38:28
Awesome. Awesome. Well, folks, please feel free to chat in questions and things like that as we've been quickly moving through this agenda. But now we're up to the demo for the day. Thomas Neto is going to do us a demo on DSCV 3, the LCM plus a pull server. Thomas, let me.
Make you a you are. I think you are now a presenter, Sir, so please have at it.

Thomas Nieto   38:55
All right, can you see the screen here?

Jason Helmick   38:56
Yep, sure can.

Thomas Nieto   38:59
All right. So currently DSC is really just a platform layer and I think the Microsoft team has really pushed that, right? Let's get the XC, let's get the contract in place, but the solution layer around that, so.
You can define your configuration documents, you can push them to a node, you can have a scheduled task, go and run them over and over again. And but that doesn't really buy you enterprise level deployment mechanisms as well as monitoring mechanisms. And So what this is, I went ahead and created a.
LCM replacement and a pull server replacement using DSV DSC V3 and it has the an API behind the scenes so and a blazer UI and So what you can do here you can see on the screen there's a dashboard it will show you your.
All your nodes, your nodes that are in compliance, nodes that are not in compliance and error status as well as the LCM state. If it's currently idle, testing or setting the configuration and historical histories of that, you have your list of nodes. You can use either a local configuration source or if you wanted to do it.
In the pull server you can do that. You can manage all your stuff from here, defining a configuration so I can pull up that real quick and create configurations which are uploading your YAML or Jason files or any other files type that DSC supports via.
Extensions and so you can see here this one's just a simple registry resource that will take one arameter called mytest and put that into the registry. And what you can do here is you can also compare that. It shows you a nice diff view if you have different versions that you want to publish.
Once that's out there, the node will then on its pull interval will wake up, see if it has a new configuration for it, a new version. It will pull down that new version, apply that new, test it first, and if it's not in compliance, then it'll do a set.
And there are two operational modes. There is a.
Monitor mode, which is kind of like the old I think just check status. I can't remember what the old name of it was, but in a remediate mood. So one just does test and one does test and set. You can configure how long the interval is between checks and if you want to report the compliance off that.
So you can see here this node has been running for every 15 minutes for a while and you can see the report that got pulled back. Here's the report details of it. It has the breakdown of that YAML file that gets returned or that Jason file. And then there's also a view here that can show you.
Each individual resource, if it's in compliance or not, or in desired state, and then if there is a difference, it'll show you a nice diff view of each of the properties to show which ones are expected and which ones have changed. So if I go back here.
Here's one here.
You can see here it has this value and is different from what it expected. Where is another one here? Let's see.
There we go. And this one is a just off of a set here. So you can see it was originally this value and now it's this value. And one of the nice things about this system is that you can have your configuration documents have parameters and those can be generic.
So you can have one configuration document that can have span regions, environments, anything like that, and then you can provide runtime values to the nodes via the parameters. And so if you're ever familiar with, I think Puppet has Hira.
There's a similar concept in this where you can have a configuration default scope. Actually, let me show this first. There's a scope system where there's a precedence and you can basically tag your nodes to either a region or environment or whatever custom scope you want. You can reorder the scopes, you can enable or.
Or disable them whatever you need for your environment to manage that and then once you have those defined, you can then go ahead and create various configuration parameters. So here I have a version one and this is in the default scope, so every single node with this configuration would.
Get this value and then at the different layer scope. So let's say my node is in production, it would also get this, but it's kind of a pain to look at it this way. And So what you can actually do is there's this provenance view so you can see at each of the different layers.
The value was default scope value at the default scope, prod scope value at the prod scope and then the highest level here was our US scope value and that was ultimately what gets passed to the node for it to pull in. And so I can do a real quick demo of just going to go ahead and.
Set this to some random value.
Let's go ahead and restart the LCM since it's waiting.
It's going to go in there, check to see that it's not in desired state and then it's going to go and run the set operation. It says one was successfully completed and so now if I look at my report.
Just look at it this way for my one node you can see one got pulled in from broke to now the value that I expected.
So there's a whole lot of stuff in here. I don't want to go too deep into it, but I think hopefully this is useful for people to start to play around with and test in their environments and see how that works. There's a whole other, you know, not only can you do configurations, but you can do composite configurations. So like imagine you have.
One team or one configuration is just configuring IIS, and you have a different configuration that configures the firewall or whatever. You can combine those for a collection of configurations to apply to that node, so you can have different teams and different responsibilities for that.
So yeah, this was a high level demo of the LCM and pull server replacement for V3. Thanks.

Jason Helmick   45:24
That's awesome. That is awesome. Thomas, thank you so much for showing that. And I I I just want to say it out loud for everybody. I we we really appreciate your contributions to the DSC project and to the DSC working group and what a great demo. Thank you so much.
Appreciate it. Hey folks, we're getting towards the end of the call, so if you have any questions, feel free to type them in. I think we're pretty much caught up, but you might be saying to yourself, hey, what if I think of a question this afternoon or tomorrow? How am I going to remember that to the next community call? Well, thanks to your advice.
We now have all of the community calls pre-created in the discussion list. So if you think of a question and you didn't type it in here and you didn't get an answer or something like that, the April community call is already created. Feel free to go out there and put your questions in and we'll get some answers for you and.
Stuff like that. All the good stuff. So with that, is there anything else in particular that is on anybody's mind? I'm checking with the PowerShell team here. I'm checking with you folks as well.
Yes, the Tetris Tetris is is awesome. Thanks Thomas for putting in the repo.
Well, that's great. OK. With that, folks, let's go ahead and end the March community call. Once again, thank you so much everyone for your feedback. We really appreciate it. And again, happy 20 years of Power Show. We'll see you next month.

Jason Helmick stopped transcription
