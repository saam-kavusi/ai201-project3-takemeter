# Project 3 Planning: TakeMeter

## Community

For this project, I chose **r/OnePiece**, a Reddit community focused on discussion of the anime and manga series *One Piece*. This community is a good fit for a text classification task because the discourse is active, text-heavy, and varied. People post theories, chapter reactions, character debates, power-scaling opinions, emotional responses, and criticism of story decisions.

This makes the community useful for a TakeMeter classifier because not all comments serve the same purpose. Some comments make reasoned arguments using evidence from the story, some are bold opinions with little support, and some are quick emotional reactions. These distinctions matter because One Piece fans often separate thoughtful theory/discussion from hype, jokes, or unsupported takes.

## Labels

I will use three labels: `analysis`, `hot_take`, and `reaction`.

### analysis

A post or comment is labeled `analysis` when it explains an idea about *One Piece* using reasoning, evidence, or connections to the story, characters, themes, worldbuilding, foreshadowing, or plot.

Example 1:
“Oda has been setting up inherited will since the beginning, so the Joy Boy reveal fits the larger theme of freedom.”

Example 2:
“The Void Century is probably connected to the Ancient Kingdom because the World Government keeps erasing historical records, which suggests the truth threatens their legitimacy.”

### hot_take

A post or comment is labeled `hot_take` when it makes a bold, controversial, or confident opinion with little, weak, or mostly unsupported reasoning.

Example 1:
“Zoro is overrated and people only like him because he looks cool.”

Example 2:
“Wano is a bad arc and people only defend it because of Gear 5.”

### reaction

A post or comment is labeled `reaction` when it is mostly an emotional response, hype comment, joke, simple agreement/disagreement, or immediate reaction to a scene, chapter, character, or theory.

Example 1:
“This chapter was peak, I got chills.”

Example 2:
“When Robin says ‘I want to live,’ chills every time.”

## Hard Edge Cases

The hardest edge case will be comments that mix emotional language with actual reasoning. For example, a comment might say that a chapter was “peak” but then explain how the reveal connects to earlier foreshadowing. That could look like both `reaction` and `analysis`.

My decision rule will be: if the main purpose of the comment is to explain a claim using story evidence or reasoning, I will label it `analysis`, even if it includes emotional language. If the comment is mostly hype, sadness, shock, humor, or a quick personal response, I will label it `reaction`.

Another hard edge case is the difference between `hot_take` and `analysis`. A comment may make a strong claim like “Shanks is overrated,” but also include one piece of evidence. My decision rule will be: if the comment gives enough specific reasoning that the argument could stand on its own, I will label it `analysis`. If the evidence is weak, vague, or mostly used to support a bold opinion without real explanation, I will label it `hot_take`.

### Difficult Examples From Initial Labeling

1. **Example:** A comment praised a chapter as “peak” but also explained that the reveal connected to earlier foreshadowing.  
   **Possible labels:** `reaction`, `analysis`  
   **Final label:** `analysis`  
   **Decision:** I chose `analysis` because the comment did more than express hype; it used story evidence to explain why the reveal worked.

2. **Example:** A comment said a character was overrated but included one short reason based on a fight or arc.  
   **Possible labels:** `hot_take`, `analysis`  
   **Final label:** `hot_take`  
   **Decision:** I chose `hot_take` because the reasoning was too brief to count as a real argument.

3. **Example:** A comment reacted emotionally to a major scene, saying it gave them chills and made them emotional.  
   **Possible labels:** `reaction`, `analysis`  
   **Final label:** `reaction`  
   **Decision:** I chose `reaction` because the comment focused on the emotional impact rather than explaining a broader story point.

## Data Collection Plan

I will collect public posts and comments from **r/OnePiece**. I will mainly use theory posts, chapter discussion threads, episode discussion threads, character discussion posts, and “overrated/underrated” style debate posts.

Before collecting the full dataset, I reviewed and labeled a small initial batch of about 30 r/OnePiece examples to check whether the labels were realistic and usable.

My goal is to collect at least 200 labeled examples with a fairly balanced distribution:

* About 67 examples labeled `analysis`
* About 67 examples labeled `hot_take`
* About 66 examples labeled `reaction`

If one label is underrepresented, I will collect from thread types that are more likely to contain that label. For example, if I need more `analysis`, I will collect from theory and foreshadowing posts. If I need more `reaction`, I will collect from chapter and episode discussion threads. If I need more `hot_take`, I will collect from overrated, underrated, power-scaling, and controversial opinion posts.

The final dataset will be saved as a CSV file with at least two columns: `text` and `label`.

## Evaluation Metrics

I will evaluate the model using overall accuracy and per-class F1 score. Accuracy is useful because it shows the percentage of test examples the model classified correctly overall. However, accuracy alone is not enough because the model could perform well on one label while struggling with another.

Per-class F1 score is important because this task has three labels, and I want to know whether the model is learning all three categories. I will also review the confusion matrix to see which labels are most often confused. For example, I expect the model may confuse `hot_take` and `reaction` when a comment is short or emotional, and it may confuse `analysis` and `hot_take` when a strong opinion includes some evidence.

## Definition of Success

A successful classifier should perform better than the zero-shot Groq baseline on the same test set. I would consider the model useful if it reaches at least 70% overall accuracy and has reasonable performance across all three labels, instead of only doing well on one label.

Good enough for this project means at least 70% overall accuracy and no class having an F1 score near zero.

For a real community tool, I would want the model to be especially careful with the difference between `analysis` and `hot_take`, because that is the distinction most related to discourse quality. If the model can correctly identify most reasoned analysis while not over-labeling every confident opinion as analysis, I would consider that a good result.

## AI Tool Plan

### Label Stress-Testing

I will use an AI tool to help stress-test my label definitions before labeling the full dataset. I will ask it to generate borderline examples between `analysis`, `hot_take`, and `reaction`. If the examples are difficult to classify, I will adjust my label definitions and decision rules before collecting all 200 examples.

### Annotation Assistance

I may use AI assistance to review labels or check whether examples seem mislabeled, but I will personally review every final label before including it in the dataset. If I use an AI tool to help pre-label or check examples, I will disclose that in the README’s AI usage section.

### Failure Analysis

After the model makes predictions on the test set, I will use an AI tool to help identify patterns in the wrong predictions. I will look for patterns such as short comments, sarcasm, emotional language, or specific label pairs being confused. I will verify any pattern myself by rereading the misclassified examples before including it in the README.
