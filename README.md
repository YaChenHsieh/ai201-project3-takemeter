# Emotion Classification Project

## Project Overview

This project builds and evaluates a three-class emotion classifier for short, first-person social media posts. The classifier predicts the dominant emotion expressed in each post:

- `sadness`
- `joy`
- `anger`

The project compares two approaches:

1. A prompt-based Groq LLM baseline
2. A fine-tuned DistilBERT classification model

The main goal was not only to measure overall accuracy, but also to identify which emotional categories were easiest or hardest for the models to distinguish.

---

## Dataset

The dataset contains short social media-style posts describing personal feelings, experiences, reactions, and opinions.

The final experiment used a balanced dataset of **600 examples** across three classes. A stratified split was used so that each class remained evenly represented in the training, validation, and test sets.

The test set contained **90 examples**, with:

| Label | Test Examples |
|---|---:|
| Sadness | 30 |
| Joy | 30 |
| Anger | 30 |
| **Total** | **90** |

The dataset uses the following label mapping:

| Numeric Label | Emotion |
|---:|---|
| 0 | sadness |
| 1 | joy |
| 2 | anger |

---

## Label Definitions

### Sadness

Posts whose dominant emotion is loneliness, disappointment, grief, regret, discouragement, helplessness, or emotional pain.

**Example posts:**

1. `i feel a lil bit gloomy`
2. `i didn t feel abused and quite honestly it made my day a little better`

### Joy

Posts whose dominant emotion is happiness, gratitude, pride, relief, success, excitement, or positive appreciation.

**Example posts:**

1. `i woke up this morning feeling hopeful and energetic`
2. `i feel really lucky for everything i have this year a job a roof over my head heat and the ability to give my kids a fun christmas`

### Anger

Posts whose dominant emotion is irritation, frustration, outrage, resentment, annoyance, blame, or hostility.

When a post contained multiple emotions, I assigned the label based on the strongest overall emotional signal rather than individual keywords.

**Example posts:**

1. `i cannot help but feel outraged to recognize that essentially children in america have no rights at all`
2. `i seem to wake up every day recently feeling immensely irritable and i cant quite work out why`

More detailed boundary rules and annotation decisions are documented in [`planning.md`](./planning.md).

---

## Annotation Challenges

The most difficult boundary was between `sadness` and `anger`. Both emotions can appear in posts about rejection, betrayal, disappointment, unfair treatment, or frustration.

I used the following general rule:

- Label a post as `sadness` when the emotional focus turns inward toward hurt, loneliness, helplessness, regret, or discouragement.
- Label a post as `anger` when the emotional focus turns outward toward blame, irritation, resentment, hostility, or perceived unfairness.

Mixed-emotion examples involving relief, pride, or bittersweet memories were labeled according to the post's dominant emotional resolution.

---

## Models

### Groq Baseline

The baseline used a general-purpose LLM through Groq. The model received the emotion definitions and was instructed to return one of the three labels.

This approach did not train a new model. It measured how well a general LLM could perform the classification task by following a prompt.

I also experimented with a longer prompt containing more detailed boundary rules. However, the stricter prompt did not clearly improve performance. One possible explanation is that the additional instructions caused the model to overanalyze ambiguous examples instead of selecting the dominant emotion directly.

### Fine-Tuned DistilBERT

The second approach fine-tuned `distilbert-base-uncased` on the labeled emotion dataset.

The tokenizer converted each post into model inputs, and the model learned to predict one of the three emotion labels. The validation set was used during training, while the held-out test set was used for the final comparison.

### Fine-Tuned DistilBERT

The second approach fine-tuned `distilbert-base-uncased` on the labeled emotion dataset.

The tokenizer converted each post into model inputs, and the model learned to predict one of the three emotion labels. The validation set was used during training, while the held-out test set was used only for the final comparison.

#### Training Setup and Hyperparameter Decisions

The model was trained using the following configuration:

