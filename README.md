# Project 3 — TakeMeter

TakeMeter is a fine-tined text classifier that evaluates discourse quality in an online Reddit music community, r/LetsTalkMusic, where discourse is active, text-heavy, and varied in quality. My goal is to assess where the model works and where it falls apart after defining the labels, collecting and annotating the data, and fine-tuning the model.

---

## Community Choice and Reasoning

r/LetsTalkMusic is a mid-sized subreddit (~600k members) oriented toward substantive, text-heavy discussion: genre deep-dives, album analyses, recommendation threads, and questions about how the music world works.

**Why it is a good fit for a classification task, and what makes it varied enough to be interesting:**

- Almost every submission is a title plus a paragraph or more of prose. This preserves real diversity in why people post: asking for recommendations, arguing a thesis about a genre, sharing a personal review, or asking a factual/help question.
- The subreddit rewards long, articulate posts, which gives the classifier rich text signal rather than three-word titles.
- The categories overlap at the edges (a recommendation request that opens with a mini-essay; a "discussion" post that is really a thinly veiled review). Thus, genuine ambiguity exists which makes classification complex and interesting.

---

## Label Taxonomy

I will use **four labels**. Each captures a distinct intent, and together they cover the large majority of submissions.

### Label A: Recommendation Request
**Definition:** A post whose primary purpose is to solicit specific music, artists, or albums from other users, usually by describing the author's existing taste or a mood/use-case they want to fill.

- *Example 1:* "I love the wall-of-sound production on My Bloody Valentine's *Loveless* — what other shoegaze or dream-pop records scratch that same itch?"
- *Example 2:* "Putting together a playlist for late-night solo drives. I want moody, atmospheric stuff with sparse instrumentation. Suggestions welcome."

### Label B: Discussion
**Definition:** A post whose primary purpose is to advance or invite a general conversation about music (a genre's history, an industry trend, a critical thesis, or an open-ended question to the community) without centering on the author's personal verdict about one specific work.

- *Example 1:* "Why did nu-metal collapse so completely after ~2003 when it dominated for half a decade? Was it overexposure, the rise of emo, or something internal to the genre?"
- *Example 2:* "I think streaming has fundamentally changed how albums are sequenced. Front-loading singles, shorter tracklists — let's talk about how the format reshaped the art."

### Label C: Review
**Definition:** A post whose primary purpose is to share the author's own evaluation or reaction to a specific album, song, or artist (a verdict, rating, or first-listen impression centered on the poster's subjective experience).

- *Example 1:* "Just finished my first listen of the new Tyler, the Creator record. Here's a track-by-track of what landed and what didn't for me."
- *Example 2:* "Unpopular opinion: Radiohead's *Kid A* is overrated. I've tried for years and it still leaves me cold. Here's why."

### Label D: Question
**Definition:** A post whose primary purpose is to obtain a factual answer or practical guidance (about music theory, gear, the industry, terminology, or "how do I get into X") rather than recommendations or open debate.

- *Example 1:* "What's the actual difference between 'post-rock' and 'math rock'? I keep seeing both applied to the same bands."
- *Example 2:* "I want to start getting into jazz but it's overwhelming. Is there a sensible listening order or entry point for a total beginner?"

---

## Data Collection Plan

**Data Collection Source:** Reddit (r/LetsTalkMusic), manually pasted 200 examples of posts into the csv file.

**Labeling Process:** I read through the examples and manually labelled them to gain a deeper understanding of the nuance and context of the data, and identify hard edge cases better.

**Label Distribution:**
discussion                51
review                    50
question                  50
recommendation_request    49

**3 difficult-to-label examples with my decisions:**

