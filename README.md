# Morphologically annotated Chukchi texts
* Parallel Chukchi-Russian sentences are collected from the dictionary by Charles Weinstein by [HSE-Chukchi-NLP](https://huggingface.co/datasets/HSE-Chukchi-NLP/russian-chukchi-parallel-corpora) project
** the set of the sentence examples is different from (Weinstein, 2018) -- there are additional sentences not included in the dataset.
* Morphological parser (HFST-based) is built by Vasilisa Andriyanets and Francis Tyers [GitHub](https://github.com/BasilisAndr/chkchn), [ACL](https://aclanthology.org/W18-4804/).

## The corpus structure
Corpus is presented in jsonl format. 
```json
{"ckt": "нутэнут ныкоргавӄэн, нытаӈвыентоӄэн.", "ru": "земля радуется, дышит.", "score": 0.668190598487854, "source": "Charles_Weinstein", "article": null, "translation": null, "sent_id": null, "morphology": [{"form": "нутэнут", "analyses": ["нутэнут<n><sg><abs>"]}, {"form": "ныкоргавӄэн", "analyses": ["ныкоргавӄэн+?"]}, {"form": "нытаӈвыентоӄэн", "analyses": ["тэӈыԓгын<n><incorp>+выенток<v><iv><stat><hab><s_sg3>"]}]}
```
`score`, `article`, `sent_id` are technical fields inherited from HF dataset; 
* `ckt` is Chukchi sentence
* `ru` is Russian translation
* `source` is the source of the sentence (for now Weinstein (2018) only, to be expand)
* `morphology` is a list of distionaries: wordforms and their analysises
```json
{"form": "нутэнут", "analyses": ["нутэнут<n><sg><abs>"]}, # pos-tag and grammatical tags
{"form": "ныкоргавӄэн", "analyses": ["ныкоргавӄэн+?"]}, # undefined forms are marked with +?
{"form": "нытаӈвыентоӄэн", "analyses": ["тэӈыԓгын<n><incorp>+выенток<v><iv><stat><hab><s_sg3>"]}]} # incorporation
```

### CQL corpus search

[`cql.py`](cql.py) searches the annotated JSONL corpus by
word, lemma, part of speech, and grammatical tags. Values are regular
expressions matched against the complete attribute value. Adjacent query
tokens must be adjacent in the sentence; quantifiers are supported.

```bash
python cql.py \
    corpora/charles-weinstein-morphology.jsonl \
    '[lemma="пыкирык" & tag!="caus"]' \
    --output results.xlsx
```

Examples:

```text
[lemma="каргок"]
[word="вагъэ"]
[pos="v" & tag="hab"]
[lemma="рэк"][word=".*чык[оу]"]
[pos="adv"]* [pos="v"]
```

Supported options:

- `--limit N` limits the number of matches;
- `--case-sensitive` enables case-sensitive matching (matching is otherwise case-insensitive);
- `--show-analyses` prints analyses for matched tokens;
- `--output results.xlsx` or `--output results.csv` exports the matches.

### Google Colab notebook

[`corpus_search_colab.ipynb`](corpus_search_colab.ipynb) is a
standalone Colab workflow for searching the published annotated corpus. It
downloads [`cql.py`](cql.py) and
`corpora/charles-weinstein-morphology.jsonl` from GitHub, so installation are not required for searching.

Open the notebook in Google Colab and run the cells from top to bottom. Edit `QUERY`, run
the query cell, and use `save(hits, "results.xlsx")` to download the result
table. The notebook also supports `.csv` output.

The notebook's current example searches several verb lemmas while excluding
causative analyses:

```python
QUERY = '[lemma="пыкирык|эймэвык|ӄытык|тыԓек|эквэтык|рэк|рэсӄивык|пэԓӄынтэтык|тыттэтык|рыԓык" & tag!="caus" | word="эймэквъи" & tag!="caus"]'
