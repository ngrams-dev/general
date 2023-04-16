**What is the target audience of NGRAMS?**
NGRAMS is for everyone who needs ngram data with frequency information. In particular, it may be of interest to NLP researchers and app makers who deal with language modeling. Non-native writers can use the web search interface to express difficulties in choosing words.

**What is NGRAMS' underlying dataset?**
It is the [Google Books Ngram Dataset v3](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html) which is freely available from Google.

**What is the difference between NGRAMS and Google Ngram Viewer?**
The query language is different. NGRAMS has operators that Google Ngram Viewer does not have, and vice versa. With NGRAMS you can fetch the entire result for a query, with Google Ngram Viewer you will get the top 10 matches. Most importantly, NGRAMS has a REST API that lets you access the data from third-party apps.

**Is NGRAMS free to use?**
Yes, NGRAMS is free of charge for both commercial and non-commercial purposes.

**Is NGRAMS open-source?**
No, but we appreciate community feeback and feature requests to improve the service.

**Is NGRAMS stable?**
NGRAMS has been in part-time development for four years and was released mid April 2023. The system is currently in beta status and things might change or even crash. Critical issues will be fixed as soon as possible. The person behind the project has more than 10 years of experience indexing ngram data. NGRAMS' server serving the REST API is operated by Germany's leading cloud provider [Hetzner.com](https://hetzner.com).

**What kind of database does NGRAMS use?**
NGRAMS uses its own custom-made NoSQL system for indexing and storing ngram data. This customization allows us to apply the best compression techniques suitable for this kind of data and tune the system for best read performance. With a general purpose NoSQL system this would not be possible — and less fun. The implementation language is C++.

**Where do the text examples in the web search come from?**
The text examples are not part of the original dataset. When you click the document icon next to an ngram in the result list, we fetch them from Google Books on-the-fly. They are meant to show the user some context where an ngram is being used. NGRAMS has no influence on the quality of the examples, and sometimes there are no examples at all.

**Why weren't American English and British English indexed?**
Actually to cut storage costs because they are the second and third largest corpora in the dataset, and we had to prioritize things in the early phase of NGRAMS. We indexed English though which contains ngrams from books published in either American or British English. It also contains ngrams which are neither in the American nor the British corpus because of the match count threshold of 40 (an ngram must appear at least 40 times to make it into the corpus). The corpora might be added in the future if there is enough interest.

**I have my own ngram data that need to be indexed.**
Talk to us at [contact@ngrams.dev](mailto:contact@ngrams.dev).
