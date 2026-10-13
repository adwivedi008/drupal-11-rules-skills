---
name: drupal-11-frontend-theming
description: Build Drupal 11 frontend and theming - Twig 3, Single Directory Components (SDC), libraries, JavaScript behaviors with once(), preprocess, and avoiding removed jQuery UI and deprecated APIs. Use when writing Twig templates, components, CSS/JS libraries, or theme code.
---

# Drupal 11 Frontend and Theming

## Twig 3
- Avoid deprecated Twig features: the `spaceless` filter, the `if` modifier on `for` loops (use `|filter`), and other items flagged by `upgrade_status`
- Never use `|raw` on untrusted values; rely on autoescape
- Keep logic in preprocess or services, not in templates
- Use `{{ attributes }}`, `{{ title_attributes }}`, and `|t` for strings

## Single Directory Components (SDC)
Prefer SDC for reusable UI. Layout:
```
components/card/
├── card.component.yml
├── card.twig
├── card.css
└── card.js
```
```yaml
# card.component.yml
$schema: https://git.drupalcode.org/project/drupal/-/raw/HEAD/core/assets/schemas/v1/metadata.schema.json
name: Card
status: stable
props:
  type: object
  required: [title]
  properties:
    title:
      type: string
      title: Title
slots:
  body:
    title: Body
```
```twig
{# card.twig #}
<article{{ attributes.addClass('card') }}>
  <h2>{{ title }}</h2>
  {% block body %}{{ body }}{% endblock %}
</article>
```
Use it from Twig: `{{ include('my_module:card', { title: 'Hello' }) }}` (component ID is `provider:component`). Define props with JSON schema; keep CSS/JS next to the component so it auto-attaches.

## Libraries
Declare assets in `MODULE.libraries.yml` and attach them where needed:
```yaml
example:
  version: 1.x
  css:
    component:
      css/example.css: {}
  js:
    js/example.js: {}
  dependencies:
    - core/drupal
    - core/once
```
```php
$build['#attached']['library'][] = 'my_module/example';
```
- No inline scripts or styles
- Attach libraries to the smallest render element that needs them

## JavaScript
- Use `Drupal.behaviors` with `once()`; `jQuery.once` is removed
- Prefer native JS and `fetch` over jQuery
- jQuery UI components were removed from core; use native elements, `<dialog>`, or a declared contrib/library dependency

```js
((Drupal, once) => {
  Drupal.behaviors.myModuleExample = {
    attach(context) {
      once('my-module-example', '.example', context).forEach((el) => {
        el.addEventListener('click', () => el.classList.toggle('is-active'));
      });
    },
  };
})(Drupal, once);
```

## Preprocess and templates
- Declare theme hooks in `hook_theme()` (or its OOP equivalent on your minimum core version) with `variables` and a template
- Put preprocess logic in `#[Hook('preprocess_...')]` classes where your core version supports it, otherwise in procedural `template_preprocess_*` / `MODULE_preprocess_*`; verify against change records for your minor version
- Pass cache metadata in preprocess when variables depend on context

## Accessibility
- Use semantic HTML and ARIA only when needed
- Keep focus management and keyboard support in JS behaviors

## Checklist
- [ ] No deprecated Twig features, no unescaped untrusted output
- [ ] Reusable UI built as SDC
- [ ] Libraries declared, no inline assets
- [ ] JS uses `Drupal.behaviors` + `once()`; no jQuery UI
