---
layout: default
permalink: faq
title: FAQ
---

# FAQ

## How do I validate my schema is correct?

To make sure your schema is valid, you can check it against the [meta-schema](http://json-schema.org/schema) that defines JSON Schema itself.  You can do so manually like this:

```php
<?php
use League\JsonReference\Dereferencer;
use League\JsonGuard\Validator;

$metaSchema = Dereferencer::draft4()->dereference('http://json-schema.org/draft-04/schema#');
$validator  = new Validator($mySchema, $metaSchema);

if ($validator->fails()) {
    // Invalid schema
}
```

Alternatively you can use a tool like [json-guard-cli](https://github.com/yuloh/json-guard-cli).

## Why aren't my default values used?

The [specification](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2) doesn't actually say what a default value is supposed to do.  Defaults are considered [metadata](https://spacetelescope.github.io/understanding-json-schema/reference/generic.html#metadata) like title or description, and metadata is ignored by the validator.

## Why can't I json_decode to an array?

If you decoded your json as an array (`json_decode($data, true)`), an empty object is decoded the same as an empty array; they both decode to `[]`.  This would make it impossible to validate "type": "object" or "type": "array" when the object/array is empty.

## How do I get the validator to work with large numbers?

If your schema contains numbers larger than PHP_INT_MAX (usually 2147483647), you need to decode your JSON with the JSON_BIGINT_AS_STRING flag.  The JSON_BIGINT_AS_STRING flag decodes the large number as a string instead of casting it to a float.

```php
<?php

$data = json_decode($data, false, 512, JSON_BIGINT_AS_STRING);
```

Comparison with large numbers uses the [bcmatch extension](http://php.net/manual/en/book.bc.php), so make sure it is enabled on your platform.  It's usually enabled by default.

If you need to compare floats with more than 10 places after the decimal place, you can set the scale of the Max or Min constraint when instantiating it, then add it to the rule set.

```php
<?php

$ruleSet = new \League\JsonGuard\RuleSet\DraftFour();
$ruleSet->set('minimum', function () {
    return new \League\JsonGuard\Constraint\DraftFour\Minimum(20);
});
$validator = new \League\JsonGuard\Validator($data, $schema, $ruleSet);
```
