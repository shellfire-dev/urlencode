# `urlencode`: functions module for [shellfire]

This module provides a simple framework for coping with the myriad ways of 'urlencoding' UTF-8 strings in URLs in a [shellfire] application:-

* Per [RFC 3986]
* As URI Templates per [RFC 6570]
* As `POST` form data (the MIME Type `application/x-www-form-urlencoded`)

An example user is the [github api] module, which uses it to url encode URI templates.

## Compatibility

* Tag [`release_2015.0117.1750-1`](https://github.com/shellfire-dev/urlencode/releases/tag/release_2015.0117.1750-1) is compatible with [shellfire] release [`release_2015.0117.1750-1`](https://github.com/shellfire-dev/shellfire/releases/tag/release_2015.0117.1750-1).

## Overview

### URI Templating, eg for the [github api]

```bash
urlencode_template_variable_name="my name"
encodedValue="$(urlencode_template "https://uploads.github.com/repos/octocat/Hello-World/releases/1/assets{?name}")
```

### Encoding a path piece as per [RFC 3986]

```bash
someUrl="http://host/path/to/$(urlencode_pathPiece 'my folder name')/file.txt"

### POST data

For example let's say you want to encode the key or value of a form to be `POST`ed:-

```bash
encodedValue="$(urlencode_form 'my key')
```

More usefully, you can do
```bash
urlencode_xWwwFormUrlEncoded 'my key' 'my value' 'another key' 'another value' >"$curl_uploadFile"
```

## Importing

To import this module, add a git submodule to your repository. From the root of your git repository in the terminal, type:-
```bash
mkdir -p lib/shellfire
cd lib/shellfire
git submodule add "https://github.com/shellfire-dev/urlencode.git"
cd -
git submodule init --update
```

You may need to change the url `https://github.com/shellfire-dev/urlencode.git` above if using a fork.

You will also need to add paths - include the module [paths.d].

## Namespace `urlencode`

This namespace exposes helper functions to convert a UTF-8 encoded string into a particular urlencoded form.

### To use in code

If calling from another [shellfire] module, add to your shell code the line
```bash
core_usesIn urlencode
```
in the global scope (ie outside of any functions). A good convention is to put it above any function that depends on functions in this module. If using it directly in a program, put this line _inside_ the `_program()` function:-

```bash
_program()
{
	core_usesIn urlencode
	…
}
```

### Functions

***
#### `urlencode_pathPiece`

|Parameter|Value|Optional|
|---------|-----|--------|
|`string`|A string to encode as per [RFC 3986] such that it is valid|_No_|

Writes the encoded form of the UTF-8 encoded string `string` to standard out.

This implementation uses a conservative function (it encodes where not strictly necessary so it works in paths, query strings and fragments). It is not suitable for encoding host names or schemes (eg `http`), but if anyone actually defines a scheme that needs percent encoding, they're mad.

***
#### `urlencode_form`

|Parameter|Value|Optional|
|---------|-----|--------|
|`string`|A string to encode as if it is a key or value as per `POST` form encoding (the MIME Type `application/x-www-form-urlencoded`) such that it is valid|_No_|

Writes the encoded form of the UTF-8 encoded string `string` to standard out. Might be used in as follows:-

```bash
{
	printf '?'
	urlencode_form name
	printf '='
	urlencode_form "Albert Einstein"
	printf '&'
	urlencode_form occupation
	printf '='
	printf 'Theoretical Physicist'
} >"$curl_uploadFile"
```

However, in that case, the function `urlencode_xWwwFormUrlEncoded()` is more useful.

***
#### `urlencode_xWwwFormUrlEncoded`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more key-value pairs|_Yes_|

Encodes the key value pairs and writes the result to standard out, eg:-
```bash
urlencode_xWwwFormUrlEncoded 'my key' 'my value' 'another key' 'another value' >"$curl_uploadFile"
```
produces `my+key=my+value&another+key=another+value`.


***
#### `urlencode_literal`

|Parameter|Value|Optional|
|---------|-----|--------|
|`string`|A string to encode as a literal as in [RFC 6570, Section 2.1](http://tools.ietf.org/html/rfc6570#section-2.1) and [RFC 6570, Section 3.1](http://tools.ietf.org/html/rfc6570#section-2.1); this is a confusing standard.|_No_|

Writes the encoded form of the UTF-8 encoded string `string` to standard out.

***
#### `urlencode_reserved`

|Parameter|Value|Optional|
|---------|-----|--------|
|`string`|A string to encode as a reserved literal as in RFC 6570.|_No_|

Writes the encoded form of the UTF-8 encoded string `string` to standard out. Not particularly useful outside of the uri templates functions.

***

## Namespace `urlencode_template`

This namespace exposes a helper function to convert a UTF-8 encoded URI template string into a [RFC 6570] URI.

### To use in code

If calling from another [shellfire] module, add to your shell code the line
```bash
core_usesIn urlencode template
```
in the global scope (ie outside of any functions). A good convention is to put it above any function that depends on functions in this module. If using it directly in a program, put this line _inside_ the `_program()` function:-

```bash
_program()
{
	core_usesIn urlencode template
	…
}
```

### Functions

***
#### `urlencode_template`

|Parameter|Value|Optional|
|---------|-----|--------|
|`templateString`|A string to encode as per [RFC 6570] such that it is valid|_No_|

Writes the encoded form of the UTF-8 encoded URI template string `templateString` to standard out. Supports Levels 1, 2, 3 and 4. Supports substitution of both single values and arrays of values (defined using `core_variable_array_initialise` and populated using `core_variable_array_append`); does not support maps (key value pairs) as this isn't available outside of bash 4. One passes variable values to the function by defining in the form `urlencode_template_variable_${variableName}`:-

```bash
urlencode_template_variable_name="my name"
encodedValue="$(urlencode_template "https://uploads.github.com/repos/octocat/Hello-World/releases/1/assets{?name}")
```



[RFC 3986]: https://tools.ietf.org/html/rfc3986 "RFC 3986"
[RFC 6570]: http://tools.ietf.org/html/rfc6570 "RFC 6570"
[swaddle]: https://github.com/raphaelcohn/swaddle "Swaddle homepage"
[shellfire]: https://github.com/shellfire-dev "shellfire homepage"
[core]: https://github.com/shellfire-dev/core "shellfire core module homepage"
[paths.d]: https://github.com/shellfire-dev/paths.d "paths.d shellfire module homepage"
[github api]: https://github.com/shellfire-dev/github "github shellfire module homepage"
[jsonwriter]: https://github.com/shellfire-dev/jsonwriter "jsonwriter shellfire module homepage"
[jsonreader]: https://github.com/shellfire-dev/jsonreader "jsonreader shellfire module homepage"
[urlencode]: https://github.com/shellfire-dev/urlencode "urlencode shellfire module homepage"
[unicode]: https://github.com/shellfire-dev/unicode "unicode shellfire module homepage"
[version]: https://github.com/shellfire-dev/version "version shellfire module homepage"
