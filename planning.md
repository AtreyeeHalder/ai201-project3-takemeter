# Planning: Post Classification for r/LetsTalkMusic

TakeMeter is a fine-tined text classifier that evaluates discourse quality in an online Reddit music community, r/LetsTalkMusic, where discourse is active, text-heavy, and varied in quality. My goal is to assess where the model works and where it falls apart after defining the labels, collecting and annotating the data, and fine-tuning the model.

---

## Community

r/LetsTalkMusic is a mid-sized subreddit (~600k members) oriented toward substantive, text-heavy discussion: genre deep-dives, album analyses, recommendation threads, and questions about how the music world works.

**Why it is a good fit for a classification task, and what makes it varied enough to be interesting:**

- Almost every submission is a title plus a paragraph or more of prose. This preserves real diversity in why people post: asking for recommendations, arguing a thesis about a genre, sharing a personal review, or asking a factual/help question.
- The subreddit rewards long, articulate posts, which gives the classifier rich text signal rather than three-word titles.
- The categories overlap at the edges (a recommendation request that opens with a mini-essay; a "discussion" post that is really a thinly veiled review). Thus, genuine ambiguity exists which makes classification complex and interesting.

---

## Labels

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

## Hard Edge Cases

Real ambiguity is expected in a text-heavy discussion like r/LetsTalkMusic. The most common collisions, and the rules I will apply to handle them during annotation:

1. **Question vs. Recommendation (D vs. A).** "How do I get into jazz?" can be answered with *facts* (a listening framework) or *recommendations* (specific albums).
   **Rule:** If the author asks for an *approach/explanation*, label D. If they explicitly ask for *specific titles/artists*, label A. When the post asks for both, default to A only if specific-title requests dominate; otherwise D.

2. **Discussion vs. Review (B vs. C).** "Why OK Computer is the most important album of the 90s." This reads like a thesis (B) but is anchored to one specific work and is fundamentally the author's verdict (C).
   **Rule:** If the argument generalizes beyond the work (about an era, genre, or trend), label B. If it stays a verdict on the single work itself, label C. The test: *could you remove the specific album and still have the post?* If yes, label B.

3. **Recommendation Request vs. Review (A vs. C).** A post that spends three paragraphs reviewing an album it loves and ends with "…so what else sounds like this?" The review (C) content is large but instrumental; the author's actual ask is recommendations (A).
   **Rule:** Classify by the *terminal ask / call to action.* If the post is structured to elicit suggestions, it is label A even if it contains heavy review prose.

4. **Discussion vs. Question (B vs. D).** "What makes a chord progression sound 'sad'?": open enough to invite debate (B), but also has a real answer (D).
   **Rule:** If there is a *correct/factual answer the author wants*, label D. If the author is inviting *opinions and there's no single right answer*, label B.

5. **Question vs. Review (D vs. C).** "Is the production on this album supposed to sound this muddy, or is it just me?" mixes a factual ask (D) with the author's own negative reaction to a specific work (C).
   **Rule:** If the author primarily wants an *answer or explanation* and the named work is just the occasion for the question, label D. If the post is really the author *delivering their verdict* on the work and the question is rhetorical or secondary, label C. The test: *would a single factual reply resolve the post?* If yes, label D; if the author clearly wants others to react to their opinion, label C.

6. **Recommendation Request vs. Discussion (A vs. B).** "What are the most underrated albums of the 2010s?" invites a broad community conversation (B) but also reads as a call for specific titles (A).
   **Rule:** If the post wants *specific titles/artists for the author's own listening*, label A. If it opens a *general conversation where the suggestions are the topic of debate* rather than a personal to-listen list, label B. The test: *is the author building their own queue, or starting a discussion?* Personal use-case or taste framing leans A; broad, opinion-canvassing framing leans B.

**General annotation protocol for ambiguity:**
- Apply the *primary-intent / terminal-ask* heuristic first; the body's *closing move* usually reveals intent.
- Maintain a running *edge-case log* (a separate `edge_cases.md`) documenting each hard call and the rule applied, so decisions stay consistent across the dataset and the rules can be refined.
- For genuinely 50/50 posts, default to the label matching the *call to action* over the label matching the bulk of the prose.
- Periodically re-review a sample of earlier annotations against the evolving rules to catch drift.

---

## Data Collection Plan

**Source.** Reddit's public API (via PRAW) and/or Pushshift-style archives for historical reach. I will pull from:
- `hot`, `new`, and `top` (all-time and yearly) listings of r/LetsTalkMusic to avoid over-sampling a single time window or popularity tier.
- Existing *user/mod flairs* as *weak* hints for seeding (not ground truth), every example is still hand-labeled.