1. **Post:** "Underground concerts are MUCH better than mainstream ones.
    I am not sure if this is the right sub to post this but I'll find someone to agree with me.

    Last night I went to a concert with 5 really popular artists (trappers and some reggaeton) in my country, but I didn't feel too much energy like people were barely singing at all and mostly only the chorus, also no one jumping no one acc moving and having fun, some guys were there trying to get some girls and not even caring about the concert, it felt really empty overall. To be honest I was not in the front of the stage more of the middle and also I don't really listen to the artists only to one but I went with some friends, who also were not even singing at all like barely, I barely knew some lyrics from tiktok but I still tried to sing with all my heart.

    Like 7 months ago I went to the concert of a underground band from my country, they had a tour through the entire country, they are like memphis rap and that type of stuff, and trust me their crowd was CRAZY compared to the other concert, with only like 50 people it felt much better than the other concert with over 5000. Before the main guys there were 2 other guys in their like "friend group" who got just as much attention but for the main guys... There was a moshpit for every single song, everyone was singing bar for bar and word for word, I wish I could relieve that kind of concert again, but they barely come to my city even if they were acc born here, wish they would show more love for us.

    P.S. this is just a rant I had and I want to hear anyone who disagrees or agrees"
    **Decision:** `discussion`
    **Reasoning:** This is an example of a Discussion vs. Review edge case, where it is a discussion since the concert anecdotes are evidence for the general claim, which passes the "remove the specific work and the post still stands" test.

2. **Post:** "Is there a sense of what "reddit music taste" entails?
    We've had some threads about how different types of websites, forums, magazines, and music critic fanbases will have tastes that lean towards certain directions, artists, and albums. I've read discussions about mu-core, rym-core, fantano-core, topster, the general ratings on 1001 albums, best-ever albums, etc. Plus magazines like Rolling Stone, Pitchfork, NME, SPIN, etc.

    For several of these, there's a general sense of people leaning towards indie, alternative, and experimental works alongside popular canonical works.

    But one category I don't quite understand is "Reddit Music Taste". I would see commenters talking about how a certain band or artist is really beloved by Reddit like Queen or David Bowie. Since they're just popular artists in general, it doesn't really tell me much.

    My best guess on what reddit music taste might entail is a sort of strong memetic admiration for an artist. It's usually artists that strike a critical acclaim and commercial success sweet spot. You might be able to predict what comment is going to be upvoted right to the top.

    Is there any consensus or sense of what reddit music even is? Have you encountered this type of comment?"
    **Decision:** `discussion`
    **Reasoning:** This is an example of a Discussion vs. Question edge case, where it is a discussion because "Reddit music taste" isn't a defined term with a factual answer. The author even floats their own speculative guess ("My best guess...") and asks whether any consensus exists — that framing concedes there may be no single right answer and crowdsources interpretations.

3. **Post:** "How do you organise your playlists?
    This is honestly something I’ve been struggling with for a bit so I think maybe I should ask for advice.

    I listen to a large variety of genres - so I tried (or am trying) to separate them like that. However, I’ve noticed that two songs being in the same genre doesn’t necessarily mean they go well together back to back. I’ve also tried mood based ones but I ran into a similar problem.

    I think I could maybe do a mix of both so its genre AND mood but I think that would be too specific and I wouldn’t be able to populate some of them very well. Some advice on how to populate them would be nice if anyone has any.

    I’d be very grateful for any advice on this, I think it’s the main thing stopping me from enjoying music more at the moment."
    **Decision:** `question`
    **Reasoning:** This is an example of the Question vs. Recommendation Request edge case, where it is a question because the author isn't asking for any music. Label A (Recommendation Request) requires soliciting specific songs, artists, or albums. This author already has the songs — "Some advice on how to populate them" means how to structure/group the playlists, i.e. a method, not "give me track suggestions."

---

## Fine-tuning Approach

**Base model:** `distilbert-base-uncased`

**Training setup:** Loads `distilbert-base-uncased` with a classification head `AutoModelForSequenceClassification` and fine-tunes it on the training data from labels.csv, whose train / validation / test split is 70% / 15% / 15%. Stratified so each split has roughly the same label distribution. Training runs for 5 epochs with a learning rate of 2e-5 and per device train batch size of 16 on a T4 GPU on Google Colab.