| Hyperparameter            |                     Value | Purpose                                                                          |
| ------------------------- | ------------------------: | -------------------------------------------------------------------------------- |
| Base model                | `distilbert-base-uncased` | Provides pretrained English language representations                             |
| Number of epochs          |                         3 | Allows the model to learn from the dataset without training for too long         |
| Training batch size       |                        16 | Balances training stability and GPU memory usage                                 |
| Evaluation batch size     |                        32 | Speeds up evaluation because gradients are not calculated                        |
| Learning rate             |                    `2e-5` | Uses a small update size suitable for fine-tuning a pretrained BERT-family model |
| Weight decay              |                    `0.01` | Helps reduce overfitting                                                         |
| Warmup steps              |                        50 | Gradually increases the learning rate at the beginning of training               |
| Evaluation strategy       |               Every epoch | Measures validation performance after each full pass through the training data   |
| Save strategy             |               Every epoch | Saves a checkpoint after each epoch                                              |
| Best-model metric         |                  Accuracy | Selects the checkpoint with the highest validation accuracy                      |
| Maximum saved checkpoints |                         1 | Reduces storage usage by keeping only the most relevant checkpoint               |

I used the hyperparameter values suggested in the project starter code as the initial training configuration. These values provided a reasonable starting point for fine-tuning DistilBERT on a relatively small dataset. In particular, three training epochs helped limit the risk of overfitting, while the learning rate of `2e-5` allowed the pretrained model weights to be updated gradually rather than too aggressively.

The model was evaluated and saved after every epoch. With `load_best_model_at_end=True`, the final model was not automatically the checkpoint from the last epoch. Instead, the checkpoint with the highest validation accuracy was loaded for final testing. This helped avoid using a later checkpoint if its validation performance had already started to decline.

#### Possible Hyperparameter Improvements

Several settings could be adjusted in future experiments:

* **Number of epochs:** Testing 2, 3, 4, or 5 epochs could show whether the model is underfitting or beginning to overfit.
* **Learning rate:** Comparing values such as `1e-5`, `2e-5`, and `3e-5` could identify a more stable or effective update size.
* **Batch size:** Testing batch sizes of 8, 16, and 32 could affect training stability, speed, and memory usage.
* **Weight decay:** Adjusting this value could improve regularization if the model overfits the training data.

For this project, accuracy was used to select the best checkpoint because the dataset was balanced across the three classes. However, using **macro F1** in a future experiment may be more appropriate because it directly considers performance on every class and would penalize a model that performs poorly on one emotion category.


---

## Evaluation Metrics

I evaluated both approaches using:

- Overall accuracy
- Per-class precision
- Per-class recall
- Per-class F1 score
- Macro F1 score
- Confusion matrix

Accuracy provides a high-level result, while the class-level metrics and confusion matrix show whether the model performs unevenly across emotions.

---

### Overall Model Comparison

| Model                 |   Accuracy | Correct Predictions |
| --------------------- | ---------: | ------------------: |
| Groq baseline         | **75.56%** |        68 out of 90 |
| Fine-tuned DistilBERT | **80.00%** |        72 out of 90 |

The fine-tuned model performed **4.44 percentage points above** the Groq baseline. Therefore, fine-tuning improved overall test accuracy in this experiment.

### Per-Class Metrics

#### Groq Baseline

| Class                | Precision |   Recall | F1-score | Support |
| -------------------- | --------: | -------: | -------: | ------: |
| Sadness              |      0.69 |     0.83 |     0.76 |      30 |
| Joy                  |      0.77 |     0.80 |     0.79 |      30 |
| Anger                |      0.83 |     0.63 |     0.72 |      30 |
| **Macro Average**    |  **0.76** | **0.76** | **0.75** |  **90** |
| **Weighted Average** |  **0.76** | **0.76** | **0.75** |  **90** |

The Groq baseline achieved its highest recall on `sadness` at **0.83**, meaning that it correctly identified most true sadness examples. However, its sadness precision was only **0.69**, showing that some examples predicted as sadness actually belonged to another class.

For `anger`, the pattern was reversed. The model achieved relatively high precision of **0.83**, but recall was only **0.63**. This means that its anger predictions were usually correct, but it failed to identify many true anger examples.

#### Fine-Tuned DistilBERT

| Class                | Precision |   Recall | F1-score | Support |
| -------------------- | --------: | -------: | -------: | ------: |
| Sadness              |      0.72 |     0.77 |     0.74 |      30 |
| Joy                  |      0.93 |     0.87 |     0.90 |      30 |
| Anger                |      0.77 |     0.77 |     0.77 |      30 |
| **Macro Average**    |  **0.80** | **0.80** | **0.80** |  **90** |
| **Weighted Average** |  **0.80** | **0.80** | **0.80** |  **90** |

