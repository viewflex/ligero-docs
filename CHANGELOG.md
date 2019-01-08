# Changelog

This document tracks changes in release versions of **Ligero**. This project adheres to the [Semantic Versioning](http://semver.org/spec/v2.0.0.html) standard.

Added demo ItemsContext to default config.


## Version [1.1.1](https://github.com/viewflex/ligero/tree/1.1.1) - 2019-01-08

### Fixed

- Added missing demo ItemsContext to contexts config array.


## Version [1.1](https://github.com/viewflex/ligero/tree/1.1) - 2019-01-03

### Added

- Option to create Publisher with no arguments.
- Declarative configuration option for Publisher.

### Changed

- Config, Request, and Query are now attributes only on PublisherApi.
- PublisherApi->getQueryView() now to defaults to 'list'.
- Renamed Rules and PostRules vars and methods to QueryRules and RequestRules.
- Assorted minor refactoring, reorg, updated comments.
- Major documentation revisions.

### Removed

- Ability to overwrite validation rules for reserved parameters.
- Demo Config, Request, and Query classes.


## Version [1.0.3](https://github.com/viewflex/ligero/tree/1.0.3) - 2017-12-31

### Added

- Abstract methods getContext() and ContextInputsAreValid() to AggregateContext for clarity.
- Context test suite.
- Comments for formatting methods.

### Changed

- BasePublisherController, ContextApiController, BaseContext methods refactored to traits.
- Readme updates.
- Test cleanup.

### Fixed

- Missing validation rule for price column in demo context and request.


## Version [1.0.2](https://github.com/viewflex/ligero/tree/1.0.2) - 2017-12-14

### Changed

- Publisher and PublisherApi methods refactored to traits for portability.
- Demo routes file must now be published to be registered.
- PSR-2, DRY'd up comments.

## Version [1.0.1](https://github.com/viewflex/ligero/tree/1.0.1) - 2017-11-29

### Added

- Changelog link in readme.

### Changed

- Organization of translation strings into nested arrays.

### Removed

- Publisher initialize() stub and some chaff comments.


## Version [1.0](https://github.com/viewflex/ligero/tree/1.0) - 2017-11-22

Initial public release.