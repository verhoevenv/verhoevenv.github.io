---
layout: post
title:  "Today I Learned: Java 8's Optional.map is actually flatmap"
tags: til java8 optional monads pragmatism
---
During a recent developer meeting I showed how Java 8's `Optional` can simplify code in if-else structures that default to a fallback value. We went from the following code:
{% highlight java %}
if (location != null) {
    Office office = officeRepository.findFor(location.getZipCode());
    if(office != null) {
        return office;
    } else {
        return retrieveDefaultOffice();
    }
} else {
    return retrieveDefaultOffice();
}
{% endhighlight %}
to the following:
{% highlight java %}
return Optional.ofNullable(location)
        .flatMap(loc -> Optional.ofNullable(officeRepository.findFor(loc.getZipCode())))
        .orElseGet(() -> retrieveDefaultOffice());
{% endhighlight %}
First of all, if this transformation is not obvious to you, please think about it for a while, maybe try it out in a Java IDE. The code is linked at the end of the article. Further on, I'll assume you understand what's going on.

I would argue for the merits of this code because it is an obvious decrease in lines of code, bringing the essential forwards while reducing boilerplate code. I would also argue there is a decrease in complexity, because it is less straining to follow the flows of happy-path and failure-recovery, compared to scanning the start and end of if-blocks. However, this is only simpler if you _know_ about what `flatMap` does for `Optional` types. I'll hand-wave this argument with [the words of Venkat Subramaniam](https://youtu.be/llGgO74uXMI?t=1056):

> Simple is not necessarily familiar

Which is why I'm trying to teach functional programming idioms to my team anyway. :)

However, on trying to explain what the `flatMap` does and why we need it, I stumbled upon an unexpected feature of Java. You might have already discovered it when experimenting with the codebase. It turns out the following is functionally equivalent, as proven by the tests we had written before we tried the whole refactoring:
{% highlight java %}
return Optional.ofNullable(location)
        .map(loc -> officeRepository.findFor(loc.getZipCode()))
        .orElseGet(() -> retrieveDefaultOffice());
{% endhighlight %}

The documentation of `Optional.map` specifies

> If a value is present, apply the provided mapping function to it, and if the result is non-null, return an `Optional` describing the result. Otherwise return an empty `Optional`.

It makes total sense for Java. The class invariant is that an `Optional` can never hold a `null` value, so either the map should fail or it should return an empty `Optional`. The second seems the better choice.

In stricter functional programming languages (aka Scala), this doesn't happen. A Scala `Option` is perfectly capable of holding `null`, and map will put it in there without raising an eyebrow. I think this happens because monadic composition[^1] is a better tool to handle the situation and `null` is less of an issue in Scala code anyway, because its use is highly discouraged.

The following is a test highlighting this difference which you can also find [in the codebase for this article](https://github.com/verhoevenv/java8-optional/blob/master/src/test/scala/com/github/verhoevenv/java8/optional/EssentialOptionalDifferenceSpec.scala). `s2j` is a conversion function between Scala's and Java's function-type.
{% highlight scala %}
"A java Optional" should "map null to Optional.empty" in {
  val javaOptional = Optional.of("a value")

  val result = javaOptional.map(s2j(_ => null))

  result.isPresent shouldBe false
}

"A scala Option" should "map null to Some(null)" in {
  val scalaOptional = Option("a value")

  val result = scalaOptional.map(_ => null)

  result.isDefined shouldBe true
  result shouldBe Some(null)
}
{% endhighlight %}

I think this is a great example of pragmatism from Java's development team. We won't be having monads or equivalent things in Java in the foreseeable future, so let's just make life easier for everyone.

You can find the code on GitHub if you want to play around with it: <https://github.com/verhoevenv/java8-optional>. The Java codebase with the three alternatives is there and the tests that prove the implementations are equivalent. I've thrown in some alternatives in Scala, including the literal translation of the map-based Java code (which behaves differently in Scala) and a monadic alternative.

[^1]: I will refrain from explaining monads per [the monad tutorial fallacy]( https://byorgey.wordpress.com/2009/01/12/abstraction-intuition-and-the-monad-tutorial-fallacy/), if you really need your fix there is [plenty of choice](https://wiki.haskell.org/Monad_tutorials_timeline)
