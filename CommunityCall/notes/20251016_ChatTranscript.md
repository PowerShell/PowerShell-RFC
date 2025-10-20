PowerShellOpenSSH Community Call-20251016-Meeting Recording
October 16, 2025

Michael Greene (POWERSHELL) 0 minutes 5 seconds

OK, our recording has started. Hello and welcome to the PowerShell community call. Everyone is welcome. We're getting started now at 9:30 AM PST. Just FYI, this meeting is recorded. The recordings will be upload up uploaded about a week after the call, sometimes a little earlier.

Michael Greene (POWERSHELL) 0 minutes 21 seconds

You can find past community calls on the YouTube channel and the link is in the chat. It's the PowerShell and DSC community community channel or team channel. Please remember our code of contact code of conduct. We do take that very seriously and there's a link to the code of conduct in the chat as well.

Michael Greene (POWERSHELL) 0 minutes 39 seconds

Also, it would be great to review our contribution guide which is in the PowerShell repo and you'll find that in the GitHub link there in the chat under PowerShell slash PowerShell with contributing dot MD. And then finally while we did not have an entry in September or October in the.

Michael Greene (POWERSHELL) 0 minutes 58 seconds

Discussions uh in the PowerShell repo. That is typically where we uh discuss what will be on the agenda leading up to the meeting. I didn't see an entry, but I went ahead and and put the link in there for future reference. Um, just so if anybody wanted to put something in for next time.

Michael Greene (POWERSHELL) 1 minute 13 seconds

Um, but that'll be the place to to keep an eye on things. So with that, uh, we have an agenda to get through. I'm gonna actually give the floor to Steven first to talk about some Ignite sessions coming up.


Steven Bucher 1 minute 27 seconds

Yeah, thanks, Michael. I'll make this pretty quick, but wanted to share. Ignite is coming up at the end of November. I believe it's November 18 through the 21st, the week before the US Thanksgiving holiday, and we will have some PowerShell representation there, myself and.

Steven Bucher 1 minute 47 seconds

Jamie and Caro will be at Ignite this year. We'll be sharing about some of the AI enhancements that we're doing to the Azure client tools to help you write and work with Azure CLI and Azure PowerShell. We have a session there. Let me find the exact breakout link, but we will be.

Steven Bucher 2 minutes 7 seconds

We will be boothing and around Ignite, so just wanted to call out. If you did want to chat with us about PowerShell while there, do find us. We'll be at the monitoring and management booth of Ignite as well. So here is the session if in case anyone is curious.

Steven Bucher 2 minutes 26 seconds

I do not know yet if it will be live streamed or not. It may be the recording may be found after the session. So yeah, keep a lookout for that. But yeah, that's all I had on as far as Ignite.

Steven Bucher 2 minutes 42 seconds

Ignite stuff.

Dongbo Wang 15 minutes 17 seconds

All right, just to make sure it's gone. Yeah, look at the path again.

Dongbo Wang 15 minutes 26 seconds

It's gone. So now let me start 76 again. It's not available, so when get install.

Dongbo Wang 15 minutes 43 seconds

And.

Dongbo Wang 15 minutes 46 seconds

It's able to run and get directly after installation in 7.6. OK, so that's the three changes I want to highlight for the 7.6 preview 5. Thank you.


Michael Greene (POWERSHELL) 15 minutes 57 seconds

Cool. And while we're on the topic of PowerShell, we did want to communicate as sometimes happens, we're expecting a delay in shipping the RC. We're thinking weeks, not months, but we wanted to, you know, just let people know to expect that and then forecasting it out if the RC is slightly behind schedule.

Michael Greene (POWERSHELL) 16 minutes 17 seconds

Schedule or behind schedule, that probably will mean that the GA date is later than we expected. So we'll try to communicate a firm date, you know, once we have that in mind, but just wanted to give people a heads up and I think next on my agenda list was Tess to talk about Open SSH.


Tess Gauthier 16 minutes 34 seconds

Thanks Michael. Hi folks. I just wanted to say that we are also working on getting out the next release for Win 32 Open SSH. That will be the 10.0 release. Some of the changes that are coming I just want to highlight. The 1st is post quantum hybrid algorithm support.

Tess Gauthier 16 minutes 54 seconds

For key exchange. So Upstream has had support for one of these algorithms for a little while, I think since 8.9 and in 9.9 added support for a second algorithm. This will be our first release supporting both of them and they will be preferred by default, so there won't be.

Tess Gauthier 17 minutes 14 seconds

Any user input required or configuration to start using these for the KEX Exchange and this release also continues the process separation of the SSH server by functionality, so there will be a new binary to isolate authentication called SHD Auth.

Tess Gauthier 17 minutes 32 seconds

So keep an eye out for the preview release coming soon and please continue to provide any feedback. Thanks.


Michael Greene (POWERSHELL) 17 minutes 39 seconds

Thanks, Tess. Appreciate it. And next, Sean Wheeler, everybody's favorite topic. Sean always does a wonderful job with our docs update. Sean, you have the floor.

