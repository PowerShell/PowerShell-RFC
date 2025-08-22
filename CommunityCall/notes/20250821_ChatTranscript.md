Transcript
August 21, 2025, 4:29PM


Jason Helmick   2:12
Summer edition of the PowerShell Community Call. Everyone is welcome to join and discuss the PowerShell project and its development. We have an agenda posted in the meeting chat in the link which I just put in the chat message, so if you want to check out the agenda.
Please feel free to pose questions either in that discussion issue or right here in the chat here. I think most people do it right in the meeting chat. Please check out and follow our code of conduct as well. And with that, let's go ahead and dive in. We've got a pretty big agenda. First thing on the.
I just saw the smooth turned up to Max and there's an egg. I look like an egg. The first agenda item is a conference that several of us were just at last week, which was Tech Mentor. Tech Mentor runs. The organizers for that run a couple of of of great conferences every year. Orlando and.
And over here on the West Coast, here in Redmond, Tech Miner has been around for, wow, a really long time and the organizers do a really nice job with this. We had a great time. I want to thank the organizers for inviting us over. So myself, Sean Wheeler and MVP Stephen Judd and oh, our friend.
And James Petty also was there and we did several sessions. We did an all day getting started with PowerShell thing, which I'm really glad we I love doing those and I'm really glad we got a chance to do that because I have got a follow on to the story.
We also got a chance to talk about things like the history of PowerShell. We we talked about Platty PS, DSC, Sean and I got a great chance to to share a I Shell with folks and talk about that. Steven couldn't make it so. So Steven, we did our best not as.
As good as you, but I think we did a pretty good job representing AI Shell for you. So we had a really good time and again, I'd like to thank the organizers for it. One one thing that it was pretty interesting that happened last week and this is one of the reasons why I always like.
Helping folks getting started with PowerShell is I've always used this phrase for about the last 20 years with people that are saying, you know, you know, you know, do I really need to help people, you know, learn this or do I really, you know, want to take the time to help somebody get better at PowerShell, that kind of stuff. And the phrase that I always say is that there's.
There's new IT babies born every month and I think it's kind of a cute phrase and it's a testament to the constant growth of the IT community. Obviously there are new IT babies born every month, but at Tech Mentor we had a great representation of this after the the the the initial.
Where we spent all day talking about getting started with PowerShell, someone came up to me and was asking me questions and he said to me all of a sudden, he said, I bet you I'm the youngest person you'll have all day today. And I said, really, how old are you? He was 15.
He was 15 at a conference here at Microsoft and the reason that he was here was because he was interested in getting into IT. The reason he was interested because, well, his dad works in IT and I just want to give a huge shout out to his dad and to Levi.
Who showed up to the conference? I think it's great that if you have interest to be able to go to a conference like this. There were folks like Rob Hinman from Microsoft talking about Server 2025, whole bunch of consultants and excellent technical people talking about solutions around Azure and on-prem solutions.
What a great environment. And it turns out that he his dad told him that, you know, if you're going to get into IT, one of the first places to get started is with PowerShell. And so he not only came to our one day session, he then came to all of our sessions and asked great questions and so.
I just want to give a huge shout out to Levi and to all of the new IT babies that are born every day and welcome them to PowerShell. And as long as you keep coming, we'll keep talking about it so.
So with that, let's go ahead and go on with the oh, and I just throw this out to Sean Wheeler if he if he wants to to say anything. But with that, let's go on with the rest of our agenda and let's talk about Dongbo. Are you around? Can we talk about the 7-6 preview?

Dongbo Wang   6:40
Yeah, hi. So we are preparing for the 776 Preview 5 release, but it's still blocked right now on some internal work. So we're working through that and hopefully we'll.
Get to release out next week or the week after next week if if we can. So that's that's a yeah it's a it's a it's a lot of preview before the RC. So we try to get things done and to verify the work we do in the like around the build.
Around the around the the the partial build infrastructure, so it takes some time.

Jason Helmick   7:22
Thanks, Domo. And as as Domo was talking about, one of the other things I want to point out and Sydney was reminding me of this the other day. These preview releases that we're getting to now, we're now starting to get to that time of the year where we're starting to look at RC as we come up to the 7-6 release.
And this is going to be a 7-6 long term service release or LTS. So it's really important that we get stable before we do the release. We ship on quality, not on a date. And as Sydney was reminding me, remind everybody to say, hey, make sure that you're testing these and you're checking out our previous right now.

