---
name: drupal-11-code-architecture
description: Structure Drupal 11 module code - naming, PSR-4 layout, controllers, entities and bundle classes, services, routing, type declarations, and PHPDoc. Use when creating or reviewing controllers, routes, entity types, services, or when deciding where code belongs in a module.
---

# Drupal 11 Code Architecture

## Naming and layout
- PSR-4 namespaces: `Drupal\my_module\...` mapped to `src/`
- Follow Drupal naming for files, classes, and methods
- Prefix custom services, routes, permissions, and plugins with the module machine name
- Typical layout: `src/Controller`, `src/Form`, `src/Hook`, `src/Plugin`, `src/Service`, `src/Entity`, `src/EventSubscriber`, `src/Repository`

## Controllers
- `final` unless extension is an explicit design goal
- Thin: delegate business logic to services
- Dependency injection via autowiring (see `drupal-11-dependency-injection`)
- Return render arrays or `Response` objects with explicit return types

```php
<?php

declare(strict_types=1);

namespace Drupal\my_module\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\Core\DependencyInjection\AutowireTrait;
use Drupal\my_module\Service\ExampleManagerInterface;

/**
 * Returns responses for My Module routes.
 */
final class ExampleController extends ControllerBase {

  use AutowireTrait;

  public function __construct(
    private readonly ExampleManagerInterface $manager,
  ) {}

  /**
   * Builds the overview page.
   *
   * @return array<string, mixed>
   *   A render array.
   */
  public function overview(): array {
    return [
      '#theme' => 'item_list',
      '#items' => $this->manager->getItems(),
    ];
  }

}
```

## Entities
- Extend core entity base classes following the class hierarchy
- Define entity types with attributes (`#[ContentEntityType]`, `#[ConfigEntityType]`) and base fields with `BaseFieldDefinition`
- Use bundle classes (`bundle_class`) for bundle-specific behavior instead of hooks or procedural helpers
- Define access in an access control handler, never ad hoc checks scattered in controllers
- Use the Repository pattern for query-heavy data access; always set `accessCheck()` on entity queries

## Services
- Register in `MODULE.services.yml`; define interfaces for public contracts
- Single responsibility per service
- Mark internal-only classes `@internal`

## Routing
Define routes in `MODULE.routing.yml` with explicit access requirements:
```yaml
my_module.overview:
  path: '/my-module/overview'
  defaults:
    _controller: '\Drupal\my_module\Controller\ExampleController::overview'
    _title: 'Overview'
  requirements:
    _permission: 'access my_module overview'

my_module.reset:
  path: '/my-module/reset'
  defaults:
    _controller: '\Drupal\my_module\Controller\ExampleController::reset'
  requirements:
    _permission: 'administer my_module'
    _csrf_token: 'TRUE'
```
- Use `_permission`, `_entity_access`, or a custom access checker; never leave a route without a requirement
- Use `_csrf_token: 'TRUE'` for state-changing GET routes
- Declare permissions in `MODULE.permissions.yml`

## Type declarations
- Explicit return types on every method, including `void`, `never`, `static`
- Explicit parameter and property types; avoid `mixed`
- Document array shapes in PHPDoc: `array<string, mixed>`, `list<int>`, `array{id: int, label: string}`

## PHPDoc
- Complete docs for classes, methods, and properties
- `@param` types may be omitted when native types say it all, but add shape details for arrays
- Include `@return`, `@throws`, `@deprecated` as needed:
  `@deprecated in drupal:11.x.0 and is removed from drupal:12.0.0. Use X instead. See https://www.drupal.org/node/NNNN`
- Document hook implementations with `@see \hook_name()`

## Checklist
- [ ] Namespace and directory follow PSR-4
- [ ] Controller is thin, final, and autowired
- [ ] Every route has an access requirement
- [ ] Entity access goes through an access handler
- [ ] Return types and array shapes documented