The fine-tuned model performed best on `joy`, with precision of **0.93**, recall of **0.87**, and an F1-score of **0.90**. This indicates that joy was the easiest class for the model to distinguish.

The largest improvement was in `anger` recall, which increased from **0.63** for the baseline to **0.77** for the fine-tuned model. This means that fine-tuning helped the model identify more true anger examples.

However, sadness remained the weakest fine-tuned class by F1-score, at **0.74**. This result is consistent with the confusion matrix, which shows continued overlap between sadness and anger.

---

## Confusion Matrix Analysis

![Fine-tuned model confusion matrix](./img/confusionMatrix600.png)

The confusion matrix was:

| True label \ Predicted label | Sadness | Joy | Anger |
|---|---:|---:|---:|
| Sadness | **23** | 1 | 6 |
| Joy | 3 | **26** | 1 |
| Anger | 6 | 1 | **23** |

The largest error pattern was a **two-way confusion between sadness and anger**:

- **6 true sadness examples** were predicted as anger.
- **6 true anger examples** were predicted as sadness.
- These 12 cases account for **two-thirds of the model's 18 total mistakes**.

Joy was easier for the model to recognize:

- 26 of 30 joy examples were correctly classified.
- 3 joy examples were classified as sadness.
- 1 joy example was classified as anger.

The model predicted `sadness` 32 times, `joy` 28 times, and `anger` 30 times. These totals are close to the balanced true distribution of 30 examples per class, so the confusion matrix does **not** show strong over-prediction of anger or another label.

These results suggest that positive emotional language was relatively distinct, while posts involving hurt, disappointment, rejection, frustration, blame, or emotional pain created overlap between sadness and anger.

---

## Analysis

My original hypothesis was that fine-tuning on project-specific examples would help the model learn the annotation rules more effectively than a general LLM baseline. In particular, I expected improvement on the boundary between sadness and anger.

The overall result supports the first part of this hypothesis. The fine-tuned DistilBERT model achieved **80.00% accuracy**, compared with **75.56%** for the Groq baseline, an improvement of **4.44 percentage points**.

However, the confusion matrix shows that fine-tuning did not completely solve the sadness-versus-anger boundary. The confusion is symmetrical rather than one-directional: the model sometimes interprets sadness as anger and sometimes interprets anger as sadness.

Possible reasons include:

1. **Overlapping language**  
   Sadness and anger frequently share words related to pain, disappointment, rejection, unfairness, and frustration.

2. **Insufficient boundary examples**  
   The training set may not contain enough carefully contrasted examples showing inward-focused sadness versus outward-focused anger.

3. **Ambiguous or mixed-emotion posts**  
   Some posts genuinely contain both hurt and resentment, making the dominant emotion difficult to determine from a short sentence.

4. **Annotation consistency**  
   If similar mixed-emotion examples were labeled differently, the model may receive an unclear decision boundary.

5. **Training configuration**  
   The final performance may still depend on the number of epochs, learning rate, random seed, and checkpoint selection.

The result is useful because it identifies a specific weakness. The fine-tuned model improved overall and learned joy particularly well, but it still needs stronger training coverage for the distinction between sadness and anger.
## Success Criteria

The original classroom success criteria were:

- At least **70% overall accuracy**
- At least **0.70 macro F1**
- No class recall below **0.70**

The fine-tuned model reached **80.00% accuracy**, and all three class recalls were above **0.70**:

- Sadness recall: **76.67%**
- Joy recall: **86.67%**
- Anger recall: **76.67%**

Therefore, the fine-tuned model met the overall accuracy target and the per-class recall target. The macro F1 requirement should be confirmed using the per-class evaluation metrics in `evaluation_results.json` or the classification report if that value is recorded there.
## Limitations

This project has several limitations:

- The dataset is relatively small.
- It contains only three emotion categories.
- Posts may express more than one emotion at the same time.
- The labels represent the dominant emotion and may simplify complex emotional states.
- Results are based on one test split and may change with another random seed.
- The fine-tuned model may require additional hyperparameter testing.
- Emotion classification is subjective, so some examples may reasonably support more than one label.

