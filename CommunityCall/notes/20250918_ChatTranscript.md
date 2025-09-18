PowerShellOpenSSH Community Call-20250918_092924-Meeting Recording
September 18, 2025


Sydney Smith (She/Her)   0:43
Hello, everyone. Good morning or afternoon or evening. I'll give it about one more minute and then we'll go ahead and get things started.
Alrighty. Well, we have a full agenda today, so we'll go ahead and get started. So again, hello and welcome to the September 2025 PowerShell Community Call. We are so thankful for all of you being here. Just a reminder that this meeting is recorded and will be uploaded.
To our YouTube channel after the call. If you do not wish to be in the recording, we ask that you leave now and catch us with the recording. Also, as always, we like to remind folks of our code of conduct that we take very seriously and encourage everyone to lead with kindness.
As always, we use a GitHub discussion to share our agenda and take community demo requests, questions and other topics, which is linked here in the chat, and we will get things started with Aditya talking about an issue we had with the PS Gallery.
I believe last week.
Um, Aditya might not be here yet.
Just checking the condense.

Dieter Beckers   2:54
Do you mean me?

Sydney Smith (She/Her)   3:02
No, we'll come back to that topic. Um.
Anam, are you here to talk about the PS resource get preview release we just had?

Anam Navied   3:11
Yeah, I can definitely do that. Let me share my screen.

Sydney Smith (She/Her)   3:14
OK, great.

Anam Navied   3:19
Hi everyone, my name is Anam and I work on PS Resource GET and just last week we had a release of 120 Dash Preview 3 and this release included some new features like pagination support for MCR. We also had some bug fixes in this release.
Most notably, there was an issue with using PS resource get on a lockdown machine or a machine where you have WDAC enforced. So there if you try to run a command like find PS resource or install, you would see an issue relating to constrained language mode. So we fixed that. So now you're able to use PS resource get on a lockdown.
Mode machine. We also had a bug fix for using publish VS resource on a new machine or a new agent, so the module prefix parameter was causing some issues there. But now if you try to do a publish on a new machine, you're able to do that and a repository file is created.
Created for you and the module prefix parameter is checked correctly to see if everything can go forward. We also had some bug fixes for script file infos and getting complete metadata when using a Container registry repository.
And then lastly, we also had a bug fix for retrieving unlisted package versions. So this is something that was supported in PowerShell get where if you have a package version that's unlisted, you could still find and install it. But that was something we had changed with PS resource get and then we received a lot of feedback from the community where.
People do want to retrieve an unlisted package version, perhaps for internal testing or validation before making it public. So now with find PS resource and install, you can retrieve an unlisted package version. Yeah, but thank you all for your feedback. It really kind of helps us drive.
Which issues we focus on, where we need to add support, and we'd appreciate feedback on this release. So thank you.

Sydney Smith (She/Her)   5:18
Awesome. Yeah, thank you, Anam, and thank you to all the community members who have contributed to this project, whether through PRS or issues. Now is the time to check out the 1-2 preview because we are hoping to get it in for PowerShell 76. It has a couple of.
Other key features that we think will be really valuable to folks that we want to get in. So we took in a community PR that added Nugat V3 dependency support in an earlier preview as well as I think believe in the last preview we added support for the.
Azure DevOps credential provider. So couple of key features to check out before we hopefully get that into the 7.6 release. And with that I think I saw Aditya join. Do you want to give a quick rundown on the PS Gallery issue? Thank you.

Aditya Patwardhan   6:10
Yes. So as some of you might already know, we had outage for gallery on September 8th about when you tried to install the package provider, it was pointing you to.
CDN and the CDN had an expired certificate, so the root cause of the issue was we had moved to a different CDN domain, but some regions were not purged appropriately.
And for some reasons it was redirecting to the old CDN. So we work with third party provider CDN provider and we were able to purge it in a couple of hours and from that point everything is fine. We also updated the documentation for.
Mentioning that the older CDN endpoint is no longer valid and should not be directly used. So I'm just dropping a link to that documentation over here in the chat, but so everything has been purged and.
We have validated everything is looking fine, so we had already moved a new CDN, but some regions were affected. So we have completely purged all of our caches for the CDN to mitigate that.

