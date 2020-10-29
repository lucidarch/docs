---

title: "CLI Reference"
date: 2020-10-28T17:04:19Z
draft: false
weight: 10
hide: ["header"]
head: "<hr />"


---

The `lucid` command line interface companion for the Lucid Architecture.

`lucid` is a set of methods to easily manage [create, delete] Lucid units such as Jobs, Operations, Features as well as Laravel's own components such as Controller, Eloquent Model, Request and Policy; ensuring that they go where they belong and are generated with their test companion classes.

---

## Setup

The executable binary can be found at `./vendor/bin/lucid` after requiring `lucid-arch/console`.

For convenience you might want to address `lucid` directly instead of having to go through `./vendor/bin` every time.
To do that you need to add `./vendor/bin` to your shell session's `$PATH`. For the current session, run:

```bash
export PATH="./vendor/bin:$PATH"
```

However, it will only be available for the current session. To make it permanent, add it to your shell profile (`~/.bash_profile`, `~/.bashrc`, `~/.zshrc`) and you will be able to simply call `lucid` from the application's root directory.

Use `list` to view a list of all available commands:

```bash
lucid list
```

Also, just like in Artisan, every command includes a "help" screen describes the command's available arguments and options.
To view a help screen, precede the name of the command with `help` or use the `--help` option:

```bash
lucid help make:feature
# or
lucid make:feature --help
```

## Commands

### `src:name`

Set the root namespace.

```bash
lucid src:name <namespace>
```

---

### `make:service`

{{% notice info %}}
{{% icon name="fa-info-circle" %}}&nbsp;For Monolith projects only.
{{% /notice %}}

Create a new service in a monolith project.

```bash
lucid make:service <name>

# example

lucid make:service HumanResources
```

Will generate the following structure:

```
src
└── Services
    └── HumanResources
        ├── Providers
        ├── Console
        ├── Http
        ├── Features
        ├── Operations
        ├── Tests
        ├── database
        ├── routes
        └── resources
```

---

### `make:controller`

Generate a controller class.


{{% tabs %}}

    {{% tab "Micro" %}}
**Signature**
```bash
lucid make:controller <controller>
```

**Example**
```bash
lucid make:controller Article
```
Generated class will be at `app/Http/Controllers/ArticleController.php`

    {{% /tab %}}

    {{% tab "Monolith" %}}
**Signature**
```bash
lucid make:controller <controller> <service>
```

**Example**
```bash
lucid make:controller Article Publishing
```
Generated class will be at `src/Services/Publishing/Http/Controllers/ArticleController.php`

    {{% /tab %}}

{{% /tabs %}}

**Empty Controller**

By default it will generate a RESTful controller with preset methods. To generate an empty controller use the `--plain` option:

```bash
lucid make:controller <controller> [<service>] --plain
```

---

### `make:feature`

Generate a Feature class.

{{% tabs %}}

    {{% tab "Micro" %}}
**Signature**
```bash
 lucid make:feature <feature>
```

**Example**
```bash
 lucid make:feature ListProducts
```
Generated class will be at `app/Features/ListProductsFeatures.php`

and its test at `tests/Features/ListProductsFeaturesTest.php`
    {{% /tab %}}


    {{% tab "Monolith" %}}
**Signature**
```bash
 lucid make:feature <feature> <service>
```

**Example**
```bash
 lucid make:feature ListProducts Commerce
```
Generated class will be at `src/Services/Commerce/Features/ListProductsFeatures.php`

and its test at `src/Services/Commerce/Tests/Features/ListProductsFeaturesTest.php`
    {{% /tab %}}

{{% /tabs %}}

The generated Feature class will automatically be suffixed with `Feature`, so there's no need for it to be specified in the command.

---

### `make:job`

Generate a Job class.

{{% tabs %}}

    {{% tab "Micro" %}}
**Signature**
```bash
 lucid make:job <job> <domain> {--Q|queue}
```

**Example**
```bash
 lucid make:job FindProduct product
```
Generated class will be at `app/Domains/Product/Jobs/FindProductJob.php`

and its test at `tests/Domains/Product/Tests/FindProductJobTest.php`
    {{% /tab %}}


    {{% tab "Monolith" %}}
**Signature**
```bash
 lucid make:job <job> <domain> {--Q|queue}
```

**Example**
```bash
 lucid make:job FindProduct product
```
Generated class will be at `src/Domains/Product/Jobs/FindProductJob.php`

and its test at `src/Domains/Product/Tests/FindProductJobTest.php`

    {{% /tab %}}

{{% /tabs %}}

The generated Job class will automatically be suffixed with `Job`, so there's no need for it to be specified in the command.

---

### `make:operation`

{{% tabs %}}

    {{% tab "Micro" %}}
**Signature**

```bash
lucid make:operation <operation> {--Q|queue}
```

**Example**
```bash
lucid make:operation NotifySubscribers
```
Generated class will be at `app/Operations/NotifySubscribersOperation.php`

and its test at `tests/Operations/NotifySubscribersOperationTest.php`
    {{% /tab %}}

    {{% tab "Monolith" %}}
