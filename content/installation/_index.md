---
title: "Installation"
date: 2020-11-13T18:14:56Z
draft: false
weight: 4
head: "<hr />"
hide: ["header"]
---

### Install Laravel

The Lucid Architecture is delivered as a Composer package [`lucidarch/lucid`](https://packagist.org/packages/lucidarch/lucid).
Its only requirement is a Laravel project so to begin the installation, first grab a Laravel instance
by simply running

```bash
composer create-project --prefer-dist laravel/laravel blog
```

Or head over to [Laravel's Installation docs](https://laravel.com/docs/installation) for other means of installation.

### Install Lucid

```bash
composer require lucidarch/lucid
```

Make sure to place Composer's project vendor bin direcotry in your `$PATH` so the `lucid` executable can be located by your system.
Usually it's done by running `export PATH="$PATH:./vendor/bin"` to have it available in your current Terminal session,
or add it to your corresponding Terminal profile (e.g. `~/.bashrc`, `~/.bash_profile`, `~/.zshrc`) to have it permanently loaded
with every session.

## Versions & Compatibility

Lucid's versioning system follows the [semantic versioning scheme](https://semver.org).
The latest Lucid version is tested against the following matrix, hence providing support to any combination of them:

- **Laravel**
{{<badge info>}} 8 {{</badge>}}
{{<badge secondary>}} 7 {{</badge>}}
{{<badge secondary>}} 6 {{</badge>}}
{{<badge secondary>}} 5.5 (LTS) {{</badge>}}

- **PHP**:
{{<badge info>}} 8.0 {{</badge>}}
{{<badge secondary>}} 7.4 {{</badge>}}
{{<badge secondary>}} 7.3 {{</badge>}}
{{<badge secondary>}} 7.2 {{</badge>}}
{{<badge secondary>}} 7.1 {{</badge>}}
{{<badge secondary>}} 7.0.21 {{</badge>}}

## Initialization

Lucid structure initialization is an optional step. If you've required the package you may already begin using `lucid` executable
to generate units such as [Jobs]({{<ref "/jobs">}}), [Features]({{<ref "/features">}}) and [Operations]({{<ref "/operations">}}).

However, if you've familiarized yourself with the predefined variants of Lucid -
[Micro]({{<ref "/micro-vs-monolith/#micro">}}) and [Monolith]({{<ref "/micro-vs-monolith/#monolith">}}) - you may wish to initialize your application.

### Micro

Initialize a Micro instance:

```bash
lucid init:micro
```

This will generate an initial Micro structure:

<pre>
app
├── Console
│   └── Kernel.php
├── <strong>Data</strong>
    └── Models
├── <strong>Domains</strong>
├── <strong>Features</strong>
├── <strong>Operations</strong>
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
</pre>

### Monolith

{{<columns>}}

Initialize a Monolith instance:

```bash
lucid init:monolith
```

<pre>
app
├── Console
├── <strong>Data</strong>
    └── Models
├── <strong>Domains</strong>
├── Exceptions
├── <strong>Foundation</strong>
├── Http
├── Policies
├── Providers
├── <strong>Services</strong>
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
</pre>

<--->

You may also specify a service to begin with:

```bash
lucid init:monolith blog
```

And the following will be added to the initial structure:

<pre>
<strong>app/Services/Blog</strong>
├── Console
│   └── Commands
├── <strong>Features</strong>
├── <strong>Operations</strong>
├── Http
│   ├── Controllers
│   └── Middleware
├── Providers
│   ├── BlogServiceProvider.php
│   ├── BroadcastServiceProvider.php
│   └── RouteServiceProvider.php
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
</pre>

{{</columns>}}

For more on the difference between Micro & Monolith and to know how to choose precisely for your application, head over to
[Micro • Monolith]({{<ref "/micro-vs-monolith">}}).

---

That's it! You're ready to build something awesome.

One last thing to remember: **enjoy the journey!**
