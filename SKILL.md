---
name: food-tracker
description: Log food eaten and track daily calories and protein against goals (2100 cal, 165g protein). Triggered by keyword "log:" or natural food descriptions.
---

# Food Tracker Skill

**DB path:** `~/.claude/food-tracker/food.db`
**Goals:** 2100 calories / day, 165g protein / day

## Trigger keywords
Fire this skill when the message contains any of:
- `log:` prefix (e.g. "log: 2 eggs and bacon") — explicit trigger
- `macros` or `totals` — user wants to see current day summary
- Natural food language: "I had", "just ate", "I ate", "just had", "for breakfast/lunch/dinner I had"

## Steps for logging food

1. **Determine the date** — use today's date (YYYY-MM-DD) unless the user specifies otherwise.

2. **Estimate macros** — use your nutritional knowledge to estimate calories and protein for each item described. Be reasonable; don't over-explain the estimates unless asked. When uncertain, use common serving sizes.

3. **Insert the record:**
```bash
sqlite3 ~/.claude/food-tracker/food.db "
INSERT INTO food_log (date, food_description, calories, protein_g)
VALUES ('YYYY-MM-DD', 'description', CALORIES, PROTEIN);
"
```

4. **Query daily totals:**
```bash
sqlite3 ~/.claude/food-tracker/food.db "
SELECT COALESCE(SUM(calories),0), COALESCE(SUM(protein_g),0)
FROM food_log WHERE date = 'YYYY-MM-DD';
"
```

5. **Reply format** (concise):
```
Logged: [food summary] (~Xcal, Xg protein)

Today so far:
  Calories: X / 2100
  Protein:  Xg / 165g
```

## Steps for viewing totals (no new food)

Just run the query in step 4 and report with the same reply format.

## Steps for viewing a past day

Query with the specified date. Show the log entries and totals.
```bash
sqlite3 ~/.claude/food-tracker/food.db "
SELECT food_description, calories, protein_g, logged_at
FROM food_log WHERE date = 'YYYY-MM-DD' ORDER BY logged_at;
"
```

## Steps for deleting an entry

**"undo" or "delete last"** — remove the most recently logged entry for today:
```bash
sqlite3 ~/.claude/food-tracker/food.db "
DELETE FROM food_log WHERE id = (
  SELECT id FROM food_log WHERE date = 'YYYY-MM-DD' ORDER BY logged_at DESC LIMIT 1
);
"
```
Then re-query totals and reply: `Removed last entry. [updated totals]`

**"delete [food]"** — find closest match by description and confirm before deleting:
```bash
sqlite3 ~/.claude/food-tracker/food.db "
SELECT id, food_description, calories, protein_g FROM food_log
WHERE date = 'YYYY-MM-DD' AND food_description LIKE '%KEYWORD%';
"
```
Show the match and ask "delete this?" before running DELETE.

## Notes
- Multiple foods in one message = one INSERT with combined estimates (keep description brief)
- Don't ask for confirmation before inserting — just log it and report
- Keep responses short; the user reads this in iMessage
