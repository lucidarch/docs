---
title: "Principles"
date: 2020-11-09T18:05:52Z
draft: false
weight: 3
hide: ["toc", "header"]
---

{{%panel%}}
## Features shall serve a single purpose

Favour creating as many of them as you wish rather than complicating one.
{{%/panel%}}

{{% panel %}}
## Jobs shall perform a single task

No job should do two things at a time, it will only get confusing the more you do it.
{{% /panel %}}

{{% panel %}}
## Domains shouldn't cross

When within a domain, strive to preserve the boundaries by not using functionality from other domains.
If encountered a case where you should, consider Foundation, Operations and Features by rethinking your design.
{{% /panel %}}

{{% panel %}}
## Features shall not call other features

Run as many jobs and operations as you like, but never a feature.
{{% /panel %}}

{{% panel %}}
## Jobs shall not call other jobs

This is your business logic, keep it concise by avoiding nesting hell.
{{% /panel %}}

{{% panel %}}
## Operations shall not call other operations

Run as many jobs as you like, but never any other unit.
{{% /panel %}}

{{% panel %}}
## Controllers serve Features

Consistency is key for a successful architecture, and so are thin controllers to a successful MVC application.
{{% /panel %}}

{{% panel %}}
## Write code that humans can read

Machines will run it nonetheless, it is us who will suffer.
{{% /panel %}}
