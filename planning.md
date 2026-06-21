# Planning: Emotion Classification Dataset

## 1. Community

I chose an online sentiment dataset composed of short, first-person social media posts. This community's discourse consists of brief text examples where individuals describe their immediate emotional states, often explicitly anchoring their feelings to personal experiences, daily reactions, memories, or opinions.

This community is an excellent fit for a classification task because emotional discourse is highly active and varied in quality, yet structured enough to operationalize. While the raw community data contains multiple complex emotional states, I am focusing on a 3-label subset for this phase of the project to ensure distinct, clean classification boundaries:

| Numeric Label | Label Name |
| :---: | :--- |
| 0 | sadness |
| 1 | joy |
| 2 | anger |

For this initial annotation and boundary-testing phase, the working dataset contains a perfectly balanced subset of 300 total examples, providing exactly 100 examples per class.

---

## 2. Labels

### Label 0: sadness

**Definition:** A post should be labeled `sadness` when the main emotion is loneliness, disappointment, grief, regret, discouragement, or emotional pain.

**Example posts:**

1. `i feel a lil bit gloomy`
2. `i didn t feel abused and quite honestly it made my day a little better`

### Label 1: joy

**Definition:** A post should be labeled `joy` when the main emotion is happiness, gratitude, pride, relief, success, excitement, or positive appreciation.

**Example posts:**

1. `i woke up this morning feeling hopeful and energetic`
2. `i feel really lucky for everything i have this year a job a roof over my head heat and the ability to give my kids a fun christmas`

### Label 3: anger

**Definition:** A post should be labeled `anger` when the main emotion is irritation, frustration, outrage, resentment, annoyance, or hostility.

**Example posts:**

1. `i cannot help but feel outraged to recognize that essentially children in america have no rights at all`
2. `i seem to wake up every day recently feeling immensely irritable and i cant quite work out why`

---

## 3. Hard Edge Cases

Some posts will be genuinely ambiguous because emotions often overlap. The hardest cases will probably be between `sadness` and `anger`, especially when the writer expresses hurt, betrayal, or self-criticism. For example, a post about being betrayed may sound sad because the writer is hurt, but it may also sound angry because the writer feels resentment or outrage.

Another difficult boundary is between `joy` and `sadness` when a post is bittersweet. A person may feel grateful or proud while also describing pain, loss, or struggle. In those cases, the label should be based on the dominant emotion in the post. 

My annotation rule is to label the post by the main emotional signal, not by every emotion mentioned. If a post contains multiple emotions, I will ask: "What is the strongest emotion the writer is expressing overall?" If the text mainly communicates hurt, loneliness, or regret, I will label it `sadness`. If it mainly communicates appreciation, success, or happiness, I will label it `joy`. If it mainly communicates frustration, outrage, irritation, or resentment, I will label it `anger`.

If an example is still unclear after applying this rule, I will mark it as a difficult example in my notes and make a consistent final decision based on the strongest available clue. I will document at least three difficult examples in the README.

---

## 4. Data Collection Plan

I will use the selected 300-example subset from the emotion dataset. The dataset has two columns:

- `text`: the short post or sentence to classify
- `label`: the numeric class label

The current dataset is balanced across three labels:

| Label | Meaning | Count |
|---:|---|---:|
| 0 | sadness | 100 |
| 1 | joy | 100 |
| 2 | anger | 100 |
| **Total** |  | **300** |

I plan to split the dataset into train, validation, and test sets using a stratified split so each label stays balanced across all three sets.

Planned split:

| Split | Percentage | Total Samples | Samples per Class (3 Classes) |
| :--- | :---: | :---: | :---: |
| **Training** | 70% | 210 | 70 |
| **Validation** | 15% | 45 | 15 |
| **Testing** | 15% | 45 | 15 |
| **Total Project Dataset** | **100%** | **300** | **100** |

If a label becomes underrepresented after filtering or cleaning, I will first collect or sample additional examples for that label. If I cannot get enough high-quality examples, I will avoid creating an "other" label and instead revise the taxonomy or reduce the number of labels so the remaining classes are still meaningful and well-supported.

---

## 5. Evaluation Metrics

I will use more than overall accuracy because accuracy alone can hide class-level problems. For example, a model might perform well overall but still do poorly on `anger` if it frequently confuses anger with sadness.

The main metrics will be:

1. **Overall accuracy**  
   This measures the percentage of all test examples classified correctly. It gives a simple high-level view of model performance.

2. **Per-class precision**  
   Precision answers: when the model predicts a label, how often is it correct? This is useful because I want to know whether the model is over-predicting a class such as `joy` or `anger`.