Sydney Smith (She/Her)   7:30
Awesome. Thank you for giving us the rundown there and thank you to all the community members as well that reached out and let us know when the issue hit. We we appreciate that as well. Always sucks to have outages, but always a learning experience as well. So thank you for that.
Steven, are you here? Do you want to talk a little bit about Mini Con?

Steven Bucher   7:53
Yeah, yeah. So if you are not familiar with Minicon, it's a it's a virtual conference hosted by the same folks who host the PowerShell Conference EU site. I just we just wanted to share the CFP is still open if people have.
Sessions they'd like to share there. We'll be there at, well, we'll be virtually there with a couple of sessions. I believe Steve has a session on DSC V3 and MCP. Dongbo and I also have a session on AI shell and MCP.
And then, uh, Autumn, I believe you have a session on partial security and stuff, but they're still being announced and they're still being, uh, there's not a formal list of all the sessions out just yet, but I would keep an eye on their LinkedIn page, which is it looks like where they're kind of, uh, posting things, but wanted to share out that.
Yeah, definitely take a look at joining us for the Mini Con, which is October 14th for if you're interested in hearing about any of those sessions. So yeah, I think that's all I got.

Sydney Smith (She/Her)   9:03
Awesome. Thank you for that. Always a fun conference in the little. I don't know if it's still in the virtual world where you walk around and see people, but that's always kind of silly and fun.

Steven Bucher   9:11
Yeah.
Yeah, it's in Gather, the little VR world or whatever.

Sydney Smith (She/Her)   9:15
Gather.
Yeah, good opportunity to connect with folks. Cool. Well, it looks like the next topic is for me. I did want to give a little update on PowerShell 7.6. As you may have noticed, we have not shipped at PowerShell's PowerShell 7.6 preview yet this month we had a few issues.
Pulling in some of our package dependencies, so we've delayed that release a couple of weeks out. So what we're looking at right now is in the first half of October, early October getting out which what will likely be our last preview release of the 7-6 cycle.
From there we'll kind of judge how things go and move into an RC state and then on to GA. So just a reminder there that PowerShell 7.6 is coming up to the RC runway, but look out for a package release in early.
October.
Let's see. Um, DSC updates from Steve.

Steve Lee (POWERSHELL)   10:26
Oh, I wasn't prepared for this. Sorry. No, no, I I guess I'll just mention. So we did release 3.2 preview 5 on Monday, I believe Monday night. So that is out. I think the main let me just take a look at the change log real quick.

Sydney Smith (She/Her)   10:29
Oh, sorry.

Steve Lee (POWERSHELL)   10:41
To refresh my, I was actually off last week. Where is it? This one. I think there's been a large number of changes. Let me just cover this. But again, a bunch of new functions. Heist from the community has been doing a good job adding those.
Let's see. I think some of the main things is the initial DCMCP support is in that release. More tools will be added later and again I'll talk more about that. If you don't know what MCP is, basically it's model context protocol. It's a way for our large language models to interact with the real world. So in this case the ability to have.
But discover DC resources and in the future and act upon them. Let me see what else. I think that was a lot of the focus for me was on the MCP side, so that I have something to talk about at Mini Con, but there's been some bug fixes.
Yeah, I mean, there's a lot of functions and stuff. I guess nothing else major. I can. I'm looking there. There's a new context function, so if you want to know during execution of configuration if you're running as a elevator or not, you can do that. You can also detect whether you're running on Windows, Linux or Mac OS, so.
There there are certain things that are being brought up by the committee and I want to give a call out to like Thomas Nando who is using this and giving us good feedback. So I'm prioritizing some feature work for people who are actually trying to put it into production. I think that's the main things.
I have for now on DSE.

Sydney Smith (She/Her)   12:10
Sounds good. Always fun to hear the new updates going on there. Sean, I have you next for any docs updates you might have this month.

Sean Wheeler   12:22
Sure. So there were some minor updates. As you may know, we dropped two minor update releases, PowerShell 7.5 dot 3 and PowerShell 7.4 dot 12.
Um, the 74 update was minor. Uh.
Maintenance work, no functionality changes there. The 753. The one big thing I want to call out that changed in here is the out grid view problem has been fixed. So now and you can run out grid view and.
Filtering works and the the searching and sorting all works again now, so that's finally been fixed and that'll get into the next this 7/6.
Hopefully in the next release. Um.
There was also an update to PS read line. Um.
We shipped 2.44 beta 4. The big update here. Andy talked about this last month. We now PS Readline now supports screen readers.
And so that that version's out and that's the version that's shipping with the, I believe shipping with the latest.
V code extension so.
Uh.
Good work there and then there was an update to AI shell up to preview 7. This again was mostly bug fixes. The big addition here was GPT5 was enabled.
And updated some documentation around MCP support. So there's a whole article article here about how to use MC PS in a I shell and just general updates across all of the documentation.
For AI shell and that's what I got.