Dongbo Wang   7:57
Um.

Jason Helmick   7:58
And let us know with feedback if there's anything that we need to be aware of before we release the long term service release. So welcome to the ship train and help us out with that. We really appreciate it. Next U is oh, I'm so excited for this. Steve, you want to talk about?

Dongbo Wang   8:00
Mhm.

Steve Lee (POWERSHELL)   8:11
Well, actually, why don't we, why don't we have Dombo continue 'cause he he was already talking and then we'll get to me next.

Jason Helmick   8:19
Oh, I'm sorry.

Dongbo Wang   8:20
Yeah. Okay. Okay.

Steve Lee (POWERSHELL)   8:20
On the air show.

Dongbo Wang   8:24
So today I will try to demo the AI shall work. We have let me.

Jason Helmick   8:28
Oh, OK. Yeah, this is great.

Dongbo Wang   8:31
Share my screen.
All right, let me get started. So AS shell, AS shell consists of an executable called ASH, so we can see that and and also a partial module called AS shell. It's the same name, so like.
Import AI shell and you can see that.
It's it has exposed 4 commands from the ASL module and their corresponding aliases are.
Are like AI run, ask AI fixation and AISH. So the start ASH start ASH shell will open ASH in the cycle of the current PowerShell tab. Invoke ASH allows the user to chat with AI shell from within PowerShell.
And when running to an error, the resolve error command can help explain the error suggests a fix using AI shell and the invoke AI command is a utility command for AI shell to run commands in a connected partial session. So.
We will first run start AI shell to open it up.
So AI Shell is deeply integrated with a PowerShell session from where it's you know started. It's using a bidirectional channel to do the to do the like integration with the PowerShell session so you can interact with the AI Shell from within PowerShell and vice versa.
Uh, let me let me let me quickly run a query and see how it does it.
I see I want to know about the configs for git.
And that's a command I want. So in this case I can run code post.
To insert the command into my PowerShell prompt and I can't like there are there are there there's key bindings. There's also a default key bindings for doing this post command action which is a control DD. So I can do that too to do the same.
You can see it's just it's it's just gonna run the same code post command and with the command posted into my PowerShell prompt. So let me switch to the PowerShell and run it.
OK.
So I I like I don't want to constantly switch between AI Shell and PowerShell. So in fact I can stay in PowerShell and run ask AI to continue to chat. So let's say I got this information about where my gig configs are are are stored.
And uh, the configurations, what the configurations are I wanna ask a question about.
About like which file is actually going to be used when I when I specify the dash dash global. So you can see that it sends the query to the AI shell from within PowerShell.
And then I can ask like what about the program file one? Like if the dash dash global is updating my user config files, what about the program file one? Oh sorry.
OK so in this case OK I know that the dash dash system is the one I'm gonna need if I want to update the program file version of the config git config file. So from within PowerShell I can do ask AI post code it will get me the git.
Because we can see that it's put into the AI shell. We'll import the AI shell module. It actually set up a predictor as well. So in this case, if I post the code from the AI shell into my PowerShell session, if there are multiple lines, then you're gonna.
Be like the the the multi line commands will be used as a prediction by the predictor come along with AI AI AI shell module. So in this case I can just you know select this one. Oh sorry, I just select this one.
And like update the the actual values and run it. Or I can like I don't have to run a command, I can I can do the same in natural language so I could say ask AI.
Uh, let's see.
Post the dash dash system command to shell and it will do the same. This is because AI shell has built in tools to interact with the connected part of session. Like I said the the AI shell command like get.
Good.
So those four commands are used for users to interact with AI shell for only in PowerShell. And at the same time, yes you can. If you are staying in the AI shell window, you can you can interact with PowerShell using the building tools within within the AI shell.
And OK, let's go back and show the the resolve error which is like the alias is fix it. So let's say I get an error when running commands. So service I run get service. I can see that there are some red test getting output in here.
So I can just run fix it. It will extract the errors and send it to the AI shell for it to explain and maybe suggest a fix and it's the same for native command. So for example.
You see I have a file untrapped. I want to clean it up so I do clean dot and it failed and in this case I can just run fix it and it can tell me why it fixed. It basically like gets the output of the command you can see over here.
Um.
Command see it gets output fatal clean requires true and dash F and then you're gonna use that use that information as a context to explain the problem and maybe suggest a fix.
OK, um.
So let's get to the AI shell side.
Crash it. Let's we you can we can like within the AI shell you can view the building commands by using the MCP commands. You're gonna show the all the like by the way a power AI shell support and and supports.
Supports MCP servers as well and currently I don't have any MCP servers set up for it and the building tools are actually represented as MCP server even though it's not, but it's represented as so. So you can see that there's a server called AS shell and those are the tools exposed by it.
Um.
So there are commands for retrieving contacts information from the Kine departure session like get working directory, get command history, get terminal content, environment variables and there are action tools that can do stuff like a copy the code into clipboard, post it into a terminal like what I just showed I can post the.
Code in the natural language and it can run commands in the terminal, basically run the commands in the connected PowerShell. So I think this is very powerful as you don't have to remember the commands in flex for what you want to do, just ask AS shell and let it generate a command and run it for you. Of course you can review the command.
Before we run it and learn how it does, how, how, how, how the like, how, how it does stuff in PowerShell for example in this way. So let's try to do something like that.
I wanna list all the storage accounts and my current subscription using AZ PowerShell.
Iran, please.
So I see that you're gonna call the building to run command terminal and.
I'm gonna run it and you can see that it's actually running a command in the AI run a proxy command. The reason we have this proxy command is I I believe many of you have used the the GitHub copilot in VS code and it also able to.
Run commands in the VS code terminal, but it has a support from the terminal layer so that when you are like the internal tools for VS code is like when when when the tool is getting called is actually telling the terminal to send keys to the car interactive.
A shell so everything is handled in the terminal like terminal syndicate to it and terminal record where the output starts and terminal like basically scrap extract out the output because it's node where where where the output starts and this send that output back to the two.
But in PowerShell we don't have the support from the Windows Terminal level, so we have to do it from the partial level and that's why we have to use this process command to actually run it like it handle the streaming behavior and it's able to get all the outputs and errors as if we were seeing in the console and send that to the.
So you can see that it gets all the results I want and it shows it in a table here and I can see like does I'm interested in this one. I'm interested in this in this storage account so I can see.
Does it contain any block containers? It is so.
So we'll try to do this. I can review the command OK it assign it to the name to the name variable and it creates a like try to get a context using the resource group name and the account name.
And then you try to do this. So let me try to run it. It seems correct.
OK, I finished. So you see that there are blobs and there are four blobs and the name of them. So as you can see, it's a great way to interact with the A shell, like if you want to do something, for example in Azure, but you don't know exactly what command to use or what flex to use.
This is a good way to get started like it can generate commands for you and write it for you and if it runs to arrow it will know that arrow and try to iterate over you know to create a better commands.
So now let me show how AI Shell can retrieve information from within the PowerShell. So let's see.
For example git stash push edge. Let's see I I'm not familiar with the git stash push command and when I see the help I see keep index. It's saying keep index. Well it's not very helpful. I don't know what it does. So in this way I can do that ASH. Oh oh so ask AI sorry.
Take my window and explain what does.
Keep index due.
So you can see that it sends a request to the AI shell site and AI shell will call this building to get terminal content to pick the window and then explain it. So it know that OK you are running a git stash push, you are having this content from help and you're asking about the keep index of the of this git commit.
Decommit.
And like like I said before, AI shells also support MCP servers, so I'm going to demo that for now. Like let me quit it and open up the MCP JSON file like MCP JSON files under the root folder of the AI shell.
Configuration folder. So it's very simple, like it just have a service and so we're trying to use the same configuration. Well, similar configuration, it's not the same because there are features we don't support, there are features in the VS codes that we don't support, but it's a very similar.
Feature as in the VS Code GitHub Copilot. So OK, let me add a new uh MCB server.
So I'm going to use the fetch CV server which is like it's able to read URL and get the content from it.
OK, so let me go back and start it again.
So now you'll see there's seeing a like there's a note we'll start it up. It's in a one MCP server config. So run MCP to see more information about it. You can see that now we have one more server which is called fetch and it's supposed 1-2 from it.
Can't fetch. Uh, so let me try it out. Um.
So let's see. OK, I can run this fetch this show profiler from this git.
Let's see if I can do it work. Yeah, I'm gonna call it.
And they're gonna like display the whole script from this show profiler.
And then I can just, you know, follow up. I I can just ask you follow up questions like what does it do? And it gonna explain like the functionality of the script.
So yeah, that's all for my demo today. I'm just trying to show that it's a bridge utility tools like assistant in your partial session for beginners, for partial beginners like the new IT babies. They're gonna be great to, I think they're gonna be a great tools to just.
DNA.
Ask a question and see how it does the work using PowerShell running on the left side and learn from it. And for experienced users they can just focus in their shell and ask questions only when needed. Like for example when I run into a case I don't know about a flag of a command, that's a time where.
I can help. Yep, that's about it. Thank you very much.

