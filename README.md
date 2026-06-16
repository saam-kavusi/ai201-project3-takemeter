# ai201-project3-takemeter

# TakeMeter: One Piece Discourse Classifier

## Project Overview

TakeMeter is a fine-tuned text classifier that evaluates the type of discourse found in an online community. For this project, I focused on **r/OnePiece**, a Reddit community centered on discussion of the anime and manga series *One Piece*. The goal of the classifier is to label posts or comments as one of three categories: `analysis`, `hot_take`, or `reaction`.

This task is useful because One Piece discussions vary a lot in quality and purpose. Some comments make detailed arguments about foreshadowing, characters, themes, or plot structure. Others are bold speculative claims with little support. Others are quick emotional responses to a chapter, scene, or episode. The classifier attempts to distinguish between these different types of discourse.

## Community Choice and Reasoning

I chose **r/OnePiece** because it is an active, public, text-heavy community with many different types of discussion. The community includes theory posts, chapter discussion threads, episode reactions, character debates, power-scaling arguments, and emotional responses to major story moments.

This made it a good fit for a classification task because the comments are not all the same style. Some posts are evidence-based and analytical, some are unsupported opinions or theories, and some are mostly emotional reactions. These distinctions matter in the community because fans often separate thoughtful discussion from hype, jokes, or unsupported takes.

## Label Taxonomy

I used three labels: `analysis`, `hot_take`, and `reaction`.

### `analysis`

A post or comment is labeled `analysis` when it explains an idea about *One Piece* using reasoning, evidence, or connections to the story, characters, themes, worldbuilding, foreshadowing, or plot.

**Example 1:**
“Oda has been setting up inherited will since the beginning, so the Joy Boy reveal fits the larger theme of freedom.”

**Example 2:**
“The Void Century is probably connected to the Ancient Kingdom because the World Government keeps erasing historical records, which suggests the truth threatens their legitimacy.”

### `hot_take`

A post or comment is labeled `hot_take` when it makes a bold, controversial, or confident opinion with little, weak, or mostly unsupported reasoning.

**Example 1:**
“Zoro is overrated and people only like him because he looks cool.”

**Example 2:**
“Wano is a bad arc and people only defend it because of Gear 5.”

### `reaction`

A post or comment is labeled `reaction` when it is mostly an emotional response, hype comment, joke, simple agreement/disagreement, or immediate reaction to a scene, chapter, character, or theory.

**Example 1:**
“This chapter was peak, I got chills.”

**Example 2:**
“When Robin says ‘I want to live,’ chills every time.”

## Data Collection and Labeling Process

I collected **200 public posts and comments** from r/OnePiece. The examples came from theory posts, chapter discussion threads, episode discussion threads, character discussion posts, and overrated/underrated debate posts.

The final dataset was saved as a single CSV file called `onepiece_dataset.csv` with two columns:

* `text`
* `label`

The final label distribution was:

| Label      |   Count |
| ---------- | ------: |
| `analysis` |      72 |
| `hot_take` |      67 |
| `reaction` |      61 |
| **Total**  | **200** |

No label made up more than 70% of the dataset, so the dataset was balanced enough to avoid a simple majority-class classifier.

## Difficult-to-Label Examples

Some examples were difficult because One Piece comments often mix emotional language, theory, and opinion.

### Difficult Example 1

A comment praised a chapter as “peak” but also explained that the reveal connected to earlier foreshadowing.

**Possible labels:** `reaction`, `analysis`
**Final label:** `analysis`
**Reasoning:** I chose `analysis` because the comment did more than express hype. It used story evidence to explain why the reveal worked.

### Difficult Example 2

A comment said a character was overrated but included one short reason based on a fight or arc.

**Possible labels:** `hot_take`, `analysis`
**Final label:** `hot_take`
**Reasoning:** I chose `hot_take` because the reasoning was too brief to count as a real argument. The main purpose of the comment was still a bold opinion.

### Difficult Example 3

A comment reacted emotionally to a major scene, saying it gave them chills and made them emotional.

**Possible labels:** `reaction`, `analysis`
**Final label:** `reaction`
**Reasoning:** I chose `reaction` because the comment focused on emotional impact rather than explaining a broader story point.

