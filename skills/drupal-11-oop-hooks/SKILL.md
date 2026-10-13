---
name: drupal-11-oop-hooks
description: Implement, convert, or reorder Drupal 11 hooks using the #[Hook] attribute in src/Hook classes instead of procedural .module functions. Use when adding a hook, migrating a .module file, or controlling hook execution order.
---

# Drupal 11 Object-Oriented Hooks

Drupal 11.1+ supports hooks as methods on autodiscovered classes. Prefer this over procedural `hook_*()` functions in `.module` files.

## When to use
- Adding any new hook (`form_alter`, `entity_presave`, `help`, `theme`, `cron`, and so on)
- Converting an existing `.module` implementation
- Changing hook order (replaces `hook_module_implements_alter()`)

## Rules
1. One class per concern in `src/Hook/`, e.g. `FormHooks`, `EntityHooks`, `ThemeHooks`, `CronHooks`.
2. Hook classes are autowired services automatically. Do not register them in `services.yml`. Inject dependencies through the constructor.
3. Use `#[Hook('hook_name')]` without the `hook_` prefix.
4. Add `declare(strict_types=1);`, explicit types, and a `@see \hook_name()` docblock.
5. Keep `.module` minimal. `hook_update_N()`, `hook_schema()`, `hook_install()` and similar stay in `.install`. Some hooks may still need procedural form on your minimum core version; check the change records before converting.
6. If an Event exists for the behavior, prefer an event subscriber over a hook.

## Template
```php
<?php

declare(strict_types=1);

namespace Drupal\my_module\Hook;

use Drupal\Core\Entity\EntityInterface;
use Drupal\Core\Hook\Attribute\Hook;
use Drupal\Core\Session\AccountProxyInterface;
use Psr\Log\LoggerInterface;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

/**
 * Entity hook implementations for My Module.
 */
final class EntityHooks {

  public function __construct(
    private readonly AccountProxyInterface $currentUser,
    #[Autowire(service: 'logger.channel.my_module')]
    private readonly LoggerInterface $logger,
  ) {}

  /**
   * Implements hook_entity_presave().
   *
   * @see \hook_entity_presave()
   */
  #[Hook('entity_presave')]
  public function entityPresave(EntityInterface $entity): void {
    $this->logger->info('Saving @type by @uid.', [
      '@type' => $entity->getEntityTypeId(),
      '@uid'  => $this->currentUser->id(),
    ]);
  }

}
```

## Ordering
```php
use Drupal\Core\Hook\Order\Order;
use Drupal\Core\Hook\Order\OrderAfter;

#[Hook('form_alter', order: Order::First)]
#[Hook('form_alter', order: new OrderAfter(modules: ['other_module']))]
```
Use these instead of `hook_module_implements_alter()`.

## Converting a procedural hook
1. Create `src/Hook/<Concern>Hooks.php`.
2. Move the function body into a method with the `#[Hook('name')]` attribute; replace `\Drupal::` calls with injected services.
3. If the module must still support Drupal 10 or early 11, keep the old function and mark it `#[LegacyHook]` pointing at the new class; otherwise delete the function.
4. Clear caches (`drush cr`) and verify with a test.

## Checklist
- [ ] Attribute name has no `hook_` prefix
- [ ] No service registration for the hook class
- [ ] Dependencies injected, no static `\Drupal::` calls
- [ ] Update/install/schema hooks left in `.install`
