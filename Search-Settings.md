The settings described on this page refer to the search facility on https://ngrams.dev. You can find them by clicking the :gear: icon in the search bar. For the corresponding settings using the REST API click [here](rest-api#search-request).

## Default

To demonstrate the effect of each setting, the query below is sent with all flags disabled. What follows is a list of the top 15 matching ngrams from the English corpus. The first column is the ngram's total match count, see [Data Model](ngram-dataset#data-model) for details.

```
hello * * * *
```

```
64100   Hello , " she said
61515   Hello , " he said
39340   Hello , my name is
36733   Hello , " I said
30904   Hello , Dolly ! _END_
18591   Hello , how are you
14915   Hello , ' she said
12641   Hello , ' he said
 8994   Hello World ! " _END_
 8693   Hello , " said the
 8506   hello to him . _END_
 8298   Hello , hello , hello
 7987   hello , " she said
 7801   hello , " he said
 7759   Hello , ' I said
```

## Case Sensitive

The search is case-senstive.

<details>
<summary>Result if enabled</summary>

```
8506   hello to him . _END_
7987   hello , " she said
7801   hello , " he said
7680   hello to me . _END_
6071   hello to her . _END_
5797   hello and good - bye
5579   hello , how are you
4740   hello to you . _END_
4351   hello to you , too
3788   hello to them . _END_
3451   hello for me . _END_
3392   hello and goodbye . _END_
3257   hello , " I said
3204   hello for me . "
2899   hello there . " _END_
```

</details>

## Collapse Result

Matching ngrams are first case-folded and then equal ngrams are merged adding their total match counts.

<details>
<summary>Result if enabled</summary>

```
72087   hello , " she said
69316   hello , " he said
39990   hello , " i said
39340   hello , my name is
30904   hello , dolly ! _END_
24170   hello , how are you
14915   hello , ' she said
12641   hello , ' he said
12165   hello , world ! "
11825   hello to you , too
11473   hello , world ! _END_
 8994   hello world ! " _END_
 8693   hello , " said the
 8506   hello to him . _END_
 8473   hello there . " _END_
```

</details>

## Exclude Punctuation Marks

Matching ngrams where wildcards got replaced with punctuation marks are removed from the result. A punctuation mark is a character with [Unicode Category P](http://www.unicode.org/reports/tr44/#General_Category_Values). See also [here](https://www.compart.com/en/unicode/category).

<details>
<summary>Result if enabled</summary>

```
1525   hello to an old friend
1498   Hello Time 2 sec Max
1207   hello to my little friend
1146   hello to a couple of
1061   hello and how are you
1022   hello to some of the
 969   hello to a few people
 926   hello to him for me
 806   hello to your mother for
 794   hello to all of you
 787   hello to her for me
 774   Hello and welcome to the
 719   hello to each other and
 710   hello to a few of
 696   hello to the rest of
```

</details>

## Exclude Sentence Boundary Tags

Matching ngrams where wildcards got replaced with sentence boundary tags are removed from the result.

<details>
<summary>Result if enabled</summary>

```
64100   Hello , " she said
61515   Hello , " he said
39340   Hello , my name is
36733   Hello , " I said
18591   Hello , how are you
14915   Hello , ' she said
12641   Hello , ' he said
 8693   Hello , " said the
 8298   Hello , hello , hello
 7987   hello , " she said
 7801   hello , " he said
 7759   Hello , ' I said
 7486   Hello , World ! "
 7474   Hello to you , too
 7247   Hello , darling , "
```

</details>

## Don't Interpret Operators

Query operators are not interpreted but processed as regular query terms.

_This leads to an empty result, because there is no matching ngram._

<details>
<summary>Another example</summary>

```
hello *
```

```
546   Hello *
492   hello *
102   HELLO *
```

</details>

## Don't Normalize Query Terms

Usually NGRAMS normalizes some characters to match the characters used in the underlying dataset. This is to maximize the number of matches. One example is the use of an accent mark in the query where an apostrophe (Unicode 39) should be used, e.g. `they´re` instead of `they're`. By enabling this flag no character is replaced.

_This leads to the same result as with default settings, because there is nothing to normalize._

<details>
<summary>Another example</summary>

The query uses Acute Accent (Unicode 180) instead of Apostrophe (Unicode 39).

```
they´re
```

This leads to an empty result, because there is no such ngram in the dataset.

</details>

## Don't Tokenize Query Terms

Usually NGRAMS tokenizes the query the same way as the underlying ngrams. This is to maximize the number of matches. Examples are contractions, e.g. `they're` becomes `they 're` and `don't` becomes `do not`, or punctuation marks, e.g. `hello!` becomes `hello !`. By enabling this flag the query is only split on whitespace-like characters.

_This leads to the same result as with default settings, because there is nothing to tokenize._

<details>
<summary>Another example</summary>

The query has a punctuation mark that won't be split off.

```
hello!
```

```
1462   Hello!
 311   hello!
 124   HELLO!
```

There are matches, but with a very low total match count compared to `hello !`. These ngrams made it into the dataset somehow without being tokenized. Only Google knows why. They can be considered noise.

</details>