**Volume target.** A balanced set of *~50 examples per label*, for a total of *~200 hand-labeled posts*.

**Fields collected.** Title, selftext (body). They will be stored as rows in a CSV file with a `label` column added during annotation. The rows will be `text` containing the title and selftext, and `label` containing the label.

**Handling underrepresented labels after 200 examples.** If, at the 200-example checkpoint, any label is clearly under target (e.g., < ~20 examples), I will take the following steps:
1. *Targeted querying.* Search r/LetsTalkMusic for label-correlated keywords (e.g., "first listen," "track by track," "review" for C; "how do I," "beginner," "difference between" for D) and the sub's review/question flairs to find more candidates.
2. *Widen the time window* via `top` (year/all) to surface rarer post types.
3. If a label remains stubbornly rare after targeted effort, *reconsider the label set*: collapse a sparse label into an adjacent one (e.g., fold a thin "Review" into "Discussion") rather than ship a class the model can't learn. I will document any such merge.
4. As a last resort for mild imbalance, address it at training time with *class weights / stratified sampling*, and report the imbalance.

---

## Evaluation Metrics

Accuracy alone is misleading here because the classes may not be perfectly balanced, and a model that just predicts the majority class could post a deceptively high accuracy while being useless on the rare classes. Per-class visibility is more useful to evaluate than a single aggregate.

**Metrics I will use and why:**

| Metric | Why it's right for this task |
|---|---|
| Per-class Precision & Recall | The core diagnostic. For an auto-flair tool, precision matters when a wrong flair actively misleads readers (e.g., labeling a Review as a Recommendation Request frustrates users seeking suggestions). Recall matters when a triage queue must not miss posts of a type (e.g., Question posts that need a mod nudge). Reporting both per class exposes exactly where the model fails. |
| Macro-F1 | The headline number. Because classes are imbalanced, macro-F1 (unweighted mean of per-class F1) treats the rare classes as equally important to the common ones, preventing a majority-class predictor from looking good. |
| Weighted-F1 / Accuracy | Reported as a secondary, deployment-realistic number, it reflects how the model performs on the actual class distribution it will see in production. |
| Confusion Matrix | Essential given the edge cases. I expect specific, predictable confusions (A <--> C, B <--> C, A <--> D). The matrix tells me which pairs the model conflates so I can target them with better features, more data, or clearer label definitions; a single F1 number hides this. |

**Validation method.** Stratified train/test split (or stratified k-fold given the modest dataset size) so every class is represented in evaluation. I will also keep a small held-out set of the documented edge cases and report performance on it separately, since aggregate metrics can hide poor handling of exactly the ambiguous posts that matter most.

---

## Definition of Success

**Baselines first.** I'll establish a majority-class baseline (likely ~30–40% accuracy) and a simple TF-IDF + logistic regression baseline. The real model must beat these convincingly to justify itself.

**What "genuinely useful" looks like:**
- *Macro-F1 ≥ 0.75* across the four classes, meaning no single class is being sacrificed.
- *No class with recall below ~0.65*, a tool that systematically misses an entire post type isn't trustworthy.
- Confusion concentrated in the *expected, defensible edge pairs* (A <--> C, B <--> C) rather than scattered randomly; random errors signal the model hasn't learned the task.

**"Good enough" for deployment in a real community tool:**
A realistic threshold for an *assistive* (not fully autonomous) tool (e.g., *suggesting* a flair that a mod or the OP can confirm) is *macro-F1 in the 0.75–0.80 range with high per-class precision on the high-stakes classes.* Because the worst-case failure is a mislabeled post, I would deploy as a *human-in-the-loop suggester* at this level.

---

## AI Tool Plan

I will use AI tools at the points where they help the annotation/evaluation workflow.

1. **Label stress-testing (before annotating).** I will give Claude Code my four label definitions and the Hard Edge Cases section and ask it to generate 5–10 posts that deliberately sit on the boundary between two labels. I'll try to classify each one myself: any post I cannot assign cleanly exposes a gap in my definitions, and I will tighten the definition or add an edge-case rule before annotating 200 examples.

2. **Annotation assistance (pre-labeling).** I will not use an LLM to pre-label a batch of posts. Labelling manually will help me gain a deeper understanding of the nuance and context of the data, and identify hard edge cases better.

3. **Failure analysis (after evaluation).** I will export the misclassified examples (text + true label + predicted label) and ask the LLM to cluster them into patterns (e.g., "Reviews framed as theses get called Discussion," or short posts losing intent signal). I'll treat these as *hypotheses only*: I verify each by pulling the actual posts it cites and checking the pattern holds against the confusion matrix before any of it goes into my evaluation write-up.