Sydney Smith (She/Her)   14:46
Awesome. Cool stuff. Thank you. I think that is the end of our agenda from our team, but we're very lucky to have a couple of folks that signed up for community demos.
Today, let me just see.
Um, Dieter?

Dieter Beckers   15:15
And that is my name.

Sydney Smith (She/Her)   15:18
OK, cool. Are you ready to go? Let me make sure you have presentation rights.

Dieter Beckers   15:21
Yes.
Cool.

Sydney Smith (She/Her)   15:26
Um, it'll probably just take me a second to find. I found you.
OK, the floor is yours.

Dieter Beckers   15:38
Thank you.
I'm really excited because this is my my first time presenting at the PowerShell community call, so wish me luck. So I created a tool called Pixel Posh.
One of the first versions was based on system dot drawing, but since I wanted to make it.
Cross system compatible. Yeah, I'm now using PS SVG, brought to you by James, who is also going to present today later after me I think. So I'll do a quick demo what it does.
So it it will generate a random backgrounds.
Like this is it can generate an SVG and then convert it to a PNG. This is one of them.
Um, yeah, I'll do a do another one.
Oh, this is this is another one. It also fetches a random.
Palette from from an API. Let me display all of the options because I have like 8 different backgrounds. It's all parameterized.
So let me just paste in the snippets. May take some time to.
To generate some of them 'cause it involves some some math.
Like the low poly ones are especially a bit heavy.
Give it a SEC.
So it uses an external tool to rasterize it to to PNG. There's support for three different tools. I'll put put everything in the chat later on, but so this is a bubble background.
Uh, the circle one.
Wave and low oly.
Another type of low poly, another type of wave.
Squares, stripes and again this is all parameterized. So next time you would generate it would have a different angle or different size or more waves, less waves and those kind of things.
So this this was for me intended to or the use case for me. What originated this tool from basically was that I was having a lot of VMS open and I I needed a way to differentiate between the VMS really easy. So yeah.
I think I have a question.
Maybe.

James Brundage   19:21
There was somebody in chat who'd asked if it worked offline.

Dieter Beckers   19:26
Yes, it does work offline. I've included I think 100 built in pallets, so.
OK, accidental entries.

Ryan.Wakefield   19:40
Well, I just want to, I just want to tell you there Dieter, that this will actually be very helpful for me. I am really doing a lot of pro presenter work for my church and so it's always been difficult to get like just fun little gradient backgrounds and stuff like that and.

Dieter Beckers   19:46
Cool.

Ryan.Wakefield   19:59
You know, telling AI to do that, but this this will be a lot of fun to kind of really leverage this and do a lot of that stuff. So I'll I'll probably be heavily using this and I'll keep you in the loop.

Dieter Beckers   20:11
Cool. Um, let me see here.
No. So so it does export to SVG as you can see here. Let me zoom out and this is all like rasterized. Sorry, it's all generated by PS SVG.
Which is created by James.
And then it rasterizes it using an external tool to PNG basically.
Uh, let's see. No other questions, I think.

Sydney Smith (She/Her)   20:54
I think you got them all.

Dieter Beckers   20:55
Yep.

Sydney Smith (She/Her)   20:57
Well, very cool. I was just going to say very cool demo. Thank you so much for presenting today and sharing. Yeah, of course. James, did you have a demo as well for today?

Dieter Beckers   20:57
Thanks all.
Thanks.

James Brundage   21:10
I did, and it's almost something of a theme. Are you gonna be able to give me some sharing rights?

Sydney Smith (She/Her)   21:16
Absolutely. Let me just find you on the list.

