---
title: "IDE Integration"
date: 2024-01-01T00:00:00Z
draft: true
weight: 7
head: "<hr />"
hide: ["header"]
---

Lucid's naming conventions and directory structure make it easy to navigate code in any IDE or editor.
This page outlines tips and extensions to improve your experience when working with Lucid Architecture.

## PHPStorm

### File Structure Navigation

PHPStorm's **Go to Class** (`Cmd+O` / `Ctrl+N`) works naturally with Lucid's naming conventions.
Since every unit follows a consistent suffix pattern (`Feature`, `Job`, `Operation`), you can type:

- `AddLink` → navigate to `AddLinkFeature`, `AddLinkJob`, etc.
- `SaveProduct` → quickly find `SaveProductJob` across domains

### Directory Bookmarks

Use PHPStorm's **Bookmarks** or **Favorites** to pin the main Lucid directories for quick access:

- `app/Features/` — all features at a glance
- `app/Domains/` — domain-grouped jobs
- `app/Operations/` — multi-step operations
- `app/Services/` (Monolith) — service-scoped features

### Live Templates

Create PHPStorm **Live Templates** to scaffold Lucid units faster.

**Job template** (`lucid-job`):

```php
use Lucid\Units\Job;

class $NAME$Job extends Job
{
    public function __construct(
        $PARAMS$
    ) {}

    public function handle()
    {
        $END$
    }
}
```

**Feature template** (`lucid-feature`):

```php
use Lucid\Units\Feature;
use Illuminate\Http\Request;

class $NAME$Feature extends Feature
{
    public function handle(Request $request)
    {
        $END$
    }
}
```

### PHP Inspections

PHPStorm's built-in **PHP inspections** work well with Lucid's strict class hierarchy.
Enable **"Method return type can be declared"** and **"Property type can be declared"** inspections
to help modernise unit code to PHP 8.1+ standards.

---

## VS Code

### Recommended Extensions

| Extension | Purpose |
|-----------|---------|
| [PHP Intelephense](https://marketplace.visualstudio.com/items?itemName=bmewburn.vscode-intelephense-client) | PHP language server — go to definition, find references, autocompletion |
| [PHP Namespace Resolver](https://marketplace.visualstudio.com/items?itemName=MehediDracula.php-namespace-resolver) | Auto-import and sort `use` statements |
| [Laravel Extra Intellisense](https://marketplace.visualstudio.com/items?itemName=amiralizadeh9480.laravel-extra-intellisense) | Route, view, config, and env autocompletion |
| [EditorConfig](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig) | Respect per-project coding style settings |

### Go to Symbol

Use **Go to Symbol in Workspace** (`Cmd+T` / `Ctrl+T`) and type a unit name with its suffix
(e.g. `SaveProductJob`) to jump directly to it — no directory browsing needed.

### Workspace Settings

Add the following to `.vscode/settings.json` in your project to exclude non-PHP directories from
PHP indexing and keep navigation fast:

```json
{
    "intelephense.files.exclude": [
        "**/.git/**",
        "**/node_modules/**",
        "**/vendor/lucidarch/**"
    ],
    "files.associations": {
        "*.blade.php": "html"
    }
}
```

### Code Snippets

Add Lucid-specific snippets to `.vscode/lucid.code-snippets` in your project:

```json
{
    "Lucid Job": {
        "prefix": "lucid-job",
        "scope": "php",
        "body": [
            "use Lucid\\Units\\Job;",
            "",
            "class ${1:Name}Job extends Job",
            "{",
            "    public function __construct(",
            "        ${2:private readonly string \\$param,}",
            "    ) {}",
            "",
            "    public function handle()",
            "    {",
            "        ${0}",
            "    }",
            "}"
        ],
        "description": "Lucid Job class"
    },
    "Lucid Feature": {
        "prefix": "lucid-feature",
        "scope": "php",
        "body": [
            "use Lucid\\Units\\Feature;",
            "use Illuminate\\Http\\Request;",
            "",
            "class ${1:Name}Feature extends Feature",
            "{",
            "    public function handle(Request \\$request)",
            "    {",
            "        ${0}",
            "    }",
            "}"
        ],
        "description": "Lucid Feature class"
    },
    "Lucid Operation": {
        "prefix": "lucid-operation",
        "scope": "php",
        "body": [
            "use Lucid\\Units\\Operation;",
            "",
            "class ${1:Name}Operation extends Operation",
            "{",
            "    public function __construct(",
            "        ${2:private readonly int \\$id,}",
            "    ) {}",
            "",
            "    public function handle()",
            "    {",
            "        ${0}",
            "    }",
            "}"
        ],
        "description": "Lucid Operation class"
    }
}
```

---

## Navigation Conventions

Regardless of IDE, Lucid's structure enables predictable navigation:

| You want to find... | Look in... |
|---------------------|-----------|
| What the app does | `app/Features/` (Micro) or `app/Services/*/Features/` (Monolith) |
| Business logic for a topic | `app/Domains/{Domain}/Jobs/` |
| Multi-step internal process | `app/Operations/` or `app/Services/*/Operations/` |
| Data models | `app/Data/Models/` |
| Feature tests | `tests/Feature/` |
| Job/Operation unit tests | `tests/Unit/Domains/{Domain}/Jobs/` |

This means any developer familiar with Lucid can navigate any Lucid project without prior knowledge of the
specific application's business logic — the structure itself communicates where to look.
