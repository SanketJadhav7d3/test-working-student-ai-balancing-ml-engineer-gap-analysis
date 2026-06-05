# Decisions Log

Append-only history of balancing changes made through this workbench.

Append entries via `audit-change-lite` after every successful commit. Newest entries at the bottom.

---

## 2026-06-05 — Mira (hero_id=1) attack bumped at L20: 608 → 700

**Commit:** `f25fa5a`

**Change:**
- `Src_Hero_Data.txt` · `hero_id=1, level=20` · `attack`: 608 → 700

**Reason:**
Producer flagged Mira as feeling weak in the mid-game. A single-cell bump to her L20 attack value (608 → 700) was requested to bring her damage output more in line with the expected power curve at that tier.

**Validation:**
- `qa-check-lite`: PASS

**Caveats / follow-ups:**
- None

---

## 2026-06-05 — Tessa (hero_id=11) attack +10% at L10–L15 (curve adjustment)

**Commit:** `fd81324`

**Change:**
- `Src_Hero_Data.txt` · `hero_id=11, level=10–15` · `attack` +10% per level (6 rows); `power` recomputed for all 6 rows
  - L10: attack 375 → 413, power 1062 → 1138
  - L11: attack 398 → 438, power 1127 → 1207
  - L12: attack 422 → 464, power 1196 → 1280
  - L13: attack 444 → 488, power 1260 → 1348
  - L14: attack 468 → 515, power 1328 → 1422
  - L15: attack 490 → 539, power 1391 → 1489

**Reason:**
Producer requested a curve adjustment for Tessa (Hero #11) to boost her attack at levels 10–15 by 10% at each level. Six rows in total. The 10% multiplier was applied to each level's existing attack value and rounded to the nearest integer; the `power` column was recomputed accordingly.

**Validation:**
- `qa-check-lite`: PASS (format, range, FK, override-aware, power consistency, regression — all clean)

**Caveats / follow-ups:**
- None

---

## 2026-06-05 — Thorne (hero_id=4) rebucketed to lower-tier unlock chain (unlock_require_id: 5010 → 5005)

**Commit:** `1639e3a`

**Change:**
- `Src_Hero_Data.txt` · `hero_id=4, level=1–30` · `unlock_require_id`: 5010 → 5005 (30 rows; same value across all levels, changed uniformly)

**Reason:**
Producer requested that Thorne (Hero #4) be moved into the lower-tier unlock chain by changing his `unlock_require_id` from 5010 to 5005. No stat columns (hp, attack, defense) were touched, so `power` was not recomputed.

**Validation:**
- `qa-check-lite`: PASS (format, range, FK, override-aware, power consistency, regression — all clean)

**Caveats / follow-ups:**
- None

---

## 2026-06-05 — [WARNING] Brogan (hero_id=7) HP nerf at L5 — forced apply below runtime floor

**Commit:** `7326985`

**Change:**
- `Src_Hero_Data.txt` · `hero_id=7, level=5` · `hp`: 1794 → 90; `power`: 906 → 736

**Reason:**
Producer requested a hard nerf to Brogan at L5. Change was initially blocked due to a known runtime HP floor of **100** (`HeroCombat.java:142`). Producer confirmed to apply forcefully despite the override.

**Override warning:**
`context/known-overrides.md` documents that `hero_id=7` HP values below 100 are silently clamped to 100 at runtime. The TXT now reads 90, but the effective in-game value will be **100** — the nerf will not fully take effect as written. Values below 50 would crash the round (INC-2042); 90 is safe in that regard.

**Validation:**
- `qa-check-lite`: SKIP — producer override; value violates floor rule intentionally.

**Caveats / follow-ups:**
- Runtime will clamp hp to 100; the TXT value of 90 has no additional effect beyond 100.
- Consider fixing the floor in `HeroCombat.java` if a sub-100 nerf is the actual goal.
