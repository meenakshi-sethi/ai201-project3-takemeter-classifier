# TakeMeter — planning.md
## AI201 · Project 3

---

## Community Choice

**Community:** r/nba (Reddit)

**Why this community:**
r/nba is one of the largest sports discussion communities on the internet (~6 million members), and it is defined by a constant stream of opinions, reactions, and analysis. The discourse quality varies enormously within a single thread: one reply offers a detailed breakdown of defensive rotations; the next is a hyperbolic claim that "the league is rigged." This variability makes the community an ideal fit for a discourse-quality classifier because:

1. The distinctions between label types are *meaningful to community members themselves* — r/nba regulars frequently call out hot takes vs. analysis in the comments ("this is just a hot take with no evidence").
2. The post volume is enormous, which means data collection is easy and examples are naturally diverse.
3. The labels map onto real community norms, not artificial categories imposed from outside.

---

## Label Taxonomy

### Label 1: `analysis`
A post that makes a **specific, evidence-grounded claim** about basketball — referencing statistics, historical comparisons, tactical observations, or cause-and-effect reasoning. The post advances an argument with at least one concrete supporting detail beyond bare assertion. The reader could, in principle, verify or dispute the claim using data.

**Example 1:**
> "Jokic's passing efficiency from the elbow is unmatched in the modern era. His hockey assists account for nearly 40% of Denver's half-court offense."

**Example 2:**
> "Boston's defensive rating drops by 6 points when Brown is on the bench. His weakside help rotations are underappreciated."

---

### Label 2: `hot_take`
A post that makes a **strong, provocative, or contrarian opinion** about a player, team, or the league — stated with high confidence and little or no supporting evidence. Hot takes are typically absolutist ("the worst ever," "will never win"), emotionally charged, and framed to provoke disagreement. The defining feature is that the claim is presented as fact but is grounded in feeling rather than evidence.

**Example 1:**
> "LeBron James is NOT the GOAT and never will be. Six rings > four rings. Simple math. Case closed."

**Example 2:**
> "Kawhi Leonard is the most overrated injury-prone player ever. Dude plays 40 games a year and gets superstar treatment."

---

### Label 3: `reaction`
A post expressing an **in-the-moment emotional response** to a game, play, trade, or news event — without making a sustained analytical claim or a strong contrarian take. Reactions are characterized by exclamation, immediacy ("just watched," "I can't believe"), personal emotional state, and present-tense urgency. They describe how the author *feels*, not what they *think* about the game at an analytical level.

**Example 1:**
> "That buzzer beater is going to live rent free in my head for years. Unbelievable."

**Example 2:**
> "Finals game 7 and I can't feel my hands. Please let this be the year."

---

## Hardest Anticipated Edge Cases

**Edge case 1: Emotionally-phrased analysis**
Some posts start with emotional language but contain a data-backed claim. Example: *"I'm FURIOUS — Boston's defensive rating without Brown is genuinely 6 points worse. They're not the same team."* The rule: if a specific stat or evidence is present, label it `analysis`, regardless of emotional tone.

**Edge case 2: Hot take dressed as analysis**
Some posts use pseudo-statistics or vague qualitative references to appear analytical. Example: *"Statistically speaking, Luka is too out of shape to win a title. The data is clear."* The rule: if no actual statistic is cited and the claim is primarily about character/effort/destiny rather than performance, label it `hot_take`.

**Edge case 3: Reaction that includes an opinion**
Example: *"That loss was devastating. This team just can't close games."* The second sentence is a mild opinion, but the post is still primarily emotional/reactive. The rule: if the dominant mode is emotional response to a specific game/moment, label it `reaction` even if a brief opinion is embedded.

---

## Data Collection Plan

**Source:** Reddit r/nba — posts and top-level comments scraped manually and via Reddit's public JSON API (`.json` appended to any thread URL). Posts collected from:
- Game threads (primary source for `reaction`)
- Daily discussion threads
- General posts and hot/new feeds (primary source for `analysis` and `hot_take`)

**Labeling process:**
1. Read each post in full before assigning a label
2. Apply the edge case rules above when posts are ambiguous
3. Skip any post shorter than 10 words (too little context to label reliably)
4. Skip posts that are primarily links, memes, or image references

**Target distribution:** ~33% per label (≥60 per label, no label > 70%)

**Estimated collection time:** ~2 hours for 200 examples at deliberate reading pace

---

## Evaluation Metrics and Reasoning

**Primary metric: Macro F1**
Because all three labels are equally important (we don't want the model optimizing for just one class), macro F1 is the right summary metric. It averages precision and recall across classes without weighting by class size.

**Secondary metric: Per-class precision and recall**
Reported separately to identify which label pair the model confuses most. Expected confusion: `analysis` ↔ `hot_take` (both are opinion-forward) more than either ↔ `reaction`.

**Baseline comparison metric:** Overall accuracy on the same held-out test set (30 examples) using Groq's `llama-3.3-70b-versatile` zero-shot.

---

## Definition of "Good Enough"

The fine-tuned model must achieve:
- **Overall accuracy ≥ 0.70** on the test set
- **No single class F1 below 0.55** (model shouldn't completely fail on any one label)
- Fine-tuned model must **outperform the Groq zero-shot baseline** on overall accuracy

If the fine-tuned model fails to beat the baseline, it signals a data quality problem (likely label inconsistency or class imbalance) that warrants investigation before submission.

---

## AI Tool Plan

**Intended uses:**

1. **Label stress-testing:** Paste 20 ambiguous examples into Claude and ask it to assign labels using the definitions above. Compare its decisions to mine and investigate disagreements to tighten the label definitions before annotation begins.

2. **Annotation assistance:** Use Claude to pre-label batches of 20 posts at a time using the full label definitions as a system prompt. Review every prediction, override errors, and track disagreement rate as a data quality signal.

3. **Failure pattern analysis:** After running the fine-tuned model, paste all misclassified examples into Claude and ask it to identify common patterns across the errors. Verify the patterns by re-reading the examples manually.

---

## Stretch Feature Pre-Planning

**Error Pattern Analysis (+1pt):**
Will identify at least one systematic pattern across misclassified examples — expected pattern: short posts (<15 words) classified as `reaction` regardless of actual label, because brief claims share surface features with brief reactions.

**Deployed Interface (+1pt):**
Will add a Gradio cell at the end of the notebook that accepts a typed post and returns predicted label + confidence. Single-cell addition, minimal effort.
