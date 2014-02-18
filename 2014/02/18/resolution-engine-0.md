## Resolution Engine...
The resolution engine is the heart of any dependency manager and it is the resolution engine of Adept that changes the game somewhat compared to it's brethren.

Before we jump into the details, my mother always told me it is always good to state your goals (Ok, she didn't say that, but it is still good practice):

  - Knows which libraries/dependencies/packages works together
  - As fast as it can be
  - Flexible so that it fits your workflow
  - Distributed - so you can work offline (even if we go offline)
  - Reliable - i.e you always get the same results
  - Historical - of course I want it to be historical in the sense that it will be remembered with time, but what we mean here is that: Adept is able to build a version of my code that is old (like 70 years)...
  - Updatable - everything can be safely updated (meta-data, artifacts)

The resolution engine is responsible for the first two points: knowing what works together and speed. We will go through the other points in later posts.

### Attributes
Adept allows module/library authors to define how users can resolve their module. 
In most dependency managers (excluding ORB) you only have one attribute which is the 'version'.

As an example: the (awesome) <a href="http://akka.io/">Akka</a> authors could express that users can resolve on **version** (i.e. 2.2.1), but also on **binary compatible versions** (i.e. 2.2).
By doing this the author of Akka is communicating to his users and saying *"you can any variant that has the same binary version works (2.2.1 or 2.2.0)"*.

This is the opposite of what you usually see where it is the users of a library that has to know how it is compatible with itself.
Obviously, this is sad. 

### Variants
TODO2

### Requirements
TBD

### {Over, Under}-constrained
TBD

### Configurations

### SBT integration status
TBD

### Think you are better than us, Punchy?

We think there is lots of room to make the resolution enigne much, much faster. 

Think you can do better? <a href="https://github.com/adept-dm/adept/blob/master/src/main/scala/adept/resolution/Resolver.scala">Have a look</a>, hack away, send us a PR and make us proud.


