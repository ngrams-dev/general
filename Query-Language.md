# Intro

NGRAMS' query language is similar to a very simple form of regular expression. A query, formed as a sequence of terms and query operators, is matched against all indexed ngrams in the selected corpus.

First example:

```
how * you *
```

returns

```
2468812   How do you know
 845786   How do you feel
 830062   How did you know
 755042   How are you ?
 739312   How do you do
...
```

Results are sorted by the ngram's total match count in descending order. See [Data Model](data-model) for details.

Because the underlying ngrams are between 1 and 5 tokens long, a query must be matchable within this range. For example, a query such as `one two three four five six` with 6 terms will never match. Sometimes a term will be split further by NGRAMS in order to match how the underlying ngrams were tokenized by Google. In this case a query can exceed the limit of 5 tokens. More about that in [Tokenization](#tokenization).

# Query Operators

Here is the complete list of query operators.

| Operator  | Name          | Description                                                                              | Example                                                                                           |
| --------- | ------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `*`       | Star          | Matches one token.                                                                       | [what \* day](https://ngrams.dev/?corpus=eng&query=what+%2A+day)                                  |
| `**`      | StarStar      | Matches zero or more tokens.                                                             | [what \*\* day](https://ngrams.dev/?corpus=eng&query=what+%2A%2A+day)                             |
| `a / b`   | Alternation   | Matches either **a** or **b** — see [Alternation](#alternation).                         | [what a sunny / rainy day](https://ngrams.dev/?corpus=eng&query=what+a+sunny+%2F+rainy+day)       |
| `"a b"`   | TermGroup     | Treats multiple terms as one entity — see [TermGroup](#termgroup).                                | [you are / "will be" doing](https://ngrams.dev/?corpus=eng&query=you+are+%2F+%22will+be%22+doing) |
| `prefix~` | Completion    | Matches tokens starting with **prefix**.                                                 | [what an aw~ day](https://ngrams.dev/?corpus=eng&query=what+an+aw~+day)                           |
| `*_ADJ`   | Star_ADJ      | Matches one adjective.                                                                   | [I feel \*\_ADJ](https://ngrams.dev/?corpus=eng&query=I+feel+*_ADJ)                               |
| `*_ADP`   | Star_ADP      | Matches one adposition (preposition or postposition).                                    | [working \*\_ADP home](https://ngrams.dev/?corpus=eng&query=working+*_ADP+home)                   |
| `*_ADV`   | Star_ADV      | Matches one adverb.                                                                      | [she sings \*\_ADV](https://ngrams.dev/?corpus=eng&query=she+sings+*_ADV)                         |
| `*_CONJ`  | Star_CONJ     | Matches one conjunction.                                                                 | [tea \*\_CONJ coffee](https://ngrams.dev/?corpus=eng&query=tea+*_CONJ+coffee)                     |
| `*_DET`   | Star_DET      | Matches one determiner or article.                                                       | [go \*\_DET way](https://ngrams.dev/?corpus=eng&query=go+*_DET+way)                               |
| `*_NOUN`  | Star_NOUN     | Matches one single noun.                                                                 | [buy some \*\_NOUN](https://ngrams.dev/?corpus=eng&query=buy+some+*_NOUN)                         |
| `*_NUM`   | Star_NUM      | Matches one numeral.                                                                     | [buy \*\_NUM bottles](https://ngrams.dev/?corpus=eng&query=buy+*_NUM+bottles)                     |
| `*_PRON`  | Star_PRON     | Matches one pronoun.                                                                     | [bring \*\_PRON flowers](https://ngrams.dev/?corpus=eng&query=bring+*_PRON+flowers)               |
| `*_PRT`   | Star_PRT      | Matches one particle.                                                                    | [to step \*\_PRT](https://ngrams.dev/?corpus=eng&query=to+step+*_PRT)                             |
| `*_VERB`  | Star_VERB     | Matches one verb.                                                                        | [I \*\_VERB you](https://ngrams.dev/?corpus=eng&query=I+*_VERB+you)                               |
| `_START_` | SentenceStart | Matches the start of a sentence.<br>See [Sentence Boundary Tags](#sentence-boundary-tags). | [\_START\_ as expected \*](https://ngrams.dev/?corpus=eng&query=_START_+as+expected+*)            |
| `_END_`   | SentenceEnd   | Matches the end of a sentence.<br>See [Sentence Boundary Tags](#sentence-boundary-tags).   | [as expected \* \_END\_](https://ngrams.dev/?corpus=eng&query=as+expected+*+_END_)                |

## Alternation

An alternation checks multiple terms at once. The `/` can be read like a logical OR operator.

`what a sunny / rainy / windy *` checks

- `what a sunny *`
- `what a rainy *`
- `what a windy *`

## TermGroup

A term group treats zero or more terms as one entity. It is only useful within an alternation, i.e. as the left or right side of the `/` operator. A term group can also be empty to let an alternation check for the empty string.

`you are / "will be" / "" doing` checks

- `you are doing`
- `you will be doing`
- `you doing`

When a query is parsed from left to right, a token like `"` or `"foo` (opening token) is interpreted as the start of a term group. The next `"` or `bar"` (closing token) is interpreted as the end of this term group. It is an error if a closing token comes before an opening token. Unintended opening and closing tokens can be avoided using an [escape sequence](#escape-sequences).

## Sentence Boundary Tags

`_START_` and `_END_` are artificial tokens that were inserted by Google after sentence detection. `_START_` is placed before the first word of a sentence. `_END_` is placed after the punctuation mark that finishes a sentence. As the ngrams in the dataset do not span across sentence boundaries, you will not find these tokens in the middle of an ngram.

# Escape Sequences

To disable the semantics of characters or whole tokens that are used as operators, you have to backslash-escape them. For example, to search for a literal `*` you have to enter `\*`, and so on. Here is the complete list of escape sequences:

| Operator  | Escape Sequence | Note                                     |
| --------- | --------------- | ---------------------------------------- |
| `*`       | `\*`            |                                          |
| `**`      | `\**`           |                                          |
| `/`       | `\/`            |                                          |
| `"a`      | `\"a`           | `a` can be any term or empty.            |
| `b"`      | `b\"`           | `b` can be any term or empty.            |
| `prefix~` | `prefix\~`      | `prefix` can be any term or empty.       |
| `*_ADJ`   | `\*_ADJ`        | Same for other part-of-speech wildcards. |
| `_START_` | `\_START_`      |                                          |
| `_END_`   | `\_END_`        |                                          |

# Tokenization

When Google compiled the dataset, they applied some tokenization (and normalization) to the raw material. For example, punctuation marks that usually follow directly after a term are split off to form a separate token, e.g. `hello!` became `hello !`. Another example is the splitting of contractions like `they're` into `they 're` for linguistic reasons.

In order to match a query against these ngrams, the query has to be tokenized the same way. NGRAMS does this by default. However, this has the effect that a query can exceed the number of tokens (5) and is not matchable anymore. In this case NGRAMS responds with an error. You then have to make your query shorter.

You can turn auto-tokenization off in [search settings](search-settings).
