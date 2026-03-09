---
title: "Micro • Monolith"
date: 2020-11-07T22:25:15Z
draft: false
weight: 9
hide: ["header"]
---

Often times we start building our applications with little forecast on how they will evolve over time, they may reach a steady maintainance stage with small tweaks here and there,
or undergo pivots that require a high degree of flexibility to not clutter and cause an abundance of technical debt.

In both cases, the familiarity with the system's architecture is the guaranteed constant to protect our codebase from going out of bounds.

---

Lucid is provided in two variants - **Micro** and **Monolith** - to accompany a wide range of projects at all stages
and provide the versatility of moving between one and the other, which is attributed to the composability of the Lucid stack,
where you could use the units you need with no commitment to others - yet like lego bricks they all fit together when the need for them arises.

## Micro

A single-purpose project that contains a moderate amount of functionalities that fit in a similar context.

It contains the fundamental units of the Lucid stack: Domains, Features, Operations and Data, complementing the Laravel framework:

```text
app
├── Console
│   └── Kernel.php
├── Data
    └── Models
├── Domains
├── Features
├── Operations
├── Exceptions
│   └── Handler.php
├── Http
│   ├── Controllers
│   ├── Kernel.php
│   └── Middleware
├── Policies
└── Providers
    ├── AppServiceProvider.php
    ├── AuthServiceProvider.php
    ├── BroadcastServiceProvider.php
    ├── EventServiceProvider.php
    └── RouteServiceProvider.php
```

### When To Use Micro?

Micro is suitable for most projects, including quick prototypes that are ought to become actual products at some point which is
where Lucid comes in to reduce the technical debt imposed over time. Or API projects that are meant to be organized for scale.

Also as the name suggests they are best suitable for Microservices, where you would have multiple instances of Laravel • Lucid Micro,
each representing a microservice in your system.

## Monolith

A multi-purpose project that exposes functionality in groups that are best separated into their own contexts physically,
mirroring their logical separation and encapsulation. To achieve this we use [Services]({{<ref "/services">}}) -
the differentiating factor of Monolith in the Lucid stack.

Having services means we're expanding the functionalities that are available at the framework level, into a
contextualised division as an extension. Enabling us to replicate the framework structure into sub-structures that contain
routes, controllers, views, resources, tests, features and operations of their own, yet all connected together by sharing code
using Domains and Data.

There are several ways to segregate services, below we mention two of the most common ways: **Multifaceted** and **Multifunctional**.

### Example: Multifaceted Project

Is a project that has multiple facets like Api, Admin, Web; where each would have their own way
of authenticating users, authorizing requests and responding to them in corresponding formats.

{{<columns>}}

```text
app
├── Console
├── Data
    └── Models
├── Domains
├── Exceptions
├── Foundation
├── Http
├── Policies
├── Providers
└── Services
    ├── Admin
    ├── Api
    └── Web
├── artisan
├── bootstrap
├── composer.json
├── composer.lock
├── config
├── database
├── package.json
├── phpunit.xml
├── public
├── resources
├── routes
├── server.php
├── storage
├── tests
└── webpack.mix.js
```

<--->

```text
app/Services/{Admin,Api,Web}
├── Console
│   └── Commands
├── Features
├── Operations
├── Http
│   ├── Controllers
│   └── Middleware
├── Providers
│   ├── {Admin,Api,Web}ServiceProvider.php
│   ├── BroadcastServiceProvider.php
│   └── RouteServiceProvider.php
├── Tests
│   ├── Features
│   └── Operations
├── database
│   ├── factories
│   ├── migrations
│   └── seeds
├── resources
│   ├── lang
│   └── views
└── routes
    ├── api.php
    ├── channels.php
    ├── console.php
    └── web.php
```

{{</columns>}}


### Example: Multifunction project

Is a project with a wide range of functions that are best separated in logic due to the degree of difference in the areas they serve,
like an application that has Chat, Product Management, Ecommerce (web UI) and an API.

The structure and usage is the same as multifacet, they only differ in concept and their logical separation.
As illustrated below, the naming of the services is different between the projects yet their internals
are exactly the same. What matters is to choose whichever approach works best for your case and maintain it consistently.

{{<columns>}}

```text
app
├── Console
├── Data
    └── Models
├── Domains
├── Exceptions
├── Foundation
├── Http
├── Policies
├── Providers
└── Services
    ├── Chat
    ├── Ecommerce
    ├── ProductManagement
    └── Api
├── artisan
├── bootstrap
├── composer.json
├── composer.lock
├── config
├── database
├── package.json
├── phpunit.xml
├── public
├── resources
├── routes
├── server.php
├── storage
├── tests
└── webpack.mix.js
```

<--->

```text
app/Services/{Chat,Api,Ecommerce,ProductManagement}
├── Console
│   └── Commands
├── Features
├── Operations
├── Http
│   ├── Controllers
│   └── Middleware
├── Providers
│   ├── {Chat,Api,Ecommerce,ProductManagement}ServiceProvider.php
│   ├── BroadcastServiceProvider.php
│   └── RouteServiceProvider.php
├── Tests
│   ├── Features
│   └── Operations
├── database
│   ├── factories
│   ├── migrations
│   └── seeds
├── resources
│   ├── lang
│   └── views
└── routes
    ├── api.php
    ├── channels.php
    ├── console.php
    └── web.php
```

{{</columns>}}

{{%notice info%}}
{{<icon name="fa-hat-wizard fa-2x">}}&nbsp;These are only two approaches to services, but there certainly are other creative ways you could benefit from them. Go crazy!
{{%/notice%}}

### When To Use Monolith?

Monolith is suitable for projects that have been pre-defined with a large set of functionalities across multiple sections
and are known to grow vertically in functionality as well as horizontally in more areas.
However this is not always easy to guess from the start, especially when having to identify the services involved,
which is why it will be easy to start with Micro and then move to Monolith when the project has gotten better definition.

Also, Monolith projects have been witnessed to grow beyond the initially anticipated scale, to then be split into Microservices
themselves, and Lucid happens to be perfect for the case where each of the Services is moved to its own Micro instance.
