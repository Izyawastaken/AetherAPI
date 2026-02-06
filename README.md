# Aethermancer Database

This folder is a per‑monster database that bundles:
- structured monster data
- animation clip frame data
- the sprite sheets needed to render those frames
- cropped icon PNGs (portrait, trigger icon, battle/overworld/proj replacements, etc.)

Use this when you want to render monsters and their animations on a website without parsing Unity files at runtime.

---

## Folder Layout

```
Database/
  index.json
  <MonsterName>/
    monster.json
    sheets/
    icons/
```

### `index.json`
Quick entrypoint for listing monsters.

Example:
```json
{
  "monsters": [
    { "name": "Cherufe", "prefab": "MonsterCherufe", "folder": "Cherufe" },
    { "name": "Dark Elder", "prefab": "MonsterDarkElder", "folder": "Dark_Elder" }
  ]
}
```

### `<MonsterName>/monster.json`
Per‑monster payload with:
- `data`: the readable monster data (stats, perks, traits, actions, poise…)
- `animations`: animation clips with ordered frames
- `icons`: icon PNGs + sprite metadata

---

## monster.json Shape

```json
{
  "name": "Cherufe",
  "prefab": "MonsterCherufe",
  "prefab_path": "Assets/GameObject/MonsterCherufe.prefab",
  "data": { ... },
  "icons": { ... },
  "animations": { ... }
}
```

### `data`
This is copied from `Aethermancer Data/monsters.json`, so it stays human‑readable:

Typical keys:
- `stats`: base health, start perk count
- `types`: type names
- `traits`: signature trait and additional traits
- `actions`: start actions, scripted actions, reset actions
- `perks`: base + void/champion perks
- `poise`: poise rules and stagger definitions

### `icons`
Each entry has a cropped PNG + the underlying sprite metadata:

```json
"icons": {
  "PortraitSpriteSmall": {
    "sprite_guid": "...",
    "file": "Database/Cherufe/icons/PortraitSpriteSmall.png",
    "sprite": {
      "name": "Portrait_Cherufe",
      "rect": { "x": 0, "y": 0, "width": 128, "height": 128 },
      "pivot": { "x": 0.5, "y": 0.5 },
      "sheet": "Database/Cherufe/sheets/SomeSheet.png",
      "sprite_asset": "Assets/Sprite/Portrait_Cherufe.asset"
    }
  }
}
```

Use `file` if you just need the cropped icon.  
Use `sprite.sheet` + `sprite.rect` if you want to render from the sheet.

### `animations`
Animation clips are keyed by clip name. Each clip contains ordered frames.

```json
"animations": {
  "CherufeWalkUp": {
    "clip": "Assets/AnimationClip/CherufeWalkUp.anim",
    "frames": [
      {
        "time": 0,
        "sprite_guid": "...",
        "sprite": {
          "name": "Cherufe_Overworld_SpriteSheet_46",
          "rect": { "x": 0, "y": 0, "width": 90, "height": 90 },
          "pivot": { "x": 0.5, "y": 0.35 },
          "sheet": "Database/Cherufe/sheets/Cherufe_Overworld_SpriteSheet.png",
          "sprite_asset": "Assets/Sprite/Cherufe_Overworld_SpriteSheet_46.asset"
        }
      }
    ]
  }
}
```

To play the animation on web:
1. Load the sheet image (`sprite.sheet`).
2. Crop the rectangle (`sprite.rect`) per frame.
3. Advance frames by `time` values (Unity times are in seconds).

---

## Quick Start (JavaScript)

```js
const index = await fetch("Database/index.json").then(r => r.json());
const first = index.monsters[0];
const monster = await fetch(`Database/${first.folder}/monster.json`).then(r => r.json());

// Example: get walk animation frames
const walk = monster.animations["CherufeWalkUp"];
const frames = walk.frames;

// Example: icon URL
const portrait = monster.icons.PortraitSpriteSmall?.file;
```

---

## Quick Start (Python)

```python
import json

index = json.load(open("Database/index.json", "r", encoding="utf-8"))
first = index["monsters"][0]

monster = json.load(open(f"Database/{first['folder']}/monster.json", "r", encoding="utf-8"))
walk = monster["animations"].get("CherufeWalkUp")
portrait = monster["icons"].get("PortraitSpriteSmall", {}).get("file")
```

---

## Notes / Caveats

- Folder names are sanitized (spaces → `_`).
- Some monsters might not have certain icons or animation clips.
- Animation frame order is time‑sorted. Use time deltas for playback.
- If you re‑run the generator, the `Database/` folder is rebuilt.

---

## Regenerate

```powershell
python build_monster_database.py
```
