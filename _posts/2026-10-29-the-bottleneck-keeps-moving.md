---
layout: post
title: "The Bottleneck Keeps Moving"
date: 2026-10-29 09:00 +0000
comments: true
categories: [ai, programming, tdd]
---

One of my favourite books about software engineering isn't actually about software engineering.

[*The Goal*](https://www.goodreads.com/book/show/113934.The_Goal) by Eliyahu Goldratt is a book about manufacturing. More specifically, it's about a factory that isn't doing very well, and the process of figuring out why. Somewhere in that process there is a constraint that determines the throughput of the entire system.

Making something that isn't the constraint faster doesn't necessarily make the system faster. Quite often all you've done is make things worse, by putting more load on the bottleneck that was already there (there's a whole field around the theory of constraints that I'm not going to explore here). What I have been thinking about is how AI changes where the bottleneck sits, and how history doesn't repeat itself, but often it rhymes.

Computing, like a lot of markets, takes something expensive and makes it cheap. You stop worrying about the cost and use it more freely. Then you stumble into the next constraint. Mainframes, then personal computers, then phones, then smart devices. Just think about the number of photos you take. It's orders of magnitude more than it was a decade ago, because storage is basically free now.

Every time this happens some people complain that the new way is wasteful, that it isn't what real programmers do, or that "640K ought to be enough for anybody" (apparently Bill Gates never said this). Why do you need more?

Let's look at what happened before, and what happened afterwards.

## When computers were expensive

Go back far enough and the computer itself is the scarce resource.

You don't have one sitting on your desk. You certainly don't have several of them around the house idling. You prepare your program on punch cards, hand them over, and wait for your turn to run it. If you've made a mistake, you've wasted a trip through the most expensive part of the system. Programming was a slow, thoughtful process. You check it. You work through it. You make absolutely sure it's correct before it gets anywhere near the expensive machine.

Spending another hour of relatively cheap human time to avoid wasting expensive machine time was rational. The good programmer was the one who could "one-shot" the punch cards.

## Then computers got cheaper

So trying things got cheaper.

Terminals appeared. Eventually you got a computer of your own, and the feedback loop started shrinking. On my ZX Spectrum you turned it on and you were in BASIC. You could bash in a few lines, try something, fail miserably, and try again. Instead of carefully preparing a program and waiting for a slot to run it, you could write something and see what happened. Write, run, break it, change it, try again.

This is a fundamentally different way of working. You didn't have to prove everything in your head before you asked the machine. There was a trade-off, though. How long it took to run and see what you'd done depended on how much effort you'd put into getting it right the first time. If I can spend ten seconds of computer time to avoid spending ten minutes simulating a computer in my head, that's a good deal. The economics had changed.

## Then we started utilising more CPU cycles

The same thing happens with programming languages. At one point it makes sense to care enormously about the instructions you're asking a machine to execute. Machine time was slow. Memory and storage were limited and expensive.

Then those things got cheaper, and programmer time started looking expensive beside them. So we built abstractions. Assembly let us stop thinking quite so directly in machine code. You still wrote the instructions. You just gave them names.

C was the game changer. You described the program, and the compiler wrote the assembly. A lot of developers looked at the output and thought it was wasteful. I can write assembly better than this. Originally, you could. I doubt you can now, and you shouldn't spend the afternoon proving it. The compilers got better, and the machines got faster.

Then Java came along and did outrageous things, like managing memory for us. I remember people arguing about how much memory it took to create an object and what a waste that was! Yes, it used more memory. Yes, there were more instructions. But in reality, you never looked at the compiled code. It enabled you to think more about the problem and the domain.

Gradually, the machine stopped being the constraint and it moved to the programmer. We deliberately utilised a resource we had plenty of, to conserve one we didn't. Good engineering often means utilising the thing you have plenty of to conserve the thing you don't. If you want something to be fast, you cache it, and you spend disk and memory to do it. There's always a trade-off.

## Then we started writing code that wasn't the product

Then over time compute got so fast and so cheap that we could do something which would have looked completely bonkers a few decades earlier.

Previously we used to have people manually testing each release using a test plan rubric. Then we wrote programs whose only job was to run other programs and tell us whether they worked. They would be run at each RTM, or nightly if you were lucky. Then we started to have enough spare capacity that we could run them on every code commit. Then XP and TDD turned up and said we should write a test for code that doesn't exist yet. Run it. Watch it fail. Then run the whole lot again. From the point of view of efficient CPU use, this looks ridiculous. We're spending millions of cycles to test something we wrote five minutes ago, quite often to be told something we already suspected.

It was brilliant and had a huge impact on how I wrote software. It never became the dominant way of writing software, and at a guess never got past a 25% adoption rate. However, in my experience code that was written this way was consistently of higher quality and enabled teams to deliver solutions faster. The bottleneck was no longer the code. The constraint moved into the scaffolding around it.

CPU cycles aren't the constraint. Feedback is.

We deliberately utilised that computation to shrink the time between doing something and finding out whether it worked.

## And now we have LLMs

Which brings us to the latest thing people will call a waste.

We can describe a problem in a still higher-level language and have a machine generate the implementation. The obvious reading is that this is another productivity win as developers can write code faster.

Imagine a team of five who can comfortably produce ten pull requests a week, and they can review, understand, test and deploy them. Everyone gets a coding agent, and suddenly they can produce thirty. We have not tripled developer productivity ([Brooks's law](https://en.wikipedia.org/wiki/Brooks%27s_law) applies here). Except the team can still only properly review ten. Twenty pull requests sit in a queue. Next week we add another twenty. People start reviewing faster because the queue is getting ridiculous. Changes get bigger because everyone has several things in flight. Context switching increases. People understand less of the code being merged and just go on vibes.

We've made the programmers three times faster. Have we made the system any better?

This is exactly what you'd expect if you put the effort into optimising something that wasn't the constraint.

At this point people **think** the constraint is review. I'm seeing a bunch of tools and posts about automated code review, bug bots, and the rest of it. That's optimising the previous constraint. The bottleneck has moved from how we build things to what we build. We've been heading this way for a while. Lean Startup and validated learning were already about getting to the *what* as cheaply as possible. The *how* is now cheap enough that the bottleneck is the *what*.

For most of the history of software, code has been expensive. Someone has to sit down and write the stuff, and that puts a limit on how much of it enters a system. AI has removed that limit. I can produce an astonishing amount of code before lunch. We've spent decades making code cheaper to produce and now it's basically free to write. 

Programming was always a means to an end: working software that solves a person's problem.

## That's not real programming

There's a cultural pattern to all of this that I find quite funny. Every time the constraint moves, a group of people stay behind to defend the old one. Like the general who always fights the previous war. Writing assembly is real programming. Garbage collection is lazy. Dynamic languages aren't serious. Tests are a waste of time. TDD means writing twice as much code. BDD isn't programming. Using an LLM is just prompting.

There's usually some truth buried in the complaint. We work through it anyway, because the point is to solve someone's problem.

## Find the constraint

The lesson from *The Goal* isn't "automate everything". It's to find the constraint before you speed anything up.

If an LLM makes implementation dramatically cheaper, the bottleneck moves with it. A good programmer pushes back and asks questions, rather than nodding and building whatever ticket they've been assigned. The job is the same as it always was: solve the user's problem. XP was ahead of its time here: an embedded customer meant that programmers could talk to the person who had the problem.

We've spent seventy years making it cheaper to tell computers what to do.

We may finally have reached the point where the expensive part is deciding what we actually want them to do.
