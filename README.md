# Unicode Character Properties in EcmaScript

Spec: https://srl295.github.io/es-unicode-properties/

## Proposal for stage 0

Allows for a function to return [Encoded Character Properties](https://www.unicode.org/reports/tr23/#CodePointProperties)
for a code point.

For applications, they can directly answer questions such as “What kind of script is 𞤘?”, “Is ġ lowercase? ”, or “What is the numeric value of ५?”.

For feature implementers, this is a building block for implementing a wide array of higher level features, such as [number parsing](https://github.com/tc39/ecma402/issues/1), [segmentation](https://github.com/tc39/proposal-intl-segmenter), regular expressions, and much more.

## Definitions

 A property (for this purpose) is *string* (including enumerated types), a *number*, or a *boolean*.

* Code Point: a *String* containing a single Unicode code point (1 or 2 UTF-16 code units).
* Name: A Unicode [Property Alias](https://www.unicode.org/reports/tr44/#Property_And_Value_Aliases) as given in UAX44. As an input, this may be either short or long form, producing identical results.
* Value: A Unicode Property Value or [Property Value Alias](https://www.unicode.org/reports/tr44/#Property_Value_Aliases). As an output, the caller must explicitly or implicitly select an “abbreviated” or a “long” alias.

## Examples

- `и` (U+0438) [ICU UBrowse](http://demo.icu-project.org/icu-bin/ubrowse?ch=0438)
- `𞤘` (U+1E918) [ICU UBrowse](http://demo.icu-project.org/icu-bin/ubrowse?ch=0438)
- `ġ` (U+0121)
- `५` (U+096B)

|  CP | Name | Long Name          | Value | Long Value         | Comments    |
| --- | ---- | ------------------ | ----- | ------------------ | ----------- |
| `и` | `Gc` | `General_Category` | `Ll`  | `Lowercase_Letter` | Enumeration |
| `𞤘` | `sc` | `Script`           | `Adlm`  | `Adlam` | Enumeration |
| `ġ` | `Lower` | `Lowercase` | true  | true | Boolean |
| `५` | `nv` | `General_Category` | 5  | 5 | Number |

## API Brainstorm For Discussion

_Note: see Issues for further discussion._

```js
"и".getUnicodeProperty("Gc", {type: "short"}) // "Ll"
"и".getUnicodeProperty("Gc", {type: "long"}) // "Lowercase_Letter"
"и".getUnicodeProperty("Gc") // "Lowercase_Letter"  - type:long is default
"и".getUnicodeProperty("General_Category") // "Lowercase_Letter" - "Gc" ≈ "General_Category"
```

## FAQ

### Why should this be in EcmaScript?

Data Size, Complexity, Performance, Updates.

As of Unicode 13, there are nearly 150_000 characters encoded across the 2_097_152 available in the 21 bit encoding space. There are over 80 character properties. Storing and accessing this data in an efficient and up to date way is not trivial. However, any conformant implementation, especially one which includes Unicode regular expressions, _already has_ all of this data, available via implementations such as ICU.

### Why not just use RegEx?

In a way, getting a property is the _inverse_ of Unicode Regular Expressions.

```js
/\p{gc=Lowercase_Letter}/u.test('и')
// implies:
"и".getUnicodeProperty("Gc") === 'Lowercase_Letter'
```

If all that is needed is matching, certainly a regex could be used, especially for a boolean operation.

```js
/\p{Lower}/u.test('e') === "e".getUnicodeProperty("Lower") // both true 
/\p{Lower}/u.test('E') === "E".getUnicodeProperty("Lower") // both false
```

However, for classifying (as in [segmentation](https://github.com/tc39/proposal-intl-segmenter)) or analyzing (as in number parsing), this becomes unwieldy.

* “Parse `١٢٣٬٤٥٦`” into numeric form:

```js
     if(/\p{NumericValue=0}/u.test('٢')) { digit = 0; } // false
else if(/\p{NumericValue=1}/u.test('٢')) { digit = 1; } // false
else if(/\p{NumericValue=2}/u.test('٢')) { digit = 2; } // true
else if(/\p{NumericValue=4}/u.test('٢')) { digit = 3; } // false
…

// vs:
digit = '٢'.getUnicodeProperty('nv') // 2
```

This could be used to convert `١٢٣٬٤٥٦` into `Number(123.456)`

_(this property was not supported by the JS engine I tested.)_

* Implement UAX29 Sentence Break Segmentation

Need to calculate the `Sentence_Break` property value for each character:

```js
     if(/\p{Sentence_Break=Extend}/u .test('q')) { … } // false
else if(/\p{Sentence_Break=Lower}/u  .test('q')) { … } // true
else if(/\p{Sentence_Break=OLetter}/u.test('q')) { … } // false
else if(/\p{Sentence_Break=STerm}/u  .test('q')) { … } // false
…
// vs:
'q'.getUnicodeProperty('Sentence_Break') // 'Lower'
```
_(this property was not actually supported by the JS engine I tested.)_

For performance reasons, an application may actually want to get the properties of every codepoint in a string. 

## History 

https://github.com/tc39/ecma402/issues/90