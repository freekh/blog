## Resolution Engine...
The resolution engine is the heart of any dependency manager and it is the resolution engine of Adept that changes the game somewhat compared to it's brethren.

Before we jump into the details, my mom always told me it is always good to state your goals (Ok, she didn't say that, but it is still good practice):

  - Knows which libraries/dependencies/packages works together
  - As fast as it can be
  - Flexible - so that it fits your work-flow
  - Distributed - so you can work offline and even if adepthub.com goes offline
  - Reliable - i.e you **always** get the same results
  - Historical - **We want Adept to be the Louvre of code!**. Haha, no just kidding (although it would be nice)... We want Adept to be able to build code that is really, really old (think 70 years)...
  - Updatable - everything can be safely updated (meta-data, artifacts)

Holy Long List, Batman! Not to worry - we will only be covering the first goals now: knowing what works together and speed. We will go through how to reach the other goals in later posts.

### Attributes
Adept allows module/library authors to define how users can resolve their module. 
In most dependency managers (excluding ORB) you only have one attribute which is the 'version'.

As an example: the (awesome) <a href="http://akka.io/">Akka</a> authors could express that users can resolve on **version** (i.e. 2.2.1), but also on **binary compatible versions** (i.e. 2.2).
By doing this the author of Akka is communicating to their users, saying *"you can use any variant of Akka that has the same binary version  (2.2.1 or 2.2.0)"*.

This is the opposite of what you usually see where it is the **users** of a library **that has to know** how a library is compatible with itself.
The problems with this approach, beyond the fact that you are forcing your users to keep updated on your versioning scheme, is that it might change at any given time.
A library author might for example involentary introduce a binary incomptiblity and then either your versioning scheme is broken or you have think about some hack.
Neither approach is particularly pleasant, which is why we think that Adept's approach is better.

Attributes goes beyond versions as well, because the actual **matching** in Adept *redonkulously* simple and **only does equality**. If you want binary version 2.2, that is what you getting.
In the same way, you can get libraries with "license" equal to "apache2" or on "QA department seal of approval" "approved".

Adept also supports attributes with several values. Imagine you require binary version 7 and binary version 6 of Java for example. 
Since Java 7 is compatible with Java 6, the authors of Java (yeah, *those* guys), could encode this Java's attributes creating a variant of Java that has an attribute "binary-version" with values: 7,6,5,... 

### Variants
Up till now we have been mentioning variants, but we haven't actually said what we mean with it - shame on me! 

A variant in Adept is typically what you would call a version in most other dependency managers: a variant defines an Id (something like: com.typesafe.akka/akka-actor), some attributes and some requirements.


### Resolution
TODO

### {Over, Under}-constrained
TBD

### Configurations

### SBT integration status
There is a lot of talk here, but sometimes it is nice to see. 

### Think you are better, Punchy?

We think there is lots of room to make the resolution enigne much, much faster. But we have hidden it and want you to discover it!

Think you can do better? <a href="https://github.com/adept-dm/adept/blob/master/src/main/scala/adept/resolution/Resolver.scala">Have a look</a>, hack away, send us a PR and make us proud.

Need help to get started - create an issue!