3. **Per-class recall**  
   Recall answers: out of all true examples of a class, how many did the model find? This is important because I do not want the model to miss many examples of a particular emotion.

4. **Macro F1 score**  
   Macro F1 averages performance across labels and treats each label equally. This is useful because the task should work well for all three emotions, not only the easiest or most common one.

5. **Confusion matrix**  
   A confusion matrix will show which labels are being confused with each other. This is especially important for understanding mistakes between `sadness` and `anger`.

These metrics are appropriate because this is a multi-class emotion classification task, and the goal is not only to get a good overall score but also to understand which emotional categories the model handles well or poorly.

---

## 6. Definition of Success

This classifier would be genuinely useful if it can reliably identify the main emotion in short first-person posts and produce balanced performance across all three labels.

For this project, I would consider the model successful if it reaches:

- at least **70% overall accuracy** on the test set
- at least **0.70 macro F1**
- no individual class recall below **0.70**

For deployment in a real community tool, I would want slightly stronger and more stable performance, especially because emotion labels can affect how posts are interpreted or prioritized. A good deployment-ready version should reach around **90% overall accuracy** and show strong per-class performance, with a clear process for sending low-confidence or ambiguous examples to human review.

A "good enough" classroom version does not need to be perfect, but it should show that the fine-tuned model learned meaningful emotional boundaries rather than only memorizing keywords such as "happy," "sad," or "angry."

## 7. AI Tool Plan

Because this project focuses on dataset design, annotation quality, and evaluation rather than generating code, I will use AI tools mainly as a support system for testing label boundaries, assisting annotation, and analyzing model failures.

### 7.1 Label Stress-Testing

Before annotating the full dataset, I will use an AI tool to stress-test my label definitions. I will give the AI my three label definitions for `sadness`, `joy`, and `anger`, along with my hard edge cases, and ask it to generate 5–10 short social media-style posts that sit near the boundary between two labels.

The most important boundaries to test are:

* `sadness` vs. `anger`, especially posts about betrayal, disappointment, resentment, or emotional hurt
* `joy` vs. `sadness`, especially bittersweet posts that include both gratitude and pain
* `joy` vs. `anger`, especially posts where someone feels proud or relieved after overcoming a frustrating experience

After the AI generates these boundary examples, I will try to classify them using my own label rules. If I cannot classify several examples cleanly, that will be a sign that my label definitions are still too vague. In that case, I will revise the definitions before annotating the full dataset. For example, I may add a rule that if the dominant emotion is hurt or loneliness, the label should be `sadness`, but if the dominant emotion is blame, irritation, or resentment, the label should be `anger`.

I will not automatically add all AI-generated examples to my final dataset. The purpose of this step is mainly to improve the label definitions and annotation rules before working with the real examples.

### 7.2 Annotation Assistance

I may use an LLM to pre-label a small batch of examples before reviewing them myself. If I do this, the AI labels will only be treated as suggestions, not final answers. I will personally review each pre-labeled example and make the final annotation decision based on my written label definitions.

If I use AI pre-labeling, I will track it clearly in my annotation notes or spreadsheet. For example, I can add a column such as:

* `ai_suggested_label`
* `final_human_label`
* `review_status`
* `notes`

This will allow me to separate the AI's suggestion from my final decision. It will also make the process transparent for the AI usage disclosure section. If the AI suggestion and my final label disagree, I will write a short note explaining why I chose the final label.

This approach is useful because it can speed up the annotation process, but it still keeps the final dataset human-reviewed and consistent with my label definitions.

### 7.3 Failure Analysis

After training and evaluating the classifier, I will use an AI tool to help analyze the model's wrong predictions. I will create a list of misclassified examples that includes the text, the true label, and the predicted label. Then I will ask the AI to look for patterns in the mistakes.

I will especially look for patterns such as:

* whether the model often confuses `sadness` and `anger`
* whether certain keywords cause the model to over-predict one label
* whether short or vague posts are harder for the model
* whether mixed-emotion posts are more likely to be misclassified
* whether the model relies too much on obvious emotion words instead of the full meaning of the sentence

The AI's analysis will not be accepted automatically. I will verify any pattern myself by checking the actual examples and comparing them to the confusion matrix, per-class precision, per-class recall, and macro F1 score. If the AI identifies a pattern that is supported by the evaluation results, I will include it in my final write-up. If the pattern is not supported by the data, I will not include it.

This step will help me write a stronger evaluation section because it connects the model's numerical performance with specific examples of where the classifier succeeds or fails.

## 8. Annotation Guidelines for Emotion Classification

This document outlines the rules for resolving boundary conflicts between three labels: **sadness**, **joy**, and **anger**. When a post sits near the boundary of two emotions, use the following linguistic and contextual triggers to determine the correct label.

