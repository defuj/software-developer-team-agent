---
name: dart-run-static-analysis
description: Execute `dart analyze` to identify warnings and errors, and use `dart fix --apply` to automatically resolve mechanical lint issues. Use during development to ensure code quality and before committing changes.
metadata:
  model: models/gemini-3.1-pro-preview
  last_modified: Fri, 24 Apr 2026 15:09:34 GMT
---

# Analyzing and Fixing Dart Code

## Contents

- [Analysis Configuration](#analysis-configuration)
- [Diagnostic Suppression](#diagnostic-suppression)
- [Workflow: Executing Static Analysis](#workflow-executing-static-analysis)
- [Workflow: Applying Automated Fixes](#workflow-applying-automated-fixes)
- [Examples](#examples)

## Analysis Configuration

Configure the Dart analyzer using the `analysis_options.yaml` file located at the package root.

- **Base Configuration:** Always include a standard rule set (e.g., `package:lints/recommended.yaml` or `package:flutter_lints/flutter.yaml`) using the `include:` directive.
- **Strict Type Checks:** Enable strict type checks under the `analyzer: language:` node to prevent implicit downcasts and dynamic inferences. Set `strict-casts: true`, `strict-inference: true`, and `strict-raw-types: true`.
- **Linter Rules:** Explicitly enable or disable specific rules under the `linter: rules:` node. Use a key-value map (`rule_name: true/false`) when overriding included rules, or a list (`- rule_name`) when defining a fresh set. Do not mix list and map syntax in the same `rules` block.
- **Formatter Configuration:** Configure `dart format` behavior under the `formatter:` node. Set `page_width` (default 80) and `trailing_commas` (`automate` or `preserve`).
- **Analyzer Plugins:** Enable custom diagnostics by adding plugins under the `analyzer: plugins:` node. Ensure the plugin package is added as a `dev_dependency` in `pubspec.yaml`.

## Diagnostic Suppression

When a diagnostic (lint or warning) yields a false positive or applies to generated code, suppress it explicitly.

- **File-level Exclusion:** Use the `analyzer: exclude:` node in `analysis_options.yaml` to exclude entire files or directories (e.g., `**/*.g.dart`) using glob patterns.
- **File-level Suppression:** Add `// ignore_for_file: <diagnostic_code>` at the top of a Dart file to suppress specific diagnostics for the entire file. Use `// ignore_for_file: type=lint` to suppress all linter rules.
- **Line-level Suppression:** Add `// ignore: <diagnostic_code>` on the line directly above the offending code, or appended to the end of the offending line.
- **Pubspec Suppression:** Add `# ignore: <diagnostic_code>` above the offending line in `pubspec.yaml` files (e.g., `# ignore: sort_pub_dependencies`).
- **Plugin Diagnostics:** Prefix the diagnostic code with the plugin name when suppressing plugin-specific issues (e.g., `// ignore: some_plugin/some_code`).

## Workflow: Executing Static Analysis

Use this workflow to identify type-related bugs, style violations, and potential runtime errors.

**Task Progress:**

- [ ] 1. Verify `analysis_options.yaml` exists at the project root.
- [ ] 2. Run the analyzer using the `analyze_files` MCP tool (if available) or the CLI command `dart analyze <target_directory>`.
- [ ] 3. Review the diagnostic output.
- [ ] 4. If info-level issues must be treated as failures, append the `--fatal-infos` flag.
- [ ] 5. Resolve reported errors manually or proceed to the Automated Fixes workflow.

## Workflow: Applying Automated Fixes

Use this workflow to resolve outdated API usages, apply quick fixes, and migrate code (e.g., Dart 3 migrations).

**Task Progress:**

- [ ] 1. Execute a dry run to preview proposed changes using the `dart_fix` MCP tool or CLI command `dart fix --dry-run`.
- [ ] 2. Review the proposed fixes to ensure they align with the intended architecture.
- [ ] 3. If additional fixes are required, verify that the corresponding linter rules are enabled in `analysis_options.yaml`.
- [ ] 4. Apply the fixes using the `dart_fix` MCP tool or CLI command `dart fix --apply`.
- [ ] 5. Format the modified code using the `dart_format` MCP tool or CLI command `dart format .`.
- [ ] 6. Run the static analysis workflow to verify all diagnostics are resolved.

## Examples

### Production-Grade `analysis_options.yaml` (Flutter)

This 231-rule template covers the core Dart lint rules, Flutter-specific rules, and additional rules from the SonarQube Dart rule set. Copy this as a starting point and adjust based on your project's needs.

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    use_build_context_synchronously: ignore
  exclude:
    - build/**
    - coverage/**
    - .dart_tool/**
    - .fvm/**
    - android/**
    - ios/**
    - macos/**
    - linux/**
    - windows/**
    - web/**
    - assets/**
    - test/** # Remove if you want lint enforced in tests

linter:
  rules:
    # ── Core style & correctness ──
    prefer_single_quotes: true
    always_declare_return_types: true
    always_put_control_body_on_new_line: true
    always_put_required_named_parameters_first: true
    annotate_overrides: true
    avoid_dynamic_calls: false # Allow dynamic when needed
    avoid_print: true # Use debugPrint instead
    avoid_empty_else: true
    avoid_function_literals_in_foreach_calls: true
    avoid_init_to_null: true
    avoid_positional_boolean_parameters: true
    avoid_private_typedef_functions: true
    avoid_redundant_argument_values: true
    avoid_return_types_on_setters: true
    avoid_single_cascade_in_expression_statements: true
    avoid_types_as_parameter_names: true
    avoid_unused_constructor_parameters: true
    avoid_void_async: true
    camel_case_extensions: true
    camel_case_types: true
    cancel_subscriptions: true
    cascade_invocations: true
    close_sinks: true
    comment_references: true
    constant_identifier_names: true
    control_flow_in_finally: true
    curly_braces_in_flow_control_structures: true
    diagnostic_describe_all_properties: true # One-sentence diagnostics
    directives_ordering: true
    empty_catches: true
    empty_constructor_bodies: true
    empty_statements: true
    file_names: true
    hash_and_equals: true
    implementation_imports: true
    join_return_with_assignment: true
    library_names: true
    library_prefixes: true
    lines_longer_than_80_chars: false # Disable if using wider format
    literal_only_boolean_expressions: true
    no_adjacent_strings_in_list: true
    non_constant_identifier_names: true
    null_closures: true
    omit_local_variable_types: true # Reduces noise in local vars
    one_member_abstracts: true
    only_throw_errors: true
    overridden_fields: true
    package_names: true
    parameter_assignments: true
    prefer_adjacent_string_concatenation: true
    prefer_collection_literals: true
    prefer_conditional_assignment: true
    prefer_const_constructors: true
    prefer_const_constructors_in_immutables: true
    prefer_const_declarations: true
    prefer_const_literals_to_create_immutables: true
    prefer_contains: true
    prefer_expression_function_bodies: true
    prefer_final_fields: true
    prefer_final_in_for_each: true
    prefer_final_locals: true
    prefer_foreach: true
    prefer_function_declarations_over_variables: true
    prefer_generic_function_type_aliases: true
    prefer_if_elements_to_conditional_expressions: true
    prefer_initializing_formals: true
    prefer_interpolation_to_compose_strings: true
    prefer_is_empty: true
    prefer_is_not_empty: true
    prefer_is_not_operator: true
    prefer_iterable_whereType: true
    prefer_mixin: true
    prefer_null_aware_method_calls: true
    prefer_null_aware_operators: true
    prefer_relative_imports: true # Cleaner imports
    prefer_spread_collections: true
    prefer_typing_uninitialized_variables: true
    provide_deprecation_message: true
    public_member_api_docs: false # Skip doc requirement
    recursive_getters: true
    secure_pubsec_urls: true
    slash_for_doc_comments: true
    sort_child_properties_last: true # Flutter: child last in widgets
    sort_constructors_first: true
    sort_unnamed_constructors_first: true
    test_types_in_equals: true
    throw_in_finally: true
    type_annotate_public_apis: true
    type_init_formals: true
    unawaited_futures: true # Catch dangling futures
    unnecessary_await_in_return: true
    unnecessary_brace_in_string_interps: true
    unnecessary_const: true
    unnecessary_lambdas: true
    unnecessary_new: true
    unnecessary_null_aware_assignments: true
    unnecessary_null_in_if_null_operators: true
    unnecessary_overrides: true
    unnecessary_parenthesis: true
    unnecessary_statements: true
    unnecessary_string_escapes: true
    unnecessary_this: true
    unrelated_type_equality_checks: true
    use_full_hex_values_for_flutter_colors: true
    use_function_type_syntax_for_parameters: true
    use_key_in_widget_constructors: true
    use_rethrow_when_possible: true
    use_setters_to_change_properties: true
    use_string_buffers: true
    use_super_parameters: true
    valid_regexps: true

    # ── From SonarQube Dart rule set ──
    annotate_redeclares: true
    avoid_bool_literals_in_conditional_expressions: true
    avoid_catches_without_on_clauses: false
    avoid_catching_errors: true
    avoid_classes_with_only_static_members: true
    avoid_double_and_int_checks: true
    avoid_equals_and_hash_code_on_mutable_classes: true
    avoid_escaping_inner_quotes: true
    avoid_field_initializers_in_const_classes: true
    avoid_implementing_value_types: true
    avoid_js_rounded_ints: true
    avoid_multiple_declarations_per_line: true
    avoid_relative_lib_imports: true
    avoid_renaming_method_parameters: true
    avoid_returning_null_for_void: true
    avoid_returning_this: true
    avoid_shadowing_type_parameters: true
    avoid_slow_async_io: true
    avoid_type_to_string: true
    avoid_types_on_closure_parameters: false
    avoid_unnecessary_containers: true
    await_only_futures: true
    cast_nullable_to_non_nullable: true
    collection_methods_unrelated_type: true
    combinators_ordering: true
    conditional_uri_does_not_exist: true
    dangling_library_doc_comments: true
    depend_on_referenced_packages: true
    deprecated_consistency: true
    deprecated_member_use_from_same_package: true
    discarded_futures: false
    do_not_use_environment: false
    document_ignores: true
    eol_at_end_of_file: true
    exhaustive_cases: true # Critical for sealed class switches
    flutter_style_todos: true
    implicit_call_tearoffs: true
    implicit_reopen: true
    invalid_case_patterns: true
    invalid_runtime_check_with_js_interop_types: true
    leading_newlines_in_multiline_strings: true
    library_annotations: true
    library_private_types_in_public_api: true
    matching_super_parameters: true
    missing_code_block_language_in_doc_comment: true
    missing_whitespace_between_adjacent_strings: true
    no_default_cases: true # Forces exhaustive switch
    no_duplicate_case_values: true
    no_leading_underscores_for_library_prefixes: true
    no_leading_underscores_for_local_identifiers: true
    no_literal_bool_comparisons: true
    no_runtimeType_toString: true
    no_self_assignments: true
    no_wildcard_variable_uses: true
    noop_primitive_operations: true
    null_check_on_nullable_type_parameter: true
    omit_obvious_local_variable_types: true
    package_prefixed_library_names: true
    prefer_asserts_with_message: true
    prefer_constructors_over_static_methods: true
    prefer_for_elements_to_map_fromIterable: true
    prefer_if_null_operators: true
    prefer_inlined_adds: true
    prefer_int_literals: true
    prefer_void_to_null: true
    require_trailing_commas: false
    sized_box_for_whitespace: true
    sort_pub_dependencies: false
    tighten_type_of_initializing_formals: true
    type_literal_in_constant_pattern: true
    unintended_html_in_doc_comment: true
    unnecessary_breaks: true
    unnecessary_constructor_name: true
    unnecessary_getters_setters: true
    unnecessary_late: true
    unnecessary_library_directive: true
    unnecessary_library_name: true
    unnecessary_null_aware_operator_on_extension_on_nullable: true
    unnecessary_null_checks: true
    unnecessary_nullable_for_final_variable_declarations: true
    unnecessary_raw_strings: true
    unnecessary_string_interpolations: true
    unnecessary_to_list_in_spreads: true
    unreachable_from_main: true
    use_colored_box: true
    use_decorated_box: true
    use_enums: true
    use_is_even_rather_than_modulo: true
    use_late_for_private_fields_and_variables: true
    use_named_constants: true
    use_raw_strings: true
    use_string_in_part_of_directives: true
    use_test_throws_matchers: true
    use_to_and_as_if_applicable: true
    use_truncating_division: true
    void_checks: true

formatter:
  page_width: 100
  trailing_commas: preserve
```

### Minimal `analysis_options.yaml` (Simple Projects)

```yaml
include: package:flutter_lints/recommended.yaml

analyzer:
  language:
    strict-casts: true
    strict-inference: true
    strict-raw-types: true

linter:
  rules:
    prefer_single_quotes: true
    prefer_const_constructors: true
    prefer_const_declarations: true
    avoid_print: true
    unawaited_futures: true
    use_super_parameters: true
    exhaustive_cases: true

formatter:
  page_width: 100
  trailing_commas: preserve
```

### Inline Diagnostic Suppression

```dart
// Suppress for the entire file
// ignore_for_file: unused_local_variable, dead_code

void processData() {
  // Suppress for a specific line
  // ignore: invalid_assignment
  int x = '';

  const y = 10; // ignore: constant_identifier_names
}
```
