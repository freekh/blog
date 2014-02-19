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

**Holy Long Lists, Batman!** Not to worry - we will only be covering the first two goals now: knowing what works together and speed. We will go through how to reach the other goals in later posts.

### Attributes
Adept allows module/library authors to define how users can resolve their module. 
In many/most dependency managers you only have one attribute which is the 'version'.

As an example: the (awesome) <a href="http://akka.io/">Akka</a> authors could express that users can resolve on **version** (i.e. 2.2.1), but also on **binary compatible versions** (i.e. 2.2).
By doing this the author of Akka is communicating to their users, saying *"you can use any variant of Akka that has the same binary version  (2.2.1 or 2.2.0)"*.

This is the opposite of what you usually see where it is the **users** of a library **that has to know** how a library is compatible with itself.
The problems with this approach, beyond the fact that you are forcing your users to keep updated on your versioning scheme, is that it might change at any given time.
A library author might for example involentary introduce a binary incomptiblity and then either your versioning scheme is broken or you have think about some hack.
Neither approach is particularly pleasant, which is why we think that Adept's approach is better.

Attributes can be other stuff than versions as well, because the actual **matching** in Adept *redonkulously* simple and **only does equality**. If you want binary version 2.2, that is what you getting.
In the same way, you can get libraries with "license" equal to "apache2" or on "QA department seal of approval" "approved".

Adept also supports attributes with several values. Imagine you require binary version 7 and binary version 6 of Java for example. 
Since Java 7 is compatible with Java 6, the authors of Java (yeah, *those* guys), could encode this Java's attributes creating a variant of Java that has an attribute "binary-version" with values: 7,6,5,... 

If you are like me and prefer to see how it looks like in code, Attribute looks like this:

```scala
case class Attribute(name: String, values: Set[String])
```

### Variants & The Domain Model
Up until now we have been mentioning variants, but we haven't actually said what we mean with it - shame on me! 

A variant is the "atom" of Adept, in the sense that is sort of the basic unit. They contain the information needed to find all of it's dependencies and artifacts (files, jars, ...). In many other dependency managers, a variant is called a version of a module, but in Adept it would not make sense because you can have many variants that have the same "version" attribute.

Specifically: a variant defines an Id (something like: com.typesafe.akka/akka-actor), it's attributes, a bunch of artifacts and some requirements.

Here's the code (we will cover the rest of the classes below):

```scala
case class Id(value: String) extends AnyVal

case class Variant(id: Id, attributes: Set[Attribute], artifacts: Set[ArtifactRef], requirements: Set[Requirement])
```


#### Requirements
A requirement is an Id linking it to a module and some constraints which restricts the possible variants of the module.
As an example: if we want Akka, binary-version 2.2, the Id is "com.typesafe.akka/akka-actor" and the constaints are: "binary-version" = ["2.2"].

Again, for us code junkies, it looks like this:

```scala
case class Constraint(name: String, values: Set[String])

case class Requirement(id: Id, constraints: Set[Constraint])

```

#### Artifacts
The last part of the domain model in the core of Adept is the Artifacts.
Artifacts represents the actual files that you want as a result: the jars, zips, ... what ever.

Since we are talking about actual files, the metadata information also contains the locations where the files are located.

The obeservant reader would notice that Variants uses ArtifactRef(erences), and not Artifacts. The reason is that the uniqueness/hash of a Variant is calculated by the sum of it's parts (id, attributes, artifacts, requirements) and we do not want a Variant to considered different if the location of an Artifact changes.
The Artifacts and ArtifactRef(erences) are linked through the unique hash (SHA-256) of the file it reprensents.

Since we know what we want (the hash), we get 2 benefits:

1. We do not care where the file comes from. This means Adept can safely have multiple locations to a file. In addition to speeding up downloads, this makes the resolution more resiliant to failure. Currently locations are simply URLs, but in the future we will expand this so that somebody hosting Adept can easily change and manipulate locations (we will use properties for this) and also add more protocols such as BitTorrent.
2. We never ever have to download an artifact we already have, because we know exactly what we have. (I think it is rather strange that Ivy and Maven doesn't do this as well but...)

After an Artifact has been downloaded (and verified), it is put in Adept's cache where it is stored with filename. This is nice, because it is easy to verify the integrity of files and it is easy to look them up. It is not all a danse on roses though, since the hashes make it hard to debug the classpaths for example, in addition some IDEs require qualified names. That is why ArtifactRefs can have filenames. This way, a build tool can use the filename to copy or link the cached files in a separate directory.

```scala
case class ArtifactRef(hash: Hash, attributes: Set[Attribute], filename: Option[String])

case class Artifact(hash: Hash, size: Long, locations: Set[String])
```


### The Rules Of The Game
Now that we covered the boring bits (domain model I am looking at you!), let's have a look at how resolution works - Jej!

The first rule of Adept is that you only have one variant per Id.
The second rule of Adept is: YOU ONLY HAVE ONE VARIANT PER ID. Ehem, sorry - got a bit carried away there...

So the resolution starts off in the <a href="https://github.com/adept-dm/adept/blob/7829bf51d43f1b3dfa9897ebb3af3392f2e87d93/src/main/scala/adept/resolution/Resolver.scala#L126">Resolver</a> with a bunch requirements. Adept will then take the Ids and look up all the Variants that matches the constraints.
If there is a **unique Variant for a give Id**, the resolver continues and looks at all of the requirements of that Variant.

Resolution is successful when it every transitive Id has exactly one Variant.



### Configurations
In my mind there is absolutely no reason 

### Artifacts

### SBT integration status
There is a lot of talk here, but sometimes it is nice to see. 

### Think you are better, Punchy?

We think there is lots of room to make the resolution enigne much, much faster. But we have hidden it and want you to discover it!

Think you can do better? <a href="https://github.com/adept-dm/adept/blob/master/src/main/scala/adept/resolution/Resolver.scala">Have a look</a>, hack away, send us a PR and make us proud.

Need help to get started - create an issue!


