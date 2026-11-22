# JustTrackIt

I built this to log my food by voice or text without any number being guessed. An LLM understands what I ate — even messy voice notes like "half a multigrain bagel" or "the usual lunch" — and the database lookup is what grounds it. Every calorie and macro comes from a food database, from a label, or from me — never from the model making it up.

## How it works

1. Tap the mic and say what you ate, or type it. Messy is fine: "1 cup cooked red rice and a protein bar, 230 calories, 18g protein".
2. The LLM interprets it: it pulls out each food, the amount, and any numbers I stated — and decides what to look up for each one.
3. It calls the lookup tools for every number: USDA FoodData Central for generic foods, Open Food Facts for packaged and branded foods (plus barcode). The app turns the lookup results into numbers itself — the model never outputs a calorie from memory.
4. I get a **Check before it logs** screen: each food with its numbers and its source (USDA / Open Food Facts / Label / You stated). If a food has no match, the app says "I don't know this one" and leaves the numbers blank for me to fill in from the label. It never invents a value.
5. If I stated a number myself, mine wins over the database.
6. If I didn't name a meal, the app asks which meal it was instead of silently filing it under "extra".
7. "The usual" works because the app remembers my past meals from my own log and the LLM can match against them.

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
- v1 used plain pattern matching and no LLM. v2 added the LLM interpretation layer — the LLM interprets, the lookup grounds.
