# The Data Liberation Front

_The code for this project can be found [here](https://github.com/lordvaider/whatsappscraper)_

For my next post, I'd initially planned to analyse some Whatsapp group chats and see what I could deduce about the chat participants and the relationships between them. There are quite a few examples out there of people who have built tools that collect and display statistics like number of messages sent, most active time of day and most used smileys. While this is a perfectly reasonable thing to do, I wanted to try and figure out more readily consumable information like the names (and nicknames) of the people in the chat, their birthdays and anniversaries, and maintain a scoreboard of who wished whom, which people seem to participate in conversations together and try to infer who is close and who isn't - Really, I can't think of a more noble usecase for my newfound datadude skills than to generate fresh leads for gossip!

Unfortunately, I got sidetracked into something else, and because I'm way behind on my publishing deadlines, I decided to write about that instead.

__Table of Contents:__

* TOC
{:toc}

# Whatsapp - The Good, the Very Good, and the Excellent

For as long as I can remember, Whatsapp let you easily export your chat in a neat .txt file with just a couple of button clicks. I already had great respect for Whatsapp for several reasons:
 - __Design:__ Beautifully minimal.
 - __Penetration__: Whenever I meet a new group of people, we always form a Whatsapp group. Everyone always seems to have and use Whatsapp.
 - __Erlang__: The backend of Whatsapp is written in Erlang. I don't know why exactly that is impressive, but I do know that Erlang is fucking cool. Some day, I'll figure out exactly why Erlang is so cool (Right after I finish reading the Escher and Bach parts of [GEB](https://en.wikipedia.org/wiki/G%C3%B6del,_Escher,_Bach)), but to give you an idea, this is often touted as the reason they were able to handle over 900M users with only 50 engineers.
 - __Obvious Features that no one else has figured out__: When you send chats on Whatsapp and there's no signal, Whatsapp will automatically send the chat when the signal resumes! I'm still not sure why Messenger can't accomplish this. 
 
 "Good for Whatsapp!" - I thought. Clearly data portability is just one more thing that these guys handle in a clean, simple and efficient manner! Unfortunately, I was in for a rude shock...
 
# Whatsapp - The Ugly

I don't remember why, but I decided to export my Whatsapp call logs as well - The more data, the better right? However, on clicking the 3 dots, I couldn't find the export call logs option. At first, I thought ah well, guess Whatsapp isn't perfect after all - They've buried the export call logs option somewhere deep inside the settings window. After a few minutes of poring over the settings, it dawned on me that _maybe there is no export call logs option in Whatsapp!_ I frantically tried to refute this possibility by looking online, but each empty Google search cemented it further into a horrifying fact.

# The 5 Stages of Grief

__Denial__: I spent an embarassingly long amount of time convinced that exporting call logs was in fact possible, and that I was just a technologically challenged simpleton that couldn't figure it out. At some point Googling the exact same phrases again and again gave way to...

__Anger__: Isn't data portability the law? Aren't telecoms obliged to provide to us our call logs under GDPR? What makes Whatsapp different? Why wouldn't they just provide the call logs anyway? This time, the Zuck had gone way too far. For corrupting the soul of Whatsapp, he would pay the ultimate price... 

__Bargaining__: I did find some apps that claimed to be able to retrieve the Whatsapp call logs, but the only testimonials I could find were from the creators of the apps themselves. I was not quite ready to hook my Whatsapp up to a shady app I downloaded off some dark corner of the internet. I looked for ways to get the Whatsapp db files from my phone, but these files were encrypted, and it was not clear how to decrypt them (There was an app that claimed to let you do this, but I _definitely_ didn't want to hook my Whatsapp up to a shady third party _crypto_ app). Maybe I could scrape the data from Whatsapp Web? Nice try, but Whatsapp Web doesn't even have the "Calls" tab (Why would it?). Is there a way to scrape data from apps? No, it seems that is a notoriously difficult problem, and apps like Whatsapp have booby traps built in specifically to prevent scraping...

__Depression__: The most furstrating part was that the data was right there! In my phone! In theory, I could just scroll through the logs one by one and enter each call log into a table. Yeah right, all I need to pull that off is a metric ton of Adderall. Maybe I could build a rig with a robotic arm that would do the scrolling and clicking, and a camera that would take pictures of the screen and extract the data? I actually entertained this idea for about 5 seconds, before remembering that every hardware project I've attempted has ended in either abortion, miscarriage or attempted suicide...

<b><s>Acceptance</s> Obsession</b>: Unfortunately, I've never been good at letting things go. I knew I had to get over the burning injustice, but just like the time Kiran Kapoor called me fartface in front of the entire 6th grade, my mind kept going back to it. The great thing about engineering problems though (As opposed to bitter middle school memories) is that if you keep thinking about them, it sometimes leads to a resolution. 


# Footnotes
* footnotes will be placed here
{:footnotes}

---

{% include comments.html %}