Jason Helmick   23:45
Thank you, Dongo. That was a great demo. Teve, it's U to you now.

Steve Lee (POWERSHELL)   23:48
All right.
So I'm going to talk about DCV 3. So we just had a preview for release last Tuesday. This is a change log on the replay. You can see there's been a lot of changes. So first I want to call out to Heist who's been doing some really great work, not just like adding features, like adding a lot of functions, but also doing a lot of doc work.
I also want to call out to Thomas who recently joined the DC working group and he's been opening a lot of good issues in the repo as well. So I think and I'm I'm not gonna like, I don't want to exclude anyone else, but those two have been kind of doing a lot of work, so very phenomenal. So I'm not going to go over all the details here, but I want to cover two things real quick.
First one is one of the things I added. I forget if I added this in the last preview or this preview, but because we've been adding more and more functions, so each preview has different set of functions in each version. Once it releases, we'll have different set of functions. You know a DC function list command, so this will show you all the functions in that build.
The the category, the function name, some argument information, and of course this is all Jason, so let's see if I do this right.
Then if you want to programmatically consume this so you can imagine in the future, you know maybe we'll have a MCP server that uses this information. Nothing happening yet other than an idea on that right now, but maybe maybe next can we call out something?
So that's one thing. The other thing I want to show is with this preview, you can now consume bicep files directly with the DSC executable. All right, now I want to do put out a warning. Bicep integration is still a work in progress.
We made some. We have some basically Hello World works, but I would not definitely use this anywhere close to production. All right, there are some known issues. I've actually caused Bicep to crash, but we'll continue down this path. But the idea here is like this is a Bicep file. Very simple.
You must have target scope desired set configuration. This is how Bicep knows this is going to emit a DSC file versus an ARM template. Here I'm just using one of our resource that we ship to make it easy to diagnose problems. This is the echo resource ARM.
Meaning bicep now requires a version string. So there's something we need to still reconcile with them because we use Sember versus date, but they require date for this to not throw an error. So here I'm just using a simple this is the friendly name example echo and I'm outputting hello world. So let's run that here.
I think I have it in my history and I'm going to run this with the debug trace level so we can kind of see some of the output. But basically you don't need to worry about using bicep first to transpile it into Jason, you just basically pass it directly to DSC.
DSC OK, if I go up here, see if I can find it. So it already found the extension that I wrote that calls out to the bicep command line. So I found that extensions for import are bound to file extensions.
So it knows that this dot bicep file needs to use this. So it's calling bicep with these arguments. And the nice thing is that the bicep command can just do a dash dash standard out so that the transpile of JSON out goes to stand out. It reads it back in and then DC just gonna enact upon it. All this other stuff is written to that so you see the output so.
I would love people to start playing with this with the cavity hunt that this is still a work in progress and there are known issues. But the eventual goal is that if you prefer, you can always write in JSON. I hate that you can write in YAML, which is a little bit better, or ideally you can write in.
Bicep, which is more human readable, writable. We get nice Intellisense and VS code and stuff like that. I mean, we do have some level Intellisense with YAML using the schema file, the Jason schema file, and then we'll continue to work towards having actual Intellisense for like properties and stuff that isn't there.
That's pretty much it. So we'll continue this journey on 3/2. I expect previews to come out roughly every three to four weeks or so when we have changes that need some validation. No specific target date for when we GA.
But this calendar year is the goal. That's it.

