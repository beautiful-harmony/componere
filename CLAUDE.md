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
- Empty `.diff` files indicate test passes

## Critical Implementation Details

### Memory Management & Reference Counting
Based on PHP Internals best practices:
- Always use proper `ZVAL_COPY()` for zval copying with reference counting
- Use `zval_ptr_dtor()` for proper cleanup to avoid memory leaks
- Follow PHP's copy-on-write semantics for shared structures
- Use `Z_TRY_ADDREF()` for safe reference counting on potentially immutable values

### Static Property Management
Critical for proper static property functionality:
- Static members table must be properly initialized during class registration
- For PHP 7.4+: Use `ZEND_MAP_PTR` macros for thread-safe static member access
- For PHP < 7.4: Direct assignment to `static_members_table` 
- Always allocate and copy from `default_static_members_table` during registration
- Handle edge cases where `default_static_members_count` might be inconsistent

### Object Casting & Property Handling
Inheritance-aware property copying:
- Initialize target object with default values first
- Copy source properties only for inherited/compatible properties
- Use name-based property matching for inheritance relationships
- Avoid duplicate property copying through multiple code paths

### Method Replacement & Scope Management
For proper method functionality:
- Preserve all signature-related flags (`ZEND_ACC_HAS_RETURN_TYPE`, etc.)
- Set proper scope (`function->common.scope`) during method addition
- For closures: Use original class entry for proper context binding
- Validate function types and reject internal functions for closure creation

### Reflection Integration
Proper reflection object creation:
- Initialize all required fields including `class` property for ReflectionMethod
- Use `zend_string_copy()` for class name assignment
- Handle both function and method reflection consistently

## Known Issues & Solutions

### Segmentation Faults
Common causes and fixes:
1. **getClosure**: Ensure proper instance binding and scope validation
2. **Static properties**: Missing static members table initialization  
3. **Property access**: NULL pointer dereference in property copying
4. **Memory cleanup**: Improper zval destruction order

### Type System Integration
- Return type validation requires preserved function flags
- Method scope must be properly linked to class entry
- Closure binding needs original class context for type checking

### PHP Version Compatibility
- Use conditional compilation for version-specific APIs
- Handle `zend_do_link_class` signature differences
- MAP_PTR handling varies between PHP versions
- Static member table structure differences

## Debug & Development Tips

### Test Analysis
- Empty `.diff` files = test passes
- Segfaults usually indicate memory management issues
- Type errors suggest scope/binding problems
- Property issues often relate to inheritance handling

### Common Debugging Commands
```bash
# Test specific functionality
make test TESTS=tests/043.phpt

# Check test status
find tests/ -name "*.diff" -exec sh -c 'if [ -s "$1" ]; then echo "FAILED: $1"; fi' _ {} \;

# Rebuild and test quickly
make && make test 2>/dev/null | grep -E "(PASS|FAIL)" | tail -10
```

### Reference Materials
- PHP Internals Book: Essential for understanding zval management and object handlers
- PHP source code: `Zend/zend_*.h` files for structure definitions
- Extension development: Focus on memory safety and reference counting