**A hyperparameter decision:** I chose to increase the number of training epochs from 3 to 5 because the model's training accuracy for 3 epochs was around 0.6 and the testing accuracy was around 0.5. I believed that the model was underfitting on the data. At 5 epochs, the training accuracy increased to 0.8 and the testing accuracy became 0.7.

---

## Baseline Description

**Prompt Used:**

> You are classifying Reddit posts from r/LetsTalkMusic.
> Assign each post to exactly one of the following categories.
>
> recommendation_request: A post whose primary purpose is to solicit specific music, artists, or albums from other users, usually by describing the author's existing taste or a mood/use-case they want to fill.
> Example: "I love the wall-of-sound production on My Bloody Valentine's *Loveless* — what other shoegaze or dream-pop records scratch that same itch?"
>
> discussion: A post whose primary purpose is to advance or invite a general conversation about music (a genre's history, an industry trend, a critical thesis, or an open-ended question to the community) without centering on the author's personal verdict about one specific work.
> Example: "Why did nu-metal collapse so completely after ~2003 when it dominated for half a decade? Was it overexposure, the rise of emo, or something internal to the genre?"
>
> review: A post whose primary purpose is to share the author's own evaluation or reaction to a specific album, song, or artist (a verdict, rating, or first-listen impression centered on the poster's subjective experience).
> Example: "Just finished my first listen of the new Tyler, the Creator record. Here's a track-by-track of what landed and what didn't for me."
>
> question: A post whose primary purpose is to obtain a factual answer or practical guidance (about music theory, gear, the industry, terminology, or "how do I get into X") rather than recommendations or open debate.
> Example: "What's the actual difference between 'post-rock' and 'math rock'? I keep seeing both applied to the same bands."
>
> Respond with ONLY the label name.
> Do not explain your reasoning.
>
> Valid labels:
> recommendation_request
> discussion
> review
> question

**Collection of Results:** Runs the zero-shot baseline using `llama-3.3-70b-versatile` and returns a label string on Google Colab. Per-class baseline metrics are calculated, namely, precision, recall, f1-score, support, along with accuracy, macro avg and weighted avg. Stored in a file comparing baseline and fine-tuning metrics called evaluation_results.json.

---

## Evaluation Report

**Overall Accuracy for Both Models:** `llama-3.3-70b-versatile` (baseline): 0.8, `distilbert-base-uncased` (finetuned): 0.7

**Per-class Metrics for Both Models:**

*Baseline:*

|                        | precision | recall | f1-score | support |
| ---------------------- | --------- | ------ | -------- | ------- |
| recommendation_request | 0.67      | 1.00   | 0.80     | 8       |
| discussion             | 0.78      | 1.00   | 0.88     | 7       |
| review                 | 1.00      | 0.62   | 0.77     | 8       |
| question               | 1.00      | 0.57   | 0.73     | 7       |
| accuracy               |           |        | 0.80     | 30      |
| macro avg              | 0.86      | 0.80   | 0.79     | 30      |
| weighted avg           | 0.86      | 0.80   | 0.79     | 30      |

*Fine-tuned:*

|                        | precision | recall | f1-score | support |
| ---------------------- | --------- | ------ | -------- | ------- |
| recommendation_request | 0.60      | 0.75   | 0.67     | 8       |
| discussion             | 0.75      | 0.86   | 0.80     | 7       |
| review                 | 0.86      | 0.75   | 0.80     | 8       |
| question               | 0.60      | 0.43   | 0.50     | 7       |
| accuracy               |           |        | 0.70     | 30      |
| macro avg              | 0.70      | 0.70   | 0.69     | 30      |
| weighted avg           | 0.70      | 0.70   | 0.69     | 30      |

**Confusion Matrix for Fine-tuned Model:**

Rows are the true label; columns are the predicted label.