Jason Helmick   28:09
Great. Thanks, Steve. Andy, do you want to talk about some of the PS read line work that you've been doing?

Speaker 1   28:15
Sure. Also, honestly very excited about that bicep work. I'd like to get back to that. Got that prototype started and for now I've actually been working over in the PS read line space. Let me get my screen shared over here. So if any of you have ever tried to use PowerShell under a screen reader.
You'll know it has not really worked historically. Let me choose the right window. One second dev box.
I have to do this through a window A because I need a Windows machine and B because that computer can be muted while a screen reader's on. So you don't all hear a screen reader as I try to show this. So what you'll see right like today in PowerShell, if you've got a screen reader up such as I've got NVDA, which is a very common open source screen reader for Windows.
And you start PowerShell. It it goes ahead and tells you that, hey, we've detected there's a screen reader and we disabled PS Readline for compatibility purposes and you could reimport it and see what happens. And I'm going to just do that here real quick. It's not imported, so I can't even do something as simple as copy that, so I got to carefully type this out.
So if we do go ahead and run PS3 line while we've got like NVDA right here, this little window is actually showing us what NVDA would be reading and what you could hear if I didn't have this remote machine muted. So I'm going to type out like get child item. Do you see what all just happened there? I've only written get dash CHIL.
But we actually got, I mean so much text over here. We got the first couple letters I started typing git dash and then it actually for some reason went back and read out branch and then read out git child item. And some of this you can definitely ascribe to the inline predictions right here, but that's not the only thing going on.
You've got. It actually went back and read out the word child, even though I haven't written well, it went and reread the CHIL that I had typed and then again it repeated itself to read out with the prediction. But if I go back over here and like move my cursor, yeah, it's reading the characters as I'm moving over them, but I'm gonna go ahead and.
I think Control D should hopefully work in Windows mode. No, it doesn't. Sorry. I usually run this in Emacs edit mode. I'm gonna just go ahead and hit it backspace. I'm specifically showing that I'm editing text within a block of text that I've written, not at the end and delete that. It has to reread the whole thing. It can't just read that I deleted a character.
It's rereading the whole thing because it has to re render it. That's that's kind of this root issue that we've known about. If I open this up at least since 2021 and probably sooner, there's there's a redraw that PS refine does, and this makes a lot of sense if you go ahead and look at the implementation.
Inside this render function for PS read line, and if we actually just go ahead and minimize the new one that I'll show you in a second of render for a screen reader, the force render it just it renders line by line. If you've made an edit, it parses everything. It figures out it literally does like a syntax tree parsing. It figures out what.
The text highlighting there should be, and then by necessity to add colors and stuff, it goes ahead and blinks the whole buffer in the background and rewrites it all, which makes a lot of sense for what it's trying to do visually. It can put the control characters that control the colors for the tokens that you may have edited into the right places and do all of these things.
Things, but that doesn't really work when the output goes to the screen reader and it's just every single character, cuz now you've literally repeated what's previously been rendered. And if you're somebody who's not looking at it but listening to it, it's practically useless, which is why we've disabled it. You just get this bunch of garbage of.
Well, I don't know what I just typed. I don't even know what it's reading because it's almost to you as a user with like visual issues, just kind of randomly reading stuff. So I actually have gone ahead and fixed this. I'm very proud to say it's actually we've moved this needle forward like a whole.
It's far from perfect, but this is actually an ask over from the VS Code team. They have been working on improved accessibility inside VS Code for terminals. Specifically, they do some really cool stuff like if you have ever used your incremental history search in a regular buffer like in a regular terminal.
Terminal here, but let's let's go ahead and do that like hit control R.
In VS Code you now get this little palette up here, which since they have full control over the window, can be really nicely rendered out to that screen reader. Look as I'm I'm moving through these items. It's actually reading these pieces, but if you had this over in PowerShell.

