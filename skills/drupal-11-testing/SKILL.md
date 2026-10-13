---
name: drupal-11-testing
description: Write and run Drupal 11 tests with PHPUnit 10+ attributes (Unit, Kernel, Functional, FunctionalJavascript), plus phpcs and phpstan. Use when adding tests, fixing failing tests, or setting up quality checks for a module.
---

# Drupal 11 Testing

## Choose the lightest test type
`UnitTestCase` > `KernelTestBase` > `BrowserTestBase` > `WebDriverTestBase`. Use a heavier type only when the behavior requires it (entities, database, rendering, JS).

## PHPUnit attributes (not annotations)
Docblock annotations are deprecated in PHPUnit 10 and removed in 12.
```php
<?php

declare(strict_types=1);

namespace Drupal\Tests\my_module\Kernel;

use Drupal\KernelTests\KernelTestBase;
use Drupal\my_module\Service\ExampleManager;
use PHPUnit\Framework\Attributes\CoversClass;
use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\Attributes\Group;

/**
 * Tests the example manager.
 */
#[Group('my_module')]
#[CoversClass(ExampleManager::class)]
final class ExampleManagerTest extends KernelTestBase {

  /**
   * {@inheritdoc}
   */
  protected static $modules = ['system', 'user', 'my_module'];

  /**
   * Tests value normalization.
   */
  #[DataProvider('provideValues')]
  public function testNormalize(string $input, string $expected): void {
    $manager = $this->container->get(ExampleManager::class);
    $this->assertSame($expected, $manager->normalize($input));
  }

  /**
   * Data provider.
   *
   * @return array<string, array{string, string}>
   */
  public static function provideValues(): array {
    return [
      'trims'     => [' a ', 'a'],
      'lowercase' => ['A', 'a'],
    ];
  }

}
```

## Rules
- Replace `@group`, `@covers`, `@dataProvider` with `#[Group]`, `#[CoversClass]`, `#[DataProvider]`
- Data providers must be `public static`
- Set `protected static $modules`; for functional tests also `protected $defaultTheme = 'stark';`
- Test files live in `tests/src/{Unit,Kernel,Functional,FunctionalJavascript}/`
- Use `declare(strict_types=1);`, explicit types, and docblocks

## Commands
Adjust paths to the project's docroot.
```bash
# PHPUnit
vendor/bin/phpunit -c web/core web/modules/custom/my_module/tests/src/Kernel

# Coding standards
vendor/bin/phpcs --standard=Drupal,DrupalPractice --extensions=php,module,inc,install,theme,yml web/modules/custom/my_module
vendor/bin/phpcbf --standard=Drupal,DrupalPractice web/modules/custom/my_module

# Static analysis (mglaman/phpstan-drupal), level 6 or higher
vendor/bin/phpstan analyse web/modules/custom/my_module
```

## Checklist
- [ ] Attributes, not annotations
- [ ] Static data providers
- [ ] Lightest suitable base class
- [ ] phpcs and phpstan clean