**Signature**

```bash
lucid make:operation <operation> <service> {--Q|queue}
```

**Example**

```bash
lucid make:operation NotifySubscribers publishing
```

Generated class will be at `src/Services/Publishing/Operations/NotifySubscribersOperation.php`

and its test at `src/Services/Publishing/Tests/Operations/NotifySubscribersOperationTest.php`

    {{% /tab %}}

{{% /tabs %}}

The generated Operation class will automatically be suffixed with `Operation`, so there's no need for it to be specified in the command.

---

### `make:migration`

{{% tabs %}}

    {{% tab Micro %}}
**Signature**

```bash
lucid make:migration <migration>
```

**Example**

```bash
lucid make:migration create_articles_table
```

Generated file will be at `database/migrations/2020_10_28_180253_create_articles_table.php`


    {{% /tab %}}

    {{% tab Monolith %}}
**Signature**

```bash
lucid make:migration <migration> <service>
```

**Example**

```bash
lucid make:migration create_articles_table publishing
```

Generated file will be at `src/Services/Publishing/database/migrations/2020_10_28_180253_create_articles_table.php`

    {{% /tab %}}

{{% /tabs %}}

---

### `make:model`

Generate an Eloquent model class.

**Signature**

```bash
lucid make:model <name>
```

**Example**

```bash
lucid make:model Product
```

Generated model file will be at `[app|src]/Data/Product.php`.

---

### `make:request`

{{% tabs %}}

    {{% tab Micro %}}
```bash
lucid make:request <request>
```

Generated file will be at `app/Http/Requests/<request>`.

    {{% /tab %}}

    {{% tab Monolith%}}
```bash
lucid make:request <request> <service>
```

Generated file will be at `src/Services/<service>/Http/Requests/<request>`.
    {{% /tab %}}

{{% /tabs %}}

---

### `make:policy`

{{% tabs %}}

    {{% tab Micro %}}
```bash
lucid make:policy <policy>
```

Generated file will be at `app/Http/Policies/<policy>`.

    {{% /tab %}}

    {{% tab Monolith%}}
```bash
lucid make:policy <policy> <service>
```

Generated file will be at `src/Services/<service>/Http/Policies/<policy>`.
    {{% /tab %}}

{{% /tabs %}}

---

### `list:services`

List the services in a monolith project

{{% notice info %}}
{{% icon name="fa-info-circle" %}}&nbsp;For Monolith projects only.
{{% /notice %}}

```bash
lucid list:services
```

```
+------------+------------+-------------------------+
| Service    | Slug       | Path                    |
+------------+------------+-------------------------+
| Commerce   | commerce   | src/Services/Commerce   |
| Publishing | publishing | src/Services/Publishing |
| Admin      | admin      | src/Services/Admin      |
+------------+------------+-------------------------+
```

---

### `list:features`

```bash
lucid list:features
```

```
+---------------------------------+---------+-----------------------------------------+----------------------------------------------------------------------+
| Feature                         | Service | File                                    | Path                                                                 |
+---------------------------------+---------+-----------------------------------------+----------------------------------------------------------------------+
| Update DevTo Collaborations    | Digest  | UpdateDevToCollaborationsFeature.php    | src/Services/Digest/Features/UpdateDevToCollaborationsFeature.php    |
| Update DevTo Articles          | Digest  | UpdateDevToArticlesFeature.php          | src/Services/Digest/Features/UpdateDevToArticlesFeature.php          |
| Add GitHub Repo To Digest      | Digest  | AddGitHubRepoToDigestFeature.php        | src/Services/Digest/Features/AddGitHubRepoToDigestFeature.php        |
| Lookup GitHub Repo             | Digest  | LookupGitHubRepoFeature.php             | src/Services/Digest/Features/LookupGitHubRepoFeature.php             |
| Person Feed                    | Digest  | PersonFeedFeature.php                   | src/Services/Digest/Features/PersonFeedFeature.php                   |
| Update Stack Overflow Questions| Digest  | UpdateStackOverflowQuestionsFeature.php | src/Services/Digest/Features/UpdateStackOverflowQuestionsFeature.php |
| Update GitHub Data             | Digest  | UpdateGitHubDataFeature.php             | src/Services/Digest/Features/UpdateGitHubDataFeature.php             |
| Login With GitHub              | Digest  | LoginWithGitHubFeature.php              | src/Services/Digest/Features/LoginWithGitHubFeature.php             |
+---------------------------------+---------+-----------------------------------------+----------------------------------------------------------------------+

```

### `delete:service`

{{% notice info %}}
{{% icon name="fa-info-circle" %}}&nbsp;For Monolith projects only.
{{% /notice %}}


```bash
lucid delete service <name>
```

### `delete:feature`

```bash
lucid delete:feature <feature> [<service>]
```

### `delete:job`

```bash
lucid delete:job <job> <domain>
```

### `delete:model`

```bash
delete:model <model>
```

### `delete:request`

```bash
lucid delete:request <request> [<service>]
```

### `delete:policy`

```bash
delete:policy <policy> [<service>]
```
