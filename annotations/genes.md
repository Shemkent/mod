# Genes & Persistent DNA
**Stage:** complete
**Keywords:** `genes`, `color_genes`, `morph_genes`, `accessory_genes`, `persistent_dna`, `portrait_info`, `tags`

> **System type: Data/Reference**
> **Cluster:** Characters & Estates

## Overview
The **genes** system defines heritable physical traits for character portraits — colour genes (hair, skin, eye), morphological genes (facial shape sliders), and accessory genes (hair styles, beards, clothing, headwear). These are the building blocks of the portrait rendering engine. **Persistent DNA** records the specific gene values for named historical characters, ensuring they always appear with the correct face regardless of which country the player runs.

## Vanilla File Locations
`in_game/common/genes/` — 10 files:
- `00_genes_color.txt` — hair, skin, eye colour inheritance
- `01_genes_morph.txt` — facial morphology sliders
- `02_genes_accessories_misc.txt` through `09_genes_special_expressions.txt` — accessories, clothing, expressions

`in_game/common/persistent_dna/` — 1 file:
- `custom_characters.txt` — DNA overrides for specific historical characters

## Block Structure

### Colour Genes
```pdx
color_genes = {
    hair_color = {
        color = hair                       # color type reference
        # no blend_range — hair colour does not blend during rendering
    }
    skin_color = {
        sync_inheritance_with = hair_color # inherit same way as hair
        color = skin
        blend_range = { 0.55 0.65 }       # interpolation range for skin colour blending
    }
    eye_color = {
        sync_inheritance_with = hair_color
        color = eye
    }
}
```

### Morph Genes (facial features)
`01_genes_morph.txt` has three top-level blocks — not just gene definitions:

```pdx
# 1. Texture atlas references for eyebrow/facial decals
decal_atlases = {
    male_eyebrows_atlas_inner_01 = {
        size = 3
        textures = {
            diffuse = "gfx/models/portraits/decals/..."
            normal  = "gfx/models/portraits/decals/..."
        }
    }
}

# 2. Age curve presets (blend shapes over the character's age)
age_presets = {
    age_preset_1 = {
        mode = add
        curve = { { 0.0 0.0 } { 0.32 0.0 } { 0.7 1.0 } }
    }
}

# 3. The actual morphological gene definitions
morph_genes = {
    pose = { }                             # reserved; engine expects this gene to exist

    poses_height = {
        inheritable = no
        all_height = {
            index = 0
            male = {
                setting = { attribute = "male_body_height"  value = { min = 0.1  max = 1.0 } }
            }
        }
    }

    gene_aging_body = {
        old_body = {
            index = 0
            male = {
                setting = {
                    attribute = "infant_proportions"
                    value = { min = 1  max = 1 }
                    age = { mode = multiply  curve = { { 0.0 1.0 } { 0.2 0.0 } } }
                }
                # ... more settings
            }
        }
    }
    # ... many more gene entries (chin, eyes, cheeks, nose, etc.)
}
```
Each morph gene entry has named templates (e.g. `old_body`, `all_height`) with `index`, gender blocks (`male`/`female`), and `setting` entries that map a renderer `attribute` to a value range with optional age curves. This is primarily renderer data — **modding morph genes requires matching 3D asset support**.

### Accessory Genes
```pdx
accessory_genes = {
    gene_hairstyles = {
        index = 5
        # Entries: { name = "hairstyle_01"  age_ranges = { ... } }
    }
}
```

### Persistent DNA Entry
```pdx
magnus_eriksson = {
    priority = 1                           # higher priority overrides lower if tags match
    tags = { swe_magnus_eriksson }         # character tags this DNA applies to

    portrait_info = {
        genes = {
            hair_color    = { 62 59 203 6 }       # dom_setting dom_value rec_setting rec_value
            skin_color    = { 56 30 56 30 }
            eye_color     = { 178 67 178 67 }
            gene_chin_size = { "template_1" 127 "template_1" 127 }
            # ... full gene vector for every defined gene slot
        }
    }
}
```

## Key Fields Reference

| Field | System | Purpose | Notes |
|---|---|---|---|
| `color` | Genes | Color type | `hair` / `skin` / `eye` |
| `sync_inheritance_with` | Genes | Inherits same dominant/recessive pair as named gene | Keeps facial colour coherent |
| `blend_range` | Genes | Interpolation range for color mixing | Renderer-side blending |
| `index` | Genes | Slot index in the gene array | Must be unique per gene type |
| `priority` | Persistent DNA | Override priority | Higher number wins on conflict |
| `tags` | Persistent DNA | Character identity tags | Matched against character `tags` field in history |
| `portrait_info.genes` | Persistent DNA | Raw gene vector | `setting value setting value` per gene slot |

## Modding Notes
- **Do not force accessory genes (hair, beard, clothing) via persistent DNA** — use `portrait_modifiers` in `gfx/portraits/portrait_modifiers/` instead. The game readme explicitly warns this.
- Persistent DNA entries are matched by `tags` against character history files; multiple tags = any tag matches.
- Adding new gene types requires corresponding ethnicity template entries and portrait renderer support — purely data-side changes to `genes/` will not render without 3D asset backing.
- Colour genes use a dominant/recessive inheritance model; `sync_inheritance_with` ensures related colours (hair/eye) are inherited consistently.
- Cross-references: `ethnicities` (probability distributions over gene ranges), `cultures` (which ethnicities apply to which cultures), character history files (tags for persistent DNA matching).

## Example

```pdx
# persistent_dna/custom_characters.txt
suleiman = {
    priority = 1
    tags = { tur_suleiman_the_magnificent }
    portrait_info = {
        genes = {
            hair_color = { 24 80 24 80 }
            skin_color = { 100 60 100 60 }
            eye_color  = { 90 40 90 40 }
            gene_chin_def    = { "template_1" 180 "template_1" 150 }
            gene_body_weight = { "template_1" 127 "template_1" 127 }
            # ... full vector continues
        }
    }
}
```
