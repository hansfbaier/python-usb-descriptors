# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!--
## [Unreleased]
-->

## [0.9.2] - 2025-08-20
### Added
- Emitter for Interface Association Descriptors.
- Missing partial helper for `InterfaceAssociationDescriptor`.
### Changed
- Interface Association Descriptor's function string is now optional.
- Use pid.codes VID/PID in unit tests.
### Fixed
- String escaping for Microsoft OS descriptors.
### Removed
- Dropped support for Python 3.8
### Security
- Bumped jinja2 to 3.1.6


## [0.9.1] - 2024-06-21
### Added
- USB Standard Feature Selectors per [USB 2.0: Table 9-6]


## [0.9.0] - 2024-06-04
### Added
- Initial release

[Unreleased]: https://github.com/greatscottgadgets/python-usb-protocol/compare/0.9.0...HEAD
[0.9.1]: https://github.com/greatscottgadgets/python-usb-protocol/compare/0.9.0...0.9.1
[0.9.0]: https://github.com/greatscottgadgets/python-usb-protocol/releases/tag/0.9.0