| True / Predicted       | recommendation_request | discussion | review | question |
| ---------------------- | ---------------------- | ---------- | ------ | -------- |
| recommendation_request | 6                      | 0          | 0      | 2        |
| discussion             | 0                      | 6          | 1      | 0        |
| review                 | 1                      | 1          | 6      | 0        |
| question               | 3                      | 1          | 0      | 3        |

### Specific Examples the Fine-tuned Model got Wrong with my Analysis

<!--at least 3 specific examples the fine-tuned model got wrong with your analysis of why each one failed. "The model got it wrong" is not analysis — use these guiding questions to go deeper:

Which labels are being confused? Look at the confusion matrix — is there one pair of labels (e.g., analysis → hot_take) that accounts for most of the errors? A directional pattern tells you exactly which boundary the model hasn't learned.
Why is that boundary hard? Is it ambiguous language, sarcasm, short posts, or posts where the topic signals one label but the structure signals another?
Is this a labeling problem or a prompt/data problem? If you labeled those examples consistently but the model still gets them wrong, the issue is in your training data distribution or the boundary itself. If you find you labeled similar posts differently, the issue is annotation inconsistency.
What would need to change to fix it? More examples for the confused class? A tighter label definition? More diverse examples that show the hard case explicitly?-->

The fine-tuned model got 9 of 30 test posts wrong. The errors are not spread evenly: the confusion matrix shows that **`question` ↔ `recommendation_request` is the dominant confused pair**, accounting for 5 of the 9 errors (3 questions predicted as recommendation_request, 2 recommendation_requests predicted as question). The `question` class is the weakest overall (recall 0.43 — only 3 of 7 correct). Every misclassification also came with low confidence (0.33–0.55), so the model wasn't confidently wrong; it was genuinely unsure on these boundaries.

**Example 1 — `question` predicted as `recommendation_request` (confidence 0.38)**

> "How do you organise your playlists? ... I listen to a large variety of genres ... Some advice on how to populate them would be nice if anyone has any."

This is the exact post I flagged as a hard `question` vs. `recommendation_request` edge case during annotation, and I labeled it `question` because the author wants a *method* for grouping playlists, not track suggestions. The model latched onto surface cues that dominate the `recommendation_request` class — a first-person taste description ("I listen to a large variety of genres") plus the words "advice" and "suggestions." It learned a lexical shortcut (taste-description + "advice/suggest" → recommendation_request) instead of the underlying intent. The low confidence confirms it sat right on the boundary.

**Example 2 — `recommendation_request` predicted as `question` (confidence 0.35) — the reverse error**

> "does anyone have any essays or articles or posts they could recommend ... about the direction of music in the digital age?"

The author literally says "recommend," but is asking for *articles and essays* about an abstract topic, not music. My own definition of `recommendation_request` scopes it to "specific music, artists, or albums," so when the thing being solicited is information rather than music, the boundary genuinely blurs — and the model defaulted to the information-seeking `question` framing. This is a boundary/definition gap, not just a model mistake.

**Example 3 — `question` predicted as `discussion` (confidence 0.55, the highest-confidence error)**

> "What is the difference between metal and rock?"

A genuine terminology question, but "difference between genre X and genre Y" reads like an open-ended genre debate, which is squarely `discussion` in my data. The model learned that genre-comparison phrasing → `discussion` and confidently misfired.

**Labeling problem or data problem?** I labeled these consistently — Example 1 matches my recorded annotation rationale verbatim — so this is **not annotation inconsistency**. The issue is the training data distribution and the boundary itself. With only ~200 examples (~50 per class, ~35 per class after the 70/15/15 split), the `question` class never saw enough cases teaching that "a factual query with a right answer" should beat topical cues like a named artist or a genre comparison.

