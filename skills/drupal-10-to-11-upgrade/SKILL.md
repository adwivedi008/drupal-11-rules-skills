---
name: drupal-10-to-11-upgrade
description: Upgrade a custom Drupal module or project from Drupal 10 to Drupal 11, covering version constraints, deprecation removal, Rector, hooks, plugins, tests, and removed core modules. Use when migrating to Drupal 11 or making a module D11 compatible.
---

# Drupal 10 to 11 Upgrade

Assume the site is on the latest Drupal 10.x (10.3+ recommended) before starting.

## Steps
1. **Audit.** Install and run Upgrade Status:
   ```bash
   composer require --dev drupal/upgrade_status
   drush en upgrade_status -y
   drush upgrade_status:analyze my_module
   ```
2. **Constraints.**
   - `MODULE.info.yml`: `core_version_requirement: ^11` (or `^10.3 || ^11` if dual support is required)
   - `composer.json`: `"php": ">=8.3"`, `"drupal/core": "^11"`
3. **Automate fixes.** Run Drupal Rector, then review every change:
   ```bash
   composer require --dev palantirnet/drupal-rector
   vendor/bin/rector process web/modules/custom/my_module
   ```
4. **Manual conversions** (see the matching skills):
   - Annotation plugins to attributes (`drupal-11-attribute-plugins`)
   - Procedural hooks to `#[Hook]` classes when minimum core is 11.1+ (`drupal-11-oop-hooks`)
   - Static `\Drupal::` calls to injected services (`drupal-11-dependency-injection`)
   - PHPUnit annotations to attributes (`drupal-11-testing`)
5. **Removed or moved functionality.**
   - Core modules removed (Statistics, Book, Forum, Tracker, Aggregator, Actions UI): require the contrib versions
   - jQuery UI components removed from core: use native JS or contrib libraries
   - `jQuery.once` to `once()`
   - Twig 3 deprecated filters/tests
6. **Dependencies.** Run `composer why-not drupal/core 11` to find blocking contrib modules; update or patch them.
7. **Verify.** Run `drush updb -y`, `drush cr`, the full test suite, phpcs, and phpstan. Fix remaining deprecation notices.

## Pitfalls
- Do not bump `core_version_requirement` to `^11` before dependencies are ready
- Do not remove legacy hook functions until the minimum core supports OOP hooks
- Keep `hook_update_N()` and `hook_schema()` in `.install`
- Check change records on drupal.org for the specific minor version you target

## Checklist
- [ ] Upgrade Status reports no errors
- [ ] PHP 8.3 and Drupal 11 constraints set
- [ ] Rector reviewed, manual conversions done
- [ ] Removed modules and libraries replaced
- [ ] Tests, phpcs, phpstan pass