This model should not be used to make high-stakes judgments about a person's mental state or well-being.

---

## Future Improvements

The next version of the project could:

- Add more sadness-versus-anger boundary examples
- Review mislabeled or genuinely ambiguous training examples
- Train with multiple random seeds
- Tune the learning rate, batch size, and number of epochs
- Use early stopping and select the best validation checkpoint
- Compare DistilBERT with another pretrained text classifier
- Add confidence scores and send low-confidence examples for human review
- Perform a more detailed error analysis using the actual misclassified posts

---

## AI Tool Usage

AI tools were used as assistants during this project, but their output was not treated as automatic ground truth. I reviewed the suggestions, compared them with my label definitions and evaluation results, and made the final decisions myself.

### Instance 1: Label Boundary Stress-Testing and Annotation Review

I gave an LLM my definitions of `sadness`, `joy`, and `anger` and asked it to generate five to ten short posts near the boundaries between the labels. I then labeled the generated examples manually.

The purpose was not to let the LLM complete the annotation for me. Instead, I used the generated examples to test whether my own label definitions were clear and consistent. Some examples were difficult to classify because they contained more than one emotion, especially combinations of sadness and anger.

After reviewing these examples, I refined my labeling rule:

* `sadness` focuses more on inward emotional pain, loneliness, helplessness, or rejection.
* `anger` focuses more on outward blame, resentment, irritation, or unfairness.

I did not accept every generated example or interpretation. Some examples were too ambiguous to support only one label, so I treated them as evidence that the label boundary needed to be clarified rather than as final training examples.

### Instance 2: Diagnosing Fine-Tuning Performance

I asked an AI tool to help me analyze why the first fine-tuned model performed poorly. The initial version used approximately 300 labeled examples and achieved only about **42% accuracy**. I shared the training results and discussed possible causes with the AI.

The AI suggested that the dataset might be too small for the model to learn the three emotion categories reliably, especially the difficult boundary between sadness and anger. I did not accept this suggestion automatically. I tested it by expanding the dataset from 300 to 600 balanced examples and retraining the model.

After the dataset was increased, the fine-tuned model achieved **80.00% accuracy**. Based on this experiment, I concluded that the smaller dataset did not provide enough coverage and that increasing the amount and diversity of training data improved model performance.

### Instance 3: Error Analysis and Future Task Design

I also used an AI tool to discuss the model's confusion matrix and wrong predictions. The analysis showed that sadness and anger were still frequently confused, even after the overall accuracy improved.

The AI initially suggested adding more boundary examples and refining the label definitions. I agreed with these suggestions, but I also revised the interpretation after reviewing the wrong predictions manually. Some posts genuinely contained multiple emotions, and even a human annotator could reasonably disagree about the dominant label.

This led me to consider whether the original single-label, three-class design may be too restrictive for short emotional posts. For a future version, I would compare the current approach with either:

* binary classification using `positive` and `negative`, or
* multi-label classification that allows more than one emotion label.

I did not replace the current task design during this project because the assignment required the original three labels. Instead, I documented binary and multi-label classification as possible future improvements.

### Annotation Assistance Disclosure

AI was used to generate stress-test examples and discuss ambiguous cases, but it did not automatically label the final dataset. I manually reviewed the examples and made the final annotation decisions.

---

## Conclusion

The fine-tuned DistilBERT model achieved the best overall result with **80.00% accuracy**, compared with **75.56%** for the Groq baseline. Fine-tuning therefore improved performance by **4.44 percentage points** on the 90-example test set.

The fine-tuned model learned `joy` particularly well, correctly classifying 26 of 30 joy examples. Its most important remaining failure pattern was the two-way confusion between `sadness` and `anger`: 6 sadness examples were predicted as anger, and 6 anger examples were predicted as sadness.

This experiment shows that fine-tuning can improve a project-specific classifier, but higher overall accuracy does not mean every label boundary has been fully learned. More carefully selected sadness-versus-anger examples, tighter annotation consistency, and additional training experiments would likely produce the largest next improvement.

### Per-Class Metrics