James Brundage   21:21
OK, while that's going on, I'm going to kind of set the stage here. I've definitely been enjoying playing with graphics and PowerShell for some time. I also, when I was a very little kid, I learned a little bit about how to program from this thing that some old timers might remember called.
Turtle Graphics. Has anybody on this call heard of Turtle Graphics?
Aside from Peter, cool. We got a few. We got all right, we got more than a few. I feel better, feel way better about this. So basically I discovered the turtle graphics are really easy to implement because their core you're basically trying to imagine your.
Turtle holding a pen. You can rotate, you can move forward and you can lift the pen. These are your three basic operations. Everything descends on down from that. John, do you have a question?
OK, going on to the very fun demos. These are these three basic rules. I built a turtle in PowerShell. Now you can actually create all sorts of geometry at the command line, starting very basic. Let's draw a triangle.
OK, you know, rotate 120 forward distance, rotate 120 forward distance, rotate 120 forward distance. Well, let's make it easier to go ahead and draw polygons.
Or let's go ahead and draw a square. Let's tilt it on its side and watch it become a rhombus. Let's draw circles, half circles. We're actually nowhere near the cool yet, but you can start to see how interesting this gets because I can take a circle.
Half circle actually rotate it four times or eight times and get this nice overlapping shape.
I can use this kind of PowerShell list multiplication syntax to create shapes.
I can overlap polygons. I can use for each loops to start creating newer shapes. This gets cooler and cooler and cooler. This one I keep coming back to. You can reflect a shape by drawing it with negative numbers. So forwards is literally a reflection backwards.
And I'm actually just going to keep scrolling down here, because if I spend the time to talk about all the demos, we will be here all day. But you can morph shapes. You can very easily tile them. You can rotate or repeat them. These are often called flowers.
So I have the feeling that this sort of sacred geometry might also help with aesthetic presentations in PowerShell. However, there are going to be a lot more practical things coming. Pie graphs are coming soon.
But flowers, they always look very nice when they morph. We can also draw arcs and flower petals, make a flower out of petal, morph a flower a million different ways, create stars of N number of points, create star flowers by rotating and repeating that.
Again, this just gets cooler and cooler. And look at the, you know, sidebar there. We're we're only about halfway through this rapid-fire demo. You can morph star flowers too. You can do rotations before you morph, and this kind of influences how it will be displayed. You can do the math right and morph different.

Gregg Lott   24:36
Yeah.

James Brundage   24:50
Types of stars flowers into other types of star flowers. It creates scissors which basically create little star like shapes. We can try to complete a Polygon of scissors between the angles of 60 and 42 and just see how crazy the math gets.
So this is even division. Not so even, not so even, not so even, not so even, not so even, not so even, not so even, not so even, not so even. Even again, great spirals. We can morph those spirals. We can rotate and repeat those spirals.
I'm actually very much underselling this because this is kind of like peeling back layers of the universe as you experiment and play around and see what you can draw. There are quite literally endless possibilities, but practical there. There's also bar graphs.
Which you can obviously make vertical by rotating them. You can also just graph written data. You can make nice reflected graphs like that. Oh yeah, and you can draw these classic fractals.
At this point, most of the fractals that I already can find representable in an L series or system I am drawing, but let's start with box fractal, board fractal, crystal fractal, ring fractal, pentaplexity.
This one I made called triplexity, **** Island, **** Curve, **** Snowflake, Levy curve or the C curve, the Hilbert curve and the Moore curve. These are used to layout processors and 3D infills. In fact, note the spelling of Moore. I'm pretty sure it's the same one.
We can actually do really cool things, however, by rotating and repeating more curves. We can do binary trees, we can mimic plant growth, and we have these fractal classics like the Sarampinski triangle.
And we can also do stepwise animations if we want to see how they're drawn. This will work to fairly complicated images, more complicated after I add some new features that will adjust browning precisions.
Do a couple of reflected Sarapinski triangles and rotate and repeat those a bunch of times and end of show. So yeah.
It's the morning. I'm no longer not the best presenter in the mornings, but this is really, really fun, cool stuff and I absolutely have been having a blast opening up the door to easy graphics in PowerShell and just so that we're clear.
This is the PowerShell that will produce that image. In fact, I'm gonna prove it to you. Hopefully not too embarrassingly.

Gregg Lott   27:38
OK.

James Brundage   27:41
Let's see. We're gonna go over here.

Dieter Beckers   27:42
There was also an important question. Uh, you cannot draw turtles.