**What would fix it:** more `question` examples, specifically diverse ones that (a) name specific artists/albums, so artist-mention stops being a `recommendation_request` shortcut, and (b) are phrased as genre comparisons, so "difference between X and Y" isn't auto-mapped to `discussion`. Adding explicit contrast pairs (near-identical posts with different true labels) would also force the model to learn the intent distinction rather than the vocabulary. A tighter definition of `recommendation_request` (does "recommend me articles" count?) would resolve Example 3.

### Sample Classifications

<!--Include a Sample Classifications subsection in your evaluation report: a markdown table or list of 3–5 example posts run through your fine-tuned model, each shown with the predicted label and its confidence score, and for at least one correctly-predicted example, a sentence explaining why the prediction is reasonable. Write these out as text (a markdown table or list) — not a screenshot — so they read cleanly in the README.-->

---

## Reflection

<!--Write a reflection on what your model captured vs. what you intended it to capture. This is distinct from listing wrong predictions — it's a higher-level observation about the gap between your label definitions and what the model's decision boundary actually captures. What did the model overfit to? What did it miss?-->

I defined my four labels by **intent** — the author's *primary purpose* in posting. What the fine-tuned model actually learned was a set of **surface-level lexical and structural correlations** that happen to track those intents most of the time. It captured: taste-description plus "advice/suggest/recommend" → `recommendation_request`; genre-comparison phrasing → `discussion`; a named work plus a subjective verdict → `review`. These shortcuts work on the easy majority of posts, but they aren't the same thing as the intent distinction I wrote into the definitions. The model overfit to *vocabulary and structure* and missed the abstraction the labels actually encode: "what is the author trying to get?"

This gap is most visible in the `question` class, which is exactly where intent and surface features diverge the most — a factual question can name specific artists (looks like `recommendation_request`), compare two genres (looks like `discussion`), or describe the author's listening habits (looks like `recommendation_request` again). Because the surface cues point everywhere, the model had no stable signal to anchor on, and `question` collapsed to 0.43 recall. The pervasive low confidence (mostly 0.33–0.55, even on correct answers) is consistent with mushy decision boundaries — the expected result of a small base model (`distilbert-base-uncased`) trained on only ~35 examples per class after splitting.

The most telling finding is that the **zero-shot baseline (`llama-3.3-70b-versatile`, accuracy 0.80) beat my fine-tuned model (0.70)**. The baseline reads the label *definitions* and reasons about intent directly, so it isn't fooled when a question happens to name an album. The fine-tuned model had to *induce* the intent boundary from a handful of examples and, lacking enough signal, fell back on lexical correlations instead. I intended to capture communicative intent, and the model captured the words and shapes that usually accompany it — close, but precisely wrong on the ambiguous edge cases that motivated this community choice in the first place. Closing the gap is a data problem more than a model problem: more `question` examples, deliberate contrast pairs, and a sharper `recommendation_request` definition.

---

## Spec Reflection

One way the spec helped me is brainstorming the hard edge cases and forming rules when I encounter them during annotation. One way implementation diverged from the spec is I previously planned to use AI for annotation assistance to pre-label a batch of posts, but then I chose to label manually to gain a deeper understanding of the nuance and context of the data, and identify hard edge cases better.

---

## AI Usage

**Instance 1**

- *What I gave the AI:* I gave Claude Code the Community, Labels, and Hard Edge Cases sections of the planning.md and asked it to suggest ways to collect posts from Reddit.
- *What it produced:* It strongly suggested to collect data through web scraping.
- *What I changed or overrode:* I wanted to stay close to the data's nuances and context, so I chose to collect data manually instead of through web scraping.

**Instance 2**

- *What I gave the AI:* I gave Claude Code the Community and Labels sections of the planning.md and asked it to suggest the most common hard edge cases.
- *What it produced:* It suggested question vs. recommendation_request, discussion vs. review, recommendation_request vs. review, discussion vs. question.
- *What I changed or overrode:* I created rules for 2 more edge cases, question vs. review and recommendation_request vs. discussion, as those were also prevalent in my dataset.