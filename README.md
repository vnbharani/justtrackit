# JustTrackIt

I built this to log my food by voice or text without any number being guessed. Every calorie and macro comes from a food database, from a label, or from me — never from the app making it up.

## How to use it

Open `index.html` in a browser — no install, no build step, no account.

1. Tap the mic and say what you ate, or type it. Include the amount: "1 cup cooked red rice", "1 protein bar, 230 calories, 18g protein".
2. Pick the meal: breakfast, 11am, lunch, 3pm snack, dinner, or extra.
3. Tap "Look it up". The app searches the databases and shows a **Check before it logs** screen: each number with its source, and a list of matches to pick from if there are several.
4. Fix anything that looks wrong — the numbers are editable — then tap "Log it".

If a food has no match, the app says "I don't know this one" and leaves the numbers blank for me to fill in from the label. It never invents a value.

For packaged food you can also type the barcode number instead of the name.

## Where the numbers come from

- **Open Food Facts** — free, no key needed. Packaged and branded foods, plus barcode lookup.
- **USDA FoodData Central** — generic foods (rice, dals, vegetables, chicken). Needs a free API key; see below.
- **You stated** — if I say a number myself, mine wins over the database.
- **Label** — numbers typed in from a packet label.

The day view shows my running totals against my targets: 1,850–1,900 kcal, 213 g carbs / 105 g protein / 60 g fat, fiber 25 g+. Entries are grouped by meal and can be edited or deleted.

## Adding a USDA key

1. Get a free key at api.nal.usda.gov.
2. Open Settings in the app, paste the key, save. It stays in my browser only.
3. Without a key, lookups use Open Food Facts only.

## Notes

- Entries live in the browser's localStorage on my device — there is no server and nothing is uploaded.
- Voice input uses the browser's speech recognition; where it isn't supported, typing works the same way.
- No sync to any other app in this version — the diary lives here.
