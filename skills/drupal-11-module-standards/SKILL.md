---
name: drupal-11-module-standards
description: Drupal 11 module-level standards - hook_schema and update hooks, events vs hooks, forms, translation, configuration and config schema, cacheability metadata, and custom extension points with api.php. Use when adding schema or updates, forms, config, caching, translatable strings, or extension points to a module.
---

# Drupal 11 Module Standards

## Schema and updates (procedural, in `.install`)
```php
/**
 * Implements hook_schema().
 */
function my_module_schema(): array {
  $schema['my_module_item'] = [
    'description' => 'Stores items.',
    'fields'      => [
      'id'    => ['type' => 'serial', 'unsigned' => TRUE, 'not null' => TRUE],
      'label' => ['type' => 'varchar', 'length' => 255, 'not null' => TRUE, 'default' => ''],
    ],
    'primary key' => ['id'],
  ];
  return $schema;
}

/**
 * Adds the "weight" column to my_module_item.
 */
function my_module_update_10001(): void {
  \Drupal::database()->schema()->addField('my_module_item', 'weight', [
    'type'     => 'int',
    'not null' => TRUE,
    'default'  => 0,
  ]);
}
```
- One update hook per schema change; never edit a shipped update hook
- Static `\Drupal::` calls are acceptable here (procedural `.install` glue)

## Events over hooks
Prefer Symfony events (`EventSubscriberInterface`) when core or contrib dispatches one. Use hooks (`#[Hook]`, see `drupal-11-oop-hooks`) when no event exists.

## Forms
- Extend `FormBase` or `ConfigFormBase`; implement `getFormId()`, `buildForm()`, `validateForm()`, `submitForm()`
- Validate in `validateForm()`, never in `submitForm()`
- Use `$form_state->setErrorByName()` for validation errors

```php
public function validateForm(array &$form, FormStateInterface $form_state): void {
  if (mb_strlen((string) $form_state->getValue('label')) > 255) {
    $form_state->setErrorByName('label', $this->t('The label is too long.'));
  }
}
```

## Translation
- All user-facing strings go through `$this->t()` (with `StringTranslationTrait`), `TranslatableMarkup`, or `formatPlural()`
- Use placeholders (`@name`, `%name`, `:url`), never string concatenation
- Use literal strings in `t()` so they can be extracted

## Configuration
- Ship defaults in `config/install` (always) and `config/optional` (when dependencies may be absent)
- Every config object needs a schema in `config/schema/MODULE.schema.yml`
- Read with `$this->configFactory->get('my_module.settings')` (immutable); edit with `getEditable()` only when saving
- Never store environment-specific secrets in exported config

## Cacheability
Bubble cache metadata on render arrays and responses:
```php
use Drupal\Core\Cache\CacheableMetadata;

$build = ['#markup' => $this->t('Hello')];
CacheableMetadata::createFromObject($config)
  ->addCacheTags(['my_module:list'])
  ->addCacheContexts(['user.permissions'])
  ->applyTo($build);
```
- Use cache tags for invalidation, contexts for variation, max-age only when needed
- Invalidate with `Cache::invalidateTags()` or the cache tags invalidator service

## Extension points
- Provide extensibility through events, plugins, or alter hooks
- For alter hooks use `ModuleHandlerInterface::alter()`; for collecting hooks use `invokeAll()`
- Document every custom hook in `MODULE.api.php`

## Checklist
- [ ] Schema changes have a numbered update hook
- [ ] Forms validate in `validateForm()`
- [ ] Strings translated with placeholders
- [ ] Config has defaults and a schema
- [ ] Cache tags/contexts bubbled; custom hooks documented in `api.php`