#### Groq Baseline

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| Sadness | 0.69 | 0.83 | 0.76 | 30 |
| Joy | 0.77 | 0.80 | 0.79 | 30 |
| Anger | 0.83 | 0.63 | 0.72 | 30 |
| **Accuracy** |  |  | **0.76** | **90** |
| **Macro Average** | **0.76** | **0.76** | **0.75** | **90** |
| **Weighted Average** | **0.76** | **0.76** | **0.75** | **90** |

The Groq baseline achieved its highest recall on `sadness` at **0.83**, meaning it recognized most true sadness examples. However, sadness precision was lower at **0.69**, which suggests that the model also assigned the sadness label to some examples from other classes.

Its weakest class was `anger`, with recall of **0.63**. This means the baseline missed a noticeable portion of true anger examples, even though its anger precision was relatively strong at **0.83**. In other words, when the baseline predicted anger, it was often correct, but it did not predict anger often enough.

#### Fine-Tuned DistilBERT

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| Sadness | 0.72 | 0.77 | 0.74 | 30 |
| Joy | 0.93 | 0.87 | 0.90 | 30 |
| Anger | 0.77 | 0.77 | 0.77 | 30 |
| **Accuracy** |  |  | **0.80** | **90** |
| **Macro Average** | **0.80** | **0.80** | **0.80** | **90** |
| **Weighted Average** | **0.80** | **0.80** | **0.80** | **90** |

The fine-tuned DistilBERT model performed best on `joy`, with precision of **0.93**, recall of **0.87**, and an F1-score of **0.90**. This suggests that positive emotional language was the most distinct category for the model.

The fine-tuned model also improved the balance across classes. Compared with the baseline, `anger` recall increased from **0.63** to **0.77**, and the overall macro F1 improved from **0.75** to **0.80**. However, `sadness` remained the most difficult class, with an F1-score of **0.74**, showing that overlap between sadness and anger was still the main challenge.

## Misclassification Analysis

The fine-tuned model made **18 incorrect predictions out of 90 test examples**. The confusion matrix shows that the most important remaining difficulty was the boundary between `sadness` and `anger`. This was a two-way confusion: **6 sadness examples were predicted as anger, and 6 anger examples were predicted as sadness**. These 12 cases account for **two-thirds of all 18 errors**.

At the same time, some wrong predictions also suggest mixed-emotion posts or possible annotation ambiguity, rather than a pure model failure.

### Example 1: Sadness Predicted as Anger

> “ill just paraphrase i ranted about not being able to trust anybody and being hurt feeling rejected etc”

- **True label:** `sadness`
- **Predicted label:** `anger`
- **Confidence:** 0.52

This example contains signals for both sadness and anger. The phrase “ranted” suggests outward frustration, which is associated with anger, while “hurt” and “rejected” suggest inward emotional pain, which is more consistent with sadness.

The model likely focused too much on the anger-related expression and did not fully capture that the emotional center of the sentence is being hurt and rejected. This is a class-boundary problem rather than a random mistake.

### Example 2: Joy Predicted as Sadness

> “i just remember being so fully stressed out and while i had fun i feel it could have been more lively”

- **True label:** `joy`
- **Predicted label:** `sadness`
- **Confidence:** 0.54

This post contains mixed emotional signals. On one hand, “had fun” supports the joy label. On the other hand, “fully stressed out” and “could have been more lively” introduce stress and disappointment.

This is a difficult example because the topic and wording are mixed. The model likely paid more attention to the negative language than to the brief positive phrase. At the same time, this example may also reveal some annotation ambiguity, because the dominant emotion is not completely obvious.

### Example 3: Anger Predicted as Joy

> “i feel like so much of my life has been rushed through like just the means to an end and now it feels like i am enjoying everything i possibly can for what it truly is”

- **True label:** `anger`
- **Predicted label:** `joy`
- **Confidence:** 0.81

This example is especially interesting because the model’s prediction is highly understandable. While the beginning of the sentence describes dissatisfaction with the past, the ending shifts strongly toward a positive emotional resolution: “now it feels like i am enjoying everything i possibly can.”

The model appears to have captured this final emotional direction, treating the dominant present emotion as joy. Since the prediction confidence is relatively high, this points to an inherently ambiguous labeling case rather than a true model failure; forcing a single-label constraint on such a multi-layered sentiment inevitably creates a discrepancy between human annotation and model judgment.

