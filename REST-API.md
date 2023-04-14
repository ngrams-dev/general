# Intro

NGRAMS has been built following API-first principles. The goal is to make the accessibility of ngram data as easy as possible. The API sends and receives data in UTF-8 encoded JSON format.

There are endpoints which enable the following types of requests:

- Send a [wildcard query](Query-Language#wildcard-query) and receive matching ngrams using a [search request](#search-request).
- Send multiple [raw queries](Query-Language#raw-query) at once and receive matching ngrams using a [batch request](#batch-request).
- Send an [ngram id](#) and receive year-based match count information for that ngram.
- Get static information about available corpora.

> The REST API is currently in beta status — expect things to change.

> By using the API, you agree to our [Terms of Service](https://docs.ngrams.dev/terms). In short, they read: NGRAMS can be used free of charge, for both commercial and non-commercial purposes. Use requires attribution.

## Base URL

```
https://api.ngrams.dev
```

## Rate Limits

We believe that performance is a feature and therefore our backend tech stack is pure native. To judge our system in the early stage, we do not apply any rate limiting. You can send as many requests as you want, as fast as you can. We will adjust this policy if necessary.

We will block clients based on IP address if we detect abnormal usage.

## Corpora

At the moment, the following corpora are available.

| Name               | Label     | Number of Ngrams |
| ------------------ | --------- | ---------------: |
| English            | `eng`     |   23 568 635 820 |
| French             | `fre`     |    4 690 610 916 |
| German             | `ger`     |    4 464 999 699 |
| Spanish            | `spa`     |    2 426 952 987 |
| Italien            | `ita`     |    2 142 886 796 |
| Russian            | `rus`     |    1 526 968 135 |
| Simplified Chinese | `chi_sim` |      204 765 477 |
| Hebrew             | `heb`     |      115 377 808 |

# Search Request

A search request allows you to send a single wildcard query and receive a set of matching ngrams. This is basically the same type of request issued when using the search interface on https://ngrams.dev. The returned ngrams are sorted by decreasing total match count. Large sets are sent in chunks making use of pagination.

## Endpoint

```
GET /{corpus}/search
```

## Path Parameters

**`corpus`** `string`

The label of the corpus to search, see [corpora](#corpora).

## Query Parameters

**`query`** `string`

The [percent-encoded](https://developer.mozilla.org/en-US/docs/Glossary/Percent-encoding) query string.

<div style="border-top:1px solid #eee"><br></div>

**`flags`** `string` _`optional`_

Enable search flags by adding the respective character sequence to the string.

- `cs` — Search is **case-sensitive**.
- `cr` — **Collapse** the **result set** by case-folding and then merging equal ngrams.
- `ep` — **Exclude** ngrams from the result set where wildcards matched **punctuation marks** as of [Unicode category P](http://www.unicode.org/reports/tr44/#General_Category_Values), see also [here](https://www.compart.com/en/unicode/category).
- `es` — **Exclude** ngrams from the result set where wildcards matched **sentence boundary tags**, namely \_START\_ and \_END\_.
- `ri` — **Raw** query: Do not **interpret** query operators. No need for escape sequences.
- `rt` — **Raw** query: Do not **tokenize** query terms. Split only on ASCII whitespace.
- `rn` — **Raw** query: Do not **normalize** query string characters.
- `ra` — **Raw** query: **all** (`ri` + `rt` + `rn`)

<div style="border-top:1px solid #eee"><br></div>

**`limit`** `number` _`optional`_ _`default: 100`_ _`max: 100`_

The maximum number of ngrams to return. The limit is applied before ngram filtering takes place, which means that the actual size of the result set could be smaller if any of the flags `cs`, `cr`, `ep`, or `es` is used.

<div style="border-top:1px solid #eee"><br></div>

**`start`** `string` _`optional`_

An opaque token to fetch the next chunk of a result set (pagination). A start token is included in a successful search result if there are possibly more matching ngrams.

## Response

The HTTP status code tells whether a request was successful. A code other than 200 is considered failure. The response to a 400 bad request contains body data with error details.

| Code                        | Body                                | Description                             |
| --------------------------- | ----------------------------------- | --------------------------------------- |
| `200 OK`                    | [`SearchResponse`](#searchresponse) | The request was successful.             |
| `400 Bad Request`           | [`ErrorResponse`](#errorresponse)   | The request failed due to client error. |
| `404 Not Found`             | no                                  | The corpus is unknown.                  |
| `500 Internal Server Error` | no                                  | Try again later.                        |

## Examples

```sh
curl 'https://api.ngrams.dev/eng/search?query=hello+*&flags=cs&limit=3'
# OR
curl -G https://api.ngrams.dev/eng/search \
--data-urlencode query='hello *' \
-d flags=cs \
-d limit=3
```

`200 OK`

<details>
<summary>Response Body</summary>

```json
// SearchResponse object,
// 2 instead of 3 ngrams due to post-retrieval case-sensitive filtering.
{
  "queryTokens": [
    { "text": "hello", "type": "TERM" },
    { "text": "*", "type": "STAR" }
  ],
  "ngrams": [
    {
      "id": "d975b1edafaf5aa521f6aee0d7efbe06",
      "absTotalMatchCount": 608657,
      "relTotalMatchCount": 2.899120077549673e-7,
      "tokens": [
        { "text": "hello", "type": "TERM" },
        { "text": ",", "type": "TERM", "inserted": true }
      ]
    },
    {
      "id": "983f5221b490f979d836276d3d986ef2",
      "absTotalMatchCount": 598094,
      "relTotalMatchCount": 2.848807002403643e-7,
      "tokens": [
        { "text": "hello", "type": "TERM" },
        { "text": ".", "type": "TERM", "inserted": true }
      ]
    }
  ],
  "nextPageToken": "157c30ede3ed098320eadbaf1a807dd17228ef6880f959e08605f29fcbfc14a894ffea81eec4fdb609a3e331772e9bd5",
  "nextPageLink": "https://api.ngrams.dev/eng/search?query=hello+%2A&flags=cs&limit=3&start=157c30ede3ed098320eadbaf1a807dd17228ef6880f959e08605f29fcbfc14a894ffea81eec4fdb609a3e331772e9bd5"
}
```

</details>

<div style="border-top:1px solid #eee"><br></div>

```sh
curl https://api.ngrams.dev/eng/search
```

`400 Bad Request`

<details>
<summary>Response Body</summary>

```json
// ErrorResponse object
{ "error": { "code": "MISSING_PARAMETER.QUERY" } }
```

</details>

## Pagination

Wildcard queries can generate result sets that contain thousands of ngrams. The API sends these big result sets in chunks called pages. The start of a page is controlled by the `start` parameter. The size of a page is controlled by the `limit` parameter.

Every search request that contains a partial result, i.e. a page, has a so called page token in its response. This page token can be used in a follow-up request — as the value of the `start` parameter — to fetch the next page.

If a response has no page token, you have reached the end of the result set.

# Batch Request

A batch request allows you to send up to 100 raw queries at once, which saves a lot of HTTP round trip time compared to single search requests. Queries in a batch request implicitly have the `ri` flag set, which means there is no interpretation of query operators. This type of request is most appropriate in situations where the existence or frequency of multiple ngrams needs to be checked quickly.

Batch requests have no means of pagination, because the list of matching ngrams per query is rather short as it only reflects variants in casing. If, in addition, the `cs` or `cr` flag is enabled, there is only one or none ngram to return per query.

## Endpoint

```
POST /{corpus}/batch
```

## Path Parameters

**`corpus`** `string`

The label of the corpus to search, see [corpora](#corpora).

## Request Body

[`BatchRequest`](#batchrequest) object

### Response

The HTTP status code tells whether a request was successful. Invalid request body data causes a request to fail entirely. If the processing of individual queries fails, the batch response will contain corresponding error information for these queries.

| Code                         | Body                              | Description                             |
| ---------------------------- | --------------------------------- | --------------------------------------- |
| `200 OK`                     | [`BatchResponse`](#batchresponse) | The request was successful.             |
| `400 Bad Request`            | [`ErrorResponse`](#errorresponse) | The request failed due to client error. |
| `404 Not Found`              | no                                | The corpus is unknown.                  |
| `500  Internal Server Error` | no                                | Try again later.                        |

## Example

```sh
curl https://api.ngrams.dev/eng/batch \
-H 'Content-Type: application/json' \
-d '@path/to/batch.json'
```

<details>
<summary><code>path/to/batch.json</code></summary>

```json
// BatchRequest object
{
  "flags": "cs",
  "queries": ["The quick brown", "fox jumps over the lazy dog"]
}
```

</details>

`200 OK`

<details>
<summary>Response Body</summary>

```json
// BatchResponse object
{
  "results": [
    {
      "queryTokens": [
        { "text": "The", "type": "TERM" },
        { "text": "quick", "type": "TERM" },
        { "text": "brown", "type": "TERM" }
      ],
      "ngrams": [
        {
          "id": "ecaf9b4576d82550a5661c85f515be24",
          "absTotalMatchCount": 18248,
          "relTotalMatchCount": 9.13534806330214e-9,
          "tokens": [
            { "text": "The", "type": "TERM" },
            { "text": "quick", "type": "TERM" },
            { "text": "brown", "type": "TERM" }
          ]
        }
      ]
    },
    {
      "error": { "code": "INVALID_QUERY.TOO_MANY_TOKENS" },
      "queryTokens": [
        { "text": "fox", "type": "TERM" },
        { "text": "jumps", "type": "TERM" },
        { "text": "over", "type": "TERM" },
        { "text": "the", "type": "TERM" },
        { "text": "lazy", "type": "TERM" },
        { "text": "dog", "type": "TERM" }
      ]
    }
  ]
}
```

</details>

# Ngram Request

An ngram request allows you to send an ngram ID and receive a full ngram object with year-based match count information. This type of request is used on https://ngrams.dev to fetch the data backing an ngram's histogram view.

## Endpoint

```
GET /{corpus}/{ngram_id}
```

## Path Parameters

**`corpus`** `string`

The label of the corpus to search, see [corpora](#corpora).

<div style="border-top:1px solid #eee"><br></div>

**`ngram_id`** `string`

An ngram ID as returned from a search or batch request. Note that the ID of an [abstract ngram](search-settings#abstract-ngram) is always considered unknown, because such ngrams have no year-based match count information.

## Response

The HTTP status code tells whether a request was successful. A code other than 200 is considered failure.

| Code                        | Body           | Description                  |
| --------------------------- | -------------- | ---------------------------- |
| `200 OK`                    | [Ngram](ngram) | The request was successful.  |
| `404 Not Found`             | no             | The corpus or ID is unknown. |
| `500 Internal Server Error` | no             | Try again later.             |

## Example

```sh
curl https://api.ngrams.dev/eng/92c668bc012dc3e387ff0c7e791528db
```

`200 OK`

<details>
<summary>Response Body</summary>

```json
// Ngram object
{
  "id": "92c668bc012dc3e387ff0c7e791528db",
  "absTotalMatchCount": 118987,
  "relTotalMatchCount": 5.6675204699428895e-8,
  "tokens": [
    { "text": "Hello", "type": "TERM" },
    { "text": "World", "type": "TERM" }
  ],
  "stats": [
    {
      "year": 1880,
      "absMatchCount": 52,
      "relMatchCount": 1.2108055367130671e-8
    },
    // There might be gaps for years without any data.
    {
      "year": 1899,
      "absMatchCount": 1,
      "relMatchCount": 1.28983869973734e-10
    },
    {
      "year": 1900,
      "absMatchCount": 49,
      "relMatchCount": 6.053137889244918e-9
    },
    // Items removed to keep it short.
    {
      "year": 2017,
      "absMatchCount": 5107,
      "relMatchCount": 1.765788720318268e-7
    },
    {
      "year": 2018,
      "absMatchCount": 4923,
      "relMatchCount": 1.7802199706983458e-7
    },
    {
      "year": 2019,
      "absMatchCount": 3798,
      "relMatchCount": 1.5816449035193755e-7
    }
  ]
}
```

</details>

# Corpus Info Request

A corpus info request allows you get static information about a corpus.

## Endpoint

```
GET /{corpus}/info
```

## Path Parameters

**`corpus`** `string`

The label of the corpus to search, see [corpora](#corpora).

## Response

The HTTP status code tells whether a request was successful. A code other than 200 is considered failure.

| Code                        | Body                         | Description                 |
| --------------------------- | ---------------------------- | --------------------------- |
| `200 OK`                    | [`CorpusInfo`](#corpus-info) | The request was successful. |
| `404 Not Found`             | no                           | The corpus is unknown.      |
| `500 Internal Server Error` | no                           | Try again later.            |

## Example

```sh
curl https://api.ngrams.dev/eng/info
```

`200 OK`

<details>
<summary>Response Body</summary>

```json
// CorpusInfo object
{
  "name": "English",
  "label": "eng",
  "stats": [
    {
      "numNgrams": 23568635820,
      "minYear": 1470,
      "maxYear": 2019,
      "minMatchCount": 1,
      "maxMatchCount": 1922716631,
      "minTotalMatchCount": 40,
      "maxTotalMatchCount": 115513165249
    },
    {
      "numNgrams": 76862879,
      "minYear": 1470,
      "maxYear": 2019,
      "minMatchCount": 1,
      "maxMatchCount": 1922716631,
      "minTotalMatchCount": 40,
      "maxTotalMatchCount": 115513165249
    },
    {
      "numNgrams": 1604084580,
      "minYear": 1470,
      "maxYear": 2019,
      "minMatchCount": 1,
      "maxMatchCount": 1446928350,
      "minTotalMatchCount": 40,
      "maxTotalMatchCount": 82544506739
    },
    {
      "numNgrams": 11777289629,
      "minYear": 1470,
      "maxYear": 2019,
      "minMatchCount": 1,
      "maxMatchCount": 84854130,
      "minTotalMatchCount": 40,
      "maxTotalMatchCount": 2907518961
    },
    {
      "numNgrams": 5089891990,
      "minYear": 1470,
      "maxYear": 2019,
      "minMatchCount": 1,
      "maxMatchCount": 14391742,
      "minTotalMatchCount": 40,
      "maxTotalMatchCount": 384260789
    },
    {
      "numNgrams": 5020506742,
      "minYear": 1470,
      "maxYear": 2019,
      "minMatchCount": 1,
      "maxMatchCount": 7167265,
      "minTotalMatchCount": 40,
      "maxTotalMatchCount": 226361873
    }
  ]
}
```

</details>

# Types

A complete list of types (schemas) used in this API.

## BatchRequest

A container for multiple queries and search flags.

### Properties

**`queries`** `string[]`

An array of query strings.

<div style="border-top:1px solid #eee"><br></div>

**`flags`** `string` _`optional`_

Enable search flags by adding the respective character sequence to the string.

- `cs` — Search is **case-sensitive**.
- `cr` — **Collapse** the **result set** by case-folding and then merging equal ngrams.

## BatchResponse

A container for multiple search results and metadata.

### Properties

**`results`** `(SearchResponse | ErrorResponse)[]`

An array of multiple types, aka union. `results[i]` is the outcome of `BatchRequest.queries[i]`. If `results[i].error` exists, the object is an instance of [`ErrorResponse`](#errorresponse), otherwise it is an instance of [`SearchResponse`](#searchresponse).

## CorpusInfo

A container for static information about a single corpus.

### Properties

**`name`** `string`

The name of the corpus — something like "English", see [corpora](#corpora).

<div style="border-top:1px solid #eee"><br></div>

**`label`** `string`

The label of the corpus — something like "eng", see [corpora](#corpora).

<div style="border-top:1px solid #eee"><br></div>

**`stats`** `CorpusStat[6]`

An array of [`CorpusStat`](#corpusstat) objects sorted by ngram length. `stats[0]` refers to all ngrams in the corpus, `stats[1]` refers to the subset of unigrams, `stats[2]` refers to the subset of bigrams, etc.

## CorpusStat

A container for statistical data about a corpus or sub-corpus.

**`numNgrams`** `number`

The number of indexed ngrams. See [ngram dataset](ngram-dataset) for details.

<div style="border-top:1px solid #eee"><br></div>

**`minYear`** `number`

The minimum year value associated with an ngram.

<div style="border-top:1px solid #eee"><br></div>

**`maxYear`** `number`

The maximum year value associated with an ngram.

<div style="border-top:1px solid #eee"><br></div>

**`minMatchCount`** `number`

The minimum value of an ngram's year-based match count.

<div style="border-top:1px solid #eee"><br></div>

**`maxMatchCount`** `number`

The maximum value of an ngram's year-based match count.

<div style="border-top:1px solid #eee"><br></div>

**`minTotalMatchCount`** `number`

The minimum value of an ngram's total match count.

<div style="border-top:1px solid #eee"><br></div>

**`maxTotalMatchCount`** `number`

The maximum value of an ngram's total match count.

## Error

A type containing information about a failed query or request.

### Properties

**`code`** [`ErrorCode`](#errorcode)

A string indicating the type of error. The values are constants to be used for programmatic error handling.

<div style="border-top:1px solid #eee"><br></div>

**`context`** `string | object` _`optional`_

Provides error-specific context information to be used for advanced programmatic error handling. The exact format is currently work in progress.

## ErrorCode

An enum that describes the type of an error. Values are string constants.

| Value                           | Description                                                                |
| ------------------------------- | -------------------------------------------------------------------------- |
| `INVALID_PARAMETER.LIMIT`       | Something is wrong with the `limit` parameter.                             |
| `INVALID_PARAMETER.START`       | Something is wrong with the `start` parameter.                             |
| `INVALID_QUERY.BAD_ALTERNATION` | The token to the left or right of the `/` operator is invalid.             |
| `INVALID_QUERY.BAD_COMPLETION`  | The `~` operator has no prefix in front of it.                             |
| `INVALID_QUERY.BAD_TERM_GROUP`  | There is an opening token without a matching closing token, or vice versa. |
| `INVALID_QUERY.NO_TERM`         | The query has no term. At least one term is required.                      |
| `INVALID_QUERY.TOO_EXPENSIVE`   | The query is too expensive to process and was rejected.                    |
| `INVALID_QUERY.TOO_MANY_TOKENS` | The query has more than 5 tokens after tokenization.                       |
| `INVALID_REQUEST_BODY`          | The JSON data in the request body is malformed or has wrong schema.        |
| `INVALID_UTF8_ENCODING`         | The query string is not in valid UTF-8 encoding.                           |
| `MISSING_PARAMETER.QUERY`       | The `query` parameter is missing.                                          |

## ErrorResponse

A container for error information and related data.

### Properties

**`error`** [`Error`](#error)

An error object.

<div style="border-top:1px solid #eee"><br></div>

**`queryTokens`** `QueryToken[]` _`optional`_

A representation of the query after tokenization, which is an array of [`QueryToken`](#querytoken) objects. This property is only available if query processing has actually taken place. It is not available if a request was rejected at an earlier stage, e.g. due to missing required parameters.

## Ngram

A representation of an ngram with full year-based match count information. The properties listed below are in addition to the properties of [`NgramLite`](#ngramlite), i.e. `Ngram` extends `NgramLite`.

### Properties

**`stats`** `NgramStat[]`

An array of [`NgamStat`](#ngramstat) objects.

## NgramLite

A light-weight representation of an ngram with basic metadata.

### Properties

**`id`** `string`

An ID that identifies an ngram uniquely within a corpus. The ID can be used to fetch the corresponding `Ngram` object with full year-based match count information. See [ngram request](#ngram-request) for details. Applications that need a unique ngram ID for the whole dataset can do so by prefixing this ID with the label of the associated corpus, i.e. `{label}_{ngram_id}`.

<div style="border-top:1px solid #eee"><br></div>

**`abstract`** `boolean` _`optional`_

Indicates whether the ngram is abstract as a result of applying a filter operation, e.g. result set collapsing. An abstract ngram does not represent an existing ngram from the dataset and hence has no associated year-based match count information. Its ID is nevertheless unique within a corpus. The property is only present if true — absense means false.

<div style="border-top:1px solid #eee"><br></div>

**`absTotalMatchCount`** `number`

The ngram's absolute total match count. See [Data Model](data-model#ngramlite) for details.

<div style="border-top:1px solid #eee"><br></div>

**`relTotalMatchCount`** `number`

The ngram's relative total match count. See [Data Model](data-model#ngramlite) for details.

<div style="border-top:1px solid #eee"><br></div>

**`tokens`** `NgramToken[1..5]`

An array of [`NgramToken`](#ngramtoken) objects of length 1 to 5.

## NgramStat

A representation of an ngram's match count relating to a single year.

### Properties

**`year`** `number`

The year the data belongs to. See [Data Model](data-model#ngramstat) for details.

<div style="border-top:1px solid #eee"><br></div>

**`absMatchCount`** `number`

The ngram's absolute match count. See [Data Model](data-model#ngram) for details.

<div style="border-top:1px solid #eee"><br></div>

**`relMatchCount`** `number`

The ngram's relative match count. See [Data Model](data-model#ngram) for details.

## NgramToken

A representation of a single token as part of an ngram. It contains basic information like text and type, as well as metadata about its relation to a query, e.g. if the token has been inserted as a result of wildcard application.

### Properties

**`text`** `string`

The token's text in UTF-8 encoding. For tokens that have a part-of-speech suffix in the original raw data, e.g. `example_NOUN`, this suffix has been removed. The POS tag information is available via the `type` property.

<div style="border-top:1px solid #eee"><br></div>

**`type`** [`NgramTokenType`](#ngramtokentype)

The token's type allows to distinguish programmatically between text-like tokens, part-of-speech (POS) tagged tokens, and sentence boundary tags. It can be used to append the original POS tag suffix to the text string or for syntax highlighting when displayed.

<div style="border-top:1px solid #eee"><br></div>

**`inserted`** `boolean` _`optional`_

Indicates whether the token was inserted as a result of applying a `*`, `**`, or `*_ADJ` and friends wildcard. The property is only present if true — absense means false.

<div style="border-top:1px solid #eee"><br></div>

**`completed`** `boolean` _`optional`_

Indicates whether the token was completed as a result of applying the `~` operator. The property is only present if true, absense means false.

## NgramTokenType

An enum that describes the type of an ngram token. Values are string constants.

| Value            | Description                        |
| ---------------- | ---------------------------------- |
| `TERM`           | The token is a regular term.       |
| `TAGGED_AS_ADJ`  | The token has a POS tag of `ADJ`.  |
| `TAGGED_AS_ADP`  | The token has a POS tag of `ADP`.  |
| `TAGGED_AS_ADV`  | The token has a POS tag of `ADV`.  |
| `TAGGED_AS_CONJ` | The token has a POS tag of `CONJ`. |
| `TAGGED_AS_DET`  | The token has a POS tag of `DET`.  |
| `TAGGED_AS_NOUN` | The token has a POS tag of `NOUN`. |
| `TAGGED_AS_NUM`  | The token has a POS tag of `NUM`.  |
| `TAGGED_AS_PRON` | The token has a POS tag of `PRON`. |
| `TAGGED_AS_PRT`  | The token has a POS tag of `PRT`.  |
| `TAGGED_AS_VERB` | The token has a POS tag of `VERB`. |
| `SENTENCE_START` | The token is the `_START_` token.  |
| `SENTENCE_END`   | The token is the `_END_` token.    |

## QueryToken

A representation of a single token as part of a query string.

### Properties

**`text`** `string`

The token's text in UTF-8 encoding.

<div style="border-top:1px solid #eee"><br></div>

**`type`** [`QueryTokenType`](#querytokentype)

The token's type tells if the token has been recognized as text-like token or some query operator.

## QueryTokenType

An enum that describes the type of a query token. Values are string constants.

| Value            | Description                         |
| ---------------- | ----------------------------------- |
| `TERM`           | The token is a regular query term.  |
| `STAR`           | The token is the `*` wildcard.      |
| `STARSTAR`       | The token is the `**` wildcard.     |
| `STAR_ADJ`       | The token is the `*_ADJ` wildcard.  |
| `STAR_ADP`       | The token is the `*_ADP` wildcard.  |
| `STAR_ADV`       | The token is the `*_ADV` wildcard.  |
| `STAR_CONJ`      | The token is the `*_CONJ` wildcard. |
| `STAR_DET`       | The token is the `*_DET` wildcard.  |
| `STAR_NOUN`      | The token is the `*_NOUN` wildcard. |
| `STAR_NUM`       | The token is the `*_NUM` wildcard.  |
| `STAR_PRON`      | The token is the `*_PRON` wildcard. |
| `STAR_PRT`       | The token is the `*_PRT` wildcard.  |
| `STAR_VERB`      | The token is the `*_VERB` wildcard. |
| `SENTENCE_START` | The token is the `_START_` token.   |
| `SENTENCE_END`   | The token is the `_END_` token.     |
| `SLASH`          | The token is the `/` operator.      |
| `PREFIX`         | The token has the `~` operator.     |
| `TERM_GROUP`     | The token is a term group.          |

## SearchResponse

A representation of the outcome of a successfully processed query.

### Properties

**`queryTokens`** `QueryToken[]`

A representation of the query after tokenization, which is an array of [`QueryToken`](#querytoken) objects.

<div style="border-top:1px solid #eee"><br></div>

**`ngrams`** `NgramLite[]`

A representation of the result set, which is an array of [`NgramLite`](#ngramlite) objects.

<div style="border-top:1px solid #eee"><br></div>

**`nextPageToken`** `string` _`optional`_

An opaque token to be used in a follow-up request to fetch the next chunk of the result set. See [pagination](#pagination) for details.

<div style="border-top:1px solid #eee"><br></div>

**`nextPageLink`** `string` _`optional`_

An absolute URL to issue a follow-up request. See [pagination](#pagination) for details.
