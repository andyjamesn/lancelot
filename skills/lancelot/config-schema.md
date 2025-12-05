# Lancelot Config Schema

The `.lancelot/config.json` file defines project-specific expertise for Lancelot agents.

## Location

```
your-project/
├── .lancelot/
│   ├── config.json      ← Project configuration
│   └── plans/           ← Generated plans
├── src/
└── ...
```

## Schema

```json
{
  "version": "1.0",
  
  "stack": {
    "backend": "Laravel 11, PHP 8.3",
    "frontend": "Vue 3.4, Inertia 1.0",
    "language": "PHP 8.3, TypeScript 5.4",
    "database": "MySQL 8",
    "styling": "Tailwind CSS 4"
  },
  
  "expertise": {
    "patterns": [
      "Laravel Actions pattern (single-responsibility action classes)",
      "Form Requests for all validation",
      "Eloquent API Resources for responses",
      "Composition API with <script setup>",
      "Pinia stores for state management",
      "Inertia useForm for form handling"
    ],
    "testing": {
      "backend": "Pest PHP with feature tests",
      "frontend": "Vitest + Vue Test Utils"
    },
    "architecture": "Monolith with Inertia SPA"
  },
  
  "conventions": {
    "backend": [
      "PSR-12 coding standard",
      "Strict types in all PHP files",
      "Final classes by default",
      "Action classes in app/Actions/",
      "Form Requests in app/Http/Requests/"
    ],
    "frontend": [
      "Components in resources/js/Components/",
      "Pages in resources/js/Pages/",
      "Composables in resources/js/Composables/",
      "TypeScript strict mode"
    ],
    "naming": {
      "actions": "VerbNounAction (e.g., CreateUserAction)",
      "components": "PascalCase.vue",
      "composables": "use*.ts"
    }
  },
  
  "avoid": [
    "Repository pattern - use Eloquent directly",
    "Options API - use Composition API",
    "Raw SQL - use Query Builder or Eloquent",
    "Global state - use Pinia stores",
    "Mixins - use composables instead"
  ],
  
  "preferences": {
    "complexity": "simple",
    "comments": "minimal, self-documenting code",
    "errorHandling": "explicit try/catch with custom exceptions"
  }
}
```

## Field Descriptions

### `stack`
The technology stack used in the project. Agents use this to apply framework-specific knowledge.

### `expertise.patterns`
Design patterns and architectural approaches used in this project. Agents will follow these patterns.

### `expertise.testing`
Testing frameworks and approaches. Guides test generation.

### `conventions`
Coding standards and file organization rules. Agents ensure new code follows these.

### `avoid`
Anti-patterns and approaches NOT to use. Agents will explicitly avoid these.

### `preferences`
General preferences for code style and complexity.

## Usage

Agents receive this config as expertise context:

```
## YOUR EXPERTISE (from .lancelot/config.json)

Stack: Laravel 11, Vue 3.4, Inertia...

Patterns to follow:
- Laravel Actions pattern
- Form Requests for validation
...

AVOID:
- Repository pattern
- Options API
...
```

## Generation

Run `/lancelot/init` to auto-generate this config by analyzing your codebase.
