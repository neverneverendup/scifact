# Data

The data formats and code used to visualize and work with the data are described below. Documentation on the format for predictions is not yet included; for now, just run `script/full-example.sh` and examine the outputs dumped to the `prediction` directory.

## Outline

- [Gold data](#gold-data)
    - [Claims](#claims)
    - [Corpus](#corpus)
    - [Code](#code)

## Gold data

The schemas for the claim and corpus data are below.

### Claims

The schema for the claim data is as follows:

`claims_(train|dev|test).jsonl`
```python
{
    "id": number,                   # An integer claim ID.
    "claim": string,                # The text of the claim.
    "evidence": {                   # The evidence for the claim.
        "doc_id": [                 # The rationales for a single document, keyed by S2ORC ID.
            {
                "label": enum("SUPPORT" | "CONTRADICT" | "NOT_ENOUGH_INFO"),
                "sentences": number[]
            }
        ]
    },
    "cited_doc_ids": number[]       # The claim's "cited documents".
}
```

NOTE: In our dataset, the labels for all rationales within a single abstract will be the same. We provide a label annotation for each rationale in case future versions of the data have conflicting rationales within a single abstract.

An example is below. This claim was written based on a citation with 3 cited documents, of which only two were found to have evidence. Document `11328820` contradicts the claim, and has 1 rationale consisting of two sentences. Document `30041340` also contradicts the claim. It has two rationales: one with two sentences, and another with a single sentence.
```python
{
  "id": 263,
  "claim": "Citrullinated proteins externalized in neutrophil extracellular traps act indirectly to disrupt the inflammatory cycle.",
  "evidence": {
    "11328820": [
      { "sentences": [7, 9],
        "label": "CONTRADICT" }
    ],
    "30041340": [
      { "sentences": [0, 1],
        "label": "CONTRADICT" },
      { "sentences": [11],
        "label": "CONTRADICT" }
    ]
  },
  "cited_doc_ids": [
    11328820,
    30041340,
    14853989
  ]
}
```

### Corpus

Below are the schema for evidence abstracts, along with an example.

Schema
```python
{
    "doc_id": number,               # The document's S2ORC ID.
    "title": string,                # The title.
    "abstract": string[],           # The abstract, written as a list of sentences.
    "structured": boolean           # Indicator for whether this is a structured abstract.
}
```

Example
```python
{
  "doc_id": 4983,
  "title": "Microstructural development of human newborn cerebral white matter assessed in vivo by diffusion tensor magnetic resonance imaging.",
  "abstract": [
    "Alterations of the architecture of cerebral white matter in the developing human brain can affect cortical development and result in functional disabilities.",
    ...
  ],
  "structured": false
}
```


### Code

We've written some functions to facilitate pretty-printing of the evidence associated with each claim. For example, running the following snippet:
```python
from lib.data import GoldDataset

data = GoldDataset("data/corpus.jsonl", "data/claims_train.jsonl")
claim = data.get_claim(123)
claim.pretty_print()
```
produces this output:
```text
Example 123: Antiretroviral therapy increases rates of tuberculosis across a broad range of CD4 strata.

Evidence sets:

####################

4883040: REFUTES
Set 0:
	- Antiretroviral therapy is strongly associated with a reduction in the incidence of tuberculosis in all baseline CD4 count categories: (1) less than 200 cells/µl (hazard ratio [HR] 0.16, 95% confidence interval [CI] 0.07 to 0.36), (2) 200 to 350 cells/µl (HR 0.34, 95% CI 0.19 to 0.60), (3) greater than 350 cells/µl (HR 0.43, 95% CI 0.30 to 0.63), and (4) any CD4 count (HR 0.35, 95% CI 0.28 to 0.44).
Set 1:
	- CONCLUSIONS Antiretroviral therapy is strongly associated with a reduction in the incidence of tuberculosis across all CD4 count strata.
```