James Brundage   27:48
I will get back to that in a second. You know, good question to ask.
Let's get my demo over from here. Copy that and.
Paste it into there. Now by default I am in pre or powerful terms getting an object. Doesn't look quite as sexy as it did on screen. There are a couple ways I can get this to improve. One I can throw it in quotes which will just output the SVG which is what the web page is doing.
And I can also go ahead and save any of these O I'm going to go ahead and type that save turtle.
Yeah.
P dot SVG or call S dot SVG.
Cool.
And go open up that. Actually I'll just invoke item.
There we go.
And if I wanted to go ahead and give it some more color, let's go ahead and say stroke 4488 FF background color is 000000.
Cool.
A little bit more power chili.
And just because I don't want to be showing U in the demo here, yes, I can save it as a PNG too.
Or a JPG.
Or Web V.
No.
So yeah.
All sorts of fun things that you can do with turtles. Now the important question, can I draw a turtle? Yes, what I cannot do at present. Let me rotate this one.
Going to get there. There's our turtle.
Now some bathrooms might notice this.
This is an aperiodic monotile, and once I get a little bit more clever, I will be able to take these aperiodic monotiles and tile the plane aperiodic periodically.
But I'm sorry, my math is not yet that good.
So yes, you can draw a turtle with turtle, and eventually you will actually be able to draw an infinite number of turtles across the space. I wanted to highlight one more relatively new feature before I go. First of all, you can get to the project at PowerShell Web.
Slash turtle.
Github.com PowerShell web Turtle The website is psturtle.com.
One of the things that recently came out was the turtles all the way down build and what this allows you to do is have a turtle which can contain turtles, which can contain turtles, which can contain turtles and add on and so on and so on.
What we can do with this is we can actually start to model behaviour. I'm going to stop for a second and go up 1000 feet. Turtle has been around since the 60s. It's the foundation of the first generations of computer graphics and behavioural modelling.
Because you can basically view the whole world as a series of turtles, and you can basically easily model the behavior of a pair of turtles or a sedentary.

Bruno   31:38
Oh, instead of just passing it at, I think that just the whole idea that that is like very widely just basic function. It should be no interaction from this sure inside functions.

James Brundage   31:41
Working together.
Hi Bruno, are you trying to talk about turtles?
Cool. Thank you. So we're going to go with this fun example here where I'm going to follow that turtle. I'm going to have a set of turtles each in a corner.
Following the next turtle. OK, so you got this turtle defining a square, then turtle in one corner, then turtle in another corner, another corner, another corner, and we're going to say each turtle.
If you're not more than one away, rotate towards the next turtle and move forward. And I'm just gonna quick pop quiz the room. Does anybody have a guess on what would happen if I have 4 little shapes in the corner each trying to go towards the next?
Shout - IT out. You'd be surprised how many people actually get this right on the first try.
OK.
Well, as each turtle curves towards the next one.
I get.
This beautiful spiral.
So this turtle goes towards this turtle, goes towards that turtle, goes towards that turtle.
So not only is - IT turtles all the way down, and we can have infinite turtles within infinite turtles, but we can actually get these amazingly beautiful shapes by simulating basic behavior.
And this makes all sorts of math, geometry, behavioral simulation, particle simulations all easier to visualize and do in PowerShell. So on that note, I keep doing what I've been doing for some time now and opening up brand new turf to us all.
And again, lest I be too shown up, I haven't really highlighted this that much, but you can just go ahead and save any of these as a pattern to get back an endless background of them.
So there is a snowflake pattern.
We can also animate these for endless variety.
And everything works inside of a webpage.
Basically, as perfectly as you'd like, so.
Any thoughts or questions about turtles before I take up too much more meeting time?
On a scale of one to 10, how cool are we?

Dieter Beckers   34:26
Over 9000.

James Brundage   34:30
OK. Thank you.

Ryan.Wakefield   34:30
Sad part is, I was thinking that too.

Michael Greene (POWERSHELL)   34:33
Hey, James, I've got.

James Brundage   34:35
Alright, uh.

Michael Greene (POWERSHELL)   34:36
James, I've got a question. I know you've done a ton of work. Can you hear me? I know you've done a ton of work in the area of like containers and that kind of stuff. This kind of stuff could be really interesting as as people are looking at huge, huge scale of.

James Brundage   34:40
Yeah.
Mhm.

Michael Greene (POWERSHELL)   34:52
Where are my IT assets? And if they move, you know, between data centers, between cloud environments, anything like that, that can be really, really difficult to understand.

