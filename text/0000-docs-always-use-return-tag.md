- Start Date: 2018-03-28
- RFC PR: (leave this empty)
- WordPress Coding Standards Issue: (leave this empty)

# Summary

The current WordPress documentation standard explicitely forbids/discourages the use of `@return void`.
This RFC proposes to always demand a `@return` tag in all function and method docblocks to demonstrate and document intent.

# Basic example

```php
/**
 * Example function with a return statement returning a value.
 *
 * @return float Description.
 */
function my_function( $a, $b ) {
	return floor( max( $a, $b ) );
}
```

```php
/**
 * Example function without a return statement.
 *
 * @return void
 */
function my_function() {
	if ( condition ) {
		// Do something.
	}
}
```

```php
/**
 * Example function with a void return statement.
 *
 * @return void
 */
function my_function() {
	if ( unfulfilled condition ) {
		return;
	}
	
	// Do something.
}
```

```php
/**
 * Example function with mixed returns, including null.
 *
 * @return int|null Description.
 */
function my_function() {
	if ( unfulfilled condition ) {
		return null;
	}

	// Do something.
	return 1;
}
```

# Status Quo

The [WordPress PHP Documentation standards](https://make.wordpress.org/core/handbook/best-practices/inline-documentation-standards/php/) currently state:

> `@return`: Should contain all possible return types, and a description for each. Use a period at the end. Note: `@return void` should not be used outside of the default bundled themes.

Ref: https://make.wordpress.org/core/handbook/best-practices/inline-documentation-standards/php/#1-functions-class-methods (last bullet)

> Tag |	Usage |	Description
> --- | ----- | -----------
> @return |	datatype description |	Document the return value of functions or methods. `@return void` should not be used outside of the default bundled themes. For boolean and integer types, use `bool` and `int`, respectively.

Ref: https://make.wordpress.org/core/handbook/best-practices/inline-documentation-standards/php/#phpdoc-tags


# Motivation

By not documenting `@return void`, it becomes neigh impossible to distinguish between the following:
* The code is incomplete/incorrect and return statements still need to be added.
* The code and documentation are as should be, a void function, where the return value is insignificant.

**By explicitely documenting `@return void`, the intent of the developer becomes clear and indisputable.**

This is especially important when documenting abstract methods and methods in interfaces, as those methods do not have any code in the primary declaration, so the documentation is the only context available.


### Other considerations

* Having documentation be explicit will make it easier for new and existing contributors to grasp the code and will save developers time.
    One does not have to dig through the history of the code to check if a `return` statement was accidentally forgotten or removed.
* Always having a `@return` tag makes the documentation more consistent.
* Having less exceptions to remember will make it easier to on-board new documentation contributors.

### References
The current proposal is largely in line with and in part inspired by:
* [PHP Void return type RFC](https://wiki.php.net/rfc/void_return_type)
* [PHP FIG PSR-5 (draft)](https://github.com/phpDocumentor/fig-standards/blob/master/proposed/phpdoc.md)


# Detailed design

> The @return tag is used to document the return value of functions or methods.
>
> #### Syntax
> ```php
> @return <"Type"> [description]
> ```

Ref: https://github.com/phpDocumentor/fig-standards/blob/master/proposed/phpdoc.md#715-return


### General

A `@return` tag MUST be present in all function and method docblocks.

Only one (1) `@return` tag is allowed in each function/method docblock.

If the function does not return anything, `@return void` MUST be used.

#### Exception:
When the PHP 7.1 `void` return type is used, the `@return` tag MAY be omitted as the `return` in that case, is sufficiently documented in the function declaration.

```php
/**
 * Function description.
 */
function my_function(): void {
	// Do something.
}
```

### Return Types

A `@return` tag MUST list all possible return types.

Valid types are: `bool`, `int`, `string`, `float`, `object`, `array`, `resource`, `callback`, `null`, `void`, a Fully Qualified Class Name, or in the case of a class in the same namespace as the current file, a relative class name.

The use of `null` as in `@return int|null` is only allowed when there is an explicit `return null;` statement in the code. An "empty" `return;` should be documented as `void`, as in `@return int|void`.

The use of `mixed` as a return type is STRONGLY DISCOURAGED in favour of explicitely naming the possible return types.

The use of `self` and `parent` as return types is FORBIDDEN. A Fully Qualified or namespace relative class name should be used instead.


### Return Description

It is STRONGLY RECOMMENDED to provide a description for each possible return type.

The description MAY be multi-line.

The description MUST be in sentence form and end with a period `.`. Multiple sentences are allowed.

If the function does not return anything, no description is necessary.


### Formatting

The format of the `@return` tag should be as follows:
```php
@return[one space]typeA|typeB[one space]Description
```

There MUST be one (1) blank comment line above the `@return` tag to make a clear distinction between other tags and the `@return` tag.

The `@return` tag MUST come after any `@param` tags.

The `@return` tag MAY come before or after any `@throws` tags.


### Differences when compared to the current WordPress PHP documentation standard
* In contrast to the current standard, this RFC states that a `@return` tag must always be present in each function/method docblock.
* In contrast to formatting implied in the code examples in the current standard, this RFC states that a `@return` tag should be preceeded by a blank line to have a clear visual distinction between function input and function output documentation.

There are no other differences.

Everything else is in line with the current WordPress PHP documentation standard, though various rules which are implied through the examples, have been made explicit for the purpose of this RFC.


### Differences when compared to the above mentioned references
* Using `return null` and documenting this as `@return null` is not forbidden in this proposal, while `return null` is [explicitely forbidden for void functions](https://wiki.php.net/rfc/void_return_type#why_isn_t_return_nullpermitted).
* [PSR-5](https://github.com/phpDocumentor/fig-standards/blob/master/proposed/phpdoc.md#715-return) denotes the return description as "_OPTIONAL yet RECOMMENDED_", while the above proposal makes it "_STRONGLY RECOMMENDED_".
* [PSR-5](https://github.com/phpDocumentor/fig-standards/blob/master/proposed/phpdoc.md#715-return) states that `@return void` "_MAY be omitted_", while the above proposal forbids to omit it.


# Drawbacks

The current WordPress Core codebase does not comply with the proposal as written here.
This should, however, not be an argument not to adopt this standard, as automated tooling can be provided to execute most, if not all, the labor-intensive changes.

It will need to be investigated what the impact of this change would be on the automated documentation generator as used by WordPress Core.
Having said that, if any changes are needed to the documentation generator tooling, this will be a one-time change and a one-time only dev time investment to make.


# Adoption strategy

### Tooling

The current PHPCS tooling for the WordPress Coding Standards does not trigger any errors when `@return void` is found, so the previous standard was not actively enforced.

When this standard is accepted, initially this could be checked for by the existing upstream `Squiz.Commenting.FunctionComment` sniff.
This sniff is already included in the `WordPress-Docs` ruleset, but the relevant error codes are currently excluded. In particular, the `Squiz.Commenting.FunctionComment.MissingReturn` and `Squiz.Commenting.FunctionComment.InvalidReturnNotVoid` error codes come to mind.

The upstream sniff does not contain auto-fixers, while for transitioning the current WP Core codebase, it would be helpful if these did exist, or at least existed for the "missing `@return void`" part. To that end, a custom WordPress native sniff could be developed, or, if PHPCS upstream would be open to the idea, the upstream sniff could be enhanced with selective auto-fixers.

### WordPress Core

When this standard is accepted, all new function docblocks being introduced into WordPress Core should comply with the updated standard.

Once the tooling is in place, a one-time fixer run can be executed to automatically add missing `@return void` tags and where necessary, to add a blank docblock line above the `@return` tag line.

### Userland WordPress projects

It is up to individual project owners to decide their own adoption strategy, though the above suggested strategy for Core could be applied to userland projects as well.


# How we teach this

For existing documentation contributors, the change is small enough to get used to relatively quickly.

For new documentation contributors, there will be one less exception to remember, so in practice, there will be less to teach, not more.

The WordPress PHP Documentation Standards would need to be adjusted to reflect this change.

The following changes would need to be made:
* Removal of the "_Note: `@return void` should not be used outside of the default bundled themes._" phrase in the two paragraphs quoted above.
* Addition of a "_A `@return` tag must be present in all function docblocks_" phrase to clarify the change for people who are looking for the old rule.
* All examples docblocks in the standard should be adjusted to include the blank line above the `@return` tag.

Additionally, it is recommended to reword the section regarding "_[Parameters that are arrays](https://make.wordpress.org/core/handbook/best-practices/inline-documentation-standards/php/#1-1-parameters-that-are-arrays)_" to include array `@return` tags.


# Unresolved questions

> Note that @return is not used for hook documentation, because action hooks return nothing, and filter hooks always return their first parameter.

Ref: https://make.wordpress.org/core/handbook/best-practices/inline-documentation-standards/php/#4-hooks-actions-and-filters

This RFC, at this time, does not include, nor apply to, hook docblocks.

A case can be made, that the `@return` for hook function calls should also be explicitely documented.

Documenting the `@return` of hooks could enhance the understanding of hooks for new developers in the WordPress eco-system and would make teaching the principle of hooks easier.

In practical terms, for action hooks, this would always result in `@return void`. For filter hooks, this would repeat the type of the first parameter with a description along the lines of "Adjusted $var1.".

