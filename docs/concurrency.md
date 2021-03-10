layout: post
title: "Concurrency in Python"
date: 2021-03-10 20:58:00 -0000
categories: concurrency python

# Concurrency in Python 🐍 🐍 🐍
…and how to cook a banging roast dinner 🍖 /🥬 

I’m going to be honest (because I already have this job and I don’t *think* I can be retroactively fired 🤞😬), but I’ve never really understood concurrency. I always knew it was something to do with doing multiple things at once, and that apparently Python isn’t particularly good at that. Whenever it’s come up, I’ve essentially gone with the technique of either ask someone that knows more than me (thanks Max!!), or if they’re busy doing real work I copy and paste N (>>5 👀) different stack overflow answers until one of them vaguely speeds things up and then claim a glorious victory before moving on none the wiser. It’s worked for years, and I stand by it (#engineering).

I don’t know whether it was lockdown boredom, imposter syndrome, or the fact that numerous people asked me to explain it, but this year I decided to finally try and understand at least the core principles behind concurrency in python. So here goes nothing…


> DISCLAIMER
> This will be one of my hand-wavey, not too worried about the details, let’s understand enough to do what we want to do explanation. Please feel free to give me a shout about any inaccuracies!


## Threads, Processes and the GIL

Classic interview scenario:


> Interviewer: Why is concurrency difficult in Python?
> Candidate: Because Python has the Global Interpreter Lock (GIL), so you need to be careful
> Interviewer: YOU’RE HIRED!

Wait…what? What’s the GIL and why do I care?
It turns out the answer has to do with the fundamental design of Python, and what this means for “Threads” and “Processes”. These are the two biggies. Two different ways to concurrency nirvana (🤘). But what are they and which should I be using?

I always read something like this:


> If you’re IO-bound, use threads. If you’re compute bound, use processes.

OK…cool. So if I want to make 5000 requests to some API (ie. I’m *IO-bound*) then use threads, but if I want to process 5000 chunks of data (i.e. I’m *compute-bound*) then I should use processes.

But…what’s the actual difference?

🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟 
So if there’s one take away from this whole thing it’s this:


> **Python runs one process at a time, and a process cannot execute multiple things at once.**

🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟

OK, so let’s break this down and try to understand what each concept means in turn.

**Processes**
When you run your script with `python my_script.py` you are doing so within *a single python process* (same as when you open the python interpreter with just `python`). The python interpreter essentially steps through your script line by line and executes it (more or less). So clearly, if it goes through line by line, instruction by instruction, it cannot do multiple things at once!
So what then “multiprocessing”? It’s just your single process (`python my_script.py`) kicking off a bunch of other python processes (kind of like it doing `python other_temp_script.py` a few times) to do things, completely separately.
Back in the day when Python was invented, the decision was made to make sure that you could never do more than 1 thing at a time in a python process. To make sure of this, they needed something to enforce this rule…

**GIL**
The “Global Interpreter Lock” forbids any python process from doing more than one thing at a time. It’s like the fun police 🚔 , making sure that everything stays chill, not too wild, nice and linear.
But what if I don’t actually want to *do* multiple things at once, but I want to *keep track of* multiple things at once…

**Threads**
Threads can be thought of as the multiple things that can happen in a single process. In the IO-bound scenario, e.g. making a bunch of requests to some API, you could create multiple threads to do this. Each thread involves making a request, waiting for a response, then returning this response. Using threads you could loop through each thread and kick off a request, then go back to the first one and see whether it’s got a reply yet. All within your single python process.


# Roast dinners 🥲 

Imagine you’re trying to make a banging roast dinner. This is the menu:


- Chicken 🍗
- Potatoes 🥔
- Broccoli 🥦
- Gravy 🇬🇧 

But there’s a catch - you’re the only person about to cook it, and ***you*** ***can only do one thing at a time*** - you’ve only got one pair of hands! You can either chop potatoes, prepare the chicken, steam the veg, or make the gravy etc etc, but you can’t physically chop carrots while stirring gravy.

**You** **are a single process.**

So what do you do? You’re stuck! You can only do one thing at a time, so your afternoon is going to look something like this:


    - prepare chicken
    - put chicken in oven
    - take cooked chicken out of oven
    
    - cut up potatoes
    - put potatoes in oven
    - take potatoes out of oven
    
    - cut up broccoli
    - steam broccoli
    
    - make gravy
    
    - combine everything
    - serve

Aside from half of the plate being stone cold, it’s going to take forever. If we plotted this cooking technique over time:

![](https://paper-attachments.dropbox.com/s_041D9188E0AD959036E0E1C4ADF482CF4D6E651C9CB2FD56DEDF0915294F68C0_1609258067281_Quick+sheets+-+page+40+1.png)


So because we only have one process (and only one thread), we’ve had to wait for each task to complete, before moving on to the next. You’ve served cold food and all of your diners have gone home as it’s now Monday morning. Shame 🔔.

But why do I need to stand and wait for the chicken to be fully cooked before I start doing something with the potatoes? Just because I only have one chef (i.e. one “process”) doesn’t mean I can’t keep track of multiple things (multiple “threads”). I’ll never actually be taking a chicken out of the oven while simultaneously cutting up broccoli, but I can still have ***multiple things happening that I can check in on.***

So if I allow my single process to use multiple threads, my afternoon now looks more like this:


    - prepare chicken
    - put chicken in the oven
    - prepare potatoes
    - put potatoes in the oven
    - cut up broccoli
    - steam broccoli
    - make gravy
    - take chicken out of oven
    - take potatoes out of oven
    - combine everything
    - serve

Much better! We’ve taken advantage of the fact that a large part of each of our tasks (i.e. cooking a chicken) consists of hanging around and waiting for things to cook! So rather than stand around waiting an hour for my chicken to be ready, I can crack on and peel some potatoes etc etc.

Again, if we plot it:

![](https://paper-attachments.dropbox.com/s_041D9188E0AD959036E0E1C4ADF482CF4D6E651C9CB2FD56DEDF0915294F68C0_1609257230320_Quick+sheets+-+page+41.png)


By allowing a single process (me) to run multiple threads at once, I’ve served the meal quicker and better. Everyone’s gotten hot food on time, and I’m once again the best chef in the world. 👨‍🍳 👩‍🍳

Can we make this even smoother?
Are we IO-bound or compute-bound in this example?
What happens if we make more chefs (do multi-processing)? Will it speed things up?

**Exercise for the reader: would multiple threads help me if I had to make 10 sandwiches?**
(Answer here)


# Conclusion

Both multi-threading and multi-processing are possible in Python. Each allows us to perform concurrent tasks, but they both fundamentally differ in how they do this and for which tasks they’re best at. In this post we haven’t dived into the many pro’s and con’s of each, or how to implement them technically, but hopefully it has provided a rough conceptual understanding of what is meant by Concurrency in Python.

And who knows, maybe after another 3 years of employment I’ll finally be brave enough to learn what a “greenlet” or a “neural network” actually is 🤷‍♂️ 