James Brundage   35:00
Oh, that is a good idea.
That is a really good idea. Let me let me actually connect dots with you here for Michael. Yeah.

Michael Greene (POWERSHELL)   35:05
I mean, looking at, looking at the logs is one thing, right? Cool.

James Brundage   35:12
OK, let me connect for everybody else and then go back to the idea.

Michael Greene (POWERSHELL)   35:13
Maybe - IT could, yeah, maybe - IT could look at log files and help visualize what happened in the log file. You know, all kinds of cool stuff.

James Brundage   35:17
Yeah, so.
Exactly. So the big power of PowerShell is that we've got this beautiful object pipeline, right? And we can render that object pipeline any number of ways that we see fit. What this does is it gives us an easy way to connect the object pipeline into actual graphical rendering without getting down to the.
I need this pixel to be this way sort of level, but instead thinking at a very high level, I want to draw this sort of shape. I want to draw that sort of shape, go draw this or that and that color. There is just endless data visualization there. So yeah, I think container topology would be an interesting one. Container layout would be another one. I keep wanting to build something to.
AST. These are all possible and again, they all just drop into a web page if you do string them. So that is a really good idea Michael. Like make sure I have it right. You would like to be able to see.
Let's say all the containers I have in Azure laid out in a graphical form.
Sorry for the leading form of that question.

Michael Greene (POWERSHELL)   36:28
Yeah. And I think it's kind of like endless opportunity. No, it's OK. I I would say like we could even broaden the perspective to what are all the different times whenever it's difficult to understand what we're looking at, you know, as as we're managing IT.

James Brundage   36:43
Uh uh.

Michael Greene (POWERSHELL)   36:45
And these explorations can really help because whether it's like I open a log file and I feel overwhelmed, I can't read all of this and what do I just search for the word error, right? Understanding you know anything from I I look at.

James Brundage   36:57
Yeah.

Michael Greene (POWERSHELL)   37:01
Within my data center, what you know, I'm looking at inventory. How can I possibly get my head around this? I don't like a lot of people don't naturally just look at a bunch of data and go, uh, I get it right. So something like this.

James Brundage   37:14
Uh.

Michael Greene (POWERSHELL)   37:17
Would be a very interesting way to render the information in a way that is consumed a little bit more in in a delightful way so.

James Brundage   37:18
Yeah.
Yeah, you're right. And I would love to take some of this offline and brainstorm and come up with fun examples. I would also love all of you to try Turtle out and see how fun it is. It should be possible to render anything in Turtle. The question ends up being how difficult.
It will be, but we can do a lot of animations and a lot of interactivity right out of the gate. One other thing that your point raises and makes me kind of ponder is how quickly I want to enable like each individual turtle to have a reference link.
Because if I was doing that whole visualizing all the containers in my cloud sort of thing, well, it'd be one thing to be able to see them all, maybe even with some color indicating some of the metrics about them, but it would be another nice thing if I could literally just click any one of them and go right over to the.
No.
And that should be possible.
Also, I should probably sink some time into figuring out better web navigation on top of this period. Brian Lindstrom, you had your hand raised a bit ago. Did you have a question?

Lindstrom, Brian C.   38:43
I believe Michael kind of answered it or your conversation with him. I was just wondering for some practical applications of it and then his talk about viewing the different containers and what not. One aspect I think would be interesting is if you use it with Intune and.
Seeing from where the groups are applied to how they are linked to the different, you know, apps or different configurations there might be interesting and kind of like a flow tree type style, but I was just curious if you had any other, you know, practical applications or maybe like real world examples that this could be used with.

