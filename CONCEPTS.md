# How JustTrackIt works — the concepts behind it

I built this to log my food by voice without any number being guessed. The design below is how it actually works, concept by concept, and where each concept comes from. Written so that anyone reading the repo can see the thinking, not just the code.

## The shape of it

The app is a fixed workflow — a small graph with four nodes — and the language model decides things *inside* each node, never the shape of the graph itself:

```
voice / text → EXTRACT → LOOK UP → DISPLAY → LOG
```

1. **Extract** — pull out each food and its amount from what I said.
2. **Look up** — find every number in a food database.
3. **Display** — show me what's about to be logged.
4. **Log** — save it, unless I say otherwise.

## 1. Tool use — give tools, don't let the model freestyle

The model gets exactly three tools: search USDA, search Open Food Facts, and look up a barcode. Each tool has a fixed input and returns matches with an id. The model returns only match ids and grams — the app turns the tool results into numbers itself.

That is the whole trick. A model that is asked "how many calories in this?" will answer from memory and sound confident doing it. A model that is only allowed to call tools and return ids *cannot* output a calorie from memory, because there is no path in the workflow where a number comes from the model. The numbers come from the tools or from me.

Reference: Chip Huyen, *AI Engineering*, chapter 6 — on giving agents tools with clear contracts instead of letting them figure everything out, which is where most hallucinations come from. Also Anthropic, "Building effective agents" (December 2024) — tool design as the main lever on reliability.

## 2. RAG — retrieval before generation

Every number in this app is retrieved, not generated. The database lookup *is* the retrieval step: the model generates a search query, the database returns records, and the answer is built from those records. If the retrieval returns nothing, the app says "I don't know" instead of filling the gap with a guess.

The fallback chain is the same idea, one level deeper. When a dish has no direct match — bendakaya fry, say — the app works out what the dish is, states its closest find, breaks it into ingredients, and retrieves each ingredient. That is corrective retrieval: when the first retrieval misses, reformulate the query and retrieve again, instead of giving up or guessing.

Reference: Huyen, *AI Engineering*, chapter 6 (retrieval-augmented generation); Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020).

## 3. Workflow, not a free agent

The four nodes above never change order and never get skipped. The model decides *within* a node — which foods to extract, which tool to call, which match is best — but it does not decide what happens next. That is a deliberate choice: a fixed workflow is predictable, testable, and cheap, while a free-roaming agent is none of those.

Anthropic draws the same line: workflows are for tasks where the steps are known, agents for tasks where the system itself has to decide the steps. Logging food is a known-steps task, so it is a workflow with agentic steps inside it.

Reference: Anthropic, "Building effective agents" (December 2024).

## 4. Parallelization — three searches at a time

The lookups run in parallel, up to three at a time. A four-item dinner does not take four sequential round trips; the three lookups go out together and the results come back together. One slow source never blocks the others. This is the same parallelization pattern Anthropic describes for independent subtasks — it buys speed without buying complexity, because the nodes are independent by construction.

## 5. Grounding by construction

Grounding here is not a check added at the end. It is built into the flow:

- The model never outputs a number. Only tool results and my stated numbers carry numbers.
- The app computes the numbers from the tool results, using the match id the model chose.
- My stated numbers always win over the database.
- An amount the model had to assume is labelled as assumed, on the screen, before anything logs.
- No match means "I don't know", with blank fields for me to fill from the label.

If any of those rules broke, the grounding would break with it — so they live in the code, not in a prompt asking the model to be careful.

## 6. The display step — human in the loop

After the lookup, the app shows what is about to be logged: each food, its numbers, and where each number came from. I can say yes, or send a note with what to change, and the flow runs again on the changed part. If I don't reply within 60 seconds, it logs as shown — silence is acceptance, and the countdown is on the screen so it is never a surprise.

The same step is what catches a wrong match. If the database returns a packaged butter naan for a chapati with ghee, I see the 320 kcal on the screen and correct it. No automated check would know my chapati better than I do.

## 7. Failure handling — retry twice, then say so

Every lookup tries twice. If it still fails, the app says plainly that it can't do it right now because of a timeout — it does not pretend, and it does not keep retrying in a loop. The failed log is queued and retried later, when the app opens again. If the retry fails twice as well, the app says it has failed and asks me for the exact numbers.

That order matters: retry, admit, queue, admit again, hand it to the human. An agent that hides its failures is worse than one that has them.

## 8. Fast log — the escape hatch

When the lookup can't find something and I want it logged fast, there is always a fast track: give the macros (protein, carbs, fat) and the calories are worked out exactly — protein × 4 + carbs × 4 + fat × 9. Or give the calories and the macros are estimated from my target split (213 g carbs / 105 g protein / 60 g fat) — stated plainly as an estimate, never hidden. Best is macros; calories alone works too.

The same idea fills the gap at the 8 pm audit: an item that stayed unknown all day can be fast-logged instead of leaving a hole in the day.

## 9. Time-aware meals and snack alerts

The app knows the time, so it infers the meal from the clock and the size of the meal: 8 am–12 pm, a big meal is breakfast (brunch on a weekend); 12–4 pm is lunch; 4 pm onwards is dinner. If the meal is much smaller than a main meal for that time, it asks whether it's a snack. Extraction itself doesn't change per meal — the time rules only help the app understand the slot.

That slot feeds the snack alerts: a light snack gets a short "you're on track", a bigger one gets a gentle note to keep the next meal lighter. Positive first, useful second.

## What I deliberately did not do

- **No image logging yet.** A photo of the food is an extra step to think about later; portion estimation from a photo is the least reliable part of the whole flow.
- **No sync to other apps.** The diary lives here, in the browser, on my device.
- **No credentials in the repo.** Whoever downloads the app provides their own keys in Settings, stored only in their own browser.

## References

- Chip Huyen, *AI Engineering* (O'Reilly, 2025), chapter 6 — RAG, tool use, and agents.
- Anthropic, "Building effective agents" (December 2024) — workflows vs agents, parallelization, tool design.
- Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (NeurIPS 2020).
- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022) — interleaving reasoning with tool calls, which is what the lookup loop does.
