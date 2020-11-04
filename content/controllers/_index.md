---
title: "Controllers"
date: 2020-10-18T13:17:41Z
draft: false
weight: 5
hide: ["header"]

---

In an effort to minimize the work of controllers, for they are not here to do work for the application but to point the
request in the right direction. In Lucid terms, to serve the intended feature to the user.

Eventually we will end up having one line within each controller method, achieving the thinnest form possible.

{{% figure alt="Lucid Controller" src="/media/images/controller/route-controller.png" %}}

**Responsibility:** Serve the designated feature.

What you should do in a controller:

- Serve a feature.
- Prepare input as required by the feature that's being served [does not include input validation]

---

## Generate Controller Class
Use `lucid` CLI to generate a controller that extends Lucid's Controller class by default, which allows us to serve features
features using the built-in `serve` method.

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
lucid make:controller Article MyService
```
Generated class will be at `src/Services/MyService/Http/Controllers/ArticleController.php`

    {{% /tab %}}

{{% /tabs %}}


{{% notice info %}}
{{% icon name="fa-asterisk"%}}&nbsp;For more details on this command see the help manual with `lucid make:controller --help` or
visit [`make:controller`]({{% ref "/cli/#makecontroller" %}} "Make Controller Command")
{{% /notice %}}

## Serve Features

{{% figure alt="Lucid Controller serve Feature" src="/media/images/feature/controller-feature.png" %}}

To serve a Feature from controllers simply call the `serve` method provided by Lucid's parent controller.

```php
use Lucid\Units\Controller;
use App\Features\UpdateArticleFeature;

class ArticleController extends Controller
{
    public function articles()
    {
        return $this->serve(ListArticlesFeature::class);
    }
}
```

### Request Input

The served feature will be able to inject `Request` class to access request properties.
This keeps our controllers clean and allows us to concentrate on what matters to the feature only.

```php
class ListArticlesFeature
{
    public function handle(Request $request)
    {
        $input = $request->input();
        // or
        $title = $request->input('title');
    }
}
```



### Feature Parameters
To pass parameters to a feature, we use the same syntax as dispatching a Laravel job:

```php
use Lucid\Units\Controller;
use App\Features\UpdateArticleFeature;

class ArticleController extends Controller
{
    public function update($id)
    {
        return $this->serve(UpdateArticleFeature::class, ['id' => $id]);
    }
}
```

The `id` key will be mapped to `$id` constructor param `UpdateArticleFeature::constructor($id)`.

{{% notice info %}}
{{% icon name="fa-info-circle" size="16px" %}}&nbsp;Parameter and constructor variable names must match. And they're case sensitive!
{{% /notice %}}


Using associative arrays as properties has the advantage of disregarding the order in which the parameters are defined
in the class we're calling. However, they should be only what the feature needs to operate.

```php
class UpdateArticleFeature extends Feature
{
    private $id;

    public function __construct(string $id)
    {
        $this->id = $id;
    }

    public function handle(Request $request)
    {
        $this->run(UpdateArticleDataJob::class, [
            'id' => $this->id,
            'title' => $request->input('title'),
            'content' => $request->input('content');
        ]);
    }
}
```

For more on writing features see the [Features section](/features).
