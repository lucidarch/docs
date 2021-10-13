---
title: "Database"
date: 2021-09-21T18:46:27Z
draft: false
weight: 18
hide: ["header"]
---

## Migrations
- Micro migrations: Are the Laravel built-in migrations so please refer to the [official docs](https://laravel.com/docs/migrations) and the `artisan migrate:*` commands.

- Service migrations: When a service is generated with the `lucid make:service` command, it will contain a `database` directory that resemble `/database` at root.

```
app/Services/Chat/database
├── factories
├── migrations
└── seeders
```


{{% notice info %}}
{{<icon name="fa-info-circle">}}&nbsp;The examples in this document assume that our service name is `Chat`, please make sure to replace it with your service name instead.
{{% /notice %}}

### Generate Service Migration

**Signature** `lucid make:migration <migration> <service>`

**Example**

```bash
lucid make:migration create_messages_table chat
```

Will generate `app/Services/Chat/database/migrations/{date}_create_messages_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateMessagesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('messages', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('messages');
    }
}
```

### Run Service Migrations

Our migrations are recognized by Laravel due to them being loaded automatically in the service provider - `ChatServiceProvider` in this example.

Which makes it straight-forward to run them:

```bash
php artisan migrate
```

### Disable Service Migrations
If your service doesn't use a [separate] database (you may always still use Laravel's default migrations) and you wish to clean it up from database stuff:

- Remove `app/Services/Chat/database` directory
- Remove the following snippet from `ChatServiceProvider::boot` in `app/Services/Chat/Providers/ChatServiceProvider.php`
    ```php
    $this->loadMigrationsFrom([
        realpath(__DIR__ . '/../database/migrations')
    ]);
    ```

## Factories


1. Register service factories in composer
	- Under  `autoload.psr-4` add

    `"App\\Services\\Chat\\Database\\Factories\\": "app/Services/Chat/database/factories/"`

    So your `composer.json` should have a similar section as this:
    ```json
    {
        "autoload": {
            "psr-4": {
                "App\\": "app/",
                "Database\\Factories\\": "database/factories/",
                "Database\\Seeders\\": "database/seeders/",
                "App\\Services\\Chat\\Database\\Factories\\": "app/Services/Chat/database/factories/"
            }
        }
    }

    ```
	- Run `composer dump-autoload`
2. Create factory class (e.g.  `php artisan make:factory --model=Message`)
3. Move factory class from `/database/factories` to `/app/Services/Chat/database/factories`
4. Change class namespace to `App\Services\Chat\Database\Factories`
5. In your model class (here it's `Message`): add `newFactory()` method. This is a method override that Laravel natively looks at before loading the default factory (see `HasFactory` trait)
    ```php
    use App\Services\Chat\Database\Factories\MessageFactory;

    protected static function newFactory()
    {
        return app(MessageFactory::class);
    }
    ```
6. Use the factory from the model as you would by default: `Message::factory()->make()`


## Seeders

1. Register service seeders in composer
	- Under `autoload.psr-4` add the following

    `"App\\Services\\Chat\\Database\\Seeders\\": "app/Services/Chat/database/seeders/"`

    So your `composer.json` should have a similar section as this:
    ```json
    {
        "autoload": {
            "psr-4": {
                "App\\": "app/",
                "Database\\Factories\\": "database/factories/",
                "Database\\Seeders\\": "database/seeders/",
                "App\\Services\\Chat\\Database\\Seeders\\": "app/Services/Chat/database/seeders/"
            }
        },
    }
    ```
	- Run `composer dump-autoload`
2. Create seeder class (e.g. `php artisan make:seeder MessageSeeder`)
3. Move seeder class from `/database/seeders` to `/app/Services/Chat/database/seeders`
4. Change class namespace to `App\Services\Chat\Database\Seeders`
5. Call `MessageSeeder::class` from `/database/seeders/DatabaseSeeder.php`
    ```php
    use App\Services\Chat\Database\Seeders\MessageSeeder;

    public function run()
        {
            $this->call([
                MessageSeeder::class,
            ]);
        }
    ```
6. Seed the database with `php artisan db:seed`
