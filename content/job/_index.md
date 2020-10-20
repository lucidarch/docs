---
title: "Job"
date: 2020-10-19T20:34:41Z
draft: true
weight: 7
hide: ["header"]

---

Jobs do the actual work. Being the smallest unit in the Lucid architecture, a Job should do one thing, and one thing only. That is: perform a single task.

Our objective with Jobs is to limit the scope of a functionality so that we know where to look for an implementation, and when we're within its context we don't tangle responsibility with jobs or any other components in the codebase.

Things to be careful of with Jobs:

- Never call another Job from within a Job, it defeats the whole purpose of having them and will create obscure nested logic that will be hard to follow and maintain.
- Jobs do not know about other [surrounding] jobs, they operate in isolation and are unaware of their surroundings. They are selfish, only concerned with themselves and their needs to operate.
- To validate your choice with jobs, simply ask yourself: "what does this job do?" and the answer should be "It *[does this]* then returns *[that]*" where:
    - *[does this]:* should not include an "and" and should be made up of few words
    - *[that]:* should either be a defined structure such as an object, or a status response. A tip here is a to avoid associative arrays as much as possible, or at all if possible, for they ramp up undefined structures and it will require more and more cognitive load to deduce meaning out of looking at the code.
- Constructor parameters of a Job - a.k.a. Job signature - should be about the Job itself only, not concerned with where it will be called from and in which conditions. Here's a personification of a Job speaking:
{{% panel %}}
*I, as a Job, in order to fulfil my task, I need "X" and "Y", and once I am done I will return "Z" to you.*
{{% /panel %}}

It is a common practice to share jobs, make their code as shareable as possible but be careful not to end up having complicated jobs just for the sake of reusability.
Good balance is key, and it is up to you and your team to find that balance .

## Generate Job Class


## Calling Jobs

Call jobs within features

call jobs within operations

Call jobs from anywhere (exception handler, command)

## Asyncronous Jobs


## Lucid Jobs vs Laravel Jobs

## FAQ
- How to ensure a job doesn't grow complexity?

- How to ensure that a job has one responsibility?

    they are the single responsibility keepers.
