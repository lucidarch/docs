---
title: "Routing"
date: 2020-10-18T12:49:47Z
draft: false
weight: 7
hide: ["toc", "header"]
head: "<hr />"

---

Out of the SOLID principles - of which we are disciples - we've taken Single Responsibility seriously and created components that define the responsibilities beyond an MVC's *controller* or any other modern application's entry point, be it a *route/request* or a *command*. Which is where it is loose and chaos builds its nest. Within a *controller* we can do **anything**, and whichever architecture we follow (or don't) we can still create a mess due to not having a specific guideline that helps contain the chaos.

Here we define the responsibility of each component starting from MVC and moving through Lucid to keep codebase organised, defined and understandable at a glance, supporting whichever design patterns we decide to adopt:

{{% figure alt="Laravel Lucid" src="/media/images/laravel-lucid.png" %}}

## Router

A router is like a door that one can open to enter a room. A door does not concern itself with what the room contains or what its purpose is. It is simply a door. However, sometimes we have some security pass for the door to open, so a router can be configured to include components that perform security checks and other forms of entrance preparation.

{{% figure alt="Lucid Routing" src="/media/images/routing/routing.png" width="800px" %}}

**Responsibility:** Expose a feature from the application over HTTP, routing the request to the corresponding *controller* method.

## Kernel

Use Lucid Units (Feature, Job, Operation) to implement your middleware functionality. Code that you may use anywhere else in your application can be easily shared in

{{% figure alt="Lucid Routing Kernel Middleware" src="/media/images/routing/kernel-decision.png" width="800px" %}}

This space is best to:

- Define the URL you want your application to allow entries through.
- Define the *controller* method that should handle the request and serve the Feature.
- Perform pre-flight / middleware work such as request authorisation, preparation [does not include input validation].
