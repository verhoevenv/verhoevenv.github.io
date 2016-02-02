---
layout: post
title:  "Notes from Domain Driven Design Europe 2016"
tags: ddd community conference notes
---
The year 2016 started off great by having a [superb Domain Driven Design conference](http://dddeurope.com/2016/) right here in Belgium, bringing together some big names of the scene (like Eric Evans, Vaughn Vernon, Alberto Brandolini, Greg Young, ...), a lot of really smart people, and me. These are my thoughts and notes on the talks I watched. I mostly wanted to write these down to be able to get back to them later. But maybe you can also learn something from this. Maybe it can serve as a conversation starter. So feel free to talk to me about these notes, or to correct me where I'm wrong!

I'll add videos when they appear online. While there are no videos in here, this will be one gigantic wall of text. Good luck.

# Opening keynote (Eric Evans)
Eric Evans gave us a view about his current thoughts on DDD, 12 years after [The Book](https://www.goodreads.com/book/show/179133.Domain_Driven_Design) came out.

While the technology has progressed a lot since then, the book and techniques are still relevant today. Even more so! The tools are so much better than 2003. This allows developers to spend relatively less time on fighting with heavyweight J2EE or ORM frameworks, and relatively more time on domain modeling.

# Modeling with Strangers
I wanted to go to the talk by Paul Rayner, but the room was packed full, no way to enter if I still wanted to be able to breathe. So I walked around a bit around the Unconference and Modeling with Strangers area. I collaborated a little on some modeling that was happening around a Norwegian video streaming service. It's an interesting experience to throw suggestions and see what resonates and what doesn't with the person explaining their domain. However, in the end, it is really hard to come to deep insights in someone else's domain and I ended up resorting to general advice like "how about more fine-grained domain events" or "errr maybe that should be in a separate bounded context?" But, on the other hand, maybe the modeling with strangers experience is useful just as a high level rubber ducking exercise for the one explaining the domain.

The whole self organized part of the conference (unconference / modeling with strangers) was just getting started, and I didn't have the time to return there later. But speaking from my SoCraTes experience, magic happens when you allow the community to connect, so I don't doubt the experience was more fulfilling for others that spent more time there.

# A board game night with geeks (Felienne Hermans)
What do you do when you have a completely theoretical question about the board game you're playing? You translate the question to a SAT problem, whip up enough code to rewrite that problem in conjunctive normal form, throw it at a SAT solver and manually interpret the output to see how it looks on the board. All in a few hours time. You wouldn't do this? Well, then you're not as [awesome as Felienne](https://github.com/Felienne/Quarto).

Felienne is a really enthusiastic speaker, explaining complex and theoretical concepts in an intuitive way. For me, it was throwback Thursday to my AI days at university, reminding me that what I learned there is still powerful knowledge. Being able to solve problems in the right domain can be a tremendous timesaver. I've heard this used at work, where a team had translated a specific business domain to the more abstract but well-known accounting domain, solve the problem there and translate back to the more specific domain. Something for Cegeka to present at the next DDD conference?

# The Precision Blade (Alberto Brandolini)
Alberto was talking about his experience with Event Storming, and the many advantages this technique offers beyond the obvious one, gathering of domain events. He is a pretty energetic speaker, gluing simple observations, obvious jokes, subtle jokes and deep insights all together in a waterfall (a whirlpool?) of words. Really fun to watch.

The one thing that struck me was Alberto's explanation for _why_ event storming works. The idea is that you get everything out of your head and into the visible shared space. Everything. It will be messy and unstructured and unsightly but it is necessary to do this before you can even hope to find the right problem to solve. Or, in a typical Brandolini way of speech: "you can't achieve simplicity without diving into chaos, just like putting all your clothes on the floor to sort them out."

# Amongst Models (Yves Reynhout)
Yves' talk was on his experience creating software to automate hospital appointment registration. It seems like a deceptively simple problem to solve. Why not just give them Google Calendar? Well, there are many good reasons, and Yves explains many of the needs and the corresponding changes in the model.

All of this is interesting, but for me it was hard to draw lessons from the talk. The main point Yves made was that models will evolve over time, as the needs of your users become clearer, but this is definitely not news. I'm not sure what else to take from this.

# Cognitive Cynefin: How Language and Bias Keep Us Complicated (Liz Keogh)
So far, the conference left me with a good feeling, but I still hadn't heard anything groundbreaking, anything that gave me a really new perspective. With the last talk of Thursday, that was about to change.

Liz starts out by talking about the metaphors we use to describe our work. We pick up work, we fit it in a sprint, we close it, we timebox. We talk about work as if it's physical. As if our work is a bunch of containers we open, close and move around. This metaphor keeps us from seeing the true diversity and implications our work has. It's not a bunch of tasks we move around in lanes. It's solving real problems for real people.

To drive this point home, Liz makes a detour into [the Cynefin Framework](http://lizkeogh.com/2012/03/11/cynefin-for-devs/) ("Say Kevin, like the name! Now say Ke-ne-vin! Great, your pronunciation is already better than mine") to give some more background on how we, as humans, approach different kinds of complexity in work. It was the first time I heard about this model, and wow, it makes a lot of sense.

I (and countless others) have seen a project derail by slicing it into stories before identifying the proper problem, have noticed TDD works well on some problems and not on others, have seen micromanagement over really simple things. Cynefin explains these things, and gives us a vocabulary to talk about how to approach problems with different complexity. This is invaluable to reach common understanding in a team about what you are doing and why you are doing it.

Eye-opening! I'm planning to learn more about Cynefin and spread the word in our company. Apparently our agile coaching division already uses the framework, which shows both that our coaches are awesome and that we need to work on internal trainings for these things. So much to learn from each other!

# Heuristics from the trenches (Cyrille Martraire)
"Talk to your domain expert", say the books. But how do I do that? Who do I talk to? What do I say?

Over the course of one hour, Cyrille delivers an insane amount of practical advise on do's and don't when trying to extract domain knowledge from your domain expert. Be genuinely curious. Don't just ask for features. Read the important books of your domain. Take notes of your talks.

One slide hit close to home: "The worst domain expert is the one whose expertise is built from the intricacies of the existing system." That person will not teach you any deep insights about the problem, just about one possible solution. It is better to have the person that has lived in the domain, that has seen the good and bad sides.

This casts the idea of an _analyst-proxy-as-stand-in-for-domain-expert_, used pretty often at Cegeka, in a dark shadow. These proxies are invaluable in our process, but _they are not the domain experts_. Cyrille confirms this after a question at the end of his talk: if you are in a strict analyst-gives-stories-to-developer process where the developers can't talk to domain experts, you are stuck doing something that is _not DDD_.

# Software Design and the Physics of Software (Carlo Pescio)
We often talk about our code as if it has certain properties. It is elegant, or rigid, or viscous. However, these properties always carry value connotations. Elegant code is good code. Viscous code is bad code. This is in stark contrast with actual physical properties in actual physical materials. Viscosity in physics isn't inherently good or bad. It's bad for the fly stuck in the honey, it's good for your engine coated in motor oil that doesn't instantly leak out. Studying these properties without any value connotation allows us to select the right material for the problem, which gave way to whole engineering disciplines.

As an example, Carlo explained how he tried to define _friction_ in the context of code, as a measure of how resistant code is to movement. The exact formulas aren't very interesting, but the main point is: the end result is just a numerical property. Sometimes you want it to be low (akin to minimize coupling), but sometimes you want it to be high. For example, you might want your code to resist movement to architectural layers where it doesn't belong.

This talk fascinates me endlessly. Yet, right now, it seems too abstract to be immediately useful. But imagine that we could engineer our code just like we engineer our materials - the right properties for the right job. I could see that happening, and I could see it as required studying material for future generations. I'm hopeful towards a more formal and rigorous approach to coding.

You can find more at [physicsofsoftware.com](http://www.physicsofsoftware.com/)

Bonus for this talk! At the end, Carlo was a bit disappointed no-one asked the question about "what all of this has to do with DDD", as this question had been asked many times before at other talks and he had prepared his answer. So he told us anyway, by way of an example he saw the previous day at the modeling with strangers. There was talk of a certain piece of data probably not being in the right bounded context, because "these two concepts change at different speeds", which he recast as another example of his theory about entanglement in software design. Carlo explained it as "this guy" making the 'different speeds' observation, with a whole "seemed like a really smart guy, I want him on my team." Little tingling sensation all over my body as he points towards me and I recall the start of the previous day...

# oDDs and enDs (Vaughn Vernon)
At the start of the day we were literally saying to each other how we had never heard of "oDD" or "enD" and how we couldn't follow all these new DDD trends. It still took us 5 minutes of the talk to understand that the title was just wordplay on "odds & ends" and "DDD". I think I missed a few of Vaughn's key points because I couldn't stop laughing after that realization.

Vaughn listed a lot of things that go wrong in our industry currently, before countering each with an antidote, some recent, some well-known. Big ball of mud? Bounded contexts! Developers as code monkeys? Show passion, and teach others! Huge joins? CQRS! The broader idea is that we need to be aware of these problems and work to their solutions, if we want to elevate the developers' situation from mere cost center to a productive partnership with the domain people.

At the end of the talk, Vaughn did a "one more thing" to announce a new book that's coming out soon: Domain Driven Design Destilled, a rather short explanation (about 250 pages) of what DDD is all about. It should be useful to get team members up to speed quickly, or to present to your management about why DDD is a good idea. There was some skepticism from the people around me, describing Vaughn as mostly a good salesman. I'll await the book, if it fulfills its promises, it's going to be a very welcome book to me.

# Enjoy Yourself Because You Can't Change Anything Anyway: Immutable Data in the Real World (Kelsey Gilmore-Innis)
Immutability is a good thing. Do more immutable objects. Kelsey talks about theory, history and practice of immutability in a quickfire and entertaining way.

Immutable objects are simpler because there's only one set of invariants and you can check them off at construction time. This helps with reasoning, debugging and performance. Further performance gains are possible because of memoization - caching the result of an operation, which is only possible with immutable results. Immutable values tend to compose better. Immutable values encapsulate a state at a fixed point in time, whereas a mutable object weaves state together with the time, in the 'complecting' sense that Rich Hickey explains in his talk [Simple Made Easy](http://www.infoq.com/presentations/Simple-Made-Easy).

Immutability was a big influence on object design in Smalltalk, with Alan Kay saying that "when you have a setter on an object, you have turned it into a data structure". This design philosophy is described in Alan's paper "The early history of Smalltalk", which I will have to read sooner rather than later.

Furthermore, immutability is at the core of functional programming. It's not coincidence that functional languages tend to favor immutable values. Functions can only be declared side-effect free on immutable data - how else can you see your function produces the same output for the same input, if input or output are not fixed? Furthermore, function composition can only work if input and output are immutable - the result of `(f.g)(x)` can only be well-defined if the result from `g(x)` is fixed for every `x` and can't change after calling `g`.

Kelsey of course explains this way better than me. Still, the insights above are things I knew but were made a lot clearer to me. So for me this was a really fun talk to watch, and I got a lot of notes out of it to explain why I like immutability. Unfortunately, a few of my colleagues got lost in the superfast explanations with pretty unrelated slides. I'll convince them yet.

# Symmetry in Design (Jim Coplien)
To fully appreciate this talk, you have to understand that the atmosphere at the conference up until now was pretty positive. Everyone was explaining what we needed to do, gave tips about DDD, was discussing their problems and insights. The general feeling is one of _"if the business wants it, we can DDD it. We just have to DDD hard enough"_.

I would like to draw the parallel to the fin-de-siècle feelings in physics. Around the year 1900, all of classical physics as we know it today (fields like mechanics, optics, thermodynamics) was discovered and was producing astounding good results. The general feeling was that we understood the main laws and only needed to clear up some little problems, maybe get some better methods for the harder problems like fluid dynamics.

However, some of those little problems (the ultraviolet catastrophe for black body radiation, thoughts experiments about the speed of light) turned out to be a little bit bigger than anticipated. The seeds for quantum mechanics and relativity were sown. A whole new form of physics was needed.

For those of dramatical inclination (like me), I will just throw this out there: maybe this talk will blossom in a revolution in DDD just like the one that happened in physics. Software development isn't a clear cut science like phsyics, however, so it might take a long time.

Jim starts out with some definitions of symmetry which seem technically correct to me (I speak as a Bachelor of Science in Mathematics) but then applies them in nebulous ways to software design, and object oriented programming in specific. Maybe I don't understand it, or maybe it's just nonsense, it doesn't really matter. Eventually, he makes the point that DDD, like many methodologies before, tries to solve problems in a hierarchical way. This is just not going to be good enough for every complex problem out there.

Jim then proposes we look into how DCI (Data-Context-Interaction) solves the problem for him in the telecom industry.

It was quite the rant, if you want to see it that way. But it seems the DDD community rather saw it as a warning and a challenge. At the end, Eric Evans stood up, and the two expressed interest in talking to each other. Out of these discussions could come an entire new understanding of the limitations of DDD, DCI and maybe software design at large.

You had to be there.

# A Decade of DDD, CQRS, Event Sourcing (Greg Young)
Greg talks about the good, the bad and the future of 10 years of event sourcing.

I don't have any event sourcing experience (sadface), so I hadn't encountered many of the things Greg talks about. When he explains his points, it all seems reasonable, but I can't really give any more insightful comments on this talk.

# The afterglow
_Truism alert!_ We get so used to doing the things we do everyday without questioning the bigger picture. And I think this is true even if you know about this phenomenon. Even if you actively try to fight it. There will always be things we don't see, ways of thinking we don't know, practices we keep doing without thinking. Any experience that shakes those habits up and causes reflection is bound to be good in the long run.

Cynefin, physics of software, DCI. A conference that raises questions you didn't even know you had is a great conference. And DDD Europe definitely did that for me.

On the other hand, these questions also provide a form of clarity. A realization of the _boundaries_ of DDD, the insight that DDD is a tool, not a cult. With that, the realization that _I_ should wield it as a tool, not spread it as a cult.

Thanks to the organization of DDD Europe for making all of this possible, and thanks to everyone that was there for actually making all of this happen.

I hope to see you again next year.

# Full disclosure / marketing note
_DDD Europe 2016 was in part made possible due to the sponsorship of Cegeka, my employer. Cegeka also payed for my ticket, as well as the tickets of about 10 other developers. If you would like to work for an employer that invests in its developers and the general software community, [Cegeka is always looking for passionate developers, analysts and coaches](http://www.cegeka.com/en/jobs)._