## Fine-Tuning Approach

The fine-tuned model used `distilbert-base-uncased` as the base model. The dataset was split into train, validation, and test sets by the starter notebook using a 70% / 15% / 15% split.

I kept the default training settings from the notebook:

| Hyperparameter |                     Value |
| -------------- | ------------------------: |
| Base model     | `distilbert-base-uncased` |
| Epochs         |                         3 |
| Learning rate  |                      2e-5 |
| Batch size     |                        16 |
| Test set size  |               30 examples |

I kept these defaults because the dataset was small, and the project starter notebook was designed for this scale of fine-tuning. With only 200 examples, increasing the number of epochs too much could increase overfitting.

## Baseline Description

For the zero-shot baseline, I used Groq with `llama-3.3-70b-versatile`. The prompt described the r/OnePiece classification task, defined the three labels, and instructed the model to output only one label name: `analysis`, `hot_take`, or `reaction`.

The first baseline prompt was too loose, and the notebook could not parse the responses. I revised the prompt to make the output format stricter by telling the model not to explain, not to use punctuation, and to return only the exact label name. After revising the prompt, all 30 test examples were parseable.

The final prompt included the following label definitions:

* `analysis`: The post/comment explains an idea about One Piece using reasoning, evidence, or connections to the story, characters, themes, worldbuilding, foreshadowing, or plot.
* `hot_take`: The post/comment makes a bold, controversial, or confident opinion with little, weak, or mostly unsupported reasoning.
* `reaction`: The post/comment is mostly an emotional response, hype comment, joke, simple agreement/disagreement, or immediate reaction to a scene, chapter, character, or theory.

The prompt also told the model to return exactly one of the labels and nothing else.

## Evaluation Results

Both models were evaluated on the same 30-example test set.

### Overall Accuracy

| Model                       | Accuracy |
| --------------------------- | -------: |
| Groq zero-shot baseline     |    0.700 |
| Fine-tuned DistilBERT model |    0.467 |

The Groq baseline performed better than the fine-tuned DistilBERT model. This suggests that the fine-tuned model did not learn the label boundaries as well as expected from the small dataset.

## Baseline Per-Class Metrics

| Label            | Precision | Recall | F1-score | Support |
| ---------------- | --------: | -----: | -------: | ------: |
| `analysis`       |      0.73 |   1.00 |     0.85 |      11 |
| `hot_take`       |      0.67 |   0.20 |     0.31 |      10 |
| `reaction`       |      0.67 |   0.89 |     0.76 |       9 |
| **Accuracy**     |           |        | **0.70** |      30 |
| **Macro avg**    |      0.69 |   0.70 |     0.64 |      30 |
| **Weighted avg** |      0.69 |   0.70 |     0.64 |      30 |

The baseline performed strongest on `analysis` and `reaction`. Its biggest weakness was `hot_take`, where recall was only 0.20. This means the baseline missed most true hot takes.

## Fine-Tuned Model Per-Class Metrics

| Label            | Precision | Recall | F1-score | Support |
| ---------------- | --------: | -----: | -------: | ------: |
| `analysis`       |      0.41 |   1.00 |     0.58 |      11 |
| `hot_take`       |      1.00 |   0.20 |     0.33 |      10 |
| `reaction`       |      1.00 |   0.11 |     0.20 |       9 |
| **Accuracy**     |           |        | **0.47** |      30 |
| **Macro avg**    |      0.80 |   0.44 |     0.37 |      30 |
| **Weighted avg** |      0.78 |   0.47 |     0.38 |      30 |

The fine-tuned model had high precision for `hot_take` and `reaction`, but very low recall for both. This means that when it predicted those labels, it was usually correct, but it rarely predicted them. Instead, it over-predicted `analysis`.

## Confusion Matrix

The fine-tuned model produced the following confusion matrix:

| True label ↓ / Predicted label → | `analysis` | `hot_take` | `reaction` |
| -------------------------------- | ---------: | ---------: | ---------: |
| `analysis`                       |         11 |          0 |          0 |
| `hot_take`                       |          8 |          2 |          0 |
| `reaction`                       |          8 |          0 |          1 |

