---
title: "Error Handling"
date: 2020-10-22T06:37:52Z
draft: true
weight: 20
hide: ["header", "toc"]

---

*[WIP]*

Features typically expose functionality to the "outside world", meaning they have contracts to fulfill when returning results
like JSON, Views (HTML) or other data exchange formats. Centralizing error handling in features guarantees consistency in returned results.

We strive to keep error handling work to a minimum within a feature; so we only handle the errors that are most specific to the feature
itself and delegate the rest of the non-specific errors to the Handler class in the case of HTTP or the caller in other cases.

### Expected Exceptions

Are the exceptions that are specific and (in most cases) would require a certain action to be taken when they occur. They are
of a case-by-case basis so it wouldn't make sense to include them among the "common" types of errors such as the error handler.

### Unexpected Exceptions

Are the types of exceptions that are best left for the central error handler class to take care of. They have to do with the "overall"
application behaviour rather than a specific part or functionality.

