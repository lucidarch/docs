---
title: "Validation"
date: 2020-11-04T17:39:15Z
draft: false
weight: 19
hide: ["header"]

---

As common as it is, validation is essential for every application.
Lucid brings a domain-driven approach that embraces Laravel's validation. In short, if you've done validation with Laravel
you're already familiar with everything here, we will only choose convenient locations for Validators and Form Requests
by placing them in domains.

## Domain-Driven Validation

Often times validation is a matter of context, and domains in Lucid are an ideal place to encapsulate functionality,
and form requests fit perfectly there! e.g. `Domains/{domain}/Requests/{Request}.php`

Eventually we'd end up with a categorical directory structure that protects our validation files from becoming unmanageable.

{{<columns>}}
**Form Requests**
```
app/Domains
├── Chat
│   └── Requests
│       └── SendMessage.php
├── Comment
│   └── Requests
│       └── AddComment.php
├── Filesystem
│   └── Requests
│       └── UploadFiles.php
└── Post
    └── Requests
        ├── CreatePost.php
        └── UpdatePost.php
```

<--->

**Validation Jobs**
```
app/Domains
├── Chat
│   └── Jobs
│       └── ValidateNewMessageJob.php
├── Comment
│   └── Jobs
│       └── ValidateNewCommentJob.php
├── Filesystem
│   └── Jobs
│       └── ValidateFilesUploadJob.php
└── Post
    └── Jobs
        ├── ValidateNewPostJob.php
        └── ValidatePostUpdateJob.php
```
{{</columns>}}


## Form Request Validation

In an effort to keep our controllers thin and reduce their clutter, as well as having features carry their requirements with
them wherever we decide to serve them from, it is recommended for **validation using Form Request to happen in features instead of controllers**, to maintain their integrity whenever they're served.

Nevertheless, it is done exactly the same as Laravel controllers in any of Lucid's units,
by injecting the form request class in the method signature as a parameter:

```php
class CustomFeature extends Feature
{
    public function handle(UpdatePost $request)
    {
        // request is valid according to the rules specified in UpdatePost
    }
}
```

This will perform Form Request Validation using `UpdatePost` class.

{{% notice info %}}
{{<icon name="fa-info-circle">}}&nbsp;The same can be done in any unit: Feature, Job and Operation.
{{% /notice %}}

{{% notice dark %}}
For more on Form Request classes see [Laravel's docs](https://laravel.com/docs/validation#form-request-validation).
{{% /notice %}}

### Generate Form Request

{{<tabs>}}

{{<tab Micro>}}

**Signature**
```bash
lucid make:request <name> <domain>
```

**Example**
```bash
lucid make:request UpdatePost post
```

Generated class will be at `app/Domains/Post/Requests/UpdatePost.php`

{{</tab>}}

{{<tab Monolith>}}
**Signature**
```bash
lucid make:request <name> <domain>
```

**Example**
```bash
lucid make:request UpdatePost post
```

Generated class will be at `src/Domains/Post/Requests/UpdatePost.php`

{{</tab>}}

{{</tabs>}}

<br />

```php
<?php

namespace App\Domains\Post\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UpdatePost extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            //
        ];
    }
}
```

### Requests as Unit Dispatchers

One of the most powerful aspects of Lucid is the degree of code reuse it introduces. Specifically when dealing with
Job and Operation classes which can be called from any [custom dispatcher class]({{<ref "/jobs/#custom-dispatcher">}}).

Here, we will be setting up `UpdatePost` Request class to authorize using `AuthorizeUserOperation`, and on [after hook](https://laravel.com/docs/8.x/validation#adding-after-hooks-to-form-requests)
to run `FaliedValidationJob`.

{{% panel %}}
**Benefit:** Doing so would increase the degree of consistency and integrity in our application,
where `AuthorizeUserOperation` could contain our consolidated authorization mechanism and
can be called from any other class such as middleware to authoriza pre-flight.
As for `FaliedValidationJob` it will ensure that all of our failed validations are handled consistently as expected.
{{% /panel %}}

```php
<?php

namespace App\Domains\Post\Requests;

use Lucid\Bus\UnitDispatcher;
use App\Operations\AuthorizeUserOperation;
use Illuminate\Foundation\Http\FormRequest;
use App\Domains\Http\Jobs\FailedValidationJob;

class UpdatePost extends FormRequest
{
    use UnitDispatcher;

    public function authorize()
    {
        return $this->run(AuthorizeUserOperation::class);
    }

    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $this->run(new FailedValidationJob($validator));
            }
        });
    }
}
```

## Validation Jobs

Another approach to validation is to validate using jobs. This is most useful with protocols other than HTTP (e.g. Console, AMQP, etc.),
or if you simply choose not to use Requests.

Our validation jobs would contain [validation logic](https://laravel.com/docs/8.x/validation#quick-writing-the-validation-logic)
so it can be called whenever validation is required.

Start by creating a job:

```bash
lucid make:job ValidatePostUpdate post
```

Then inject `Illuminate\Http\Request` and fill in your validation logic:

```php
<?php

namespace App\Domains\Post\Jobs;

use Lucid\Units\Job;
use Illuminate\Http\Request;

class ValidatePostUpdateJob extends Job
{
    public function handle(Request $request)
    {
        $validData = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        return $validData;
    }
}
```
