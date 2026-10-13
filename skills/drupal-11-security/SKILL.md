---
name: drupal-11-security
description: Apply Drupal 11 security practices - output sanitization, render arrays and Twig autoescape, entity query access checks, query placeholders, input validation, CSRF, and route access. Use when handling user input, building queries, rendering markup, adding routes or forms, or reviewing code for vulnerabilities.
---

# Drupal 11 Security

## Output
- Rely on render arrays and Twig autoescape; never print raw user data
- `#markup` only for trusted strings; use `#plain_text` for anything user-controlled
- Use `Xss::filter()` / `Html::escape()` only when render arrays cannot do the job
- Never use `|raw` in Twig on untrusted values; never build HTML by concatenating user input

```php
// Bad
$build['#markup'] = '<p>' . $request->query->get('name') . '</p>';

// Good
$build['name'] = [
  '#type'       => 'html_tag',
  '#tag'        => 'p',
  '#value'      => $name,
];
// Or
$build['#plain_text'] = $name;
```

## Queries
- Use the Database API with placeholders for every dynamic value; never interpolate into SQL
- Always set `accessCheck()` explicitly on entity queries

```php
// Entity query: explicit access check.
$ids = $this->entityTypeManager
  ->getStorage('node')
  ->getQuery()
  ->accessCheck(TRUE)
  ->condition('type', 'article')
  ->range(0, 10)
  ->execute();

// Database API: placeholders.
$result = $this->connection->select('my_table', 't')
  ->fields('t', ['id', 'label'])
  ->condition('t.uid', $uid)
  ->execute();

// Raw expressions: named placeholders only.
$this->connection->query('SELECT id FROM {my_table} WHERE label = :label', [':label' => $label]);
```
Use `accessCheck(FALSE)` only for deliberate, documented system-level operations.

## Input
- Validate and sanitize all input (query string, POST, headers, uploaded files)
- Use Form API validation (`validateForm()`) and typed config/entity field constraints
- Cast and whitelist values (`match`, enums, `in_array(..., TRUE)`)
- For uploads, restrict extensions, use managed files, and never trust client MIME types

## Access and CSRF
- Every route needs `_permission`, `_entity_access`, or a custom access checker
- Check entity operations with `$entity->access('update', $account)`
- Forms get CSRF protection automatically through Form API; state-changing GET routes need `_csrf_token: 'TRUE'`
- Generate CSRF-protected links with `Url::fromRoute()`, not hand-built URLs

## Secrets and logging
- Never hardcode credentials; use environment variables, `settings.php`, or a secrets manager
- Never log passwords, tokens, or full request bodies
- Do not expose stack traces or internal paths in user-facing messages

## Checklist
- [ ] No unescaped user data in markup
- [ ] Placeholders in all queries, `accessCheck()` set on entity queries
- [ ] Routes have access requirements; state-changing GET routes have CSRF tokens
- [ ] Input validated and whitelisted
- [ ] No secrets in code or logs