Jason Helmick   32:56
Yes.

Speaker 1   32:59
Under screen reader or other shells. This is not unique to PowerShell and this is why they started doing this. If you're in Z shell, if you're in Bash, all these other shells and you hit control R I haven't loaded PS read line. Give me a second here. Import module PS read line and if I hit control R well we've got this like status line prompt back I search and.
And all of this actually will then show you the same issue we've got is not necessarily unique to us because now we're changing multiple lines as we're going through history and it's just not an accessible experience. So they've brought a bunch of accessibility into VS Code. The thing is.
The way they do this is through their shell integration script, and this is kind of why it came back down to my boat. If you remember, I mostly maintain the PowerShell extension for VS Code along with Patrick and Sydney. And one of the things that this VS Code team wanted to do a couple of years ago was, hey, we have the shell integration and we need to inject it into PowerShell. And I was like, well, you know what?
We do the same thing. We have to inject something into the PowerShell REPL loop and we do it inside of PS Read line. There's a hook inside of PS Read line that the PowerShell extension PowerShell Editor services uses to make itself work and that we taught the VS Code team to also use. Well, that kind of requires PS.
Read line to be available. And as you can tell, PowerShell itself is the one going, hey, you're on a screen reader, we're just not going to load PS Read line because we know how broken it is. So when it came down to brass tacks, it was, you know what? We can't really continue to do this. We need PS Readline to be somewhat accessible to.
Being a screen reader more so than it currently is because PS Readline itself completely nonaccessible. And honestly, even if you don't have PS Readline loaded, which is what PowerShell dumps you into, it still doesn't work great. It was not at all on par with other shells. You would also often get a bunch of repeated characters with that basic REPL implementation inside of PowerShell, mostly because.
We don't really maintain it. We're expecting you to use PS read line, so we go over to a pull request here. I've been working on this pretty much since I broke my arm. July 18th was when I first opened this PR, but that was after a month of looking at it.
We've got actually almost everything working. I mean, my list here is things that are not known to be supported and things that are only partially supported. And like a big one to call out is colors aren't going to work right. You have to rerender to move those tokens around to colorize stuff properly. Inline predictions just flat out not going to work.
Work right? If you have to read out the text that's in the buffer, which is kind of our limitation, well, if you have text pass where you are, it's gonna get read back out. Same with list view and menu completions. But things that do work is, well, all of the tests other than the tests around those features that I had to disable text selection works multiple.
Mostly works. There's caveats here, no continuation prompt. Visually wrapped lines are now working properly. We had to do a couple prototypes to get that right and I actually in kind of the last hour, not this literal last hour, but last week it felt like last hour I was like, you know what? I actually have a way to do the status prompt in here really cleanly in.
This implementation. So what is key digit argument and that incremental history search does work in this screen reader implementation. Those tests pass.
Let me go back to VS Code here. A little demo. So I've now run PowerShell. You can see it still didn't import PS read line. I've done this with my launch config that actually is forcibly loading my PS read line that I just built and enabled the new screen reader option and loaded up the shell integration script from VS Code. So all of these.
Things can get plugged in. Now I can actually do something like get. Oh, it's debugging. I'm sorry, pause, pause the debugger, turn off my breakpoints. That is why it's reading more stuff.
OK, all breakpoints disabled. But look at that. I deleted GET and I actually read it out as I deleted TEG because I hit backspace, backspace, backspace and it only read that stuff out and it's actually doing exactly what we needed to do. It's reading the text as I've written it. If I go back over it, that's fine. It's reading the characters that I'm mousing over or.
Cursing.
I can delete a character and instead of the whole buffer being re rendered, there's differential rendering and that's kind of the key point. We get the common prefix between what we previously rendered and what we're being asked to render now, so we can minimize the amount of text that changes if you know much about.
Uh, control sequences. I've deleted a line in here. Um, if you know much about control sequences, what we do is once we know where the text changed in that buffer, we literally set the cursor to that location and we emit A0J control sequence, which is hey clear from where the cursor is onward.
And then we write out a substring. We've substringed out from the common prefix that we found, which was math was the answer to all of this, the difference. So I can go back over here. Let me finish. Let me actually do something cool and hit tab and actually get a completion here. It actually did complete get child item. And you know what? That's a whole different buffer. So it did have to.
Read read get child item, but that's not being reread. We did change the buffer so it's working exactly as intended right there. Now I can cursor over these. I'm gonna delete the C and the trick is we're not gonna see GET dash read, we're just gonna see HILDITEM.
We deleted the C, it read that we deleted the C, and then it read what has changed in the buffer after that. Would I like that to be a little better? Yes, but we're pretty much bound to. We have to admit what's been changed, but we cannot admit anything more. And that's like the long and short of this implementation it it has been through.
It's been through a lot. If I move over to the commits here, we've been working on it over and over. There were a couple of different ways we could use to detect screen readers. The VS code uses Electron and I went through the Electron code base to figure out how they detect screen readers to be like, hey, can we do this right? And I tested their version and learned.
The hard way that it won't work for us 'cause we're not a windowed app, so but we kind of had to go back to the original.
Their comment in the Electron code base, since it's open source, you can read it, says this way we do it is known to be a little buggy, but we don't have another option. So we had to change that background. It's the system parameter screen reader check. Predictions obviously don't really work in here, not the inline predictions anyway. That's kind of like a futurist there.
An accessible way we could do that like VS Code does, cuz if I'm down here and hit control R since I'm in VS Code and I've actually been able to load the shell integration script, which means I get VS Code's screen reader optimizations. But they've been doing a lot with completions and buffers too. So what I'm really hopeful is that since we've moved this needle forward and we can.
Actually load PS read line for VS Code, let them inject the shell integration script and keep working with them on that, that they can go, hey, that inline prediction engine is running, the predictions are there. Let's go ahead and present these accessibly since they have a lot more control over that, since they kind of own. You can almost see this as a whole windowing system cuz they write VS code, it's its own self-contained window.
With panels that they can do. So I'm looking forward to future improvements there. But yeah, we've been through it and we got this working. And then kind of the question came up, what about tests? And that was a whole fun week and 1/2 or two in and of itself. But then once we had the test in place and actually were able to run.
Set this up to use a fixture. So we reran all of our existing tests. We were able to go back and improve this implementation even more. I've gone through a couple of reviews, have found a couple of bugs. So far I've been very proud and we're really looking forward to getting this merged in and having it show up in VS Code Insiders as soon as possible.
Well, honestly, it is the goal here. So yeah, that's the new screen reader support. It's it's been fun. I'm looking forward to more of this. You know what? It's relevant to my work. It's VS Code.
Hey, I'm gonna stop sharing that screen into my demo.

