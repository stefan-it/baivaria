# Baivaria

![alt text](logo.png)

*Baivaria* is an encoder-only language model for Bavarian achieving new SOTA results on Named Entity Recognition (NER) and Part-of-Speech Tagging (PoS).

This repository documents the creation of Baivaria.

# 📋 Changelog

* 18.09.2025: Initial version of this repository.

# Data Selection

We use the following Bavarian corpora for the pretraining of Baivaria:

* [Bavarian Wikipedia](https://huggingface.co/datasets/bavarian-nlp/barwiki-20250901)
* [Bavarian Bible](https://huggingface.co/datasets/bavarian-nlp/gemini-bavarian-bible)
* [Bavarian Awesome Tagesschau](https://huggingface.co/datasets/bavarian-nlp/gemini-bavarian-tagesschau-v0.1)
* Bavarian Occiglot
* [Bavarian Books](https://huggingface.co/datasets/bavarian-nlp/bavarian-books-ocred-v0.1)
* [Bavarian Finepdfs](https://huggingface.co/datasets/HuggingFaceFW/finepdfs)

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

# Pretraining

Pretraining a Bavarian model from scratch would be very inefficient, as there's not enough pretraining dataset.

For Baivaria we follow the main idea in the "[Don't Stop Pretraining: Adapt Language Models to Domains and Tasks](https://arxiv.org/abs/2004.10964)" paper from Gururangan et al. and perform a domain-adaptive pretraining and continue pretraining from a strong German encoder-only model.

We use the recently released [GERTuraX-3](https://huggingface.co/gerturax/gerturax-3) as backbone and continue pretraining with our Bavarian corpus. Additionally, we perform a small hyper-parameter search and report micro F1-score on the [BarNER](https://arxiv.org/abs/2403.12749) NER dataset. The best
performing ablation model is used as final model and is released as *Baivaria* in version 1.

Thanks to the [TRC program](https://sites.research.google/trc/about/), the following ablation models could be pretrained on a v4-32 TPU Pod:

| Hyper-Parameter     | Ablation 1 | Ablation 2 | Ablation 3 | Ablation 4 |
| ------------------- | ----------:| ----------:| ----------:| ----------:|
| `decay_steps`       |     26.638 |     26.638 |     26.638 |     26.638 |
| `end_lr`            |        0.0 |        0.0 |        0.0 |        0.0 |
| `init_lr`           |     0.0003 |     0.0003 |     0.0005 |     0.0003 |
| `train_steps`       |     26.638 |     26.638 |     26.638 |     26.638 |
| `global_batch_size` |       1024 |       1024 |       1024 |       1024 |
| `warmup_steps`      |        266 |          0 |       1598 |       2663 |

Ablation 3 is using the proposed hyper-parameters from the "[Don't Stop Pretraining: Adapt Language Models to Domains and Tasks](https://arxiv.org/abs/2004.10964)" paper.

Now we report the MLM accuracy and loss for the training, including the downstream task performance on the [BarNER](https://arxiv.org/abs/2403.12749) dataset. For fine-tuning, we use the last checkpoint and the hyper-parameters as specified in the [GERTuraX Fine-Tuner](https://github.com/stefan-it/gerturax-fine-tuner) repo:

| Metric          | Ablation 1   | Ablation 2   | Ablation 3   | Ablation 4   |
|:--------------- | ------------:| ------------:| ------------:| ------------:|
| MLM Accuracy    |        72.24 |        72.17 |        72.99 |        71.61 |
| Train Loss      |       2.9175 |       2.9248 |       2.8785 |       2.9689 |
| BarNER F1-Score | 80.21 ± 0.31 | 80.83 ± 0.28 | 80.59 ± 0.35 | 80.06 ± 0.41 |

# Results

Not many datasets for Bavarian exists for an evaluation on downstream tasks. We are using the following ones:

* [BarNER](https://arxiv.org/abs/2403.12749) NER dataset
* [MaiBaam](https://arxiv.org/abs/2403.10293) Part-of-Speech Tagging dataset

## BarNER

We use the [GERTuraX Fine-Tuner](https://github.com/stefan-it/gerturax-fine-tuner) repo and its hyper-parameter to fine-tune Baivaria for Bavarian NER.

Results on the development set:

| Configuration   |   Run 1 |   Run 2 |   Run 3 |   Run 4 |   Run 5 | Avg.         |
|-----------------|---------|---------|---------|---------|---------|--------------|
| `bs32-e20-lr3e` |   81.31 |   80.94 |   80.63 |   80.49 |   80.78 | 80.83 ± 0.28 |
| `bs16-e20-lr2e` |   80.87 |   80.76 |   79.96 |   80.07 |   80.04 | 80.34 ± 0.39 |
| `bs32-e20-lr2e` |   80.03 |   80.03 |   80.07 |   80.65 |   80.25 | 80.21 ± 0.24 |
| `bs16-e20-lr3e` |   80.07 |   79.27 |   79.31 |   80.28 |   80.31 | 79.85 ± 0.46 |
| `bs32-e20-lr4e` |   79.27 |   79.82 |   80.28 |   80.38 |   78.98 | 79.75 ± 0.55 |
| `bs16-e20-lr4e` |   79.04 |   79.09 |   80.29 |   79.08 |   79.62 | 79.42 ± 0.48 |
| `bs16-e20-lr1e` |   78.34 |   80.66 |   79.83 |   78.98 |   79.07 | 79.38 ± 0.80 |
| `bs32-e20-lr1e` |   80.87 |   78.96 |   77.52 |   79.45 |   78.39 | 79.04 ± 1.12 |

For the best configuration (`bs32-e20-lr3e`) we report the micro F1-Score on the test dataset:

| Configuration   |   Run 1 |   Run 2 |   Run 3 |   Run 4 |   Run 5 | Avg.        |
|-----------------|---------|---------|---------|---------|---------|-------------|
| `bs32-e20-lr3e` |   76.68 |   75.89 |   76.66 |   74.11 |   75.17 | 75.70 ± 0.97 |

The [BarNER](https://arxiv.org/abs/2403.12749) paper reports a micro F1-Score of 72.17 ± 1.75 for their best German model (GBERT Large). However, this model is a 24-layer BERT model with 337M parameters. Baivaria is outperforming that model by +3.53% on average while at the same time Baivaria only has 125M parameters, as Baivaria is a 12-layer ELECTRA model.

## MaiBaam

We also use the [GERTuraX Fine-Tuner](https://github.com/stefan-it/gerturax-fine-tuner) repo and the same hyper-parameter search as used for Bavarian NER.

We follow the same fine-tuning strategy as shown in the [MaiBaam](https://arxiv.org/abs/2403.10293) paper: we fine-tune the model on the latest [German GSD](https://github.com/UniversalDependencies/UD_German-GSD) universal dependencies dataset and evaluate the best performing configuration on the MaiBaam test dataset.

Baivaria shows the following performance on the German GSD development dataset:

| Configuration   |   Run 1 |   Run 2 |   Run 3 |   Run 4 |   Run 5 | Avg.         |
|-----------------|---------|---------|---------|---------|---------|--------------|
| `bs16-e20-lr4e` |   97.74 |   97.76 |   97.76 |   97.74 |   97.70 | 97.74 ± 0.02 |
| `bs32-e20-lr3e` |   97.72 |   97.69 |   97.72 |   97.74 |   97.73 | 97.72 ± 0.02 |
| `bs32-e20-lr1e` |   97.74 |   97.68 |   97.72 |   97.68 |   97.76 | 97.72 ± 0.03 |
| `bs16-e20-lr3e` |   97.64 |   97.72 |   97.75 |   97.68 |   97.78 | 97.71 ± 0.05 |
| `bs32-e20-lr2e` |   97.77 |   97.70 |   97.67 |   97.68 |   97.70 | 97.70 ± 0.03 |
| `bs16-e20-lr1e` |   97.68 |   97.65 |   97.72 |   97.71 |   97.75 | 97.70 ± 0.03 |
| `bs32-e20-lr4e` |   97.65 |   97.69 |   97.71 |   97.68 |   97.75 | 97.70 ± 0.03 |
| `bs16-e20-lr2e` |   97.76 |   97.72 |   97.72 |   97.60 |   97.66 | 97.69 ± 0.06 |

We use the best configuration (`bs16-e20-lr4e`) and report accuracy on the MaiBaam test dataset:

|   Run 1 |   Run 2 |   Run 3 |   Run 4 |   Run 5 | Avg.         |
|---------|---------|---------|---------|---------|--------------|
|   90.52 |   90.08 |   90.12 |   90.27 |   90.40 | 90.28 ± 0.16 |

Additionally, we report F1-Score on the MaiBaam test dataset using the best configuration:

|   Run 1 |   Run 2 |   Run 3 |   Run 4 |   Run 5 | Avg.         |
|---------|---------|---------|---------|---------|--------------|
|   74.19 |   72.75 |   73.78 |   74.97 |   72.54 | 73.65 ± 0.91 |

The best performing model (UDPipe trained on GSD dataset) in the [MaiBaam](https://arxiv.org/abs/2403.10293) paper reports an accuracy of 80.29% and a micro F1-Score of 62.45%. Baivaria improves this by +9.99% and +11.2% respectively.

## Overall

In the overall section we compare results of Baivaria to current state-of-the-art results in the corresponding papers.

For NER:

| Model                                                       | F1-Score (Final test dataset) |
| ----------------------------------------------------------- | --------------------------- |
| GBERT Large from [BarNER](https://arxiv.org/abs/2403.12749) | 72.17 ± 1.75                |
| Baivaria v1                                                 | 75.70 ± 0.97                |

For PoS Tagging:

| Model                                                        | Accuracy (Final test dataset) | F1-Score (Final test dataset) |
| ------------------------------------------------------------ | ----------------------------- | ----------------------------- |
| GBERT Base from [MaiBaam](https://arxiv.org/abs/2403.10293)  | 80.29                         | 62.45                         |
| Baivaria v1                                                  | 90.28 ± 0.16                  | 73.65 ± 0.91                  |

# ❤️ Acknowledgements

Baivaria is the outcome of working with TPUs from the awesome [TRC program](https://sites.research.google/trc/about/)
and the [TensorFlow Model Garden](https://github.com/tensorflow/models) library.

Many thanks for providing TPUs!

Made from Bavarian Oberland with ❤️ and 🥨.
