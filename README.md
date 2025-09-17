# Baivaria

*Baivaria* is an encoder-only language model for Bavarian achieving new SOTA results on Named Entity Recognition (NER) and Part-of-Speech Tagging (PoS).

This repository documents the creation of Baivaria.

# Data Selection

We use the following Bavarian corpora for the pretraining of Baivaria:

* [Bavarian Wikipedia](https://huggingface.co/datasets/bavarian-nlp/barwiki-20250901)
* [Bavarian Bible](https://huggingface.co/datasets/bavarian-nlp/gemini-bavarian-bible)
* [Bavarian Awesome Tagesschau](https://huggingface.co/datasets/bavarian-nlp/gemini-bavarian-tagesschau-v0.1)
* Bavarian Occiglot
* [Bavarian Books](https://huggingface.co/datasets/bavarian-nlp/bavarian-books-ocred-v0.1)
* [Bavarian FinFinepdfsepds](https://huggingface.co/datasets/HuggingFaceFW/finepdfs)

The following table shows some stats for all corpora - after filtering:

| Corpus Name                 | Quality Measures       |Documents | Sentences | Tokens      | Plaintext Size |
|:--------------------------- |:---------------------- |---------:| ---------:| -----------:| --------------:|
| Bavarian Wikipedia          | High-quality Wikipedia |   43,627 |   242,245 |   7,001,569 |            21M |
| Bavarian Bible              | Gemini-translated      |    1,189 |    35,156 |   1,346,116 |           3.8M |
| Bavarian Awesome Tagesschau | Gemini-translated      |   10,036 |   335,989 |  10,528,908 |            35M |
| Bavarian Occiglot           | Gemini-translated      |  149,774 | 6,842,935 | 214,697,892 |           834M |
| Bavarian Books              | OCR'ed Books           |    4,361 |    53,656 |   1,147,435 |           3.2M |
| Bavarian Finepdfs           | OCR'ed PDFs            |    1,989 |    73,970 |   2,381,873 |           6.7M |

Overall, the pretraining corpus has 210,976 documents, 7,583,951 sentences, 237,103,793 tokens with a total plaintext size of 903M.
