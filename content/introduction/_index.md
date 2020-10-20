---
title: "Introduction"
date: 2020-10-14T09:27:24Z
draft: false
weight: 1
hide: ["header"]

---

Software architecture to build services that last with minimum technical dept; based on proven principles and practical guidelines.

---

## The Motive

Developing large-scale applications has always been easy to think about but when it comes to implementation lots of questions arise especially when working in a collaborative team where every decision is peer-discussed and made over coffee or beer on a napkin. Some of these discussions go around fundamental subjects such as:

- Where should this good piece of art code reside (folder, file, class, etc.)?
- How should we apply these sets of design patterns to implement this feature?
- How can you describe the underlying architecture that is running… EVERYTHING!?

Asking such questions is a brilliant start, knowing that every project or product we start working on starts from a mere idea - a spark - to help fulfil a human function. Nevertheless, deep in the back of our heads we know that at some point in the future it will grow and we don't want it to grow on us, we'd rather have it grow with us. Especially in an agile culture we want the pieces of our applications to be easily interchangeable with the least friction possible and keeping technical debt to a minimum.

What is an architecture anyway? But **a pattern of connected structures cooperating according to a set of guidelines and principles to serve their purpose.**

From that sentence, Lucid will be the practical **set of guidelines** **and principles** to build backend services that last.

The following questions will also help ensure that you should incorporate Lucid in your current or upcoming project:

- Have you ever been onboarded to a code that is completely custom to the point that it
- Have you seen a Laravel project, even though it has been provided with Laravel's impeccable simplicity, somehow managed to be ruined by endless whirlpools of huge controllers and deeply dependant and connected classes?

These situations and many more are best described in the following picture, which analogy is our application in a few months time.

{{% figure width="600" src="/media/images/composition-intertwined.png" %}}
{{% caption "Technical debt due to intertwined object relationships in a legacy application." %}}

If you would like the image above to become something as below...

{{% figure width="600" src="/media/images/composition-clear.png" %}}
{{% caption "Clear and precise composition of objects." %}}

{{% alert info %}}
{{% icon name="fa-archway" size="16px" %}} You're in the right place!
{{% /alert %}}

## Lucid • Laravel

Within controllers and commands, our application is prone to disorder due to the high degree of autonomy in MVC. Lucid fills that gap to complete the chain of Laravel's MVC framework.

The relationship between Laravel and Lucid is that they complete each other. Lucid will be the bond between the application's entry points and the components that do the actual work (classes, interfaces, design patterns etc.) securing code from meandering in drastic directions.

![Laravel & Lucid](/media/images/laravel-lucid.png)

In fact, the very first announcement of Lucid to the public was in Laracon EU 2016 two years after its in-house use. See video below if you would like to watch the announcement:

{{% youtube wSnM4JkyxPw %}}

{{% caption "Abed Halawi - The lucid architecture for building scalable applications - Laracon EU 2016" %}}

## The Architecture

At a glance...

![Lucid Architecture Overview](/media/images/architecture-overview.jpg)

### Framework

Provides the “kernel”, request/response lifecycle, dependency injection and the core functionalities for the Foundation to build on. Knows nothing about the application logic. This is [Laravel](https://laravel.com).

### Foundation

Provides support to the concrete components of the architecture (services, domains and data) by extending the framework with fundamental (abstract) classes; due to that, and the fact that the concrete components will only deal with the foundation, any change at the framework level will be transparent and have the least effect possible on the application overall and as a result **the Lucid architecture grants a reduced amount of [technical debt](https://en.wikipedia.org/wiki/Technical_debt) in the case of a drastic update/upgrade or even migration to another framework**.

Examples of what the Foundation might include:

- `FoundationController` to be extended by the controllers in services
- Entity class that extends a base class such as an Eloquent model
- Decorators for validation, authentication and other fundamental yet shared functionality

### Services

They implement the features but do not implement the logic of the feature themselves, their responsibility is to compose Jobs and Operations from Domains. Think of a Job as a step if the feature was a user story; and an Operation is a group of steps that are often executed together to accomplish a single purpose.

{{% panel %}}
**Example**: a project consisting of an API and an Admin where you would implement each in its own scope for separation of concerns yet use common code between them using Domains and Data.
{{% /panel %}}

**Terminology**

The terminology of a service is used in the form of “Our application provides data through an API service, it also has an Admin service with a user interface to perform advanced data manipulation procedures.

**Hierarchy**

The hierarchy of services and their components is as described in the pyramid below, given the example of an HTTP request:

![Lucid Architecture Hierarchy](/media/images/architecture-hierarchy.png)

### Domains

They implement the business logic and expose them through jobs, they are isolated and MUST NOT inter-communicate [no domain calls another]. Their objective is to categories the jobs that accomplish unit-level tasks in our application within a definite scope.

{{% panel %}}
**Example**: in a task management app there’s the `User` domain and `Task` domain each having its own validators, generators, builders, factories, utility classes etc. and they utilise the data layer to perform data-specific procedures in addition to Jobs that are (favourably) the only way to communicate with domains. Potential Jobs in a task management app in the `Task` domain could be: `SaveTaskJob`, `MarkTaskCompletedJob`, `AssignTaskToUserJob` , where each of these is a class of its own. More on Jobs in Definitions.
{{% /panel %}}

{{% notice info %}}
We suffix Job classnames with `Job` for readability and easier navigation, also extending the PSR-2 coding standard of suffixing interfaces with `Interface` and traits with `Trait`.
{{% /notice %}}

### Data

Owns the data (duh!), which means models (entities), repositories, value objects, enums... you name it. You may be asking why separate the data layer from the domains layer? aren’t they related? The answer is of course they are! Yet domains remain to be isolate, self-satisfied and no cross-domain relationships exist while data is organically related; if we included them in domains we would’ve ended up working with cross-domain dependencies which defies the idea of domains being self-contained.

Another reason for having a separate data layer is the trend of the 21st centry: 💫**data science**✨ which makes this layer the place for your statistics and analytics algorithms, **a central layer for the elements of data**.

{{% panel %}}
**Principle**: Domains, Jobs and Operations are isolated classes with single responsibilities, they perform one thing and one thing only;
no inter-domain, inter-job or inter-operation dependencies should exist whatsoever.
{{% /panel %}}

Congratulations for making it thus far! It may feel a little rough though, and that's why there is more to these concepts for you to dig into.
