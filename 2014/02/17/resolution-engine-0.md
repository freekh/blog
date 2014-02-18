# the Resolution Engine...
The resolution engine is the heart of any dependency manager and it is the resolution engine of Adept that changes the game somewhat compared to it's brethren.

Before we jump into the details, my mother always told me it is always good to have goals (Ok, she didn't say that, but I hear it is good practice). With that in mind these are some of the goals of Adept:
- Knows which libraries/dependencies/packages works together
- Flexible so that it fits your workflow
- As fast as it can be
- Reliable - i.e you always get the same results
- Historical - of course I want it to be historical in the sense that it will be remembered with time, but what we mean here is that: Adept is able to build a version of my code that is old (like 70 years)...

The resolution engine is responsible for the first two points. We will go through the other points in later posts.

## the Attributes
Adept allows module/library authors to define how users can resolve their module.
As an example: the author of the (awesome) <a href="http://akka.io/">Akka library</a> could say you can resolve on **version** (i.e. 2.2.1), but also on **binary compatible versions** (i.e. 2.2).
By doing this the author of Akka is communicating to his users, saying that you can change any 2.2.x version with the other and it will work.
The way it is now: 





We still think there is lot's of room for optimization in the resolution enigne. If you think you can do better - <a href="https://github.com/adept-dm/adept/blob/master/src/main/scala/adept/resolution/Resolver.scala">have a look</a>, hack away and send us a PR.

Du må gå igjennom: multi-attribute og custom attribute og hvorfor -> varianter -> resolve -> under/over-constrained -> scanning ->Configurations -> next article: conflict resolution.
