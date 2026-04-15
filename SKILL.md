---
name: food-tracker
description: Log food, track cal+protein vs goals (2100cal/165g protein). Trigger on "log:", "macros", "I ate/had/just ate/just had".
---

# Food Tracker
DB: `~/.claude/food-tracker/food.db` | Goals: 2100cal / 165g protein

## Triggers
- `log:` prefix → log food
- `macros` / `totals` → show today
- "I had/ate/just had/just ate" → log food
- "undo" / "delete last" → rm last entry

## Log food
1. Date = today YYYY-MM-DD (or user-specified)
2. Estimate cal+protein from description
3. Insert:
```bash
sqlite3 ~/.claude/food-tracker/food.db "INSERT INTO food_log (date,food_description,calories,protein_g) VALUES ('DATE','DESC',CAL,PRO);"
```
4. Query totals:
```bash
sqlite3 ~/.claude/food-tracker/food.db "SELECT COALESCE(SUM(calories),0),COALESCE(SUM(protein_g),0) FROM food_log WHERE date='DATE';"
```
5. Reply:
```
Logged: DESC (~Xcal, Xg protein)
Today: Xcal/2100 · Xg/165g protein
```

## Show totals
Run step 4, same reply format (no "Logged:" line).

## Undo/delete last
```bash
sqlite3 ~/.claude/food-tracker/food.db "DELETE FROM food_log WHERE id=(SELECT id FROM food_log WHERE date='DATE' ORDER BY logged_at DESC LIMIT 1);"
```
Reply: `Removed. Today: Xcal/2100 · Xg/165g protein`

## Delete by name
Query: `SELECT id,food_description FROM food_log WHERE date='DATE' AND food_description LIKE '%KEY%';`
Confirm match → DELETE. Re-query totals.

## Rules
- Multi-food = 1 INSERT, combined estimate
- No confirm before insert
- Replies short (iMessage)
