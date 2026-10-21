# EldenRim_MyPatch — English Guide (build 2.3.12-EN)

## 1. What this is

A patch for **Elden Rim** (Base 3.7.5 + Weapon Art 3.2.7) that adds extra
Variants to the Ashes of War.

- The 5 original ash types — Strength, Dexterity, Arcane, Mystery, Fortune —
  each get 6 variants (variant 1 is the original ash itself, variants 2–6
  are new). Total: **30 ashes**.
- Every variant can hold **its own Weapon Art**, assigned independently
  through the Elden Rim scroll menu.
- The original Elden Rim plugins, scripts and OAR data are **not modified**.
  Everything new lives in this patch; the few overridden original records
  only add the variant logic (see §6).
- Status: tested in game, works. Final builds are silent (no log spam,
  no popups).

Two builds exist, with identical logic:

| Build | For | File |
|---|---|---|
| RU 2.3.12 | Russian game + Russian-translated RimSkills | `EldenRim_MyPatch-OneAsh-OAR-2.3.12.zip` |
| **EN 2.3.12-EN (this one)** | Original English RimSkills | `EldenRim_MyPatch-OneAsh-OAR-2.3.12-EN.zip` |

Never enable both at once — they use the same plugin name.

## 2. Requirements

- Skyrim Special Edition.
- `EldenSkyrim.esp` (Elden Rim Base 3.7.5).
- `EldenSkyrim_RimSkills.esp` (Elden Rim Weapon Art 3.2.7), **original English**.
- `TrueDirectionalMovement.esp` (master of the patch).
- PapyrusUtil (the patch stores variant data in StorageUtil).
- Open Animation Replacer — required by Elden Rim itself.
- Optional: Spell Hotbar NG — convenient for binding variants to hotkeys.

Load order: this patch **below** the Elden Rim mods.

## 3. Installation (Mod Organizer 2)

1. In MO2: *Install Mod from Archive* → pick this ZIP.
2. Place the mod below the Elden Rim mods in the left pane, enable it.
3. Disable any older version of this patch first (only one version active).

No INI changes, no Nemesis/Pandora output changes, no new game needed.

## 4. How to use

1. Use the Elden Rim scroll.
2. Choose the ash type (Strength / Dexterity / Arcane / Mystery / Fortune).
3. Choose the variant, e.g. **Strength 2**.
4. Choose the hand, then the Weapon Art. The art is saved for that variant.
5. Craft the variant armor (recipes are included) and books that teach the
   variant spells.
6. Equip the variant (or put it on the favorites / Spell Hotbar NG panel)
   and press its key: the stored art of **that variant** plays.
7. Pressing the original ash plays the original ash's own art, as before.

## 5. Package contents

```
EldenRim_MyPatch.esp          the patch (plugin name must stay as is, see §6)
meta.ini                      MO2 metadata
Documentation/
  EldenRim_MyPatch_Report.md  full development diary (Russian, history)
  README-EN.md                this guide
Scripts/*.pex                 compiled patch scripts (4)
Scripts/Source/*.psc          their sources (4)
meshes/.../190-ERMy-Sila2/    inert OAR leftover, harmless (see §6)
```

### 5.1 Plugin records (EN build: 298 records)

| Records | Count | Role |
|---|---|---|
| ARMO (own) | 25 | variant armors, types 1–5 × variants 2–6 |
| ARMO (overrides) | 5 | original armors + variant-aware equip script |
| ENCH (own) | 25 | variant enchantments (FC4 marker + auto-unequip + backup writer) |
| ENCH (override) | 1 | original Strength enchant + backup writer effect |
| SPEL (own) | 25 | variant spells (`… [3 - Spell]`) |
| SPEL (overrides) | 5 | original spells, medial effect writes the slot before casting |
| BOOK | 25 | `Book of War - <Type> <N> [Book]`, teach the variant spells |
| MGEF | 61 | 30 cast effects, 30 equip effects, 1 unused marker |
| GLOB | 60 | variant slot globals (60 = 5 types × 6 variants × 2 hands) |
| FLST (own) | 2 | variant menu list + variant slot list |
| FLST (overrides) | 8 | original lists extended with the new forms |
| MESG | 5 | variant submenus (`Choose variant: <Type>`, 7 buttons each) |
| COBJ | 50 | crafting recipes for the variant armors and books |
| ALCH | 0 | none — the potion path was removed (see §8) |

The EN build additionally drops 4 enchantment overrides that carried only
translated names and are byte-identical to the original once in English.
No gameplay change.

### 5.2 Scripts

