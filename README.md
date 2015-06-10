# d3-dsv

A parser and formatter for delimiter-separated values, most commonly [comma-separated values](https://en.wikipedia.org/wiki/Comma-separated_values) (CSV) and tab-separated values (TSV). These tabular formats are popular with spreadsheet programs such as Microsoft Excel, and are often more space-efficient than JSON for large datasets.

To use in the browser:

```html
<script src="dsv.js"></script>
<script>

console.log(dsv.csv.parse("foo,bar\n1,2")); // [{foo: "1", bar: "2"}]

</script>
```

To use in Node.js, `npm install d3-dsv`, and:

```js
var dsv = require("d3-dsv");

console.log(dsv.csv.parse("foo,bar\n1,2")); // [{foo: "1", bar: "2"}]
```

Supports [CSV](#csv) and [TSV](#tsv) out of the box. To define a new delimiter, use the [dsv constructor](#dsv):

```js
var psv = dsv.dsv("|");

console.log(psv.parse("foo|bar\n1|2")); // [{foo: "1", bar: "2"}]
```

<a name="dsv" href="#dsv">#</a> <b>dsv</b>(<i>delimiter</i>)

Constructs a new DSV parser and formatter for the specified *delimiter*.

<a name="csv" href="#csv">#</a> <b>csv</b>

A parser and formatter for comma-separated values (CSV), defined as:

```js
var csv = dsv(",");
```

<a name="tsv" href="#tsv">#</a> <b>tsv</b>

A parser and formatter for tab-separated values (TSV), defined as:

```js
var tsv = dsv("\t");
```

<a name="dsv_parse" href="#dsv_parse">#</a> *dsv*.<b>parse</b>(<i>string</i>[, <i>accessor</i>])

Parses the specified *string*, which must be in the delimiter-separated values format with the appropriate delimiter, returning an array of objects representing the parsed rows. The string is assumed to be [RFC4180-compliant](http://tools.ietf.org/html/rfc4180).

Unlike [*dsv*.parseRows](#dsv_parseRows), this method requires that the first line of the DSV content contains a delimiter-separated list of column names; these column names become the attributes on the returned objects. For example, consider the following CSV file:

```
Year,Make,Model,Length
1997,Ford,E350,2.34
2000,Mercury,Cougar,2.38
```

The resulting JavaScript array is:

```js
[
  {"Year": "1997", "Make": "Ford", "Model": "E350", "Length": "2.34"},
  {"Year": "2000", "Make": "Mercury", "Model": "Cougar", "Length": "2.38"}
]
```

Field values are always strings; they will not be automatically converted to numbers, dates, or other types. In some cases, JavaScript may coerce strings to numbers for you automatically (for example, using the `+` operator). By specifying an <i>accessor</i> function, you can convert the strings to numbers or other specific types, such as dates:

```js
var data = csv.parse(string, function(d) {
  return {
    year: new Date(+d.Year, 0, 1), // convert "Year" column to Date
    make: d.Make,
    model: d.Model,
    length: +d.Length // convert "Length" column to number
  };
});
```

Using `+` rather than [parseInt](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/parseInt) or [parseFloat](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/parseFloat) is typically faster, though more restrictive. For example, `"30px"` when coerced using `+` returns `NaN`, while parseInt and parseFloat return `30`.

<a name="dsv_parseRows" href="#dsv_parseRows">#</a> <i>dsv</i>.<b>parseRows</b>(<i>string</i>[, <i>accessor</i>])

Parses the specified *string*, which must be in the delimiter-separated values format with the appropriate delimiter, returning an array of arrays representing the parsed rows. The string is assumed to be [RFC4180-compliant](http://tools.ietf.org/html/rfc4180).

Unlike [*dsv*.parse](#dsv_parse), this method treats the header line as a standard row, and should be used whenever DSV content does not contain a header. Each row is represented as an array rather than an object. Rows may have variable length. For example, consider the following CSV file:

```
1997,Ford,E350,2.34
2000,Mercury,Cougar,2.38
```

The resulting JavaScript array is:

```js
[
  ["1997", "Ford", "E350", "2.34"],
  ["2000", "Mercury", "Cougar", "2.38"]
]
```

Field values are always strings; they will not be automatically converted to numbers. See [*dsv*.parse](#dsv_parse) for details.

An optional *accessor* function may be specified as the second argument. This function is invoked for each row in the DSV content, being passed the current row and index as two arguments. The return value of the function replaces the element in the returned array of rows; if the function returns null, the row is stripped from the returned array of rows. In effect, the accessor is similar to applying a [map](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/map) and [filter](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/filter) operator to the returned rows. The accessor function is used by [*dsv*.parse](#dsv_parse) to convert each row to an object with named attributes.

<a name="dsv_format" href="#dsv_format">#</a> <i>dsv</i>.<b>format</b>(<i>rows</i>)

Converts the specified array of *rows* into delimiter-separated values format, returning a string. This operation is the inverse of [*dsv*.parse](#dsv_parse). Each row will be separated by a newline (`\n`), and each column within each row will be separated by the delimiter (such as a comma, `,`). Values that contain either the delimiter, a double-quote (`"`) or a newline will be escaped using double-quotes.

Each row should be an object, and all object properties will be converted into fields. For greater control over which properties are converted, convert the rows into arrays containing only the properties that should be converted and use [*dsv*.formatRows](#dsv_formatRows).

<a name="dsv_formatRows" href="#dsv_formatRows">#</a> <i>dsv</i>.<b>formatRows</b>(<i>rows</i>)

Converts the specified array of *rows* into delimiter-separated values format, returning a string. This operation is the reverse of [*dsv*.parseRows](#dsv_parseRows). Each row will be separated by a newline (`\n`), and each column within each row will be separated by the delimiter (such as a comma, `,`). Values that contain either the delimiter, a double-quote (") or a newline will be escaped using double-quotes.

### Content Security Policy

If a [content security policy](http://www.w3.org/TR/CSP/) is in place, note that [*dsv*.parse](#dsv_parse) requires `unsafe-eval` in the `script-src` directive, due to the (safe) use of dynamic code generation for fast parsing. (See [source](https://github.com/d3/d3-dsv/blob/master/src/dsv.js).) If `unsafe-eval` cannot be used, then [*dsv*.parseRows](#dsv_parseRows) can be used as a workaround.
