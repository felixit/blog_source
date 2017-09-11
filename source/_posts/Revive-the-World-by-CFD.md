---
title: Revive the World by CFD - from Zero to CPR (1.Basic math)
date: 2017-09-10 17:21:27
tags:
---

**A day on April 8th, 2024**

"Allen, look! the Robot No.2 holds a plate of corn cakes and some water, standing there, and... seems like walking towards us." Felix shouts in a low voice.
"Are you sure? Felix, our room is too small, I had a nightmare last night. So hungry now..." Allen opens his eyes gradually and say.
"I am pretty sure. Last time, I was grabbed by him. His upgraded alloy legs are very strong and flexible, like a human. If I have that kind of legs, I will never be trapped here. The Robot No.2 is receiving some commands now. Really hope to eat a full meal otherwise I will starve to death too."
"You have repeated your story for more than 100 times. I do not care how you come, I just wonder how we die."
"I appreciate that you can remind me this joke from more than 20 years ago. Shh~, be quiet, he comes." Felix says again.
<!-- more -->
"Here are the energy and water for you guys." Robot No.2 says and then leaves. Obviously, the food here, no matter what it is, is always called 'energy'. From the view of robots, to name those kinds of carbon-based low-energy biological materials is a big task, so 'energy' seems to be elegant and not offending. The corn cakes and water are put in many metal plates carved with the room number following some other strange number prefix. Felix grabs the plates and gives one cake to Allen and himself too, then says:"Could No.2 be able to say something else?"
Allen eats for a while and then sits up:"Absolutely. It is not necessary to talk with you, right? Arresting you or not is none of your business."
Felix laughs with some sadness. A while later, with serious attention, he asks:"Do you know what day is today?" It sounds like there is something important to announce.
"April, 8th, what happened?" Allen responds.
"The total eclipse is at 13:17. It will last 114 minutes in total." Felix says with great confidence.
"The last total eclipse was in 2017. Really cherish the memory of that period of time. But... do you want to see the eclipse today? You are really interesting..." Allen drinks a cup of water and says.
"No, don't you want to escape from here? The Union Nation conference is going to be held soon, we should tell everybody what we experience and know, right?"
"You think I am enjoying the life here? If I can escape, I will never waste my life here even for 1 second. There are 4 robots here, they are equipped with best sensors. Do you hope they will enjoy the eclipse while we can escape at the meantime?"
"Allen, no, I am not kidding. Do you know the closest city has hundreds of miles from here?"
"Sure. Felix, you mean our place is the best location to see the eclipse? Good idea."
"To save time, I just say in short. Our place is surrounded by desert. No underground fossil resources also. Where the power of those robots come from?"
"Everyone knows this. Solar power. They are mostly equipped with the solar panel on their teeth. You mean they will not have enough power during the total solar eclipse?" Allen says with shaking his head.
"Of course no. They have batteries inside. The solar eclipse will only last less than 10 minutes, which is piece of cake for those batteries."
"Do not hide anymore, tell me your ideas." Allen is going to be impatient.
"The key is the sensor! If their sensors detect that the solar intensity is lower than some certain value, to the sake of power, they will transfer to night mode. I mean they will start to go on patrol and has only one-third of normal detection distance, which means we have least distance to escape. The artificial intelligence is just this kind of thing. From possibility and statistics, they are perfect, which does not mean that they are perfect at all time. This is the best chance for us." Felix says and his eyes are glinting. To see the rising sun is a dream for him. Maybe this can be realized tomorrow.
Allen thinks for a while, and then says: "Reasonable, but even in the night mode, the robots are still going to patrol our area. At any single time, any inch of the area is under monitored by at least one robot. Do you still remember the Alpha Go in almost 10 years ago? In the 19*19 problem space, it can beat the Lee and Ke easily. Our small area is just a baby discrete optimization problem."
Felix sits closer to Allen, and then say:"Correct, you are my best friend. That is the night mode patrolling. Are you curious about that the time we are closed into the room by the robots every day is different? Why different? From my long-time investigation, it is an adaptive system! It adapts according to the solar intensity. Recently, at about 8:32 pm, the solar intensity goes down to less than 50W/m^2, then the robots will start to go to the designated 4 locations respectively, which are distributed uniformly in this closed district. To ensure the best performance and lowest energy cost rate, the embed system needs some time to do auto-cleaning and backend optimization.  No computers do not need rest completely, do you know?"
Allen understands a little, but says:"Are you sure they do not have the emergency mechanism? For example, the heavy cloudy weather or the sand storm. Also, so what?"
Felix does not pay attention to that, seems very confident:"Those phenomena can not drop the solar intensity so much. Our chance is here! I find that the bio-detector does not work when they are doing backend optimization. Because one night, I saw them through the window and found that the lights of bio-detector are off during that period of that time. That is almost 10 minutes. Normally, we were confined to the room, but today is different. The sun and the moon give us the best chance, we can not miss it."
Allen seems to be very excited too, so they decided to talk about it in detail. It is clear that they must succeed, otherwise, they may not be able to escape here forever, or even do not need to think about how to escape here anymore. They go into the toilet out of the sights of No. 2 Robot. 
The first problem is that when the bio-detectors will become inactive. This is a critical element for this race. When the solar intensity drops down to 5% of its peak value during that day, the robots will start the auto-cleaning program and go to the designated locations. The picture shows below.
![Solar eclipse](http://upload-images.jianshu.io/upload_images/6821307-eb05e9e15701845c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/330)
To model this problem, we can put the sun into the Cartesian coordinate system
![Eclipse in Cartesian coordinate system](http://upload-images.jianshu.io/upload_images/6821307-6cd3184e64bd20bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)
Based on the figure above, we can get the following relation between the relative solar intensity with the length of $c$, as well as time $t$.
{% math %}
\begin{equation*}
\left\{\begin{aligned}[c]
cos(\frac{1}{2}\theta)&=\frac{R-0.5c}{R}\\
S_{sector}&=\frac{1}{2}\theta R^2\\
S_{\Delta}&=2\frac{1}{2}(R-0.5c)\sqrt{R^2-(R-0.5c)^2}\\
S_{shade}&=\pi R^2-2(S_{sector}-S_{\Delta})\\
Percent& =\frac{S_{shade}}{\pi R^2}\\
\end{aligned}\right.
\end{equation*}
{% endmath %}

"Allen, we need to know the time when the solar intensity becomes 5% of its peak, so we can let the 'percent' to be 5% and solve the above equations." Felix says.
"Hmm, a little hard... although it is a pure algebra equation system. We need a numerical way instead we will miss the eclipse." Allen says.
"Sure. I always take a programming calculator. We can plot the relation between time and solar intensity percent. See this."
![Solar intensity percentage with time](http://upload-images.jianshu.io/upload_images/6821307-77d60a438cdee52f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)
"If we zoom in, we can see after 54.7 minutes, which means at 14:01:42, the robots will start to go into night mode. I have measured the preparing process (bio-detector off) many times, it always takes 8.5-10 minutes. To be safer, we assume it will take 8 minutes. The time between 14:01:42 and 14:09:42 is our escaping time. For the daylight mode, it needs 25% solar intensity percent to be active, so that will be a while later then the night mode is turning on."

The next question is the path for escaping. To avoid this closed district coming into notice by the satellites, there are no walls or fences at all. From the top view, this closed district is almost like a common monitoring station, nothing special. Ironically, Allen and Felix may be regarded as two quietly brilliant scientists working in Antarctic stations, who make great contributions but nearly have little public attention. Now both of them would rather not be paid attention by those robots too.
![Top view](http://upload-images.jianshu.io/upload_images/6821307-451e9cc965a7930e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

The shortest distance between our current location and the nearest location out of monitoring (point B) is 3.0 km. Theoretically, the averaged speed must be 6.25 m/s for 8 minutes. After the blue monitoring circles become active, the escaper must be out of the B point.
"Is this OK for you? Felix? After we go beyond the B point, the left part will be easier for us, because we also need to escape the 'Day Monitor' circle." Allen asks with deep concern.
"This speed is really challenging, but sprint is my strong point. My limit is 8.5 m/s, but I need some warm-up to reach that limit. The eclipse will start soon. Do not hesitate too much, just run together." Felix begins to comfort the Allen, but they all know a turning point will come soon. whether to fail or to have some hope of the entire life will be determined in those 8 minutes.
Allen nods with some hints. To not attract too much attention, they go out to the small yard for activity, pretending to take a rest, the heaviest rest they did in the past more than 30 years.

Just a while later, the eclipse starts. Allen and Felix breath with great tension. A famous sentence--'Opportunities are only for those who are ready'--starts to hang in Felix's mind and heart, but he does not have time to know whether he is ready or not. To expose their conspiracy to all over the world, escaping here is the first step, although hundreds of miles stretch this area among the entire desert, that is the following question of the next step.
The sky is darker and darker. The surrounding has no sound. Hopefully, this is just a movie, then the audience can enjoy the 'di-da-di-da' in the background count down.
"Something wrong! The auto-cleaning and backend optimization may not take 5 minutes! They do not have much stuff to clean and optimize for the only morning instead of the entire day!" Allen suddenly shouts with a little louder voice.
"What's the meaning? You mean..." Felix seems to already understand it.
"We must increase our speed to at least 9m/s, which exceeds our limit!" Allen stares at Felix.
Felix suddenly understands and interrupts Allen. The total eclipse will come soon. The only chance. Escape or surrender? If we fail, nothing can be done in exchange of that. "Maybe the robots need longer time to do the cleaning, maybe machines also have some bugs, maybe the monitoring range is not as far as one-third of its maximum in the night mode..." Numerous ideas go in and out from Felix's mind, which leads to extreme strain.
"14:01:42! The total eclipse starts! Felix, run to point A and then stop!" Allen seems to send a command to Felix.
"What? point A? Me? Where will you go? Not together?" 
"No time left, please, please run to point A and stop! After the bio-detector become active, then continue to run to the east!"
"What? Will not we be exposed? Why not together?" 
Felix just asks again and then Allen already runs at full speed to the south. There is no time for thinking.

On the Northern Hemisphere, the sun locates at the normal south direction at noon. A view of Allen's back is running to the darkened sun. It looks like a very little single life is disappearing into a black hole.

To be continued...