---

### 8.1 Sadness vs. Anger

If a post contains both sadness and anger, I will label it based on the dominant emotional focus.

* **Label as Sadness:** If the post mainly expresses hurt, loneliness, regret, or emotional pain. The user has turned inward, exhibiting helplessness, discouragement, or a sense that "nothing matters anymore," even if the triggering event was unfair.
* **Label as Anger:** If the post mainly expresses blame, resentment, irritation, or hostility toward someone or something. The user is actively pushing outward, using sarcasm, expressing a sense of violation, or retaliating against someone's behavior.

#### Examples
> * **Anger (Boundary Case):** *"They didn't even bother to call and tell me the funeral arrangements. I had to find out through a mutual friend's story."*
>   * *Reasoning:* Even though a funeral implies grief (sadness), the text explicitly targets "them" with blame ("didn't even bother") and resentment for being excluded.
> * **Sadness (Boundary Case):** *"Six months of working late every single night just to get replaced by an external hire. I'm sitting in my car trying not to cry, but honestly, what was the point of any of it?"*
>   * *Reasoning:* Getting replaced triggers frustration, but the dominant focus is the feeling of absolute discouragement ("what was the point") and emotional pain ("trying not to cry").

---

### 8.2 Joy vs. Sadness

If a post contains a mix of joy and sadness (such as bittersweet moments or milestone transitions), I will label it based on the ultimate resolution of the post.

* **Label as Joy:** If the post mainly expresses happiness, gratitude, pride, relief, success, or excitement. Even if the user mentions crying, feeling overwhelmed, or experiencing nostalgic heartache, these elements function as a byproduct of a positive milestone or a massive relief.
* **Label as Sadness:** If the post mainly expresses the pain of loss, longing, or feeling abandoned, where positive memories or past joy are only mentioned to contrast and highlight how miserable or lonely the current state is.

#### Examples
> * **Joy (Boundary Case):** *"The biopsy came back completely clear! I broke down sobbing the second the doctor hung up. I didn't realize how heavy this blanket of fear has been for the last three weeks."*
>   * *Reasoning:* Keywords like "sobbing" and "fear" signal sadness, but the underlying emotion is profound **relief** caused by good news, which falls under Joy.
> * **Joy (Boundary Case):** *"Watching my daughter walk across the graduation stage today. Where did the time go? My heart is completely bursting, but I can't stop the tears from falling."*
>   * *Reasoning:* The passage of time causes nostalgic sadness, but the core driving emotion of the post is immense parental **pride** and celebration.

---

### 8.3 Joy vs. Anger

If a post blends joy and anger (such as spiteful triumphs, "I told you so" moments, or malicious compliance), I will label it based on the poster's current state of mind.

* **Label as Joy:** If the post focuses on celebrating a successful outcome, validation, or the lifting of a burden. The negative past or the toxic behavior of others is merely used as a backdrop to amplify the current feeling of triumph, success, or satisfaction.
* **Label as Anger:** If the post remains fixated on malicious intent, ongoing hostility, or a desire to harm/curse others. Even if the user claims they are "happy" about something bad happening to an enemy, the driving energy of the text is pure vindictiveness and aggression.

#### Examples
> * **Joy (Boundary Case):** *"They all said this idea was a massive waste of time and that I’d fail within a month. Well, look who just signed a six-figure contract today!"*
>   * *Reasoning:* The first half expresses irritation toward doubters, but the actual purpose of the post is to share and celebrate a massive personal **success**.
> * **Joy (Boundary Case):** *"After months of being treated like a second-class citizen by my landlord, the city finally fined him for the building violations. Absolute best news I’ve heard all week."*
>   * *Reasoning:* While fueled by hostility toward the landlord, the text resolves into a celebration of justice and **relief** ("best news I’ve heard all week").

---

### 💡 The Golden Rule: The "End-Drop" Principle
If you find yourself stuck between two labels, **look at the final sentence or phrase of the post.** On social media, users naturally structure their text so that the ultimate emotional takeaway or the "punchline" sits at the very end. Whichever emotion the text lands on in its closing words is almost always the dominant label.

## 9. Baseline Reflection and Hypothesis

The Groq baseline classifier performed reasonably well overall, reaching **75.56% accuracy** on the 90-example test set. This is clearly above random guessing for a three-label task, so the baseline is useful as a comparison point. However, because this baseline model was not trained on my project-specific labeled examples, its performance mainly reflects how well a general LLM can follow the emotion classification prompt.

