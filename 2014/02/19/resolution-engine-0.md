## Adept's Resolution Engine
The resolution engine is the heart of any dependency manager and it's the resolution engine of Adept that changes the game somewhat compared to its brethren.

Before we jump into the details, my mom always told me it is good practice to focus on goals not design (or was it finish my meals?).

Anyways, here goes - Adept shall:

  - Know how things work together - i.e. which libraries/dependencies/packages are **compatible** or not
  - Be as **fast** as possible
  - Be **flexible** - so that it fits your work-flow
  - Be **distributed** like Git - so you can work offline and even if adepthub.com goes offline
  - Be **reliable** - i.e you **always** get the same results
  - Be **historical** - **We want Adept to be the Louvre of code!** Haha, no just kidding (although that does sound nice)... We want Adept to be able to build code that is really, really old (think 70 years)...
  - Be **updatable** - everything can be safely updated (meta-data, artifacts)

**Holy Long Lists, Batman!** Not to worry - we will only be covering the first two goals now: knowing what works together and speed. We will go through how to reach the other goals in later posts.

### Attributes
Adept allows module/library authors to define how users can resolve their module. 
In many/most dependency managers you only have one attribute which is the *'version'*.

As an example: the (awesome) <a href="http://akka.io/">Akka</a> authors can express that users can resolve on **version** (i.e. 2.2.1), but also on **binary compatible versions** (i.e. 2.2) of Akka.
By doing this the author of Akka is communicating to their users, saying *"you can use any variant of Akka that has the same binary version  (2.2.1 or 2.2.0)"*.

This is the opposite of what you usually see where it is the **users** of a library **that has to know** how a library is compatible with itself.
The problems with this approach, beyond the fact that you are forcing your users to know and be updated on your versioning scheme, is that it might change at any given time.

A library author might for example involuntary introduce a binary incompatibility and then either your versioning scheme is broken or you have to hack your way around it (and remove/replace the release).

Neither approach is particularly pleasant, which is why we think that Adept's approach is better.

Attributes can be other things than just versions as well, because the actual **matching** in Adept *redonkulously* simple and **only does equality**. If you want binary version 2.2, that is what you're getting.
In the same way, you can get libraries with *"license"* equal to *"apache2"* or have only the ones with *"QA department seal of approval"* equal to *"approved"*.

These attributes can also have several values. Imagine you require binary version 7 and binary version 6 of Java for example. 
Since Java 7 is compatible with Java 6, the authors of Java (yeah, *those* guys), could encode this Java's attributes creating a variant of Java that has an attribute "binary-version" with values: 7,6,5,... 

If you are like me and prefer to see how it looks like in code, Attribute looks like this:

```scala
case class Attribute(name: String, values: Set[String])
```

### Variants & The Domain Model
Up until now we have been mentioning variants, but we haven't actually said what we mean with it - shame on me! 

A Variant is the "atom" of Adept, in the sense that it's sort of the basic unit. They contain the information needed to find all of their dependencies and artifacts (files, jars, ...).  Specifically, a Variant defines an Id (something like: com.typesafe.akka/akka-actor), it's Attributes, a bunch of Artifacts and some Requirements.
In many other dependency managers, a Variant would be a "version" of a module, but in Adept it would not make sense because you can have many Variants that have the same "version" attribute.

Here's how the code looks like (we will cover the rest of the classes below):

```scala
case class Id(value: String) extends AnyVal
case class Variant(id: Id, attributes: Set[Attribute], artifacts: Set[ArtifactRef], requirements: Set[Requirement])
```


#### Requirements
A Requirement is an Id, linking it to a set of Variants, and some Constraints which filters out the Variants of the given Id.
As an example: if we want Akka, binary-version 2.2, the Id is "com.typesafe.akka/akka-actor" and the Constraints are: "binary-version" = ["2.2"].

Again, for us code junkies, it looks like this:

```scala
case class Constraint(name: String, values: Set[String])
case class Requirement(id: Id, constraints: Set[Constraint])
```

#### Artifacts
The last part of Adept's core domain model are the Artifacts.
Artifacts represents the actual files that you want as a result: the jars, zips, ... what ever.