Jason Helmick   41:13
That's awesome.
Thanks, Andy. That's that's awesome. And and having been with you as you've worked on this, great work. Really appreciate it. With that, Mikey, how about a doc's update?

Mikey Lombardi   41:35
Sure. Yep, here we are. Sorry, my machine was taking a while to catch up. So I've been heads down doing a lot of background docs work for DSC and just today actually we published the standard out.

Jason Helmick   41:36
Mikey, OK.
Uh.

Mikey Lombardi   41:55
Schemas, the documentation for what DSC expects a resource to admit to standard out. Previously we described how you can put things in your resource manifest to define it, but we hadn't explained what what data does DSC actually expect me to send back, right?
We kind of referenced it, talked about it, but we didn't have reference documentation and schemes for that. We have that now, so that should make integration work quite a bit easier. Let me grab that link for you and I'm currently working through a resource design guide.
Which to be clear, will not tell you how to write code to do these things because I'm not going to write different versions of that article in C#, Go, Rust, C++, Python, PowerShell, etcetera. So what this design guide is going to do?
Is talk more about the things you need to think of when you're implementing a DSC resource and the specifics of what you need to do for things like emitting messages and practices around defining semantically useful exit codes.
When should you implement different DSC operations? Do you need to implement test at all? For most DSC resources now, the answer is no. You can use synthetic testing, but we're addressing those types of concerns and design questions in a cohesive guide to make.
The questions that you have around what am I even doing here, a whole lot easier to answer. And then you know best the context of your software component that you are writing resource for. So we don't really try to tackle that very deeply and we're not tackling how to write Rust.
Or C# or whatever, because that's better left to the eventual helper libraries around that and and sample guides for. Here's how you implement your first resource in Go or your first resource in C# or whatever.
But it's all just kind of moving. Uh, it it takes a while.

