---
name: drupal-11-attribute-plugins
description: Create or convert Drupal 11 plugins (Block, FieldType, FieldWidget, FieldFormatter, QueueWorker, Action, entity types, custom plugin types) using PHP attributes instead of deprecated annotations. Use when adding a plugin or migrating annotation-based plugins.
---

# Drupal 11 Attribute-Based Plugins

Annotation discovery is deprecated and will be removed in Drupal 12. Use PHP attributes for all new plugins and convert existing ones.

## Core attribute classes
| Plugin type | Attribute |
|---|---|
| Block | `Drupal\Core\Block\Attribute\Block` |
| Field type | `Drupal\Core\Field\Attribute\FieldType` |
| Field widget | `Drupal\Core\Field\Attribute\FieldWidget` |
| Field formatter | `Drupal\Core\Field\Attribute\FieldFormatter` |
| Queue worker | `Drupal\Core\Queue\Attribute\QueueWorker` |
| Action | `Drupal\Core\Action\Attribute\Action` |
| Content entity type | `Drupal\Core\Entity\Attribute\ContentEntityType` |
| Config entity type | `Drupal\Core\Entity\Attribute\ConfigEntityType` |
| Form / render element | `Drupal\Core\Render\Attribute\FormElement` / `RenderElement` |

Verify the exact namespace for your core minor version in the API docs; some attributes arrived in later 11.x releases.

## Rules
1. Use `new TranslatableMarkup('...')` for translatable attribute values.
2. Keep plugin IDs prefixed with the module name.
3. Plugin classes are usually `final` unless designed for extension.
4. Use explicit return types and `#[\Override]` where overriding.
5. Do not mix annotation and attribute on one class.

## Example: Block
```php
<?php

declare(strict_types=1);

namespace Drupal\my_module\Plugin\Block;

use Drupal\Core\Block\Attribute\Block;
use Drupal\Core\Block\BlockBase;
use Drupal\Core\StringTranslation\TranslatableMarkup;

/**
 * Provides an example block.
 */
#[Block(
  id: 'my_module_example',
  admin_label: new TranslatableMarkup('Example block'),
  category: new TranslatableMarkup('My Module'),
)]
final class ExampleBlock extends BlockBase {

  /**
   * {@inheritdoc}
   */
  #[\Override]
  public function build(): array {
    return [
      '#markup' => $this->t('Hello'),
    ];
  }

}
```

## Converting an annotation plugin
1. Replace the `@Block(...)` docblock with the `#[Block(...)]` attribute above the class.
2. Convert `@Translation("...")` to `new TranslatableMarkup('...')`.
3. Convert annotation keys to named arguments (same names).
4. Remove the annotation block; keep the human-readable class docblock.
5. Clear caches and confirm the plugin still appears (`drush cr`, then the relevant UI).
Drupal Rector can automate much of this; run it, then review the diff.

## Custom plugin type
1. Create an attribute class extending `Drupal\Component\Plugin\Attribute\Plugin`:
```php
#[\Attribute(\Attribute::TARGET_CLASS)]
final class MyPlugin extends Plugin {

  public function __construct(
    public readonly string $id,
    public readonly TranslatableMarkup $label,
    public readonly ?string $deriver = NULL,
  ) {}

}
```
2. In the plugin manager constructor, pass the attribute class name to `parent::__construct()` in place of the annotation class. Set `alterInfo()` and `setCacheBackend()` as usual.
3. Register the manager as a service in `MODULE.services.yml`, and add the plugin interface and base class.
4. Document the plugin type and add tests.

## Checklist
- [ ] No `@Annotation` docblocks on new or converted plugins
- [ ] `TranslatableMarkup` used in attributes
- [ ] Plugin ID prefixed with module name
- [ ] Cache cleared and plugin discovered
