# food-tracker

A Claude Code skill for tracking daily food intake via iMessage (or any channel).

**Goals:** 2100 cal / day · 165g protein / day

## What it does
- Log food by describing what you ate — Claude estimates calories + protein
- Tracks by date in a local SQLite DB
- Shows running daily totals vs goals
- Supports undo / delete of entries

## Install

```bash
# 1. Copy skill to your skills directory
cp -r food-tracker ~/.claude/skills/food-tracker

# 2. Init the database
mkdir -p ~/.claude/food-tracker
sqlite3 ~/.claude/food-tracker/food.db "
CREATE TABLE IF NOT EXISTS food_log (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  date TEXT NOT NULL,
  food_description TEXT NOT NULL,
  calories INTEGER NOT NULL,
  protein_g REAL NOT NULL,
  logged_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX IF NOT EXISTS idx_date ON food_log(date);
"
```

## Usage

```
log: 2 eggs and toast
→ Logged: 2 eggs and toast (~310 cal, 18g protein)
   Today so far:
     Calories: 310 / 2100
     Protein:  18g / 165g

macros
→ Today so far: ...

undo
→ Removed last entry. ...
```

## Customize goals
Edit the `**Goals:**` line in `SKILL.md` to change your targets.