Since we are talking about actual files, the metadata information also contains the locations where the files are located.

The observant reader would notice that Variants use ArtifactRef(erences), and not Artifacts. The reason is that the uniqueness/hash of a Variant is calculated by the sum of its parts (Id, Attributes, Artifacts, Requirements) and we do not want a Variant to considered different if the location of an Artifact changes.
The Artifacts and ArtifactRef(erences) are linked through the unique hash (SHA-256) of the file it represents.

Since we know what we want (the hash of the Artifact/ArtifactRef/file), we get 2 freebies:

1. We can **NOT** care where the file comes from. This means Adept can use **multiple locations** to a file. In addition to **speeding** up downloads, this makes the resolution more resilient to failure (because you have multiple sources). Currently locations are simply URLs, but in the future we will expand this so that somebody hosting Adept can easily change and manipulate locations (we will use properties for this). Our evil, cunning plan also involves adding more protocols than http, such as BitTorrent. Mwahahaha
2. We never ever, EVER, have to download an artifact we already have, because we know for a fact whether we have the one we want or not (I think it is rather strange that Ivy and Maven doesn't do this as well but...). I want to say this makes Adept fast, but really it is the dependency managers that constantly re-downloads stuff that are slow so let's just say that Adept is faster :)

After an Artifact has been downloaded (and verified), it's put in Adept's cache where it's stored with the hash as the filename. This is nice for Adept, because it's easy to verify the integrity of files and it is easy to look them up. It is not so nice for you though :_(, since the hashes make it hard to debug the classpaths for example, in addition some IDEs require file names to be in a certain format. That is why ArtifactRefs can have filenames. This way, a build tool can use the filename property to copy or link the cached files to separate directories with meaningful filenames.

```scala
case class ArtifactRef(hash: Hash, attributes: Set[Attribute], filename: Option[String])
case class Artifact(hash: Hash, size: Long, locations: Set[String])
```


### The Rules Of The Game
Now that we covered the boring bits (domain model I am looking at you!), let's have a look at how resolution works - Yeah!

The first rule of Adept is that you only have one variant per Id.
The second rule of Adept is: YOU ONLY HAVE ONE VARIANT PER ID. Ehem, sorry - got a bit carried away there...

Anyways, so the resolution starts off in the <a href="https://github.com/adept-dm/adept/blob/7829bf51d43f1b3dfa9897ebb3af3392f2e87d93/src/main/scala/adept/resolution/Resolver.scala#L126">Resolver</a> with a bunch requirements. Adept will then take the Ids and look up all the Variants that matches the constraints.
If there is a **unique Variant for a give Id**, the resolver continues and looks at all of the requirements of that Variant.

Resolution is successful when every (transitive) Id has exactly one Variant. There is beauty in simplicity and we think that this is as simple it can be (in fact it is implemented on ~215 LOC including comments).

If no Variants are found for one or more Ids, we say that resolution is **over-constrained** (too... many... con... straints).

On the other hand, if there is more than Variant for one or more Ids, resolution is **under-constrained** (not enough Constraints).

#### Under-constrained
When resolution is under-constrained, there is still a chance to resolve because there might yet be a set of Variants that is unique to the set of input Requirements. Therefore, Adept will try out all the different ways the valid Variants of all the under-constrained Ids, starting with only 1 Id, the 2 Ids etc etc.  If there is an **unique** set of Variants that resolves a set of Ids (i.e. for Ids A, B, C, there are exactly 1 variant), Adept considers the graph to be resolved. 
We call this process *implicit resolve*.

An example can be found in the test <a href="https://github.com/adept-dm/adept/blob/7829bf51d43f1b3dfa9897ebb3af3392f2e87d93/src/test/scala/adept/resolution/ResolverTest.scala#L265">"basic under-constrained path find"</a>.

To understand what's going on, imagine you require a variant A, which requires C binary-version 2.0 and any variant of B. Now, there are 2 Variants of B both requiring C. However, one requires (B 1.0.0) binary-version 1.0 and the other (B 2.0.0) binary-version 2.0. That means that there is really only one Variant B that can be used: 2.0.0, and we are resolved.

#### Implications 1: Conflict Resolution
Now you might be thinking: "Gosh, that sounds nice and simple (except perhaps that under-constrained part) and all, but most of my dependency graphs depends on more than exactly one version per module - so how the hell does Adept handle this?". 

The answer lies in the way Variants are stored, which is in a versioned Git repository.

By convention, there will be **maximum** one **compatible** Variant per commit. So for a given commit, you could for example have 2 variants with version = 1.0.1 and 1.2.0. If there is a new release you would bump the binary-compatible one (1.0.1 is replaced with 1.0.2). 

When creating a Resolver you have to specify which commits of the repositories you want to use. There is a separate file per Variant which tells Adept all the repositories it needs again. 

The take-away here though is that Adept will use the *latest* commit for a repository that is specified twice.

Here, Adept shifts the responsibility of knowing what is considered to be the *latest* or *best* from the dependency manager to the *author*.
This opens up a lot of doors: an author can for example rollback or deprecate Variants without messing up things for people who already are using the old commit.
All in all, we think that is pretty cool!

There is more to say about this, but this blog post is too long already and the implementation is still in progress so you will have to wait till next blog post.

#### Implications 2: Knowing What Works With What
Now you should be able to see how Adept can know how libraries are compatible, but if you couldn't (you bozo!) let's take an example to illustrate.

Let us say that Akka requires the Scala (as in the programming language) library to compile (it really does).
The thing is that not all Scala library Variants are binary compatible :( 
Therefore, there would be multiple Variants of the Scala library to choose from, each with their different binary-versions: 2.10, 2.9.3, 2.11, ... (Scala is **NOT** backwards compatible so only one value per binary-version Attribute).

Imagine you also want to use Scala (and why wouldn't you?!?!), which means you have a requirement on a binary version of Scala.

The first thing to note now is that Adept will be **over-constrained** if you specifically want a binary version of Scala that is different than the Akka Variant which is chosen. 
***In other words, Adept fails (in a nice manner) if you try do something wrong***

The other (cooler thing) is that providing there is only one Akka Variant that is using a particular Scala library binary version, Adept will resolve because of the implicit resolve. This can be really nice for a language such as Scala (which is not backwards compatible) because you could compile different versions of your modules without having to implement your own weird naming scheme - ***Adept will find the right Variant if there is one.***

Note: there is absolutely nothing particular here with the Scala library: you can use the same line of reasoning on any framework or library which have binary compatible matrices.

### TL;DR;
Adept **rocks**! :P

It is the last dependency manager you (or your build tool) will ever need, because it's resolution engine is simple yet powerful and does everything Ivy, Maven et al. does and a bunch of stuff that they can't do such: resolving (and discovering) compatible libraries being one thing.


The resolution engine is "finished" and we are/I am working on how repositories are handled next week. After this way we will follow up with another post on why Adept is reliable. 
In the future we will also follow up with a post that shows how Adept can natively handle Ivy Configurations (Maven Scopes).

Also, have a look at the screencast below to see how it looks like when I tried out the sbt plugin:

<div style="margin:auto; text-align:center;">
<iframe width="400" height="315" src="//www.youtube.com/embed/5WLSyJBKfbU" frameborder="0" allowfullscreen></iframe>
</div>
*We wanted to find some nice music for it, but couldn't (ok, I am lying we were just too lazy) so please play your favorite tunes in the background ;)*


### Think you are better, Punchy?
We think there is lots of room to make the resolution engine much, much faster. Do you grok the resolution engine and think you can do better?

Have a look at <a href="https://github.com/adept-dm/adept/blob/7829bf51d43f1b3dfa9897ebb3af3392f2e87d93/src/main/scala/adept/resolution/Resolver.scala">the resolver</a> and its <a href="https://github.com/adept-dm/adept/blob/7829bf51d43f1b3dfa9897ebb3af3392f2e87d93/src/test/scala/adept/resolution/ResolverTest.scala">test</a>, hack away, send us a PR and make us proud.
(We don't have any perf test harness yet so if you think you are better, you will have to find a way to prove it!)

Need help to get started or think we are wrong? We are really, really, really nice guys (ehem) so don't worry and create a <a href="https://github.com/adept-dm/adept/issues">GitHub issue</a>!


