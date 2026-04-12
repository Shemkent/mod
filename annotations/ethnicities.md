# Ethnicities
**Stage:** complete
**Keywords:** `ethnicities`, `ethnicity_template`, `template`, `skin_color`, `eye_color`, `hair_color`, `hair_styles`, `expressions_mouth`, `expressions_eyes`, `expressions_brow`, `expressions_forehead`

> **System type: Data/Reference**
> **Cluster:** Characters & Estates

## Overview
Ethnicities define the probability distributions over gene expression for characters of a given regional background. Where `genes/` defines *what* genes exist, ethnicities define *how likely* each gene value range is for a given group — including facial morphology, colour distributions, hair styles, and expression presets. All concrete ethnicities inherit directly from `ethnicity_template` (or from a regional intermediate that itself inherits from `ethnicity_template`). Characters' ethnicities are resolved via graphical culture type assignments, not directly from `cultures/`.

## Vanilla File Locations
`in_game/common/ethnicities/` — 53 files:
- `00_ethnicities.txt` — `ethnicity_template` base definition
- Per-region files with granular naming: `00_ethnicities_african_bantu.txt`, `00_ethnicities_african_mande.txt`, `00_ethnicities_american_andean.txt`, etc.

## Block Structure

```pdx
african_mande_ethnicity = {
    template = "african_ethnicity"   # inherit from a regional intermediate (which inherits from ethnicity_template)
                                     # all concrete ethnicities ultimately trace back to ethnicity_template

    # --- Colour fields: 2D bounding box sampling on a colour texture ---
    # Each entry is: weight = { x_min y_min \n x_max y_max }
    skin_color = {
        10 = {
            0.4 0.62   # top-left corner of sampling box (x y)
            0.6 0.76   # bottom-right corner
        }
    }

    eye_color = {
        80 = { 0.72 0.75    # brown eyes — high weight
                0.95 0.99 }
    }

    hair_color = {
        20 = {
            0.0 0.7    # black hair
            0.1 0.85
        }
    }

    # --- Hair style distribution ---
    hair_styles = {
        1 = { name = ashanti_hair    range = { 0.0 1.0 } }
    }

    # --- Facial expression presets (four independent blend-shape channels) ---
    expressions_mouth = {
        40 = { name = no_expression    range = { 0.0 1.0 } }
        10 = { name = smile_01         range = { 0.0 1.0 } }
        20 = { name = displeased_01    range = { 0.0 0.5 } }
    }

    expressions_eyes = {
        50 = { name = no_expression    range = { 0.0 1.0 } }
        10 = { name = squint_01        range = { 0.0 0.3 } }
    }

    expressions_brow = {
        50 = { name = no_expression     range = { 0.0 1.0 } }
        10 = { name = frown_angry_01    range = { 0.0 0.8 } }
    }

    expressions_forehead = {
        100 = { name = no_expression      range = { 0.0 1.0 } }
        10  = { name = eyebrow_raise_01   range = { 0.0 0.5 } }
    }

    # --- Morphological gene distributions (weighted ranges on the [0,1] slider) ---
    gene_eye_height = {
        25 = { name = template_1    range = { 0.2 0.4 } }
        50 = { name = template_1    range = { 0.4 0.45 } }
        25 = { name = template_1    range = { 0.45 0.55 } }
    }

    gene_cheek_def = {
        10 = { name = template_1    range = { 0.2 0.4 } }
        50 = { name = template_1    range = { 0.4 0.6 } }
        30 = { name = template_1    range = { 0.6 0.8 } }
    }
    # ... many more gene_* morph entries
}
```

## Key Fields Reference

| Field | Purpose | Notes |
|---|---|---|
| `template` | Base ethnicity to inherit from | All fields not overridden fall back to the template |
| `skin_color` / `eye_color` / `hair_color` | 2D bounding-box sampling on the colour texture | Each entry: `weight = { x_min y_min \n x_max y_max }`; coordinates are fractions of the texture atlas |
| `hair_styles` | Weighted hair style preset selection | `weight = { name = style_id  range = { min max } }` |
| `expressions_mouth` | Weighted mouth expression preset | `weight = { name = preset  range = { intensity_min intensity_max } }` |
| `expressions_eyes` | Weighted eye expression preset | Same format as mouth |
| `expressions_brow` | Weighted brow expression preset | Same format |
| `expressions_forehead` | Weighted forehead expression preset | Same format |
| `gene_<feature>` | Probability distribution over a morph gene's slider range | `weight = { name = template_1  range = { min max } }` |

## Modding Notes
- **Culture → ethnicity linkage is indirect.** Cultures assign a graphical culture tag (e.g. `english_gfx`), and `gfx/graphical_culture_types/` entries carry `ethnicities = { weight = ethnicity_name }` blocks. Ethnicities are not referenced directly in `common/cultures/`.
- The `template` field triggers direct single-level inheritance. Most regional ethnicities inherit from `ethnicity_template` directly or via a regional intermediate (e.g. `african_ethnicity`); there is no deep override hierarchy.
- Colour fields (`skin_color`, `eye_color`, `hair_color`) use 2D bounding-box coordinates on a colour texture atlas, not the `range = { min max }` format used by morph genes. The coordinates are `{ x_min y_min  x_max y_max }` on a `[0, 1]` normalised texture space.
- Fields not specified in a concrete ethnicity fall through to the parent `template` value; the base `ethnicity_template` must define all fields to avoid missing-data errors.
- Adding a new ethnicity requires a corresponding graphical culture type entry pointing to it. Portrait assets are inherited from gene definitions, not from the ethnicity itself.
- The four expression channels (`mouth`, `eyes`, `brow`, `forehead`) are independent. Omitting any one inherits the template's distribution for that channel.

## Example

```pdx
# From 00_ethnicities_african_mande.txt
african_mande_ethnicity = {
    template = "african_ethnicity"

    skin_color = {
        10 = { 0.4 0.62    0.6 0.76 }   # deep brown range on the skin colour atlas
    }
    eye_color = {
        80 = { 0.72 0.75    0.95 0.99 }  # predominantly dark brown
    }
    hair_color = {
        20 = { 0.0 0.7    0.1 0.85 }     # black
    }
    hair_styles = {
        1 = { name = ashanti_hair    range = { 0.0 1.0 } }
    }
    gene_cheek_def = {
        10 = { name = template_1    range = { 0.2 0.4 } }
        50 = { name = template_1    range = { 0.4 0.6 } }
        30 = { name = template_1    range = { 0.6 0.8 } }
    }
    # ... additional gene_* and expression overrides
}
```
*A real vanilla ethnicity inheriting from `african_ethnicity`; overrides colour atlas coordinates and a specific hair style while all unspecified morphology falls back to the template.*