The strongest pattern is that the model over-predicted `analysis`. It correctly classified all 11 true `analysis` examples, but it also mislabeled 8 `hot_take` examples and 8 `reaction` examples as `analysis`. This shows that the main boundary the model failed to learn was the difference between true evidence-based analysis and comments that merely mention One Piece story details.

The confusion matrix image is included in this repository as `confusion_matrix.png`.

## Wrong Prediction Analysis

### Wrong Prediction 1

**Text:**
“zoan users are on full form, the one that died are the zoan fruit instead of user.”

**True label:** `hot_take`
**Predicted label:** `analysis`
**Confidence:** 0.37

**Analysis:**
This was labeled as `hot_take` because it makes a speculative claim without enough support. The model likely predicted `analysis` because the comment sounds like a theory about Devil Fruits and Zoan users. The topic signals “theory,” but the structure does not provide much reasoning or evidence. This shows that the model learned to associate speculative One Piece lore claims with `analysis`, even when the claim is unsupported.

### Wrong Prediction 2

**Text:**
“Tritoma is Luffy’s mom”

**True label:** `hot_take`
**Predicted label:** `analysis`
**Confidence:** 0.36

**Analysis:**
This is a bold theory claim with no explanation, so I labeled it `hot_take`. The fine-tuned model predicted `analysis`, probably because it saw a theory-like statement about character relationships. This failure shows that the model did not reliably learn the difference between an unsupported theory and actual analysis. A better model would need more examples where unsupported theories are clearly labeled as `hot_take`.

### Wrong Prediction 3

**Text:**
“The last 10 minutes were so beautiful. From the fight scenes to the flashbacks was amazing. Him running to the crew was such a nice moment. Same with the mouse and the flashback to him cooking as a kid. 10/10”

**True label:** `reaction`
**Predicted label:** `analysis`
**Confidence:** 0.37

**Analysis:**
This comment is a `reaction` because its main purpose is emotional response and appreciation. It talks about scenes, flashbacks, and animation, but it does not make an argument about the story. The model likely predicted `analysis` because the comment includes specific details from the episode. This suggests the model learned a shortcut: comments with specific One Piece details often get labeled as `analysis`, even when the comment is mainly emotional.

### Wrong Prediction 4

**Text:**
“Robin has gotten through and survived 3 buster calls before reuniting with her savior. She’s been through so much, so glad she’s happier now”

**True label:** `reaction`
**Predicted label:** `analysis`
**Confidence:** 0.38

**Analysis:**
This comment references story events, but the main purpose is emotional reflection on Robin’s journey. It was labeled `reaction` because it expresses sympathy and happiness for the character. The model predicted `analysis` because the comment includes plot-specific references such as Buster Calls. This is another example of the model confusing story detail with analysis.

## Failure Pattern Analysis

Before writing my final evaluation, I used an AI tool to review the fine-tuned model’s wrong predictions and identify common failure patterns. The clearest pattern was that the model over-predicted `analysis`. Many examples that were truly `hot_take` or `reaction` were classified as `analysis`.

One major issue was that unsupported theories often looked similar to analysis. For example, claims like “Tritoma is Luffy’s mom,” “The One Piece is a space ship,” or “Garp, Dragon and Roger conducted a plan together” were labeled as `hot_take` in my dataset because they were bold speculative claims without much evidence. However, the fine-tuned model predicted them as `analysis`, likely because they sounded like theory-based discussion.

Another pattern was that emotional reactions with story details were also mislabeled as `analysis`. Comments about emotional scenes, favorite episodes, Robin’s survival, or beautiful animation were often true `reaction` examples, but the model predicted `analysis` because the comments referenced specific events or characters.

Overall, the model seemed to learn a shallow shortcut: if a comment mentioned plot, characters, or theories, it often predicted `analysis`. It did not reliably distinguish between evidence-based reasoning, unsupported speculation, and emotional reflection.

## Sample Classifications

