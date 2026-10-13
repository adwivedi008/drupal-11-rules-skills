---
name: drupal-11-dependency-injection
description: Register and inject services in Drupal 11 using autowiring, #[Autowire], AutowireTrait, interface aliases, and event subscribers. Use when creating a service, controller, form, or plugin that needs dependencies, or when removing static \Drupal:: calls.
---

# Drupal 11 Dependency Injection and Autowiring

## Services
Prefer autowired services with the FQCN as the service ID and an interface alias:
```yaml
services:
  _defaults:
    autowire: true
  Drupal\my_module\Service\ExampleManager: ~
  Drupal\my_module\Service\ExampleManagerInterface: '@Drupal\my_module\Service\ExampleManager'
```
Module-prefixed IDs (`my_module.example_manager`) remain acceptable where compatibility matters, but type-hint the interface in consumers.

## Resolving ambiguity
```php
use Symfony\Component\DependencyInjection\Attribute\Autowire;

public function __construct(
  #[Autowire(service: 'logger.channel.my_module')]
  private readonly LoggerInterface $logger,
  #[Autowire(param: 'my_module.default_limit')]
  private readonly int $limit,
  private readonly EntityTypeManagerInterface $entityTypeManager,
) {}
```
Use `#[Autowire]` when a type hint alone is ambiguous (logger channels, cache bins, container parameters).

## Controllers, forms, plugins
Use `Drupal\Core\DependencyInjection\AutowireTrait` to skip the boilerplate `create()` when the constructor is fully autowirable:
```php
final class ExampleController extends ControllerBase {

  use AutowireTrait;

  public function __construct(
    private readonly ExampleManagerInterface $manager,
  ) {}

}
```
Write an explicit `create()` (`ContainerInjectionInterface` or `ContainerFactoryPluginInterface`) only when you need arguments the container cannot autowire.

## Event subscribers
```php
final class ExampleSubscriber implements EventSubscriberInterface {

  public static function getSubscribedEvents(): array {
    return [
      KernelEvents::REQUEST => ['onRequest', 30],
    ];
  }

  public function onRequest(RequestEvent $event): void {}

}
```
Register with `tags: [{ name: event_subscriber }]` in services.yml. Prefer subscribers over hooks when an event exists.

## Rules
- Single responsibility per service; define an interface for any public contract
- Mark internal-only classes `@internal`
- Constructor promotion with `private readonly`
- Never call `\Drupal::service()`, `\Drupal::config()`, or `\Drupal::entityTypeManager()` in classes that can be injected; static calls are acceptable only in procedural glue (`.install`, legacy `.module`)
- Use lazy services (`lazy: true`) for expensive dependencies used rarely

## Checklist
- [ ] `autowire: true` set, interface alias added
- [ ] `#[Autowire]` used where type hints are ambiguous
- [ ] `create()` only when necessary
- [ ] No static service calls inside injectable classes
