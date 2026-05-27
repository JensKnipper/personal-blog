---
draft: true
---

Test strategy

Due to different experience levels in testing software, we aim to use a simple solution.
The requirements are:
 - it is simple to apply to existing code
 - through a short glance at the class it should be clear how it should be tested
   - this helps when testing code, especially with fewer experience
   - it is easy for a reviewer to check if the test strategy has been applied correctly

## Why defining a test strategy in the first place?
https://martinfowler.com/articles/practical-test-pyramid.html
In the past the test pyramid used to be the reference for good testing. 
It avoided anti patterns like the lollypop, the ice cream cone or the hour glass, which suffer from bad test coverage or lots of long running tests.
In more current architectures, like microservices, there might be a lot more integrations to other services than business logic.
Thus the pyramid changes it's shape and may look like a house or a diamond.
Though the pyramid itself is seen as outdated, the test strategy behind it is still valid.
   
## A simple test strategy
Sticking to the test pyramid implies a strategy which is defined in the following tweet:
https://twitter.com/noahsussman/status/836612175707930625
A quick summary would be:
 - whenever possible, write a unit test
 - when it cannot be unit tested, write an integration test
 - when it cannot be unit test or integration tested, write an e2e test
 - when it cannot be unit tested or integration tested or e2e tested, test manually
This is more or less the inverted test pyramid as a "funnel" to filter all the bugs.
Of course this is architecture dependent and depending on yours the test pyramid or the "funnel" might look different, eg. it might contains more or other steps.

But how do we ensure everyone always knows how to test his or her changes?
This is actually the tricky part and requires some experience.
But we can mitigate this by structuring our code properly and defining some simple rules. 
For example if something resides in this package, write this kind of test. 
It's as simple as it gets.
The only tricky part is how do we structure our code so the testing becomes self explainatory?

## hexagonal architecture

Original article here:
https://alistair.cockburn.us/hexagonal-architecture/
A nice graphic from here
https://medium.com/idealo-tech-blog/hexagonal-ports-adapters-architecture-e3617bcf00a0

A structure where this can be found is the hexagonal architecture or ports and adapters pattern.
The idea behind the architecture is to extract external dependencies from your domain logic using ports and adapters.
This allows independent testing of your services.
An adapter can be is usually something where your application is in contact with the outside. 
This can be something where you application gets triggered like a rest endpoint, a cron job or a cli interface. 
The other option is something where your application triggers another service like a database, another rest endpoint or the file system.
Our own services should not invoke the adapters directly, but only their ports. 
The ports are interfaces which are implemented by the adapters.
This way our service is more decoupled from external services.

We defined the following rules for us to structure our code:
 - create an adapter
  - when application is triggered from the outside
  - when external service is triggered
 - domain logic calls adapters through ports

It is actually quite easy to set up, especially if you already have some kind of structure in your code. 
For example Spring nudges you to have a class design which resembles some kind of layered architecture.
That way all you have to do is create some packages, move classes around and create some interfaces.
Done, you now transfered your code into hexagonal architecture.

We did not fully implement the hexagonal architecture, but deliberately left some parts out.
For us it is a way to simplify our testing strategy, the better code structure is just a plus.
Because of that we left out the DTO mapping from in the adapters to the model. 
Though we are aware of the greater risk for data leaks and the stronger encapsulation.

To check the integrity of our architecture we use ArchUnit. 
Though ArchUnit has no official support we can just check for the onion architecture and leave some parts out, as the onion architecture also uses the ports and adapter pattern.
We are aware that ArchUnit is just restricting us from simple violations of the architecture boundaries. 
It is still possible to circumvent the tests.

## The final strategy
We are basically using the simple test strategy of the test pyramid for our tests.
To make it easy to spot gaps in our tests we structure our code according to the hexagonal architecture.
In the end we came up with these simple rules for tests
 - everything in the "domain.model" package should not contain any logic, thus should not be tested
 - everything in the "domain.service" package should be unit tested
  - we should use test fakes which implement the ports to imitate external services in a very simple way
 - everything in the package "adapter" should be integration tested
 - if it's an application, like a microservice, start it up and shoot a request to one or more endpoints
 - if it was not able to test something, eg. because you have communication through the database (ouch!), write an e2e test
 