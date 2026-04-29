# Custom Focus Tree Research — Hearts of Iron IV

Research notes for building a custom national focus tree for HOI4. Aimed at someone making their first focus tree mod.

## 1. Where files go

A HOI4 mod lives in:

```
Documents/Paradox Interactive/Hearts of Iron IV/mod/<your_mod_name>/
```

Inside the mod, the relevant folders for a focus tree are:

```
common/national_focus/<your_tree>.txt        # the focus tree itself
common/ideas/<your_ideas>.txt                # any national spirits granted by focuses
common/continuous_focus/<your_tree>.txt      # optional continuous focus tree
localisation/english/<your_tree>_l_english.yml   # focus names + descriptions
gfx/interface/goals/GFX_<your_icon>.dds      # custom icons (optional)
interface/<your_tree>.gfx                    # sprite type registration for custom icons
```

Each `.txt` file in `common/national_focus/` represents one focus tree. Use the same folder structure inside your mod to override or add files; the launcher merges them with the base game.

You also need a `descriptor.mod` at the mod root and a matching `.mod` file beside the mod folder so the launcher sees it.

## 2. The `focus_tree` block

Top-level wrapper. One per file.

```pdx
focus_tree = {
    id = my_custom_tree
    country = {
        factor = 0
        modifier = {
            add = 10
            tag = GER
        }
    }
    default = no
    continuous_focus_position = { x = 50 y = 1100 }
    initial_show_position = { focus = my_first_focus }
    shared_focus = some_shared_focus_id

    focus = { ... }
    focus = { ... }
}
```

Fields:

- `id` — unique identifier for the tree.
- `country` — weighted picker. The tree with the highest score is shown for that country. Use `factor = 0` + a `modifier` that boosts it for your tag so it only applies to the country you want.
- `default = yes/no` — if `yes`, this tree is used by every country that doesn't get a more specific one (the generic tree uses this).
- `continuous_focus_position` — pixel coords of the continuous-focus side panel. Place it somewhere that doesn't collide with your branches (e.g. far right or below the last row).
- `initial_show_position` — which focus the camera centers on when the tree first opens. Can be `focus = <id>` or `x = N y = N`.
- `shared_focus` — pulls in a shared focus block by id (see §5).

## 3. The `focus = {}` block

Every individual focus. The fields you'll use most:

```pdx
focus = {
    id = MYTAG_industrial_effort
    icon = GFX_goal_generic_construct_civ_factory
    x = 3
    y = 0
    relative_position_id = MYTAG_political_effort   # optional; x/y become relative to this focus
    cost = 10                                       # in 7-day "focus weeks"; 10 = 70 days

    prerequisite = { focus = MYTAG_political_effort }    # AND between blocks, OR within a block
    prerequisite = { focus = MYTAG_other_a focus = MYTAG_other_b }
    mutually_exclusive = { focus = MYTAG_some_other_focus }

    available = {
        has_war = no
        has_political_power > 50
    }
    bypass = {
        has_idea = MYTAG_already_done
    }
    cancel = { is_subject = yes }
    cancel_if_invalid = yes
    continue_if_invalid = no
    available_if_capitulated = no
    allow_branch = { has_dlc = "Waking the Tiger" }   # whole branch hides if false

    will_lead_to_war_with = SOV
    search_filters = { FOCUS_FILTER_INDUSTRY FOCUS_FILTER_POLITICAL }

    ai_will_do = {
        factor = 5
        modifier = { factor = 0 has_war = yes }
    }

    select_effect = {
        # runs the moment the player clicks the focus
    }

    complete_tooltip = {
        add_ideas = MYTAG_industrial_focus
    }

    completion_reward = {
        add_political_power = 50
        add_ideas = MYTAG_industrial_focus
        add_tech_bonus = {
            name = industry_bonus
            bonus = 0.5
            uses = 1
            category = industry
        }
    }
}
```

Field reference:

