# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Componere is a PHP extension that allows runtime composition/patching of classes. It provides powerful tools for modifying class definitions at runtime, including adding methods, properties, and interfaces to existing classes.

## Core Architecture

The extension is structured around several key components in the `src/` directory:

- **definition.c/h**: Core class definition manipulation functionality
- **patch.c/h**: Runtime class patching capabilities  
- **method.c/h**: Method addition and modification
- **value.c/h**: Property/constant handling
- **cast.c/h**: Type casting utilities
- **reflection.c/h**: Reflection API integration
- **common.h**: Shared utilities and PHP version compatibility macros

## Build System

This is a standard PHP extension using autotools:

### Development Commands
```bash
# Configure and build
phpize
./configure --enable-componere
make

# Run tests
make test

# Clean build artifacts
make clean
```

### Version Compatibility
- Requires PHP 7.1+
- Code uses conditional compilation for PHP 8.0+ compatibility
- Version-specific arginfo headers are generated from .stub.php files

## Code Generation

The extension uses PHP's stub system for generating argument info:
- `.stub.php` files in `src/` define function signatures
- `*_arginfo.h` files are auto-generated (don't edit manually)
- Both legacy and modern arginfo variants exist for PHP version compatibility

## Testing

Uses PHP's standard test framework:
- Tests are in `tests/` directory as `.phpt` files
- Test execution via `make test` or `php run-tests.php`
- Failed tests generate `.diff`, `.exp`, `.log`, and `.out` files

## Key Implementation Notes

- Heavy use of PHP version compatibility macros in `common.h`
- Object handlers are customized to deny property access on core objects
- Extension integrates with PHP's optimizer (OPcache) by adjusting optimization levels
- Uses libtool for shared library building