Sean Wheeler   44:08
Yeah, let me just interject here too. I want to thank Mikey for all his hard work on this. This he he owns the DSC space and I couldn't manage those docs without him.
Um. And there's been a lot of behind the scenes work there. Um.
Not a lot new to talk about in the PowerShell docs this this past month. It's just been in maintenance mode and I've been busy with the roll out of our new pipeline based on the new plan EPS and.
Preparing for tech mentor, so Mikey's been holding down the Fort and working hard on this DSC stuff.
And I think that's our docs update.

Mikey Lombardi   45:07
One last thing to say on the docs update, Heiss has already been praised a little bit earlier, but I want to go ahead and shout him out again. I took a couple weeks of vacation last month and while I was away I came back to find that the team had done a bunch of implementations.

Sean Wheeler   45:14
Yep.

Mikey Lombardi   45:25
And Heiss had come around behind them and drafted reference documentation for those new implementations, which is how I initially figured out what had happened while I was away, which is great. It's just been super helpful both in filing docs issues and actually drafting stuff and.
Providing fixes. We just merged a fix to documentation today from Heiss where in the course of my drafting I had forgotten to fill out a field and so I just left it as I sometimes do when I'm scaffolding docs with the body text blah.
Which I did not pay any attention to and accidentally shipped and somebody said hey.
Why does this just say blah? And so I went and wrote the documentation for that and I was able to review it and get that pulled in. So thank you for saving me from my own mistakes and for all those contributions.

