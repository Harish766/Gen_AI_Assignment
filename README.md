# ICT4403 Assignment 2: Autoregressive text generation

This project compares Simple RNN, Bidirectional RNN, LSTM, and GRU models for next-word prediction and sequential text generation. The implementation is in `assignment2.ipynb` and is designed for the Python notebook extension in VS Code.

## Dataset

Lewis Carroll's *Alice's Adventures in Wonderland*, Project Gutenberg eBook 11: https://www.gutenberg.org/ebooks/11 . The notebook downloads the UTF-8 plain text from https://www.gutenberg.org/cache/epub/11/pg11.txt and stores it in `data/alice.txt`. It strips Gutenberg front and back matter, lowercases the text, and tokenizes words and basic punctuation. The corpus is small, so generated text may be locally plausible without staying coherent for long.

## Run in VS Code

1. Confirm Python is installed. The VS Code notebook extension needs a Python interpreter and a kernel.
2. Open this folder in VS Code. In its PowerShell terminal, create an environment with `py -m venv venv`, then run `.\venv\Scripts\python.exe -m pip install -r requirements.txt`.
3. Verify the environment with `.\venv\Scripts\python.exe -c "import torch; print(torch.__version__)"`. If this reports a missing `shm.dll` dependency, install the [Microsoft Visual C++ Redistributable (x64)](https://aka.ms/vc14/vc_redist.x64.exe), restart VS Code, and repeat this check.
4. Open `assignment2.ipynb`, select `venv` as the notebook kernel, and choose **Run All**. The first run needs internet access to download the book.
5. Inspect the printed dataset statistics, model metrics, plots, and text samples. The notebook writes comparison files to `results/`.

The same chronological 80/10/10 train/validation/test split, training vocabulary, context length, embedding size, hidden size, batch size, optimizer, learning rate, and epoch count are used for all four models. Each model uses its best validation-loss checkpoint. Test data is used only for final evaluation and seed phrases. Model parameter counts and training times are reported because the recurrent cells have different complexity.

## Results and interpretation

The book produced 31,175 word-or-punctuation tokens. The training vocabulary contains 1,302 entries (including `<unk>`), and the test split has 3,102 prediction windows. Each model was trained for five epochs under the same settings. The test metrics below come from `results/metrics.csv`; the learning curves are in `results/training_curves.png`.

| Model | Parameters | Best validation loss | Test loss | Test perplexity | Test token accuracy | Training time (s) |
|---|---:|---:|---:|---:|---:|---:|
| Simple RNN | 225,174 | 4.318 | 4.362 | 78.41 | 21.37% | 5.14 |
| Bidirectional RNN | 365,718 | 4.321 | 4.377 | 79.61 | 20.83% | 5.30 |
| LSTM | 271,830 | 4.401 | 4.428 | 83.75 | 20.99% | 5.20 |
| GRU | 256,278 | 4.241 | 4.285 | 72.63 | 21.60% | 4.80 |

Accuracy measures exact next-token prediction. Basic punctuation is tokenized separately, so it is included in this figure; 14.4% of the test targets are punctuation. Roughly 11.0% of test targets are outside the training vocabulary and map to `<unk>`. For context, always predicting the most frequent training token (a comma) would obtain 8.38% accuracy on the same test targets. The 21.60% GRU score is therefore meaningful within this setup, while still leaving considerable uncertainty about the next token. The validation curves were still decreasing at epoch five, so the comparison reflects the chosen training budget rather than a fully tuned optimum.

### Generated text comparison

`results/generated_samples.txt` contains 36 samples: four models, three held-out seeds, and three temperatures. Each sample has 40 generated tokens. The observations below describe the saved samples; coherence and contextual consistency are qualitative judgments, not automatic scores.

| Model | Coherence and contextual consistency | Diversity and repetition |
|---|---|---|
| Simple RNN | Produces Alice-style dialogue and familiar characters, but sentences often become ungrammatical and drift away from the seed. | At 0.5 it repeats phrases such as “said the mock turtle”; diversity rises at higher temperatures. |
| Bidirectional RNN | Keeps some book vocabulary, but story continuity is weak; a 0.5 sample repeats “i think” several times. | More varied at 1.0 and 1.5, with the same loss of syntax at high temperature. |
| LSTM | Produces short dialogue-like clauses, but transitions between ideas are disconnected. | At 0.5 it repeats “she went on”; high-temperature samples contain many distinct words but weaker phrasing. |
| GRU | Produces comparable short clauses and character references; longer continuations still lose the seed context. | Some phrase repetition at 0.5; 1.0 gives a more useful diversity/readability balance than 1.5. |

### Temperature analysis

The table averages the three seeds across all four models using `results/sample_statistics.csv`. Distinct-1 and distinct-2 are fractions of unique generated unigrams and bigrams; repeated-bigram fraction measures repetition. These are diversity proxies and do not measure coherence.

| Temperature | Mean distinct-1 | Mean distinct-2 | Mean repeated-bigram fraction | Observation |
|---:|---:|---:|---:|---|
| 0.5 | 0.590 | 0.846 | 0.154 | Favors common phrases and produces the most repetition, with some locally plausible dialogue. |
| 1.0 | 0.829 | 0.989 | 0.011 | Reduces repetition while retaining some recognizable Alice-style vocabulary. |
| 1.5 | 0.950 | 1.000 | 0.000 | Produces the most varied words but often loses grammatical and contextual consistency. |

The same text seeds were used for each model and temperature. Individual samples are stochastic, so their exact wording also depends on random draws; the diversity trends across all 12 samples at each temperature support the temperature comparison. For this dataset and five-epoch budget, **GRU is the strongest overall choice**: it has the lowest validation and test loss, lowest test perplexity, highest test accuracy, and shortest measured training time. Its samples are not consistently more coherent than the other models, and none maintains a convincing story across 40 generated tokens. The small single-book corpus and short training run limit the strength of the qualitative result.

## Submission checklist

- Confirm the notebook runs from the first cell to the last and `results/` contains the reported plots, metrics, and samples.
- Review the measured results and conclusion above, then upload this notebook, README, requirements, results, and any other required supporting files to a clearly named GitHub repository. Give the evaluator access if private.
- Create `YourName_RegNum_FGAI_Assignment2.docx` with your name, registration number, assignment title, project title, dataset name and link, and GitHub repository link.
- Export that Word document as a PDF and upload the PDF to the LMS before the deadline.