| Field | Purpose |
| --- | --- |
| `id` | Unique within the file; convention is `TAG_short_name`. |
| `icon` | Sprite name from `interface/*.gfx`. Vanilla ones live as `GFX_goal_*` / `GFX_focus_*`. |
| `x`, `y` | Grid position. `x` is column, `y` is row, top-left origin. |
| `relative_position_id` | If set, `x`/`y` are offsets from that focus instead of absolute. Use this for branches so moving a parent moves all children. |
| `cost` | Time in 7-day chunks. `cost = 10` → 70 days. |
| `prerequisite` | Required parents. Multiple blocks AND together; multiple `focus = X` inside one block OR together. |
| `mutually_exclusive` | Locks out the listed focus(es) once this one is taken. |
| `available` | Trigger block evaluated each tick; focus is greyed out until true. |
| `bypass` | Auto-completes (no reward) when the trigger is true — use for "obsolete now" cases. |
| `cancel` | Aborts an in-progress focus if true. |
| `cancel_if_invalid` | If `available` becomes false mid-research, cancel it. |
| `continue_if_invalid` | If true, keep going even when `available` is false. |
| `available_if_capitulated` | Defaults to `no`; allow research while capitulated. |
| `allow_branch` | Trigger; if false, the focus and its descendants are hidden entirely. Good for DLC-gating. |
| `will_lead_to_war_with` | UI-only "this will cause war" warning, used by the AI too. |
| `search_filters` | Categories for the in-game focus search (e.g. `FOCUS_FILTER_POLITICAL`). |
| `ai_will_do` | Weight the AI uses to pick this focus. Higher = more likely. |
| `select_effect` | Effect block that fires immediately on click. |
| `complete_tooltip` | Shown on hover; usually mirrors what `completion_reward` actually does, written for tooltip clarity. |
| `completion_reward` | Effect block that fires when the focus finishes. The meat of the design. |

### `available` / triggers — common ones

```pdx
has_war = no
has_political_power > 75
has_government = fascism
has_country_flag = my_flag
date > 1937.1.1
NOT = { has_war_with = ENG }
controls_state = 64
OR = { has_idea = X has_idea = Y }
```

### `completion_reward` / effects — common ones

```pdx
add_political_power = 100
add_stability = 0.05
add_war_support = 0.10
add_manpower = 50000
add_ideas = MYTAG_some_spirit
remove_ideas = MYTAG_old_spirit
swap_ideas = { remove_idea = old add_idea = new }
set_country_flag = my_flag
country_event = mytag.1
news_event = news.42
add_tech_bonus = { name = doctrine bonus = 0.5 uses = 2 category = land_doctrine }
add_equipment_to_stockpile = { type = infantry_equipment_1 amount = 1000 producer = MYTAG }
add_resource = { type = oil amount = 10 state = 64 }
set_research_slots = 4
declare_war_on = { target = POL type = annex_everything }
annex_country = { target = LUX }
puppet = LUX
give_guarantee = FRA
transfer_state = 64
hidden_effect = { ... }    # runs but doesn't show in tooltip
```

Conditional logic inside effects:

```pdx
completion_reward = {
    if = {
        limit = { has_war_support < 0.5 }
        add_war_support = 0.1
    }
    else_if = {
        limit = { has_idea = war_economy }
        add_political_power = 150
    }
    else = {
        add_ideas = war_economy
    }
}
```

## 4. Layout tips

- Use `relative_position_id` aggressively. If your tree has a "Political" branch under a root focus, set the root's `x/y` absolutely and make every child relative to its parent. Reflowing the tree later becomes a one-line change.
- The grid is wide. Most vanilla trees span 0–25 in `x`. Leave 2 columns of breathing room between branches.
- `y` increments visually downward. Children typically sit at `y = parent_y + 1` (relative `y = 1`).
- Diagonal `prerequisite` lines are drawn automatically — you don't position them.

## 5. `shared_focus` — reusable focuses

Defined at the top level of any `common/national_focus/*.txt` file (outside any `focus_tree`):

```pdx
shared_focus = {
    id = generic_industry_effort
    icon = GFX_goal_generic_production
    cost = 10
    x = 0
    y = 0
    available = { ... }
    completion_reward = { add_political_power = 120 }
}
```

A tree imports it with `shared_focus = generic_industry_effort` inside the `focus_tree`. Useful for branches reused across many countries (the generic tree uses this heavily).

## 6. Continuous focuses

Continuous focuses are the always-running side-panel ones (Free Trade, Construction, Silent Workhorse, etc.). They live in `common/continuous_focus/`:

```pdx
continuous_focus_palette = {
    id = my_palette
    position = { x = 50 y = 1100 }   # overridden by per-tree continuous_focus_position
    continuous_focus = {
        id = MYTAG_construction_drive
        icon = GFX_goal_generic_construct_civ_factory
        available = { has_war = no }
        cost = 0.7
        modifier = {
            production_speed_buildings_factor = 0.10
        }
    }
}
```

If your `focus_tree` has `continuous_focus_position = { x = N y = N }`, that overrides the palette's default position for that country. Make sure the chosen coords don't sit on top of your main branches — that's the bug Kaiserreich repeatedly files (continuous panel covering a focus).