| Example post/comment                                                                                                                                                    | Predicted label | Confidence | Correct?                        |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | ---------: | ------------------------------- |
| “Crocodile is like a sandstorm that wears down the heart and mind of Alabasta people. Just like how he drains water, he also drains the people’s patience and trust...” | `analysis`      |       0.39 | Yes — true label was `analysis` |
| “zoan users are on full form, the one that died are the zoan fruit instead of user.”                                                                                    | `analysis`      |       0.37 | No — true label was `hot_take`  |
| “Tritoma is Luffy’s mom”                                                                                                                                                | `analysis`      |       0.36 | No — true label was `hot_take`  |
| “The last 10 minutes were so beautiful. From the fight scenes to the flashbacks was amazing...”                                                                         | `analysis`      |       0.37 | No — true label was `reaction`  |
| “Robin has gotten through and survived 3 buster calls before reuniting with her savior...”                                                                              | `analysis`      |       0.38 | No — true label was `reaction`  |

The first sample classification is reasonable because the comment connects Crocodile’s powers and role in Alabasta to a broader interpretation of how he affects the people and Robin. It uses explanation and story reasoning, so `analysis` is a reasonable prediction.

The other examples show the main model failure pattern. The model often predicted `analysis` when a comment mentioned One Piece lore, characters, or story details, even when the comment was actually an unsupported theory or an emotional reaction.

## Reflection: What the Model Learned vs. What I Intended

I intended for the model to learn the difference between three kinds of discourse: reasoned analysis, unsupported hot takes, and emotional reactions. The label definitions were based on the purpose of the comment, not just the topic. For example, a comment about Devil Fruits could be `analysis` if it used evidence, or `hot_take` if it made a bold unsupported claim.

However, the fine-tuned model seemed to learn a simpler pattern. It often treated comments that mentioned plot details, theories, characters, or lore as `analysis`. This caused it to over-predict `analysis` and miss many `hot_take` and `reaction` examples. The model captured surface-level One Piece topic signals, but it missed the deeper distinction between evidence-based reasoning, unsupported speculation, and emotional response.

This likely happened because the dataset was small and the boundaries between labels are genuinely difficult. A better version of this project would include more examples of unsupported theories labeled as `hot_take` and more emotional comments with story details labeled as `reaction`. I would also tighten the label definitions even further and add more hard boundary examples to the training data.

## Spec Reflection

The planning spec helped guide the implementation by forcing me to define the labels and hard edge cases before training the model. This was useful because it gave me a consistent rule to follow when labeling ambiguous comments. For example, I decided that a comment should only be labeled `analysis` if it used enough reasoning or evidence for the argument to stand on its own.

One way the implementation diverged from the original plan is that the fine-tuned model did not outperform the Groq baseline. My original definition of success was at least 70% accuracy with reasonable performance across all classes. The Groq baseline reached 70% accuracy, but the fine-tuned model only reached 46.7% accuracy. This showed that the task was harder than expected for a small 200-example dataset, especially because many hot takes and reactions still contained story details that looked like analysis to the model.

## AI Usage

I used AI assistance in multiple parts of this project.

First, I used AI to help stress-test and refine my label taxonomy. I asked for help identifying whether `analysis`, `hot_take`, and `reaction` were clear enough labels for r/OnePiece discourse. The AI helped me think through edge cases, such as comments that mixed emotional language with actual reasoning. I revised the label rules so that `analysis` required reasoning or evidence, while `reaction` focused on emotional response and `hot_take` focused on unsupported bold claims.

Second, I used AI to help review the fine-tuned model’s wrong predictions and identify failure patterns. I pasted the misclassified examples into an AI tool and asked it to look for common themes. The main pattern it identified was that the model over-predicted `analysis`, especially for unsupported theories and emotional comments with story details. I verified this myself by rereading the wrong predictions and comparing them to the confusion matrix before including it in this README.

I also used AI support while writing the final README structure, but I reviewed and edited the content to make sure it matched my actual dataset, metrics, and project decisions. I did not use AI to blindly accept labels without review; the final labels in the dataset were reviewed by me.

## Files in This Repository

* `planning.md` — project planning, label definitions, edge cases, evaluation plan, and AI tool plan
* `onepiece_dataset.csv` — final labeled dataset with 200 examples
* `evaluation_results.json` — saved model evaluation results
* `confusion_matrix.png` — confusion matrix image from the fine-tuned model
* `README.md` — final project report
