# JustTrackIt

I built this to log my food by voice or text without any number being guessed. An LLM understands what I ate — even messy voice notes like "half a multigrain bagel" or "the usual lunch" — and the database lookup is what grounds it. Every calorie and macro comes from a food database, from a label, or from me — never from the model making it up.

## How it works

1. Tap the mic and say what you ate, or type it. Messy is fine: "1 cup cooked red rice and a protein bar, 230 calories, 18g protein".
2. The LLM interprets it: it pulls out each food, the amount, and any numbers I stated — and decides what to look up for each one. It also figures out the meal from the time and the size of the meal: 8 am–12 pm is breakfast (brunch on a weekend), 12–4 pm lunch, 4 pm onwards dinner. If the meal is much smaller than a main meal, it asks if it's a snack.
3. It calls the lookup tools for every number — three at a time, in parallel: USDA FoodData Central for generic foods, Open Food Facts for packaged and branded foods (plus barcode). The app turns the lookup results into numbers itself — the model never outputs a calorie from memory.
4. If a dish has no match (bendakaya fry, say), the app works out what the dish is, shows its closest find, breaks it into ingredients and looks each one up.
5. I get a **Check before it logs** screen: each food with its numbers and its source (USDA / Open Food Facts / Label / You stated). It shows what's being logged — yes, no, or send a note with what to change. If I don't reply in 60 seconds, it logs as shown. If a food has no match, the app says "I don't know this one" and leaves the numbers blank for me to fill in from the label. It never invents a value.
6. If I stated a number myself, mine wins over the database.
7. "The usual" works because the app remembers my past meals from my own log and the LLM can match against them.
8. If a lookup times out, it tries twice, then says so plainly and queues the item to retry later. If the retry fails twice too, it says it has failed and asks me for the exact numbers — or I can use **Fast log**: give the macros and the calories are worked out from them, or give the calories and the macros are estimated from my target split (stated plainly).

The concepts behind all of this — tool use, RAG, workflows, grounding, failure handling — are written up in [CONCEPTS.md](CONCEPTS.md).

## Where the numbers come from

- **Open Food Facts** — free, no key needed. Packaged and branded foods, plus barcode lookup.
- **USDA FoodData Central** — generic foods (rice, dals, vegetables, chicken). Needs a free API key; see below.
- **You stated** — if I say a number myself, mine wins over the database.
- **Label** — numbers typed in from a packet label.

The day view shows my running totals against my targets: 1,850–1,900 kcal, 213 g carbs / 105 g protein / 60 g fat, fiber 25 g+. Entries are grouped by meal and can be edited or deleted.

## Audit and suggestions

- **Audit today** — the day against my goals, plainly: what was met and what was missed, with the numbers. It also shows on its own after 8 pm.
- **How am I doing?** — mid-day check. If protein or fiber is tracking short, or calories are drifting over, it suggests concrete foods from my staples (nonfat Greek yogurt, dals, moong, chickpeas, tofu, nuts, fruit) to close the gap — with numbers looked up from the databases, never invented. All vegetarian, no eggs, no avocado, no soy chunks, nonfat yogurt only, no cheese.

## Keys

No keys are ever in the code — whoever downloads the app provides their own keys in Settings, stored only in their own browser. If you ever see a key in a file here, that's a bug.

1. **Anthropic key** — needed for the LLM to interpret what I ate. Get one at console.anthropic.com, paste it in Settings. Without it, the app falls back to simple pattern matching and says so plainly. The key is sent only to Anthropic, straight from the browser — no middleman.
2. **USDA key** — free at api.nal.usda.gov. Without it, lookups use Open Food Facts only.

## Notes

- Entries live in the browser's localStorage on my device — there is no server and nothing is uploaded.
- Voice input uses the browser's speech recognition; where it isn't supported, typing works the same way.
- No sync to any other app in this version — the diary lives here.
- v1 used plain pattern matching and no LLM. v2 added the LLM interpretation layer — the LLM interprets, the lookup grounds. v3 added time-aware meal inference, three parallel lookups, the dish-to-ingredients fallback, the review screen that logs on silence, retry-twice with a retry queue, fast log, and snack alerts. See [CONCEPTS.md](CONCEPTS.md) for the thinking behind each piece.
