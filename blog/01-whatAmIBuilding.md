# What am I building?

So what am I building? For the purpose of exploring what it takes - it doesn't really matter - the concepts should for a lot of different domains. Obviously, there will be a places where this does not work (e.g. high-frequency trading, highly parallelised machine learning). For this example, I want to choose a domain that:

- is common enough for everyone to understand,
- simple enough to not get in the way of exploring the goal - that is what does it take to build a web app from start to deployment
- but not so simple that we can't demonstrate the technology to its full potential - or at least close

I've chosen a web chat app this example. Everyone should have used something very similar multiple times, at its simplest is just sending some messages, but has a lot of potential avenues to explore more complicated ideas (editing and deleting messages, multi-media messages, private and public chats, etc). I am also building assuming that what I build will grow into something much larger (even though in reality it probably won't). Making a ToDo App is not difficult and so not really interesting. The challenge is to build for the future and different curve balls that it will throw.

## How?

I want to talk about some of the high level goals and some tools (or even classes of tools) that I think I will use to achieve them.

At the highest level I have 2 goals:

1. Build something from the ground up to build my personal knowledge across the entire width of providing a technical solution.
2. Provide a 'jumping-off' point for future green-field projects. If you do development as a job then you have stakeholders, and stakeholders like to see user-facing progress. Having a UAT environment is not user-facing and therefore may not be seen as important - until it suddenly is really, really important. I want to take a considered view of best way is to bootstrap a project without any of those pressures.

### North star(s)

I have a few guiding visions that I want to achieve and inform all my choices.

- **Fast feedback** - one of the biggest impediments to software development speed is feedback. There are a lot of ways to slow down feedback, from slow running tests, waiting for a pipeline to run to check something in dev environment, through to the [obligatory xkcd](https://xkcd.com/303/). There are a few ideas that I would like to try out here. More on this as I go.
- **The "-ilities"** - depending on exactly who you ask you will get a different number of "-ilities", but these are the dependability, maintainability, scalability of engineering. Basically they are the 'doing it right' part. Without the pressure of stakeholders I want to explore how to bake these in. I will note though that this is the part with the biggest tradeoffs. For example, choosing a micro-service architecture over n-tiered tier will improve scalability but impact manageability.

### Tools

#### Domain driven design

Domain driven design (DDD) provides a framework for communicating both within engineering and to outside (e.g. to business owners or customers). By providing some categories of types of code and advocating for ubiquitous language across all levels of the business we can aid in clear communication. Some categories include (non-inclusive):

- Entities: containers for data that have a key (e.g. uuid, email address) as well as the rules above
- Aggregates: groups of entities that must adhear to invariants and the rules to enforece them (e.g. a chat in status of _X_ must have users that satisfy _Y_ condition)
- Services: define the business rules for orchestrating the changes of or retrieval of entities and agregates

While using DDD may seem way over-kill for a simple messaging app I want to explore with a mindset of building something much larger.

#### Hexagonal and Clean Architecture

Hexagonal (aka Ports and Adaptors - IMO a much better name) is an architectural pattern that mandates that IO and core business logic are separated. Ports are descriptions of how to do IO (e.g. interfaces) and adaptors are how this description is implemented. I have seen plenty of examples where ports and adaptors are not properly segregated and how this can cause problems - for example, the only way to set up an end-to-end test is to use the UI to get the data in the right starting state, and then run the test. This causes very slow running tests. By enforcing ports and adaptors from the start my expectation is that I can avoid several pitfalls - e.g. replacing the persistence layer for fast running tests, adding a CLI or scripting presentation layer instead of serving everything via REST.

Clean arcitecture takes this idea and moves it one step further and pairs nicely with DDD and P&A. Instead of creating a hard boundary betwen the core application and IO, we instead create several hard boundaries between different parts of the domain.

#### Nx

One technology that I would like to explore to achieve this is Nx. It is a typescript technology to enable partitioning of a large project into small reusable chunks, build (and possibly deploy them) independently, but still be able to compose them into applications. Nx analyses the dependencies between packages and builds (or tests, cleans, etc...) exactly what needs to be built and nothing more - everything else is cached. Decomposing like this from the start is cheap (as opposed to refactoring out a service from a jumble of code), and helps to promote good boundaries. With this foundation it can be deployed as a monolith, series of microservices or several other architectural styles.

The next level-up of Nx that I would like to try is Nx-Cloud. Where Nx caches results of builds and tests etc locally, Nx-Cloud caches these things from _anywhere_. Team-mate Tom makes some changes to libray XYZ and run the tests to prove they are good changes - those test results are cached in the cloud. So when I pull, I pull not just the tests that he wrote but also the fact that these tests pass and Nx knows I do not need to run them. This results in a massive boost to `test all` before running a deployment or pushing some code.

My hypothesis is that Nx will drive at two key points that I want to get right:

1. Dev-experiernce - only building and testing what has changed is going to make the feedback cycle very quick
2. The dependency graph feature is going to aid in enforcing strong boundaries between in hexagonal and clean architecture.