James Brundage   39:19
Practical applications, whatever we find fit to visualize real world examples. I mean, I got to tell this story because you kind of teed me up there. But again, Turtle is very old.
And it was used at first, its first real-world example. In fact, before you could even render Turtle to the screen, Turtle inside of the language logo was used to get us to the moon. Like that's how we calculated orbital trajectories in 1966.
And 6768, so on, so forth. You know what I mean. So.
Being able to very easily access this mathematical world has a lot of real-world applications, even if you can't visualize it easily. If you can visualize it easily, it starts to make all sorts of other things clear.
Um.
Including, honestly, how a lot of math itself works. It's the whole point of building out the sort of turtle was to kind of reignite that childlike curiosity within myself and others as far as.
Interacting with computers and graphics. Again, little five-year-old me first encountered a turtle on Apple Two, and before I even knew what programming was, I was doing bits of it.
So I'm hoping to kind of bring that back for everybody, and I'm hoping to kind of open up brand new areas of PowerShell utility throughout the world. You should also be able to write a formatter in PowerShell that uses Turtle. That should be no problem.
You obviously can write your own script that uses Turtle. I'm using Turtle to generate various web pages. You might already kind of guess that the background for the Turtle site is in fact Turtle.
One extraordinarily cool practical use, at least for itself. This whole page. These are all actually just directly run examples, and so instead of trying to.
Come up with.
The most perfectly awesome demo page only. I'm really basically trying to say how can I have the most complete documentation and because it's rendering something graphical right below it.
Not only is it complete, it's understandable. It gives me a great virtuous cycle of practical use of Turtle to improve Turtle.
So anytime you need to visualize anything, anytime you need to calculate anything that would be a little bit harder to imagine, you know, without the aid of a turtle, this is great. I'm going to go back to the two properties that we show.
So let's do a, you know, few other interactive examples here. I'm going to say turtle forward 42.
We have our heading. I have a changed heading. I have X and my Y coordinate, so I know how far I've moved. I'm going to rotate 90. Cool. Now I know my heading. Now I'm going to say something like Polygon.
42 Let's do 8 now let's do 12.
Good old dodecahedron.
Can you tell that I have too many random files that I've just been creating this way? Now Polygon here I have.
That, but it is still an object. I can see my heading, I can see my position. It doesn't really look like I'm at 0, but that's just because I am an infinitesimal fraction away from zero. But what I can also do is I can say something like select object expand property points.
So I can start to ask these mathematical questions about this. OK, like if I did a Polygon with 12 sides and 42 each side, what is your width and height?
So one other practical use of it, silly as it might be, is your command line graphical calculator.
Where you can actually poke at the data because we love objects in PowerShell and this is still an object.
And if I do something very complicated or fairly complicated, let's do a Sierpinski triangle up to 42 and 5.
And save that.
I'm gonna keep reusing S there.
Wrong. That one's on me.
Yeah.
So we can see that at the end of all of those.
Steps of this here in Penske Triangle. I'm really not that far from my original heading.
But I have drawn 1458 points.
And let's be a little silly for a second.
All of them well within the 42 that I originally said.
So yeah, plenty of practical fun, plenty of mathematical fun, plenty of ways to visualize. Also, I will point out, even though both Turtle and PS SVG are both built on SVGS.
PS SVG is sort of built for completeness of the SVG standard and Turtle is built for ease of use.
So Turtle was meant to be very easy to walk up to explore and work with. Maybe knocking on wood even easier than a general PowerShell. Maybe not.
If you have people in your life that are interested in computing but don't really know about PowerShell, that's a great kind of gateway drug to sort of show them. If you have people in your life that could benefit from understanding more of computing and mathematics.
But don't really kind of know how to learn. This is a great way to start and fabled Holy Grail of PowerShell people, as you know it is if you wanted to impress a family member or a random person on the street with PowerShell.

   46:09
Fable.

James Brundage   46:22
Finally figured out how I promise this is really cool.
And I believe I have talked far too much. Normally at this point in the the powerful community call, somebody's coming out with a crook. So I feel like this has to be extra cool because nobody's trying to actually shut me up yet.

Sydney Smith (She/Her)   46:42
James, it's it's been a great demo. You have a lot of fans in the chat as well who have enjoyed all the turtle talk today. But yeah, we can probably wrap things up unless anybody had any outstanding questions or topics that we wanted to talk about. But thanks for presenting today.

James Brundage   46:59
Oh, no worries. And you know, have fun with the turtles in the PowerShell. Turtle power.

Sydney Smith (She/Her)   47:05
Absolutely.

James Brundage   47:07
No.
I also did this at least at least 5% for the jokes.

Sydney Smith (She/Her)   47:14
Um.

James Brundage   47:15
So.

Sydney Smith (She/Her)   47:17
Good stuff. I'm not seeing any other questions, so we'll probably wrap things up there, but looking forward to to seeing folks at the mini-con in a few weeks and then.
And we'll catch you next month as well. So thanks everyone.

James Brundage   47:39
Thanks all.

Jason Helmick stopped transcription
