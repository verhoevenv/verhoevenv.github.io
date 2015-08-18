---
layout: post
title:  "From Guava's FluentIterable via StreamSupport to Java 8 Streams"
tags: java8 java streams functional guava
---
If you're programming in Java, you probably noticed the recent move towards streams, lambdas and a more functional style of writing code, greatly facilitated by Java 8's stream API and anonymous function syntax. Great if you're doing a clean new project on Java 8, but what about existing codebases?

Our team is currently on Java 7, but looking to move towards Java 8 in the course of the next year. To get into the habit of things, we've started using the concepts of functional programming in Java 7. We have been using [Guava's FluentIterable](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/FluentIterable.html) for this, which I would heartily recommend to anyone under similar circumstances.

However. The Guava API is somewhat lacking in some parts, with the team [taking the stand](https://github.com/google/guava/issues/218#issuecomment-91661751) they will not improve the API because Java 8 is the more logical upgrade path. Meanwhile, each call to the Guava libraries we add to our code will cost us time later on when we want to remove the dependency. So teams that _are_ going to migrate to Java 8 (like us!) don't have it in their best interest to keep using Guava.

What's the alternative? We're currently experimenting with using [StreamSupport](http://sourceforge.net/projects/streamsupport/), the backported version of the Java 8 stream API to Java 6 and 7.

The rest of this post will explain our findings, and some strategies to help the migration. I will keep this post updated when I uncover new information. Of course, feel free to remark on things I have missed!

#Contents
{:.no_toc}

* kramdown will replace this with table of content
{:toc}

#The theory, or "Why do we have to jump through all these hoops?"

##Java 8 vs Java 7
First of all, Java 8 brought a lot of improvements that you simply will have to miss in Java 7. This means that code will not be as concise under Java 7. In some cases, you might need completely different workarounds. Please note that I'm not aiming to list all Java 7/8 differences, only those relevant to the streams API.

###Lambdas
The most obvious one is lambdas for functional interfaces:

{% highlight java %}
//Java 8
List<String> strings = Arrays.asList("abc", "b", "de");
Collections.sort(strings, (a, b) -> (a.length() - b.length()));
System.out.println(strings);

ExecutorService pool = Executors.newCachedThreadPool();
pool.submit(() -> 42);
{% endhighlight %}


{% highlight java %}
//Java 7
List<String> strings = Arrays.asList("abc", "b", "de");
Collections.sort(strings, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return (a.length() - b.length());
    }
});
System.out.println(strings);

ExecutorService pool = Executors.newCachedThreadPool();
pool.submit(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        return 42;
    }
});
{% endhighlight %}

Much of the boilerplate can be hidden by extracting the anonymous inner class implementation to another class, possibly with its own factory method, which has the bonus advantage of forcing you to name the otherwise anonymous piece of code.

{% highlight java %}
//Java 7
List<String> strings = Arrays.asList("abc", "b", "de");
Collections.sort(strings, byLengthReversed());
System.out.println(strings);

ExecutorService pool = Executors.newCachedThreadPool();
pool.submit(new AnswerCalculator());


//The definitions below could be moved to other files
private static Comparator<String> byLengthReversed() {
    return new ReverseLengthComparator();
}

private static class ReverseLengthComparator implements Comparator<String> {
    @Override
    public int compare(String a, String b) {
        return (a.length() - b.length());
    }
}

private static class AnswerCalculator implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return 42;
    }
}
{% endhighlight %}

###Method references
This would take a long time to explain in depth, so I will just leave you with an example:

{% highlight java %}
//Java 7
Supplier<List<?>> listFactory = new Supplier<List<?>>() {
    @Override
    public List<?> get() {
        return new ArrayList<>();
    }
};{% endhighlight %}


{% highlight java %}
//Java 8
Supplier<List<?>> listFactory = ArrayList::new;
{% endhighlight %}

Oracle provides an [indepth overview](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html).