The baseline result suggests that the task is learnable, but not completely easy. Emotion classification can be difficult because short social media-style posts often contain overlapping emotional signals. For example, sadness and anger can both appear in posts about disappointment, rejection, unfair treatment, or frustration. Joy can also overlap with sadness or anger when a post expresses relief after stress, pride after struggle, or satisfaction after a negative situation is resolved.

I also tested a more detailed prompt that included boundary-case annotation guidelines. However, a stricter prompt did not clearly improve the baseline performance. My hypothesis is that longer and more detailed instructions may have made the model overthink ambiguous examples instead of making a simple dominant-emotion decision. This shows that prompt engineering does not always improve a baseline classifier, especially when the labels have subtle emotional boundaries.

My hypothesis going into fine-tuning was that a model trained directly on my labeled examples would learn my specific annotation patterns better than the general Groq baseline. I especially expected fine-tuning to improve the boundary between `sadness` and `anger`.

The final evaluation supports that hypothesis overall. The fine-tuned model reached **80.00% accuracy**, which is **4.44 percentage points higher** than the baseline. This indicates that fine-tuning helped the model learn the dataset-specific emotional patterns more effectively. However, the confusion matrix also shows that the improvement was not uniform across all labels: the model still has difficulty separating `sadness` and `anger`, which remains the most important boundary to improve.

## 10. Fine-tuned Model Observation

### Data size 600

#### Classification Performance

![finetune600.png](./img/finetune600.png)

The fine-tuned DistilBERT model achieved an overall accuracy of **0.8000** on the 90-example test set. This is higher than the Groq baseline accuracy of **0.7556**, so the fine-tuned model improved over the baseline in this run. The measured improvement was **0.0444**, which means the fine-tuned model performed about **4.44 percentage points better** than the baseline.

Based on the confusion matrix, the fine-tuned model correctly classified **72 out of 90** test examples.

* **Sadness:** Out of 30 true sadness examples, the model correctly classified **23** as sadness, giving sadness a recall of about **0.77**. It misclassified **1** sadness example as joy and **6** as anger. The six sadness-to-anger errors show that some posts expressing hurt, disappointment, or frustration still look like anger to the model.

* **Joy:** Joy was the strongest class by recall. Out of 30 true joy examples, the model correctly classified **26** as joy, giving joy a recall of about **0.87**. It misclassified **3** joy examples as sadness and only **1** as anger. This suggests that the model learned positive emotional signals relatively well.

* **Anger:** Out of 30 true anger examples, the model correctly classified **23** as anger, giving anger a recall of about **0.77**. It misclassified **6** anger examples as sadness and **1** as joy. This mirrors the sadness errors and shows that the main remaining difficulty is the two-way boundary between sadness and anger.

The model predicted `sadness` **32** times, `joy` **28** times, and `anger` **30** times. Because the prediction totals are relatively balanced, the confusion matrix does not show a strong overall bias toward one class.

#### Confusion Matrix Analysis

![confusionMatrix600.png](./img/confusionMatrix600.png)

1. **Joy as the strongest class:**  
The confusion matrix shows that joy is the best-performing label by recall. Out of 30 true joy examples, **26** were correctly predicted as joy. Only **3** were misclassified as sadness and **1** as anger. This suggests that joy contains clearer positive signals such as happiness, relief, gratitude, pride, or excitement.

2. **Sadness and anger have equal recall:**  
Both sadness and anger were correctly classified **23 out of 30 times**, so each has a recall of about **0.77**. Neither class is clearly weaker than the other based on recall alone.

3. **Main confusion pattern: sadness ↔ anger:**  
The largest error pattern is a two-way confusion between `sadness` and `anger`. The model classified **6 true sadness examples as anger** and **6 true anger examples as sadness**. Together, these **12 errors account for two-thirds of all 18 mistakes**. This shows that the model has learned joy relatively well but still struggles with the boundary between the two negative emotions.

4. **No major class over-prediction:**  
The model produced **32 sadness predictions**, **28 joy predictions**, and **30 anger predictions**. These totals are close to the balanced true distribution of 30 examples per class, so there is no strong evidence that the model systematically over-predicts anger or any other label.

> **Conclusion & Root Cause:**  
> Overall, the fine-tuned model outperformed the baseline and learned the `joy` class especially well. Its main remaining weakness is distinguishing `sadness` from `anger`. This boundary is difficult because both classes can contain negative words related to rejection, disappointment, unfairness, frustration, or emotional pain. The confusion is also symmetrical rather than one-directional: sadness is sometimes interpreted as anger, and anger is sometimes interpreted as sadness. To improve the model, I would add more training examples that explicitly contrast inward-focused sadness, such as helplessness, loneliness, or regret, with outward-focused anger, such as blame, irritation, resentment, or hostility.
