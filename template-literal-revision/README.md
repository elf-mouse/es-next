# Template Literal Revision

## 1. Tag functions and escape sequences

With tagged template literals, you can make a function call by mentioning a function before a template literal:

```
> String.raw`\u{4B}`
'\\u{4B}'
```

String.raw is a so-called tag function. Tag functions receive two versions of the fixed string pieces (template strings) in a template literal:

- Cooked: escape sequences are interpreted. `\u{4B}` becomes 'K'.
- Raw: escape sequences are normal text. `\u{4B}` becomes '\\u{4B}'.

The following tag function illustrates how that works:

```js
function tagFunc(tmplObj, substs) {
    return {
        Cooked: tmplObj,
        Raw: tmplObj.raw,
    };
}
```

Using the tag function:

```
> tagFunc`\u{4B}`;
{ Cooked: [ 'K' ], Raw: [ '\\u{4B}' ] }
```

## 2. Problem: some text is illegal after backslashes

The problem is that even with the raw version, you don’t have total freedom within template literals in ES2016. After a backslash, some sequences of characters are not legal anymore:

- `\u` starts a Unicode escape, which must look like `\u{1F4A4}` or `\u004B`.
- `\x` starts a hex escape, which must look like `\x4B`.
- `\` plus digit starts an octal escape (such as `\141`). Octal escapes are forbidden in template literals and strict mode string literals.

That prevents tagged template literals such as:

```
latex`\unicode`
windowsPath`C:\uuu\xxx\111`
```

## 3. Solution

The solution is drop all syntactic restrictions related to escape sequences. Then illegal escape sequences simply show up verbatim in the raw representation. But what about the cooked representation? Every template string with an illegal escape sequence is an undefined element in the cooked Array:

```
> tagFunc`\uu ${1} \xx`
{ Cooked: [ undefined, undefined ], Raw: [ '\\uu ', ' \\xx' ] }
```
