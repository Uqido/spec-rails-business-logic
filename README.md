# Spec: Rails Business Logic v0.0.1

A set of rules and conventions for standardizing the modularization of *Ruby on Rails* business logic.

## Dependencies

- https://github.com/Uqido/spec-specification-repository/tree/0.0.1

## Adoption Impact

This specification standardizes the modularization of *Ruby on Rails* business logic, with the aim to:
- Help both human and AI reviewers to identify and enforce frequently encountered issues in Rails codebases
- Encourage the decoupling of business logic from MVC components, allowing controllers, models, and views to focus solely on their intended responsibilities
- Improve the testability of business logic by organizing it into simple, top-down, dependency-injectable components

## Normative Content

Business logic **shall** be encapsulated in dedicated classes, rather than scattered across models and controllers.
This typically follows the Command Query Separation (CQS) pattern.
More specifically, business logic classes **shall** adhere to one of the following conventions:

- *Services*: **shall** have one or more methods with any semantics, with dependencies obtained at construction time via
  dependency injection (*). The constructor **shall** be side-effects free and taking *only* zero or more keyword arguments.
  Named as XyzService. The name **should** express the responsibility it's supposed to fulfill, such as TranslationService, etc...
  Methods having no logical return value **shall** return `self`.

- *Commands*: as services, but they **shall** have exactly one public method named `call`, taking no arguments, pure wrt to
  the initializer arguments. Named as XyzCommand. The name **should** express the side effect it's supposed to result in, such
  as UpdateMyFoobarCommand, etc...

- *Getters*: as commands, but its call method **shall** be side-effects free(**) and reentrant. Named as XyzGetter. The name
  **should** express the returned content, such as FoobarnessGetter, etc...

- *Factories*: as services, but they **shall** have exactly one public method named `create`, taking *only* zero or more keyword
  arguments, pure wrt to the initializer arguments, with the sole responsibility of instantiating a concrete class.
  Named as XyzFactory, where Xyz is the name of such class.

Class authors **should** give precedence, in order: Getters, Commands, and then Services.

A few notes:

- (**) By side-effects-free we mean as per the observable state only; side-effects to things such as caching arguments and such
  are allowed, provided it's clear what's what..

- (*) Dependencies **should** be passed as instances (or factory instances), don't pass classes. If avoiding allocations is crucial
  for efficiency sake, we can always use cache arguments where appropriate.

- Usage of rails initializers for managing services defaults is discouraged; use factories and env vars instead. The reason being
  that factories play well with auto-loading and it's easier to keep track of critical env var in docker compose files.

- Unless specified otherwise, services (and hence commands and getters) are neither transactional, nor exception safe nor
  synchronized. Anyway, naming **should** not suggest surprising behaviour to an average Rails user; for example, given an
  active model Foo, NewFooGetter should save no record on success, CreateFooCommand should instead, and so on ...

- Unless specified otherwise, public methods of services (and hence commands and getters) taking and returning ActiveRecords or
  ActiveRelation objects should take care of keeping them in a cache coherent state; in other words, `reloads` on service
  arguments and return values **should** *never*, ever be a caller responsibility (of course, this does not apply to parent
  associations, such as passing an `parent.children` is allowed to leave `parent` in a cache incoherent state).
  To put it yet in an another way, it **should** always be crystal clear whether a reload is needed or not as a side-effect of
  calling a service method.

Business logic classes **should** be defined under the `business_logic` directory under `app` (*zeitwerk* will manage
autoload, no extra configuration needed).
