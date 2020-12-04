---
title: "Contribution Guide"
date: 2020-10-28T21:36:30Z
draft: true
weight: 20
hide: [header]
---

## Bug Reports

To encourage active collaboration, Lucid Architecture strongly encourages pull requests, not just bug reports. "Bug reports" may also be sent in the form of a pull request containing a failing test or steps to reproduce the bug.

However, if you file a bug report, your issue should contain a title and a clear description of the issue. You should also include as much relevant information as possible and a code sample that demonstrates the issue. The goal of a bug report is to make it easy for yourself - and others - to replicate the bug and develop a fix.

Remember, bug reports are created in the hope that others with the same problem will be able to collaborate with you on solving it. Do not expect that the bug report will automatically see any activity or that others will jump to fix it. Creating a bug report serves to help yourself and others start on the path of fixing the problem.

The Lucid Architecture source code is on GitHub as [lucidarch/lucid](https://github.com/lucidarch/lucid).

## Support Questions

Lucid Architecture's GitHub issue trackers are not intended to provide help or support. Instead, use one of the following channels:

- Official [Slack workspace](https://lucid-slack.herokuapp.com/) in the `#support` channel
- [StackOverflow](https://stackoverflow.com/questions/tagged/lucid-architecture)

## Core Development Discussion

You may propose new features or improvements of existing Lucid Architecture behaviour in the Lucid Ideas [issue board](https://github.com/lucidarch/ideas/issues).
If you propose a new feature, please be willing to implement at least some of the code that would be needed to complete the feature.

Informal discussion regarding bugs, new features, and implementation of existing features takes place in the `#internals` channel of the [Lucid Slack workspace](https://lucid-slack.herokuapp.com/).
Abed Halawi, the maintainer of Lucid, is typically present in the channel on weekdays from 8am-5pm EEST (Eastern European Summer Time), and sporadically present in the channel at other times.

## Which Branch?

All repositories follow Laravel's versions in branching, where installing the branch `5.5` in any of the repositories will install Lucid with Laravel v5.5.
What's on the `master` branch is always the latest Laravel version.

The main repositories are the following:

- Main: [Laravel (monolith)](https://github.com/lucidarch/laravel) and [Microservice](https://github.com/lucidarch/laravel-microservice)
- [Foundation](https://github.com/lucidarch/foundation-laravel): Has Lucid units and other abstract classes such as Validator, Repository etc.
- [Console](https://github.com/lucidarch/console-laravel)**:** contains the Console companion, the CLI and the dashboard

## Compiled Assets

If you are submitting a change that will affect a compiled file, such as most of the files in `resources/sass` or `resources/js` of the `lucid-architecture/laravel` repository, do not commit the compiled files. Due to their large size, they cannot realistically be reviewed by a maintainer. This could be exploited as a way to inject malicious code into Lucid Architecture. In order to defensively prevent this, all compiled files will be generated and committed by Lucid maintainers.

## Security Vulnerabilities

If you discover a security vulnerability within Lucid, please send an email to Abed Halawi at [halawi.abed@gmail.com](mailto:abed.halawi@gmail.com).
All security vulnerabilities will be promptly addressed.

## Coding Style

Lucid Architecture follows the [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) coding standard and the [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) autoloading standard.

### PHPDoc

Below is an example of a valid Lucid Architecture documentation block. Note that the `@param` attribute is followed by two spaces, the argument type, two more spaces, and finally the variable name:

```php
/**
 * Register a binding with the container.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */
public function bind($abstract, $concrete = null, $shared = false)
{
    //
}
```

## Code of Conduct

The Lucid Architecture code of conduct is derived from the Laravel code of conduct. Any violations of the code of conduct may be reported to Abed Halawi (halawi.abed@gmail.com):

- Participants will be tolerant of opposing views.
- Participants must ensure that their language and actions are free of personal attacks and disparaging personal remarks.
- When interpreting the words and actions of others, participants should always assume good intentions.
- Behavior that can be reasonably considered harassment will not be tolerated.