### Example 4: Anger Predicted as Sadness

> “i feel so resentful about having to take care of us and not getting to do what i want to do”

- **True label:** `anger`
- **Predicted label:** `sadness`
- **Confidence:** 0.67

The word “resentful” is a strong anger cue, but the rest of the sentence also expresses limitation, frustration, and lack of freedom. The model may have interpreted this as discouragement or helplessness, which are closer to sadness.

This example shows why the sadness-versus-anger boundary is difficult: both emotions can appear in posts about unfair responsibilities, personal sacrifice, and frustration. The correct label depends on whether the emotional focus is outward resentment or inward discouragement.


### Overall Error Analysis

The wrong predictions suggest that not all errors come from the same source. Some are genuine `sadness` versus `anger` boundary errors, while others involve mixed emotions or potentially ambiguous labels. Short social-media-style posts often provide limited context, and human emotions are naturally complex. A single post may express both sadness and joy, or both anger and sadness, which means that even human annotators may reasonably disagree about the dominant label.

This raises a broader question about the original task design. Although the three-class setup provides more detailed emotional categories, a binary classification task using `positive` and `negative` labels may have produced more consistent annotations and a clearer decision boundary. Both sadness and anger could be grouped as negative emotions, reducing the ambiguity between closely related emotional states.

However, binary classification would also remove useful distinctions between different types of negative emotion. A useful future experiment would therefore be to compare the current three-class model with a binary positive-versus-negative classifier. Another possible approach would be multi-label classification, which would allow a post to receive more than one emotion label when multiple emotions are genuinely present.

## Spec Reflection

### How the Spec Guided My Implementation

One way the spec guided my implementation was through label stress-testing. Before completing the full annotation process, the spec instructed me to give an LLM my label definitions and ask it to generate five to ten examples near the boundaries between the labels.

I asked the LLM to generate short posts that were difficult to distinguish, especially examples near the boundary between `sadness` and `anger`. I then labeled these examples manually rather than accepting the LLM's output as the final answer.

This exercise helped me realize that broad label definitions were not enough. I also needed clear boundary rules. For example, I defined `sadness` as focusing more on inward emotional pain, loneliness, helplessness, rejection, or discouragement, while `anger` focused more on outward blame, resentment, irritation, hostility, or perceived unfairness.

Some of the generated examples were difficult even for me to classify consistently. This showed that the label definitions still needed refinement before I annotated the full dataset. As a result, the spec helped me identify ambiguous cases early and develop more consistent annotation rules.

### How My Implementation Diverged from the Spec

My implementation diverged from the original plan in the size of the dataset. The initial project plan used approximately 300 labeled examples. However, when I fine-tuned the model using this smaller dataset, the training process was unstable, the evaluation results varied considerably, and the final accuracy was only approximately **42%**.

This result suggested that 300 examples did not provide enough coverage for the model to learn the three emotion categories and their boundaries reliably. In particular, the smaller dataset did not contain enough diverse examples to distinguish overlapping negative emotions such as `sadness` and `anger`.

Because of this observation, I expanded the dataset from 300 to 600 labeled examples while keeping the three classes balanced. After increasing the amount of training data, the fine-tuned model achieved **80.00% test accuracy**, which was substantially higher than the result from the smaller dataset.

This divergence was therefore an evidence-based implementation decision. Although the original dataset size was enough to build and run the training pipeline, the evaluation results showed that additional data was necessary to produce a more stable and useful classifier.

### Demo on fine-tuned model with label and confidence visible

|index|Text|True Label|Predicted Label|Confidence|Result|
|---|---|---|---|---|---|
|0|ive just spent the last half hour feeling ridiculously angry over insensitive comments from my partner but that all changed a few minutes ago to real pride over how much i have changed|anger|anger|0\.53|Correct|
|1|i was feeling kind of rebellious and my post was a little on the|anger|anger|0\.48|Correct|
|2|im just feeling rebellious|anger|anger|0\.5|Correct|
|3|ill just paraphrase i ranted about not being able to trust anybody and being hurt feeling rejected etc|sadness|anger|0\.49|Incorrect|
|4|i feel like this is like fake bogart said at one point in the show|sadness|joy|0\.36|Incorrect|