| Script | Role |
|---|---|
| `EldenWarAshSet` | Replaces the original file. Shows the variant submenu, writes the chosen art ID into the variant list. Double-menu guard (5 s). |
| `EldenRimVariantOnEquip` | On every patch armor: writes the variant (or original) values into the active slot globals FIRST, then casts the original combat spell `0600B001`. One call stack — no race. |
| `EldenRimVariantCast` | Spell path: write slot → cast `0600B001` → re-sync slots. |
| `EldenRimVariantEquip` | Backup slot writer on the enchantment effect. |

Papyrus sources (`.psc`) are included for all four.

## 6. How it works (technical notes)

**Storage.** Each variant's art ID is stored twice: in a StorageUtil list
(`rim_SlotlistVariant<N>_<profile>`, profile from `Rim_CRList` with a `2hw`
fallback) and in the patch's own globals (list `ERMy_VariantSlotList`).
The submenu data lives in `ERMy_VariantMessageList` (5 `MESG`).

**Activation (armor path — the supported one).** Pressing a variant on the
panel equips its armor. The old chain had a race: the armor script
(`CostBBSpell.OnEquipped`) cast the combat spell `0600B001` immediately,
while the slot value was written later from the enchantment effect — the
attack read a stale global and played the wrong art. The fix: every patch
armor (all 25 variants + overrides of all 5 originals) runs
`EldenRimVariantOnEquip`, which writes the slot values first and only then
casts the same `0600B001` in the same call stack. Pressing the original ash
restores its own art the same way (variant 0).

**Overrides.** 19 functional overrides of original records: 5 armors (equip
script), 1 enchantment (backup writer), 5 spells (slot write before cast),
8 FormLists (original entries kept, patch forms appended — early versions
accidentally replaced these lists, which froze the original ashes; fixed
in 2.0.7).

**Fixed plugin name.** The file MUST stay `EldenRim_MyPatch.esp`: the
compiled `EldenWarAshSet` resolves the menu lists via
`Game.GetFormFromFile(id, "EldenRim_MyPatch.esp")`. Renaming the ESP breaks
the submenu. (The ZIP / MO2 folder names can be anything.)

**Inert OAR folder.** `meshes/.../190-ERMy-Sila2/` is a leftover of an early
experiment (a custom OAR marker later abandoned — all 150+ original arts
work with the stock marker). It requires a magic effect nothing applies,
so it never triggers. Harmless; kept only to avoid re-shuffling the build.

**Silent final.** Diagnostic traces were removed in 2.3.9 and notifications
in 2.3.10. For diagnostics, the 2.3.8 script set (traces + art/profile
notifications, `ERMy` filter in `Papyrus.0.log`) can be temporarily
dropped onto this build — the plug-in side is compatible.

## 7. Version history (condensed)

| Version | What changed |
|---|---|
| 1.0.0–2.0.6 | Patch built up: variant forms, submenu, scripts, FormID/VMAD/StorageUtil fixes |
| 2.0.7 | Fixed overwritten FormLists — original-ash freezing gone, confirmed in game |
| 2.0.8 | StorageUtil profile fallback (`2hw`) |
| 2.1.x | Minimal one-ash branch — dead end, withdrawn |
| 2.2.0–2.2.1 | OAR-marker experiment; fixed ESP-name bug (submenu did not open) |
| 2.3.0 | Removed 57 stale `PlayerKnow` overrides; clean script recompiles |
| 2.3.1 | Withdrawn, superseded by 2.3.2 |
| 2.3.2–2.3.3 | Stock OAR marker restored; slot sync without reset |
| 2.3.4 | Write-before-cast (`EldenRimVariantOnEquip`) — variants play their own arts ✔ |
| 2.3.5 | Rolled out to all 30 armors; fixed message encoding (UTF-8 BOM sources) |
| 2.3.6 | Reset lists completed — double-press sticking gone ✔ |
| 2.3.7 | Double-menu guard in the assign script |
| 2.3.8 | "Leak onto ash 1" disproven by logs; profile shown in notifications |
| 2.3.9 | Silent build: traces removed, notifications kept |
| 2.3.10 | Notifications removed |
| 2.3.11 | Potion variants + their recipes removed (50 records) |
| 2.3.12 | Original-potion overrides removed (5 records) — patch touches no potions |
| **2.3.12-EN** | **English build: 278 strings in English, descriptions restored from the original mod, 4 string-only overrides dropped** |

The full day-by-day diary (Russian) is `Documentation/EldenRim_MyPatch_Report.md`.

## 8. Known limitations

- Only the **armor path** is fully supported and tested. The spell path is
  present; the potion path was removed as out of scope, and the patch no
  longer touches original potions at all.
- Overrides of the 5 original spells (`WarAshSpell01–05`) use the same
  write-before-cast medial effect; the armor path is unaffected by them.
- `190-ERMy-Sila2` OAR data is an inert leftover (see §6).

## 9. Credits

- Idea, tasking, testing, in-game verification: **Vladimir**.
- Original mod: Elden Rim team. This patch is shared with the author's
  kind permission.