## 7. Localisation

File: `localisation/english/<anything>_l_english.yml`. Must start with `l_english:` and the file itself must be UTF-8 **with BOM** or HOI4 silently ignores it.

```yml
l_english:
 MYTAG_industrial_effort:0 "Industrial Effort"
 MYTAG_industrial_effort_desc:0 "Push the smokestacks of [Root.GetName] to new heights."
```

Convention: the localisation key is the focus `id`. The `_desc` suffix is the long description shown on hover. The `:0` is the version number; bump it if you want the launcher to consider it changed.

Variable replacements like `[Root.GetName]`, `[From.GetLeader]`, `[?var|2]` work the same here as anywhere else in HOI4 scripting.

## 8. Custom icons

1. Make a 95×95 `.dds` (BC3/DXT5) and drop it in `gfx/interface/goals/GFX_my_icon.dds`.
2. Register it in an `interface/*.gfx` file:

```pdx
spriteTypes = {
    spriteType = {
        name = "GFX_my_icon"
        texturefile = "gfx/interface/goals/GFX_my_icon.dds"
    }
}
```

3. Reference it in the focus: `icon = GFX_my_icon`.

If you don't want custom art, browse `Hearts of Iron IV/gfx/interface/goals/` in the install for vanilla `GFX_goal_*` and `GFX_focus_*` sprites.

## 9. Testing & debugging

- Launch with `-debug` (Steam → HOI4 → properties → launch options) to enable the console and reload commands.
- Console commands worth knowing:
  - `tdebug` — shows tooltips with internal IDs (focus IDs, state IDs, etc.).
  - `reload focus` — reload focus trees without restarting.
  - `reload loc` — reload localisation.
  - `focus.autocomplete` — instant complete focuses (toggle).
  - `focus.ignoreprerequisites` — ignore prerequisites.
  - `tag MYTAG` — switch country to test the tree.
- Check `Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log` after launching. Missing prerequisites, broken triggers, and bad effect names show up there.
- The tree won't appear at all if `country = {}` resolves to 0 weight for everyone — confirm with `tdebug` that your `tag = MYTAG` modifier is firing.

## 10. Common pitfalls

- **Two focuses sharing an `id`** — silently breaks the second one. Always prefix with your country tag.
- **Loc file without BOM** — strings show as the raw key in-game. Save as UTF-8 BOM, not plain UTF-8.
- **`prerequisite` syntax confusion** — multiple `prerequisite = { ... }` blocks AND together; multiple foci inside one block OR together. Mixing this up is the #1 reason "the focus won't unlock."
- **`relative_position_id` cycle** — A relative to B, B relative to A → game freezes on load. Pick one anchor per branch.
- **Effect typos** — `add_politcal_power` is not `add_political_power`. Game logs it once at startup and silently does nothing. Watch the log.
- **Continuous focus overlap** — pick `continuous_focus_position` coords away from your branches. Vanilla parks it bottom-right at roughly `{ x = 1500 y = 1100 }` on a 25-wide tree.
- **`allow_branch` hides children too** — useful for DLC gates, but easy to accidentally hide an entire branch you wanted available.

## 11. Minimal working tree

See `examples/common/national_focus/my_custom_focus.txt` and `examples/localisation/english/my_custom_focus_l_english.yml` in this repo for a runnable template you can drop into a mod folder and load.

## Sources

- [National focus modding — HOI4 Wiki](https://hoi4.paradoxwikis.com/National_focus_modding)
- [National focus — HOI4 Wiki](https://hoi4.paradoxwikis.com/National_focus)
- [Generic national focus tree — HOI4 Wiki](https://hoi4.paradoxwikis.com/Generic_national_focus_tree)
- [Continuous focus — HOI4 Wiki](https://hoi4.paradoxwikis.com/Continuous_focus)
- [Idea modding — HOI4 Wiki](https://hoi4.paradoxwikis.com/Idea_modding)
- [Modifiers — HOI4 Wiki](https://hoi4.paradoxwikis.com/Modifiers)
- [AI modding — HOI4 Wiki](https://hoi4.paradoxwikis.com/AI_modding)
- [Console commands — HOI4 Wiki](https://hoi4.paradoxwikis.com/Console_commands)
- [Common/national focus — HOI4 Modding Wiki (Fandom)](https://hoi4-modding.fandom.com/wiki/Common/national_focus)
- [Effects — HOI4 Modding Wiki (Fandom)](https://hoi4-modding.fandom.com/wiki/Effects)
