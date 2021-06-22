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

![png](/images/2020-06-21/joylobo.PNG)

<p style="text-align: center;">
<i><a href="https://3idots.fandom.com/wiki/Joy_Lobo">Joy Lobo</a> also abandoned his hardware project, but he showed impressive follow through on his suicide.</i>
</p>

<b><s>Acceptance</s> Obsession</b>: Unfortunately, I've never been good at letting things go. I knew I had to get over the burning injustice, but just like the time Kiran Kapoor called me fartface in front of the entire 6th grade, my mind kept going back to it. The great thing about engineering problems though (As opposed to bitter middle school memories) is that if you keep thinking about them, it sometimes leads to a resolution. 

# The Solution
The eureka moment was realizing that I didn't need a robotic hand, I could just mirror my phone screen on my computer, write some code to control the mouse and simulate the actions of scrolling through the call logs and taking screenshots of each one. Once I had the screenshots, I could use OCR to extract the text data from them and chuck it into a table. Easy right? Right guys?

![gif](/images/2020-06-21/tumbleweed.gif)

<p style="text-align: center;">
<i>Guys???</i>
</p>

## Mirroring my Phone on the Computer
This was easily achieved with a little [scrcpy](https://github.com/Genymobile/scrcpy) magic - The hardest part of this step was removing the USB cable from my phone charger and connecting it to my computer. 

## GUI Automation
The next step was figuring out how to control my mouse and keyboard with code. In principle, this was also fairly easy - [pyautogui](https://pyautogui.readthedocs.io/en/latest/index.html) is a great python module that lets you do exactly that. Some of the challenges I faced:

1. The biggest challenge with GUI Automation (Or automation of any kind really) is the inability of the machine to adapt to even the most minor surprises. The machine will keep executing the same instructions, and if something unexpected happens in the middle of that, things could go completely wrong from that point on. For example, I recieved a phone call in the middle of one of my runs, and that threw off the carefully balanced workflow of steps so that the code started saving screenshots of my homescreen again and again (It could've been worse). If you ry this yourself, keep your phone on flight mode, and don't leave the machine completely unsupervised; Sit nearby with a book so that you can keep an eye on things.

2. The time interval between pyautogui operations had to be optimised to account for the non-trivial and slightly random latency of GUI functions like switching between windows, or opening up the save image dialog box. I had about 800 call logs to save, and given that I was sitting around watching over the process, I could only spend a maximum of 4 seconds per log.

3. While the whole point of GUI automation was to bypass manual drudgery, there was still the one-time drudgery of figuring out the pixel locations for the mouse clicks in the workflow. To make this procss slightly smoother, I called print(pyautogui.position()) in a for loop with a sleep, so that it printed out the mouse position at regular intervals. I then positioned the mouse every where I needed it. I also tried to use keyboard shortcuts as much as possible when executing tasks.

4. Finally, I found that in some cases the 'Enter' key didn't work with pyautogui for some reason. Someone had written a detailed answer on Stackoverflow as to why this is, but I only read the answer below that one, which told me to use [pydirectinput](https://pypi.org/project/PyDirectInput/) instead. I used it, it worked, I moved on.

You can find the code I used to perform the screen grabbing [here](https://github.com/lordvaider/whatsappscraper/blob/main/imgscraper.py). After running this, I had a folder of images like the following, one for each call log.

![png](/images/2020-06-21/Example1.PNG)

## Optical Character Recognition 
Optical Character Recognition or OCR is the use of technology to distiguish and identify characters in images and convert them to text. It is one of the earliest areas of AI research and there are some truly impressive OCR tools out there today - Two examples that blew me away were Google Lens, which can read in foreign language text from your phone camera and translate in real time, and mathpix which can be used to generate latex for mathematical formulas. 

My use case was a particularly simple one. The text was digitally generated and formatted, not handwritten or in some funky font, and the image itself was a screenshot, not a picture of a physical sheet of paper (Which complicates things because the wrinkles in the paper and the perspective of the camera angle distorts the text). Despite all of this, I found that the OCR module I used ([pytesseract](https://pypi.org/project/pytesseract/)) did not give perfect results. The two issues that came up were:

__Correctness__: Given the simplicity of my use case, I was expecting perfect accuracy. However, I still got errors in about 2% of my images. There are ways to boost the accuracy by pre-processing the image, but I adopted the lazy method of going into my phone settings and changing the "Display Size" setting to the largest possible, which removed all but a couple of the errors.

__Output Consistency__: This was a much more serious problem (Mostly because it forced me to write more code than I'd initially expected to). The format of the string that I received as an output was not consistent across all my images. In some cases, the call type and the duration were part of the same line. In others they were in different lines - The figures below show the 2 main classes of text output I received in this case.

![png](/images/2020-06-21/Template1.PNG)

<p style="text-align: center;">
<i><b>Class 1:</b> Each token printed in a separate line</i>
</p>

![png](/images/2020-06-21/Template2.PNG)

<p style="text-align: center;">
<i><b>Class 2:</b> Call Type and Call Duration in the same line, Call Time and Call Size in the same line</i>
</p>

## Parsing OCR Text to Structured Data
The eventual goal of the project was to convert the text ouput of the OCR engine for each screenshot into an object of the "Record" class below:

    Record:
        Name (string)
        Status (string)
        Date (date)
        Calls (list of calls)

With each call being an onject of the following class:

    Call:
        cType: (Incoming/Outgoing/Missed)
        Time (HH:MM)
        Duration (H:MM:SS/Not answered/Empty(Iff cType == Missed))
        Size ( DD.D MB/ DDD Kb /Empty(Iff cType == Missed OR Duration == Not answered))

As stated in the previous section, the fact that there are 2 different types of text output complicates things. In such cases, the easiest solution is what I call the "Waterfall" approach - Parse the text assuming it's a class 1 output. If that fails, parse it assuming it's a class 2 output. If that also fails, throw an error. If your code throws a lot of errors, that's a clear sign that there are more than 2 classes of outputs, and you need to add at least one more level to your waterfall.

I didn't quite use the waterfall approach in my code - Never go full waterfall if you can avoid it. However, I did something pretty close to it (You can have a look at my code here). In  a nutshell, I checked each line of the input against a list of regular expressions to determine what kind of token it is and then used it's position relative to the other tokens to insert it into the output structure.

__Example__: Consider the line L = "19:42". It matches the regular expression R = \[0-9\]{2}:\[0-9\]{2}, that is to say, it is two digits, followed by colon, followed by two digits. This means that it is either a Call Time or a Call Duration. If the previous line was "333 kB" then we know that this token is preceded by a Call Size, which means it's a Call Time (Call Durations are never preceded by Call Sizes). Assuming that all tokens are present in the text input in the correct order, we find the first duration-less Call in the output structure, and populate it with 19:42.

The most unsatisfying thing about the above approach is that I have made a lot of assumptions about the input, but these assumptions aren't clearly stated in the code (I hate when people don't state their assumptions). Further, because the assumptions are baked into the structure of the code, if I want to add/remove/change one of them, I need to restructure the entire code. Not fun.

Another unsatisfying aspect of this approach is it's "All or nothing" nature. If the OCR process ommitted the ":" and incorrectly read in the input as "1942", this code would just throw. It would be nice if we could flag it as corrupted, but still match with the duration token, based on the fact that it's still a pretty close match to the regexp, and the fact that it comes after a Call Size token.

### Sidebar: Probabilistic Approach to Data Extraction

In this section, I've tried to flesh out a probabilistic approach to the data extraction problem, that tries to resolve the issues described above. STATUTORY WARNING: This section is speculative and hand-wavy. Consume at your own risk.

In a broad sense, the idea is to represent the state of knowledge about each line of the input, and what kind of token it represents, using a probability distribution. Our knowledge about the format of the input text can be specified with some conditional probabilities, and then it's just a question of applying Bayesian inference to get the resulting posterior distribution for each line. Once this is done, the maximum probability token for each line can be selected. It might clarify things to view the example above in the context of this approach.

__Example:__ Again, we have the line L = "19:42" and want to figure out what kind of token it is. For simplicity, let us assume we know it is either a Call Duration, a Call Time, or a Call Size, and the prior probability of each possibility is 0.33. We can repesent this state of knowledge with the vector $$[0.33, 0.33, 0.33]$$

In order to get the final distribution or each line, we must go through a list of tests. Each test comes with a list of conditional probabilities (The probability of a particular test outcome given that the line is of a particular type) - Given these, the posterior distribution can be inferrred from the test outcome. The tests and the associated conditional probabilities encode our assumptions about how the system behaves.

One such test could be to compare L with the regular expression R = \[0-9\]{2}:\[0-9\]{2}. The associated conditional probabilities are:

$$P(L$$ matches $$R\| L =$$ Duration$$)$$ = 0.99

$$P(L$$ matches $$R\| L =$$ Time$$)$$ = 0.99

$$P(L$$ matches $$R\| L =$$ Size$$)$$ = 0.01

We never set any of the probabilities to 1 or 0, becaue there is [always a chance](https://www.lesswrong.com/posts/QGkYCwyC7wTDyt3yT/0-and-1-are-not-probabilities) that the OCR output was corrupted (Or that we are living in The Matrix).

On applying the test, our probability distribution get's updated by Baye's Theorem into $$[0.4975, 0.4975, 0.005]$$

Another test could be based on the token type of the previous line. The conditional probabilites in this case are:

$$P(L_{i-1} =$$ Size$$\| L_i =$$ Duration$$)$$ = 0.5

$$P(L_{i-1} =$$ Size$$\| L_i =$$ Time$$)$$ = 0.05

$$P(L_{i-1} =$$ Size$$\| L_i =$$ Size$$)$$ = 0.01

$$P(L_{i-1} =$$ Duration$$\| L_i =$$ Duration$$)$$ = 0.3

$$P(L_{i-1} =$$ Duration$$\| L_i =$$ Time$$)$$ = 0.01

$$P(L_{i-1} =$$ Duration$$\| L_i =$$ Size$$)$$ = 0.6

Let's say that the inferred posterior distribution for the previous line $$L_{i-1}$$ is $$[0.05, 0, 0.95]$$. 

Then we can further update the probability distribution for $$L_i$$ as follows:

$$P(L_i) = P(L_i\|L_{i-1} =$$ Size$$)\*P(L_{i-1} =$$ Size$$)$$ + $$P(L_i\|L_{i-1} =$$ Duration$$)\*P(L_{i-1} =$$ Duration$$)$$

where:

$$P(L_i\|L_{i-1}$$ = Size) $$\propto P(L_{i-1} =$$ Size$$\|L_i)\*P(L_i)$$

$$P(L_i\|L_{i-1}$$ = Duration) $$\propto P(L_{i-1} =$$ Duration$$\|L_i)\*P(L_i)$$

The final posterior probability is $$[0.9786, 0.0202, 0.0011]$$ or a 97.8% chance that the token is a Call Duration.

### Advantages over the waterfall
Basically, the probabilistic approach gives us a way to chuck everything that we know about the system into a big pot and stir it till inferences appear. It also makes explicit our expectations and assumptions about how the system is supposed to behave. We can add or remove assumptions by simply adding or removing the corresponding tests. The strength or weakness of these assumptions can be tweaked by changing the actual values of the conditional probabilities. Finally, it gets around the brittleness of the Boolean logic approach - Even if a couple of tests fail, the information provided by the other tests could still lead us to the right answer.

# Why?
"But datadude, you infuriatingly indirect imbecile", I hear you say, "Surely there's a more direct way to fetch this data than to build this Rube Goldberg contraption involving mirroring, GUI automation and OCR?"

I'm the first to admit that there are many shortcomings with the approach I've taken. Probably the biggest one is that it isn't portable - Most people can't use this approach to get their whatsapp call logs. Hell, even I probably won't go through this process on a weekly basis to keep my call records updated! Secondly, there is a lot of ugly ad hocery involved in converting the OCR text into structured data. Whatsapp has this data in a neat table somewhere inside it, and if you're looking for something clean and scalable, you need to hunt down that table.

Why then did I do it this way? It just seemed more fun and horizon-broadening! I've thought about both GUI automation and structured data extraction in different contexts before, and this seemed like a good project to play around with these concepts. 

# Conclusion
I don't think data portability is everyone's top concern right now, but I think that will soon change. At the moment, a lot of companies seem to be getting away with [shoddy](https://chrislukic.com/2021/06/16/techniques-to-prevent-adoption-of-your-api/), or [non-existent](https://www.alias.dev/report) data portability solutions. When consumers start rightfully demanding their data, I don't think these companies will meekly hand it over - We will probably need to resort to creative tactics to get it. Even if they do hand over the data, it will have to converted from the structured format they use into the structured formats that each of us wants, and that isn't an easy problem. 

The battle to get my call logs out of Whatsapp was a microcosm of the upcoming Portability Wars and to be honest, it was actually kinda fun. Still, I live in hope that Whatsapp will see the error of their ways and just add the damn "Extract Call Logs" button, so we can all move on to bigger and better things.

---

{% include comments.html %}