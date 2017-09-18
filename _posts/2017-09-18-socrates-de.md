---
layout: post
title:  "Lessons from SoCraTes Germany 2017"
tags: socrates craftsmanship testing community conference notes
teaser-pic: /assets/2017-08-28-socrates-de/socrates2017_logo.png
excerpt: >
    <p>Germany is where the first <a href="https://www.socrates-conference.de/">SoCraTes unconference</a> took place, back in 2011. It is, to my knowledge, the SoCraTes conference with the most attendees, which naturally correlates with the broadest knowledge and amount of sessions. As an aspiring software crafter I couldn’t resist wanting to participate in 2017’s conference. Some of the things I learned or contributed to can be found below. Notes and topics are ordered roughly by importance for myself as judged arbitrarily by myself. They are not in order of the time at which the sessions occurred.</p>
---
![socrates_2017_logo]{:.teaser-pic}

Germany is where the first [SoCraTes unconference](https://www.socrates-conference.de/) took place, back in 2011. It is, to my knowledge, the SoCraTes conference with the most attendees, which naturally correlates with the broadest knowledge and amount of sessions. As an aspiring software crafter I couldn't resist wanting to participate in 2017's conference. Some of the things I learned or contributed to can be found below. Notes and topics are ordered roughly by importance for myself as judged arbitrarily by myself. They are not in order of the time at which the sessions occurred.

# Craft, not craftsmanship
Firstly, notice how I said aspiring software _crafter_, not craftsman, in the previous paragraph. This is deliberate. The unconference happened less than a month after the infamous [Google memo](https://en.wikipedia.org/wiki/Google%27s_Ideological_Echo_Chamber) was released, and the debate was still raging heavily. It probably still is whenever you read this. Personally, I don't want to join this discussion, as I could only speak from privileged position, but I'm happy to follow along and try to adapt my habits to create a welcoming and safe space for everyone.

One of the issues that was discussed is the inherently gendered word crafts_man_ship. [Changing craftsmanship to craft is a logical step I'm supporting](https://twitter.com/Singsalad/status/896640390241103872). It'll be a hard one to change, though, as the craftsmanship word is ingrained deeply by now. Please correct me when I make mistakes.

Thanks everyone for having this discussion, but especially thank you Franzi for the role you are playing in it.

# Property based testing
It has long been my view that property based testing is a good complement to unit testing. Unit testing is great to guide you to a solution which adheres to a few examples. Property based testing is great at keeping you away from the vast space of solutions that are almost but not exactly doing what you want. I've picked up a few new tricks in [Jan Stępień](https://twitter.com/janstepien)'s session.

One way to drive code with property based testing is this: the existing test doesn't change (so you just test one property), but you **iteratively increase the generator in complexity**, so that it generates more classes of examples. For example, say you have a property of trees. You'd first implement the generator to only generate leaves. Make the smallest change to fix the code. Then add trees of depth one. Make the smallest change to fix the code. Add recursive trees. Fix the code. Boom, TDD transformations. And none of that 'how do I know I have enough examples' because _your generator creates them all_.

Property based tests are awesome whenever you are mapping between models that should be equivalent. Jan's example was that of a URL structure which should correspond to AND-ed filters in the code. Think like 'the color should be red' AND 'the price should be less than 50 EUR' AND 'the locale is be-nl', next to ```/color-red/price-less-50-EUR/be-nl/```. Another example is persisting to database and retrieving again, though that might be a slow thing to implements lots of tests for. With fancy words, there is a bijection between the models, and a property of a bijective function $$f$$ is that there is an inverse function $$g$$ and that for all $$x$$ we have $$g(f(x)) = x$$. If I store the object and retrieve it again I have the same object. If I do ```filter.toUrl().asFilter()``` I have the same filter again. You get the idea. Sometimes we're not interested in the inverse function so this doesn't apply, but sometimes we need to guarantee the transformation works in both directions. **If your model is sufficiently complex, it's pretty much impossible to cover all combinations of these transformations with unit test**, but property based tests will be able to give you more than enough certainty about your code. In Jan's example, the code broke down for a combination of a Polish locale and a price filter, the price filter code assumed all prices were in EUR but the Polish locale works with złoty.

At the core of property based tests lies a difficult decision: what property do I test? Two cases seem ideal:
* You can test a property where the result of the unit under test can be abstracted away, usually by applying some transformation to it so that it conforms to an aspect of the input
* You can compare answers to an oracle that knows the answer

The second thing seems strange (if we can derive the answer, then why are we writing the code?) but can come up in a technical way (the remote DB should act the same as an in-memory DB for these operations) or even a business way (the new code should answer the same as the legacy code).

A last thought: event sourcing might be a good match for property based testing, as one property is that 'given a bunch of events generated by my system, my downstream systems should be able to handle them'. Business will find a way to generate combinations of events that you never thought possible, so better tighten the feedback loop and let your test system generate a whole lot of combinations.

# Is legacy unavoidable?
Have you ever had the feeling that everything you work on eventually becomes legacy code? We've been thinking a bit about this and we've come to the conclusion that this is pretty much unavoidable. Yes, you might be applying best practices, splitting up things, using the latest frameworks, continuously refactor your code. There are many factors at play pushing you back, though.

One of the biggest ones we identified is that our frameworks become outdated so fast. As the software community, we invent newer and better ways of working all the time. Eventually the framework you are using is outdated, and new team members won't even have the faintest idea on how to work on your system without a huge introduction and lots of cursing and apologizing from your part. I would estimate the time at which this happens between 5 years for stable technologies (say Java frameworks) and 1 year for that crazy front-end world, probably even less on the bleeding edge.

What to do? Not much. **Keep the life cycle of your system in mind**, event when starting. How long do you expect this system to last? Be realistic about this, 'forever' means infinite costs. **Optimize for replaceability, not maintainability**: it's often easier to recreate loosely coupled parts of the system with the newest insights, than try and polish whatever rotten parts you already have. This is not necessarily a call for microservices but they do seem to help a whole lot in that regard.

Refactoring, good documentation, proper tests, all these things can definitely help to delay legacy, and I'd argue often beyond the expected life duration of a project. Not doing those things will definitely ensure you have legacy code even while still very actively working on it, which your clients will not appreciate. So this is also not a call to drop those practices because it's unavoidable anyway.

Thanks [Cédric Crevier](https://twitter.com/CedricCrevier) for bringing this discussion to SoCraTes. It did change my perspective!

# On hiring
[Raimo Radczewski](https://twitter.com/rradczewski) hosted an excellent session examining the [Medium.com engineering hiring handbook](https://medium.engineering/mediums-engineering-interview-process-b8d6b67927c4), which is well worth a read if you are doing hiring interviews. Their approach is well-balanced and comes with great examples. I love that they explicitly grade 'curiosity' as a desirable attribute in a candidate.

# Clojure and dev workflows
I was very impressed in seeing the workflow Jan Stępień follows when working on Clojure code. It's valuable to try to shorten the feedback loop while coding. Small applications that compile and deploy quickly are one step. Unit testing and TDD is another step. But none of this comes close to the speed you can gain if you add a REPL in the loop. It's truly amazing to see how productive Jan is when he is live editing a running system in a REPL, and I would like to experiment with this myself. I also wonder if JShell, the REPL that will come with Java 9, will cause shifts in development patterns.

# Other tidbits
* The clean architecture and hexagonal architecture are everywhere these days and almost seem like required knowledge by now. I'm only at the _shu_ phase so I can't agree or disagree, but I need a lot more practice as it seems here to stay.
* Awareness of Domain Driven Design is growing, but we should remain vigilant that the core message doesn't get lost. DDD is about working together with business on a shared language and domain model. It's not about having the most microservices or prescribing the colors of the post-its in the mandatory event storming session. The architecture is a side effect. The tools are not the methodology.
* Blockchains are the new monads.
* The cost of an Ethereum transaction increases by its size. Therefore, it makes sense to minimize the size of a transaction. One way to do this is by only including proof of ownership of some other data, for example a hash of a signed document uploaded to a third party. This is also a way to solve use cases where privacy of data is important. Thank you [Caroline](https://twitter.com/LineyJane) for hosting this discussion!
* Corda! It's a distributed ledger (but not a blockchain) where a private network of actors can sign transactions privately between them, but with the same proof-of-transaction guarantees as your next-door blockchain. It's written in Kotlin! And it has a bunch of [really awful-awkward-awesome banker jokes](https://github.com/corda/corda/blob/8f0ea714b39ee9c92f76fcd4b699bd217636eba4/node/src/main/kotlin/net/corda/node/internal/NodeStartup.kt#L280) from which is randomly selects one at startup. Thanks to Rocío Molina Atienza for hosting a session on this!
* Idris is magic, and [Type Driven Development with Idris](https://www.manning.com/books/type-driven-development-with-idris) is an example of great technical writing. Once again, thanks to Jan (who else, right?) for his session on this.
* Orion isn't even visible in summer. I've been confusing it with Cassiopeia all my life.
* On that note, anything goes at SoCraTes. We've gone stargazing (thank you [Andreas Gnyp](https://twitter.com/andreasgnyp), there were a lot of board games, there was dancing and volleyball and some woke up at 6 AM (or stayed awake?) to watch the Mayweather vs McGregor boxing match.
* And related to that, I contributed to a developer Fiasco ruleset. You can find the rules [on GitHub](https://github.com/haslers/fiasco-dev-edition). Thanks, [Toni Rheker](https://twitter.com/offbyoni), for hosting this session. We'll be sure to play this at SoCraTes BE!
* I always used to say that I don't know any German except for the word _Bahnhof_ from travelling by train in Germany once. Turns out that's an actual German idiom: _Ich verstehe nur Bahnhof_, meaning something like "I don't understand anything about it".
* A lot of friction is reduced by the [awesome scheduling app](https://openspaceorg.github.io). It's so great to use. No more half-readable and hopefully not-outdated photos of the marketplace. Seriously, any open space where the marketplace is not visible from all locations should probably jump on this. Thank you, [Jason Peper](https://twitter.com/jason_peper), for your work on this!


[socrates_2017_logo]: /assets/2017-08-28-socrates-de/socrates2017_logo.png