Sean Wheeler 17 minutes 50 seconds

All right, let me share my screen. Gonna drop some links. So the the big update we've already talked about what's new in PowerShell 7.6.

Sean Wheeler 18 minutes 5 seconds

With the new preview, I will add to Doug Bo's comment about the out grid view that has already been back ported in 7.5. It was in the 7.53 release, so it actually.

Sean Wheeler 18 minutes 21 seconds

It made it into that release before it made it into 7.6 preview, so that's already fixed. 7.4 didn't have the problem because it's on a different version of.net.

Sean Wheeler 18 minutes 35 seconds

Um.

Sean Wheeler 18 minutes 37 seconds

A couple things to to note. There's some updates to versions of modules that ship in 7.6. Most notably this version of PS read line has the fix in it that.

Sean Wheeler 18 minutes 52 seconds

Improves the user experience with screen readers, so that's all.

Sean Wheeler 19 minutes 1 second

That's been included now and.

Sean Wheeler 19 minutes 7 seconds

Take a look at all the myriad of changes that have gotten in so far, so many contributions from the public. The other big update that I want to point out is I finally got around to updating the documentation for how to use the new version of Plat EPS.

Sean Wheeler 19 minutes 26 seconds

The commandlet reference documentation has been here for a while, but now I have articles that walk you through the process of creating or updating your content.

Sean Wheeler 19 minutes 41 seconds

Several new articles here, so a brief overview of the types of help that PowerShell supports, how to create new markdown content for your module, how to update existing.

Sean Wheeler 19 minutes 58 seconds

This edit article I want to point out explains the structure of the markdown. That's very important to plan EPS. You you can't change the structure the.

Sean Wheeler 20 minutes 14 seconds

It has to follow this pattern, but within each of these sections you have a lot more freedom of format than you had before. And then I've explained each of these sections and things to watch out for and provided examples.

Sean Wheeler 20 minutes 33 seconds

Of what's supposed to go in, there's a lot more diagnostic information with the new version of Ponty PS. There's this diagnostics.

Sean Wheeler 20 minutes 47 seconds

Property so you can now see.

Sean Wheeler 20 minutes 53 seconds

What the new plan APS thinks about the structure of your content? Um.

Sean Wheeler 20 minutes 58 seconds

In this example, this warning is OK. The notes section header exists, but there's no content. You're allowed to have an empty notes section, but you're not allowed to not have the header.

Sean Wheeler 21 minutes 14 seconds

And then the last step, converting it to mammal and.

Sean Wheeler 21 minutes 19 seconds

How to publish it? So if you're going to ship the content with the module where all that needs to be laid out in the module folder and if you want to create updatable help, how to do that. So that's what's new in docs.

21:39

21 minutes 39 seconds

Thanks, Sean.


Michael Greene (POWERSHELL) 21 minutes 41 seconds

Excellent work as always. One note I wanted to add since we're at the end of all the agenda items except for demos, typically following the calendar year, the October call just given the the schedule would end up being the.

Michael Greene (POWERSHELL) 22 minutes 1 second

Final, I guess, you know, community call of the year that we are leading and that leaves space for one more occurrence that the community can lead. I don't think there's been any discussion yet of if anybody would like to lead a community call and you know, if so, who.

Michael Greene (POWERSHELL) 22 minutes 18 seconds

So we can start a discussion over in the PowerShell repo and work that out. But I just wanted to set that expectation and get people thinking about it. I know Jan had mentioned to Jason or or I guess Jason asked me to ask if if Jan was on the call to talk about.

Michael Greene (POWERSHELL) 22 minutes 36 seconds

Oh, my posh. So I wanted to.


Steve Lee (POWERSHELL) 22 minutes 38 seconds

No, he he informed me that he's having Internet issues right now, so he'll have to defer to next time.


Michael Greene (POWERSHELL)

22 minutes 42 seconds22:42

Michael Greene (POWERSHELL) 22 minutes 42 seconds

OK, that sounds fine. I saw in the chat, James, you had mentioned that you might be interested in doing a demo if time allows. Is that still the case?

Michael Greene (POWERSHELL) 22 minutes 57 seconds

James Brundage is who I was.

Michael Greene (POWERSHELL) 23 minutes 1 second

If he has dropped as well, then we may just.

Michael Greene (POWERSHELL) 23 minutes 5 seconds

Give it a few minutes. Is anybody else interested in doing a demo today? If not, totally fine.

Michael Greene (POWERSHELL) 23 minutes 19 seconds

OK. I will end the awkward silence. Thank you everybody for joining. I really appreciate all of the updates and we'll look forward to talking to you again either at the community call led by the community if we do one yet this year and if not.

Michael Greene (POWERSHELL) 23 minutes 34 seconds

We will see you again in 2026.

Michael Greene (POWERSHELL) 23 minutes 38 seconds

Thank you very much. Enjoy the rest of your day.


