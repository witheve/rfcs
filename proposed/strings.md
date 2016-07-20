Discussion: https://github.com/witheve/rfcs/issues/5

I'm still trying to build search-as-you-type input with Eve. However, Eve seems to lack any string functions.

The bare minimum would be to have an expression that checks for substring in a string. Something like this JS function:

```js
var contains = (search, string) => string.indexOf(search) !== -1;
```

The most flexible would be to have regexp match expression. Something like this:

```js
var matches = (search, string) => !!string.match(new RegExp(search));
```

The middle ground is to allow prefix-matching for words inside string:

```js
var matches = (search, string) => !!string.match(new RegExp('\\b' + search + '\\w*\\b'));

matches('ab', 'abc def'); // => true
matches('bc', 'abc def'); // => false
matches('de', 'abc def'); // => true
```

The only thing I could currently do is to pre-build the index with external tools, generating huge amount of `[#word-prefix-match]` objects:

```
build the index
  freeze
    [#word-prefix-match "a" "apple computer"]
    [#word-prefix-match "ap" "apple computer"]
    [#word-prefix-match "app" "apple computer"]
    // …
    [#word-prefix-match "c" "apple computer"]
    [#word-prefix-match "co" "computer"]
```

And I can't even build this index with Eve code: there are no `split` or `prefix-match` functions.

P.S. Bonus points is to somehow allow to ignore common words like `a` or `an`, so that `an` wouldn't match `an apple`, but it would match `anne`.