###Generics
Java 8 comes with [improvements in type inference for generics](http://openjdk.java.net/jeps/101) related to generic parameters. Note that it seems like the proposed type inference on method chains did not make it. The example below demonstrates the change that was implemented.

Given the following class definition:
{% highlight java %}
class Enhancer<T> {
    static <Z> Enhancer<Z> withSparklyEffects() {/*..*/ return new Enhancer<>();}
    static <Z> Enhancer<Z> addLayer(Z object, Enhancer<Z> effect) {/*..*/ return new Enhancer<>();}
    T result() {/*..*/return null;}
}
{% endhighlight %}

Sample usage looks like:
{% highlight java %}
//Java 7
private String sparkles() {
    return addLayer("A pony", Enhancer.<String>withSparklyEffects()).result();
}
{% endhighlight %}

{% highlight java %}
//Java 8
private String sparkles() {
    return addLayer("A pony", withSparklyEffects()).result();
}
{% endhighlight %}

###Default interface methods
Java 8 adds the ability to define default methods on interfaces.
{% highlight java %}
public interface Die {
    int roll();
    default Die plus(int modifier) {
        return new Die() {
            @Override
            public int roll() {
                return Die.this.roll() + modifier;
            }
        };
    }
}
{% endhighlight %}
{% highlight java %}
public class SixSidedDie implements Die {
    public static SixSidedDie d6() {
        return new SixSidedDie();
    }
    @Override
    public int roll() {
        return new Random().nextInt(6) + 1;
    }
}
{% endhighlight %}
{% highlight java %}
public static void main(String... args) {
    Die modifiedD6 = d6().plus(2);
    System.out.printf("1d6+2 => %d\n", modifiedD6.roll());
}
{% endhighlight %}

##StreamSupport

###Package differences
StreamSupport lives in its own package `java8.util.stream`, which resembles the Java 8 package `java.util.stream`. This makes it possible for the two to co-exist, but it makes removing the StreamSupport dependency a little bit tricky once you switch to Java 8. Often, you will be able to simply remove the `8` from the package name, but this won't always work. Some examples of this are below.

###Starting a stream
StreamSupport is not entirely able to recreate the Java 8 API, for one simple reason: you can't change the existing Java 7 implementations of the Java standard library. In particular, and most crucially, the [`java.util.Collection.stream()`](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html#stream--) method can't be added to Java 7.

StreamSupport solves this by moving this method to a wrapper method:

{% highlight java %}
//Java 8
private static long countWithStream(List<?> list) {
    return list.stream().count();
}
{% endhighlight %}


{% highlight java %}
//StreamSupport
private static long countWithStream(List<?> list) {
    return StreamSupport.stream(list).count();
}
{% endhighlight %}

Note that this is the same approach as Guava:

{% highlight java %}
//Guava
private static long countWithStream(List<?> list) {
    return FluentIterable.from(list).size();
}
{% endhighlight %}

If you want to remove the dependency on StreamSupport after introducing Java 8, you will have to change all these calls. We haven't tried this yet, but I am pretty certain we will be able to solve this problem with [IntelliJ's structural search and replace](https://www.jetbrains.com/idea/help/structural-search-and-replace.html).

###Predicate methods
Java 8 defines Predicate composition as default methods on the Predicate interface: [`Predicate.and`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html#and-java.util.function.Predicate-), [`Predicate.or`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html#or-java.util.function.Predicate-), [`Predicate.negate`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html#negate--).

{% highlight java %}
//Java 8
private static Predicate<Integer> between5And10() {
    return greaterThan5().and(lessThan10());
}

private static Predicate<Integer> greaterThan5() {
    return integer -> integer > 5;
}

private static Predicate<Integer> lessThan10() {
    return integer -> integer < 10;
}
{% endhighlight %}

Streamsupport moves these to a java8.util.functions.Predicates class, and makes them wrapper methods.

{% highlight java %}
//StreamSupport
private static Predicate<Integer> between5And10() {
    return Predicates.and(greaterThan5(), lessThan10());
}

private static Predicate<Integer> greaterThan5() {
    return new Predicate<Integer>() {
        @Override
        public boolean test(Integer integer) {
            return integer > 5;
        }
    };
}

private static Predicate<Integer> lessThan10() {
    return new Predicate<Integer>() {
        @Override
        public boolean test(Integer integer) {
            return integer < 10;
        }
    };
}
{% endhighlight %}

Probably another job for structural search and replace.

##Wrappers
The Guava concepts of [Function](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Function.html) and [Predicate](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicate.html) are, of course, the same concepts as those in Java 8, and thus StreamSupport. While it is preferable to refactor an existing Function/Predicate to the new interface, sometimes there are a few too many usages to immediately change all depending code. It should however be no surprise we can translate from one implementation to the other. We use the following wrappers:

{% highlight java %}
import java8.util.function.Function;
import java8.util.function.Predicate;

public class Guava2Java {

    private Guava2Java() { }

    public static <T> Predicate<T> j(final com.google.common.base.Predicate<T> guavaPredicate) {
        return new Predicate<T>() {
            @Override
            public boolean test(T t) {
                return guavaPredicate.apply(t);
            }
        };
    }

    public static <T, R> Function<T, R> j(final com.google.common.base.Function<T, R> guavaFunction) {
        return new Function<T, R>() {
            @Override
            public R apply(T t) {
                return guavaFunction.apply(t);
            }
        };
    }
}
{% endhighlight %}

We choose the short name to be minimally intruding:

{% highlight java %}
private static List<String> makeAList() {
    return StreamSupport.stream(Arrays.asList("abc", "def", "bubble"))
            .map(j(guavaVersionOfToUppercase()))
            .collect(Collectors.<String>toList());
}

private static com.google.common.base.Function<String, String> guavaVersionOfToUppercase() {
    return new com.google.common.base.Function<String, String>() {
        @Override
        public String apply(String s) {
            return s.toUpperCase();
        }
    };
}
{% endhighlight %}

#The practice, or "How do I..."

##Collect to a list

{% highlight java %}
//Guava
private static List<Integer> createList() {
    return FluentIterable.from(Arrays.asList(4, 5, 6))
            .filter(greaterThan5())
            .toList();
}
{% endhighlight %}

{% highlight java %}
//StreamSupport
private static List<Integer> createList() {
    return StreamSupport.stream(Arrays.asList(4, 5, 6))
            .filter(greaterThan5())
            .collect(Collectors.<Integer>toList());
}
{% endhighlight %}

{% highlight java %}
//Java 8
private static List<Integer> createList() {
    return Arrays.asList(4, 5, 6).stream()
            .filter(greaterThan5())
            .collect(Collectors.toList());
}
{% endhighlight %}

Note that you need the generic type in the StreamSupport version.

##Collect to SortedSet
{% highlight java %}
//Guava
private static SortedSet<Integer> createSet() {
    return FluentIterable.from(Arrays.asList(4, 5, 6))
            .filter(greaterThan5())
            .toSortedSet(Ordering.natural());
}
{% endhighlight %}

{% highlight java %}
//StreamSupport
private static SortedSet<Integer> createSet() {
    return StreamSupport.stream(Arrays.asList(4, 5, 6))
            .filter(greaterThan5())
            .collect(AdditionalCollectors.<Integer>toSortedSet());
}

private static class AdditionalCollectors {
    private static <T> Supplier<SortedSet<T>> sortedSetSupplier() {

        return new Supplier<SortedSet<T>>() {
            @Override
            public SortedSet<T> get() {
                return new TreeSet<>();
            }
        };
    }

    private static <T> Collector<? super T, ?, SortedSet<T>> toSortedSet() {
        return Collectors.toCollection(AdditionalCollectors.<T>sortedSetSupplier());
    }
}
{% endhighlight %}

{% highlight java %}
//Java 8
private static SortedSet<Integer> createSet() {
    return Arrays.asList(4, 5, 6).stream()
            .filter(greaterThan5())
            .collect(Collectors.toCollection(TreeSet::new));
}
{% endhighlight %}

Generalizing these solutions to also accept a comparator is an excellent exercise for the reader.

##Combine predicates
{% highlight java %}
//Guava
public Predicate<Integer> isOddNatural() {
    return and(isPositive(), not(isEven()));
}
{% endhighlight %}

{% highlight java %}
//StreamSupport
public Predicate<Integer> isOddNatural() {
    return and(isPositive(), negate(isEven()));
}
{% endhighlight %}

{% highlight java %}
//Java 8
public Predicate<Integer> isOddNatural() {
    return isPositive().and(isEven().negate());
}
{% endhighlight %}

#Conclusion
Functional programming in Java 7 is not great, but it is workable. Upgrading to Java 8 should not be a problem, but removing all dependencies and cleaning all the boilerplate code might prove to be a hard job. StreamSupport can help, but you can still expect to have some work.