Jason Helmick   46:21
That's awesome. Thanks to both of you. And and and and Sean, I thought maybe you didn't get the memo, but as somebody pointed out in chat, you're the thought leader. So there you go. We have a demo. I think we have a demo. Andre, are you here?
Oh, awesome. Awesome. Awesome is. Oh, I think I see. Let me make you a presenter and away you go.

Andrei Vernigora   46:39
Yep, I guess.
Yeah, I'm gonna try to.

Jason Helmick   46:50
There you go. Welcome.

Andrei Vernigora   46:53
Complete.
OK.
Oh, it doesn't want me to share my screen. Just give me a SEC.

Jason Helmick   47:18
Is it letting you share your screen?
Andre, are you still with us?

Andrei Vernigora   47:31
Yeah, sorry. Can you hear me?

Jason Helmick   47:34
Yes, I can hear you.

Andrei Vernigora   47:35
OK, um, so let me share my screen.
And let me know when you can see it.

Jason Helmick   47:43
Yeah, I can see it.

Andrei Vernigora   47:46
Great. So we're going to quickly talk about PowerShell being not only an automation tool, but also it can a data analysis and data visualization tool which supports supports data-driven decisions and.
There are three pieces to this picture actually. First is a PowerShell which can process data and kind of manipulate and.
Transform data and then two other pieces is.net Interactive and Jupyter Notebooks and.net Interactive offers Barsha kernel for Jupyter Notebooks so we can kind of use them to.
To visualize and run code, and on the other hand is something called Vega, which is basically a visualization grammar based on Jason and that allows us to create a whole bunch of nice interactions.
Active visuals in a relatively simple manner. So if we look at it, all of this combined works something like this, right? For example, if we already have some kind of a.
Um.
Jupiter spec, which basically looks something like this, is just a whole bunch of Jason. We can visualize it relatively simply, right? Just by piping something into a special command that comes with.
dot net interactive and I I put like a pull request into the repo to fix some of this, but now everything works so and it becomes interactive because of how it works.
But if we want to make it more exciting.
There is a way to work with graphs and some of the bicep stuff that I'm working on and for example this this data source is basically something.
Uh.
Out of the Internet and it has all the parent child relationships within it. So if we want to represent like graph based data, we can just pull the data from somewhere. In this case this is just the.
Simple demo data, but we can pull it, we can kind of process it, and in this case I'm just grouping those things, creating vertices, creating edges in the graph, edges in the graph, and then it basically.
Displays all this stuff relatively easily, right? We can change the change the size of it and so on and so forth and make it more.
Nice looking and the last but not least is as Steve mentioned bicep today and as part of my kind of daily work, I work a lot with with bicep.
So sometimes we have pretty big bicep templates which which become complex really quickly. So what I'm trying to do is I'm trying to.
Make a tool which can kind of unfold all of these bicep templates, find the connections inside of them and show all of these details in kind of more concise way, let's say. So if you give it.
AM.
You know a path to the root file. It can basically build a graph and then visualize a graph in this manner, which shows that all of these files.
Good.
That are sitting behind the main entry point and all the other elements like parameters and so on. But the most exciting piece is DSM, which stands for Design Structure Matrix.
Which kind of allows us to.
Break these into clusters and show connections between clusters. So it basically breaks your system into different pieces, tries to clusterize them based on certain algorithms and then.
Tries to minimize the interdependencies between this cluster, so it kind of shows how your system might be structured if you wanted to minimize interdependencies, but what I'm trying to.
Get to here is that PowerShell is not only a tool that gives you an ability to automate stuff, but it also gives you an ability to.
Collect the data, rocess the data and visualize the data using, you know, some of the other tools out there.
That's probably all I wanted to.
Say today. Thanks.

Jason Helmick   53:18
Thank you Andre that that is really fascinating demo and thank you so much for sharing that. If you if you can, if if you'd like, please drop in to the chat. I know there's some questions and comments for you in there. If you could drop in a link to the repo so that folks can.

Andrei Vernigora   53:31
Sure.

Jason Helmick   53:34
Can check out your project. This is really interesting. Thank you.

Andrei Vernigora   53:40
Sure.

Jason Helmick   53:41
Well, folks, we have once again reached the end of our monthly kind of show. Let's plan on seeing each other. What is it? September is what we're going to be rolling into. So, Oh yeah, so that'll be as we're getting closer to our release train, things like that.

