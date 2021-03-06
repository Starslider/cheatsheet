### nwise

For each 10 lines, emit a new array

```
seq 1 20 | jq \
  -n -R \
  'def nwise(stream; $n):
    foreach (stream, nan) as $x ([];
      if length == $n then [$x] else . + [$x] end;
      if (.[-1] | isnan) and length>1 then .[:-1]
      elif length == $n then .
      else empty
    end);
  nwise(inputs; 10)'
```

### Reduce

Declare a root object `0`, for each object in `.[]`, save it as the variable `$item`.

The result of `. + ($item | tonumber)` is saved for the next iteration.

IE:
  * `0 + 1`
  * `1 + 2`
  * `3 + 3`
  * `6 + 4`
  * ...

```
seq 1 20 | \
  jq  \
    -n -R \
    '[inputs] | reduce .[] as $item (0; . + ($item | tonumber))'
```

### Keys

Output the keys of each fields

```
jq -n '{"blih": true, "blah": false} | keys'
````

### Format strings and escaping

|   |   |
|---|---|
|@text|Calls tostring, see that function for details.|
|@json|Serializes the input as JSON.|
|@html|Applies HTML/XML escaping, by mapping the characters <>&'" to their entity equivalents &lt;, &gt;, &amp;, &apos;, &quot;.|
|@uri|Applies percent-encoding, by mapping all reserved URI characters to a %XX sequence.|
|@csv|The input must be an array, and it is rendered as CSV with double quotes for strings, and quotes escaped by repetition.|
@tsv|The input must be an array, and it is rendered as TSV (tab-separated values). Each input array will be printed as a single line. Fields are separated by a single tab (ascii 0x09). Input characters line-feed (ascii 0x0a), carriage-return (ascii 0x0d), tab (ascii 0x09) and backslash (ascii 0x5c) will be output as escape sequences \n, \r, \t, \\ respectively.|
|@sh|The input is escaped suitable for use in a command-line for a POSIX shell. If the input is an array, the output will be a series of space-separated strings.|
|@base64|The input is converted to base64 as specified by RFC 4648.|
|@base64d|The inverse of @base64, input is decoded as specified by RFC 4648. Note: If the decoded string is not UTF-8, the results are undefined|

### Map array value to string array

```
jq -R '(
  ["date",
    "time",
    "x-edge-location",
    "sc-bytes",
    "c-ip",
    "cs-method",
    "cs(Host)",
    "cs-uri-stem",
    "sc-status",
    "cs(Referer)",
    "cs(User-Agent)",
    "cs-uri-query",
    "cs(Cookie)",
    "x-edge-result-type",
    "x-edge-request-id",
    "x-host-header",
    "cs-protocol",
    "cs-bytes",
    "time-taken",
    "x-forwarded-for",
    "ssl-protocol",
    "ssl-cipher",
    "x-edge-response-result-type",
    "cs-protocol-version",
    "fle-status",
    "fle-encrypted-fields"]) as $fields |
  . |
  split("\t") |
  to_entries |
  reduce .[] as $item ({}; ."\($fields[$item.key])" = $item.value)'
```

### Not select with multiple values

Pass the select if both `. != "1"` and `. != "2"` are true

```
seq 1 10 | jq -R 'select([. != ("1", "2")] | all)'
```

### Select with multiple values

Pass the select if `. == "1"` or `. == "2"`

```
seq 1 10 | jq -R 'select(. == ("1", "2"))'
```

### Alternate operator //

```
jq -n 'null // true'
```

### Default value for variable

```
jq --arg blah "" -n '$blah | select($blah != "") // "hey"'
```

### Build array from inputs

```
jq -nR '[inputs | select(length>0)]'
```

### Update operator and default value

```
jq -n '{"blih": 1} | select(.blah) // .blah |= 10'
```

### iterate through object quickly

```
These functions convert between an object and an array of key-value pairs. If to_entries is passed an object, then for each k: v entry in the input, the output array includes {"key": k, "value": v}.

from_entries does the opposite conversion, and with_entries(foo) is a shorthand for to_entries | map(foo) | from_entries, useful for doing some operation to all keys and values of an object. from_entries accepts key, Key, name, Name, value and Value as keys.
```

```
 seq 1 10 | jq -nr '[inputs] | with_entries(.key |= (. | tostring) | .value = {})'
```

### json to csv

```
inputs | to_entries | map(.value) | @csv
```
