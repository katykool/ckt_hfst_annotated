# Morphologically annotated Chukchi texts

* Parallel Chukchi-Russian sentences are collected  by the [HSE-Chukchi-NLP](https://huggingface.co/datasets/HSE-Chukchi-NLP/russian-chukchi-parallel-corpora) project
  * online dictionary by Charles Weinstein
    * The set of sentence examples differs from (Weinstein, 2018): there are sentences not included in the dataset.
  * articles from "Krayniy Sever" (Murɣin nutenut)
 
* The morphological parser (HFST-based) is built by Vasilisa Andriyanets and Francis Tyers: [GitHub](https://github.com/BasilisAndr/chkchn), [ACL](https://aclanthology.org/W18-4804/).

## The corpus structure

The corpus is in JSONL format. 

```jsonc
{"ckt": "нутэнут ныкоргавӄэн, нытаӈвыентоӄэн.",   // Chukchi sentence
 "ru": "земля радуется, дышит.",                  // Russian translation
 "score": 0.668190598487854,                      // technical field
 "source": "Charles_Weinstein",                   // source of the sentence (for now Weinstein (2018) only, to be expanded)
 "article": null,                                 // technical field
 "translation": null,                             // technical field
 "sent_id": null,                                 // technical field
 "morphology": [                                  // wordforms and their analyses
   {"form": "нутэнут", "analyses": ["нутэнут<n><sg><abs>"]},
   {"form": "ныкоргавӄэн", "analyses": ["ныкоргавӄэн+?"]},
   {"form": "нытаӈвыентоӄэн", "analyses": ["тэӈыԓгын<n><incorp>+выенток<v><iv><stat><hab><s_sg3>"]}
 ]}
```

Morphological analyses ([tagset here](tagset.txt)):

```jsonc
{"form": "нутэнут", "analyses": ["нутэнут<n><sg><abs>"]},       // stem, part-of-speech tag, grammatical tags
{"form": "ныкоргавӄэн", "analyses": ["ныкоргавӄэн+?"]},         // unanalysed forms are marked with +?
{"form": "нытаӈвыентоӄэн",                                       // incorporation: several stems joined with +
 "analyses": ["тэӈыԓгын<n><incorp>+выенток<v><iv><stat><hab><s_sg3>"]}
```

A form can have several analyses if it is ambiguous.

## CQL corpus search

[`cql.py`](cql.py) searches the annotated JSONL corpus by word, lemma, part of speech and grammatical tags, using a sinplified CQL (Corpus Query Language). Python 3 is required; `pip install openpyxl` is needed for `.xlsx` output.

```bash
python cql.py 
    corpora/charles-weinstein-morphology.jsonl 
    '[lemma="пыкирык" & tag!="caus"]' 
    --output results.xlsx
```

Options:

- `--limit N` limits the number of matches;
- `--case-sensitive` enables case-sensitive matching (matching is case-insensitive by default);
- `--show-analyses` prints analyses of matched tokens (console output only);
- `--output results.xlsx` or `--output results.csv` exports the matches instead of printing them.

### Query syntax

Query consists of elements in square brackets: `[attribute="regex"]`.

| attribute | matched against |
|-----------|-----------------|
| `word`  | the wordform |
| `lemma` | the lemma of any analysis of the token, including incorporated stems (both `тэӈыԓгын` and `выенток` in the example above) |
| `pos`   | the first tag after each stem (`n`, `v`, `adv`, `part`, ...) |
| `tag`   | any tag of any analysis (`sg`, `abs`, `stat`, `hab`, `s_sg3`, `incorp`, `caus`, ...) |

Adjacent query tokens are adjacent in the sentence. Quantifiers `?`, `*`, `+`, `{n,m}` are supported.

```text
[lemma="пыкирык"]                 any form of the lemma
[word="пыкиргъэ"]                 exact surface form
[pos="v" & tag="hab"]             a habitual verb
[lemma="рэк" | word="эймэквъи"]   OR between conditions
[pos="v" & tag!="caus"]           NOT: no analysis of the token has the tag caus
[lemma="рэк"][word=".*чык[оу]"]   two adjacent tokens
[pos="adv"]* [pos="v"]            zero or more adverbs followed by a verb
```

Parentheses are not supported, so a condition shared by both branches has to be repeated.
```text
[lemma="рэк" & tag!="caus" | word="эймэквъи" & tag!="caus"]
```

Things to keep in mind:

- `lemma`, `pos` and `tag` are collected over all analyses of a token. `tag!="caus"` therefore excludes a token if *any* of its analyses contains `caus`.
- Incorporation yields **one** token with several stems, so `[pos="n"][pos="v"]` does not find an incorporated noun + verb; search for the verb stem instead.

### Output table

Exported tables contain all fields of the corpus (`morphology` as a JSON string) plus a last column `match` with the matched word(s). There is one row per match, so a sentence with two matches gives two rows.

## Google Colab notebook

[`corpus_search_colab.ipynb`](corpus_search_colab.ipynb) is a Colab notebook for searching the annotated corpus. It downloads [`cql.py`](cql.py) and `charles-weinstein-morphology.jsonl` from GitHub, so no installation is required.

Open the notebook in Google Colab, edit `QUERY`, run the cells, and use `save(hits, "results.xlsx")` to download the result table (`.csv` is supported too).

The notebook's example searches several verb lemmas and a word form, excluding causative analyses:

```python
QUERY = '[lemma="пыкирык|эймэвык|ӄытык|тыԓек|эквэтык|рэк|рэсӄивык|пэԓӄынтэтык|тыттэтык|рыԓык" & tag!="caus" | word="эймэквъи" & tag!="caus"]'
```
