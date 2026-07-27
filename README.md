# Logo Kit — Figma Plugin Specification (v10, Branded Edition)

**Status:** Implementation-ready  
**Supersedes:** v9  
**Runtime:** Figma Plugin API, main thread + UI iframe  
**Network:** None  
**Backend:** None  
**External AI:** None  
**Product credit:** Leslie Williams, Ovalay Studios  
**Creator link:** https://x.com/shugardadddy  

---

## 0. What changed from v9

v10 introduces typography without turning the plugin into a typography-detection tool.

The plugin now allows the user to choose fonts already available in their Figma environment and use them across the generated handover document.

Two font sources are supported:

- Portable fonts that are broadly available inside Figma
- All fonts available to the current user, including local and organisation fonts

These must not appear as two visibly separate modes.

The experience should feel like one intelligent font picker that:

- Prioritises portable choices
- Still exposes the full font library
- Clearly communicates portability risk
- Does not force the user to understand Figma font infrastructure
- Does not visually split the UI into “Safe” and “Advanced” sections

### Binding changes

1. Add a Typography step to the generation flow.
2. Add one unified font picker.
3. Rank portable fonts first without creating separate tabs.
4. Surface local or organisation fonts through subtle metadata.
5. Add Display and Body font roles.
6. Add a Typography presentation page.
7. Use the selected typography throughout the generated document.
8. Never attempt to identify fonts from the logo artwork.
9. Never download or embed font files.
10. Never imply that a selected local font will be editable for every recipient.

---

# 1. Product definition

Logo Kit converts selected logo artwork into a polished, client-presentable **Logo Handover Document** inside Figma.

The document contains:

- Cover
- Identity overview
- Logo treatments
- Clear space
- Minimum size
- Background use
- Recommended usage
- Brand palette
- Colour scales
- Accessibility and pairings
- Gradients where available
- Typography
- Notes and warnings

The output is a sequence of designed presentation frames.

It should feel like a compact identity document assembled by a strong brand designer.

---

# 2. Product promise

> Turn finished logo artwork into a polished brand handover document.

# 2.1 Product attribution

Logo Kit is a product by **Leslie Williams from Ovalay Studios**.

The attribution must appear in two places:

1. Inside the plugin UI, as a quiet persistent footer.
2. Inside every generated Logo Handover Document, on the Cover and Notes pages.

Required copy:

> Logo Kit by Leslie Williams · Ovalay Studios

The creator name must link to:

```text
https://x.com/shugardadddy
```

## 2.1.1 Plugin UI treatment

Place the attribution in the bottom region of the plugin UI.

Requirements:

- Keep it visually secondary to the current task.
- Do not render it as a promotional banner.
- Do not place it inside the primary action area.
- Preserve it across every step of the flow.
- Make “Leslie Williams” or the full attribution clickable.
- Open the link externally through the Figma Plugin API.
- Do not require network access for the plugin’s core functionality.

Suggested treatment:

```text
Logo Kit by Leslie Williams · Ovalay Studios
```

On activation:

```ts
figma.openExternal('https://x.com/shugardadddy');
```

## 2.1.2 Generated-document treatment

### Cover

Add a small creator credit beneath the document descriptor or at the bottom edge of the frame:

```text
Created with Logo Kit
by Leslie Williams · Ovalay Studios
```

The “Leslie Williams · Ovalay Studios” range must link to:

```text
https://x.com/shugardadddy
```

Use a proper Figma text hyperlink on the relevant range:

```ts
creditText.setRangeHyperlink(
  linkStart,
  linkEnd,
  {
    type: 'URL',
    value: 'https://x.com/shugardadddy'
  }
);
```

### Notes page

Repeat the attribution once in the closing metadata block:

```text
Generated with Logo Kit by Leslie Williams · Ovalay Studios
```

The linked range must use the same URL.

### Visual rules

- Keep the credit small and deliberate.
- Maintain sufficient contrast.
- Do not obscure or compete with the client’s identity.
- Do not place the credit over the logo artwork.
- Do not repeat the credit on every page.
- Do not allow the user to edit the creator name or destination URL in v10.
- The client’s brand must remain the dominant visual subject.

## 2.1.3 Attribution data

Store the attribution as fixed product metadata:

```ts
interface ProductAttribution {
  productName: 'Logo Kit';
  creatorName: 'Leslie Williams';
  studioName: 'Ovalay Studios';
  creatorUrl: 'https://x.com/shugardadddy';
}
```

This metadata is not user content and must not be inferred from the source artwork.

---

# 3. Scope

## Included

- One or more selected logos
- Structural logo-type assignment
- Generated treatments
- Clear-space guidance
- Minimum-size guidance
- Background-use guidance
- Recommended usage
- Colour extraction
- Colour confirmation
- Colour scales
- Accessibility
- Suggested pairings
- Valid gradients
- Display typeface selection
- Body typeface selection
- Typography page generation
- Presentation-mode selection
- Page-based document generation
- Local-only processing

## Excluded

- Font detection from logo artwork
- Font recognition from outlines
- Font matching
- Font recommendations based on brand personality
- Font downloads
- Font-file embedding
- Font licensing verification
- Automatic font substitution
- Favicons
- App icons
- Asset Library
- Export settings
- Automatic file export
- Mockups
- Social templates
- Figma variables
- Paint styles
- Developer tokens
- CSS
- External AI
- Network calls

---

# 3.1 Colour naming

Every extracted colour must receive a human-readable name in addition to its HEX, RGB and HSL values.

Use an offline colour-name dataset bundled with the plugin. The implementation should use `color-name-list` as the naming source and perform nearest-colour matching locally. A perceptual distance method is preferred over raw RGB distance. `color-namer` is acceptable where bundle size remains reasonable because it ranks names using Delta-E colour difference.

The plugin must not invent names with a language model and must not make network requests to identify colours.

## 3.1.1 Naming pipeline

For every confirmed palette colour:

1. Convert the extracted colour to a canonical HEX value.
2. Convert it to CIELAB.
3. Search the bundled colour-name dataset.
4. Select the closest name using Delta-E, preferably CIEDE2000.
5. Store the matched name, source value and distance score.
6. Allow the user to edit the displayed name before generation.

```ts
interface NamedColour {
  hex: string;
  rgb: { r: number; g: number; b: number };
  hsl: { h: number; s: number; l: number };
  name: string;
  matchedHex: string;
  deltaE: number;
  namingSource: 'COLOR_NAME_LIST';
  userEdited: boolean;
}
```

## 3.1.2 Naming rules

- Prefer concise, recognisable colour names.
- Avoid duplicate names inside the same palette. When two colours resolve to the same name, add a restrained modifier based on relative lightness, saturation or hue, such as `Deep`, `Light`, `Muted`, `Warm` or `Cool`.
- Do not use novelty names, offensive names or names that read like product copy.
- Do not label a colour as Pantone, RAL, brand-standard or print-certified.
- Display the generated name as editable suggested metadata, not objective truth.
- Preserve the exact numeric value even when the name is edited.

## 3.1.3 Colour confirmation UI

Each colour row must show:

- Large swatch
- Suggested colour name
- Editable name field
- HEX
- RGB
- HSL
- Quiet confidence treatment when the nearest match is distant

When the match is weak, use:

> Closest available colour name

Do not display raw Delta-E values in the primary UI. They may be recorded in the manifest.

## 3.1.4 Generated document usage

Use the colour name consistently on:

- Brand Palette
- Colour Scales
- Accessibility & Pairings
- Gradients
- Notes, where a fallback or duplicate-name modifier was applied

A palette label should follow this hierarchy:

```text
Ultramarine Blue
#4A90D9
RGB 74, 144, 217
```

Do not show generic labels such as `Primary`, `Secondary` or `Accent` without the actual colour name. Role labels may appear as secondary metadata.

# 4. Typography model

Typography in Logo Kit refers to fonts selected by the user for the generated handover document.

It does not refer to identifying the lettering inside the logo.

```ts
interface TypographySelection {
  display: SelectedFont;
  body: SelectedFont;
  mono: SelectedFont;
  useSingleFamily: boolean;
}

interface SelectedFont {
  family: string;
  style: string;
  source: FontSource;
  availability: FontAvailability;
  portability: FontPortability;
}

type FontSource =
  | 'FIGMA_HOSTED'
  | 'ORGANISATION'
  | 'LOCAL'
  | 'UNKNOWN';

type FontAvailability =
  | 'AVAILABLE'
  | 'UNAVAILABLE'
  | 'LOAD_FAILED';

type FontPortability =
  | 'HIGH'
  | 'LIMITED'
  | 'UNKNOWN';
```

### 4.1 Roles

Support:

- Display
- Body
- Mono

Display and Body are user-selectable.

Mono is selected automatically from the available font list.

Default preference order:

1. Roboto Mono
2. IBM Plex Mono
3. Source Code Pro
4. Any available monospace font
5. Fall back to Body

### 4.2 Single-family option

Allow:

> Use one family throughout

When enabled:

- Display and Body use the same family
- The user may choose different styles or weights
- The generated document remains visually coherent
- Mono remains separate where available

---

# 5. Unified font picker

Do not show:

- A “Safe Fonts” tab
- An “All Fonts” tab
- A segmented control
- Two separate lists
- An advanced-mode toggle

The picker must feel like one font library.

## 5.1 Font ordering

Rank fonts in this order:

1. Recent selections
2. Recommended portable fonts
3. Other Figma-hosted fonts
4. Organisation fonts
5. Local fonts
6. Unknown-source fonts

Do not label these as visible sections.

The ordering should be felt, not explained.

## 5.2 Search

The user may search by:

- Family
- Style
- Weight
- Source label

Search results appear in one list.

## 5.3 Row design

Each font row contains:

- Live family preview
- Family name
- Style
- Subtle availability indicator
- Optional portability note

Examples:

```text
Inter
Regular
```

```text
Neue Montreal
Medium
Available in this workspace
```

```text
Helvetica Neue
Regular
Available on this device
```

Do not use large warning badges.

## 5.4 Source communication

Use quiet secondary labels:

| Source | User-facing copy |
|---|---|
| FIGMA_HOSTED | No label by default |
| ORGANISATION | Available in this workspace |
| LOCAL | Available on this device |
| UNKNOWN | Availability may vary |

Portable fonts should look like the default state.

Non-portable fonts should not look dangerous or disabled.

## 5.5 Portability warning

Show one contextual note beneath the selected font only when needed.

For organisation font:

> Other editors may need access to the same organisation font library.

For local font:

> Other editors may need this font installed to edit the document correctly.

Do not show a modal.

Do not interrupt selection.

Do not call the font “unsafe.”

## 5.6 Show-more behaviour

The initial picker may show:

- Recent fonts
- High-portability fonts
- Strong common options

The rest remain accessible through:

> Browse all fonts

This expands the same list.

It must not switch modes.

---

# 6. Font discovery

Use:

```ts
const availableFonts = await figma.listAvailableFontsAsync();
```

The plugin must build a searchable local index.

## 6.1 Discovery result

```ts
interface AvailableFontRecord {
  fontName: FontName;
  family: string;
  style: string;
  inferredWeight: number | null;
  inferredWidth: FontWidth | null;
  inferredItalic: boolean;
  source: FontSource;
  portability: FontPortability;
  searchTokens: string[];
}
```

## 6.2 Source inference

Figma may not expose a perfectly reliable source classification for every font.

When source cannot be determined:

- Set source to `UNKNOWN`
- Set portability to `UNKNOWN`
- Use neutral copy
- Never claim the font is portable

## 6.3 Portable-font allowlist

Maintain a local allowlist of broadly available Figma-hosted fonts.

Examples may include:

- Inter
- Roboto
- Open Sans
- Lato
- Montserrat
- Source Sans 3
- IBM Plex Sans
- Noto Sans
- Playfair Display
- Merriweather
- Poppins
- Space Grotesk
- DM Sans
- Manrope

The allowlist is a ranking aid.

It is not a guarantee that every recipient has identical access.

Do not describe allowlisted fonts as universally available.

## 6.4 No network dependency

The allowlist must ship inside the plugin bundle.

The plugin must not fetch font metadata.

---

# 7. Font loading

Before creating text nodes:

```ts
await figma.loadFontAsync({
  family: selected.family,
  style: selected.style
});
```

## 7.1 Loading order

Load:

1. Display
2. Body
3. Mono
4. Any fallback variants needed by generated components

## 7.2 Failure handling

When loading fails:

1. Try another available style in the same family.
2. Try the nearest weight.
3. Fall back to the saved portable fallback.
4. Record `W_FONT_FALLBACK`.
5. Update the manifest with the font actually used.

Do not abort the whole generation because one font style failed.

## 7.3 Source artwork

Never load or modify fonts inside source logo text nodes unless required to clone the original safely.

Never change the source logo font.

---

# 8. Typography step

Add the step after presentation mode.

```text
VALIDATING
   ↓
ASSIGNING_TYPES
   ↓
CONFIRMING_PALETTE
   ↓
CHOOSING_PRESENTATION
   ↓
CHOOSING_TYPOGRAPHY
   ↓
REVIEWING
   ↓
GENERATING
   ↓
COMPLETE
```

## 8.1 Layout

The step contains:

- Live document preview
- Display font picker
- Body font picker
- Use-one-family control
- One short portability note when relevant

Do not show the picker as a technical font-management interface.

## 8.2 Live preview

Show:

- Cover title
- Page heading
- Body paragraph
- Technical label
- Colour value

Example content:

```text
Union
Logo Handover

Primary Logo
The main identifier for first-touch brand communications.

Suggested minimum clear space
32px
```

The preview must update immediately.

## 8.3 Defaults

Default Display:

- Use the strongest available portable sans or serif appropriate to the selected presentation style.
- Do not infer from the logo.
- Default preference: Inter Semi Bold.

Default Body:

- Inter Regular.

Default Mono:

- Roboto Mono Regular.

The user can change them.

## 8.4 Do not make recommendations look intelligent

Avoid copy like:

> We chose this font because it matches your brand.

The plugin does not know that.

Use:

> Suggested document typography

This recommendation is based on portability and document readability only.

---

# 9. Typography application

The selected fonts shape the generated document.

## 9.1 Display typeface

Use for:

- Cover brand name
- Cover descriptor where appropriate
- Page titles
- Large numeric callouts
- Recommended minimum-size statement
- Major section statements

## 9.2 Body typeface

Use for:

- Page leads
- Body copy
- Usage guidance
- Notes
- Descriptions
- Warnings
- Captions where mono is unnecessary

## 9.3 Mono typeface

Use for:

- HEX
- RGB
- HSL
- Measurements
- Aspect ratios
- Manifest id
- Technical values

## 9.4 Style mapping

Map available styles deterministically.

```ts
interface FontRoleStyles {
  displayRegular: FontName;
  displayEmphasis: FontName;
  bodyRegular: FontName;
  bodyMedium: FontName;
  bodyBold: FontName;
  monoRegular: FontName;
}
```

When a precise style is unavailable:

- Choose the nearest available style
- Do not synthesise a fake style
- Do not assume style names are standardised

---

# 10. Typography page

Add one Typography page to the document.

Default position:

```text
After Brand Palette
Before Colour Scales
```

The page documents the fonts selected for the handover document. It does not claim they were detected from the logo.

The current single-column specimen is too sparse and must not be used as the default composition. The page should feel like an editorial type specimen, with hierarchy, rhythm and practical examples of use.

## 10.1 Page title

> Typography

Lead:

> Suggested type system for this handover document.

If the user explicitly confirms the typefaces as brand fonts, allow the lead to become:

> Brand type system

Do not default to that claim.

## 10.2 Required content

Every Typography page must include:

- Display family and selected style
- Body family and selected style
- Mono family where available
- Oversized display specimen
- Alphabet and numeral specimen
- A realistic heading and paragraph composition
- A small hierarchy demonstration
- Weight or style comparison where variants exist
- Technical-value example using the mono face
- Quiet availability note when relevant

Do not place all content in one narrow column with a large empty right side.

## 10.3 Layout system

Use a 12-column editorial grid. Select one of the following compositions based on the selected font relationship.

### A. Typeface Pairing

For different Display and Body families:

- Display specimen spans columns 1–7.
- Body specimen spans columns 8–12.
- A full-width hierarchy strip sits across the lower third.
- Technical and availability metadata sits in a restrained footer band.

Display area contains:

- Oversized `Aa` or a short brand-neutral word
- Family name
- Style name
- Uppercase alphabet
- Numerals
- One expressive heading

Body area contains:

- Family and style
- Sentence-case alphabet
- Paragraph at intended reading size
- Caption and small-label examples
- Line-height and size examples

### B. Single Family

When one family is used throughout:

- Use one oversized editorial composition across the top two-thirds.
- Show Display, Medium, Regular and Mono roles as a horizontal comparison.
- Include one realistic content block demonstrating H1, H2, body, caption and value styles.
- Use scale and alignment rather than cards to distinguish roles.

### C. Limited Variant

When only one or two usable styles are available:

- Do not fake a weight comparison.
- Build contrast through size, spacing, case and colour.
- Explain quietly that the document uses the available styles.

## 10.4 Hierarchy specimen

Include a practical mini-layout instead of only an alphabet dump.

Example:

```text
Build recognition through consistency

A clear identity system helps a brand remain recognisable across every context.

Section label                 01
Minimum clear space           32 px
```

The hierarchy specimen should demonstrate:

- Page title
- Section heading
- Body paragraph
- Caption
- Technical value

## 10.5 Type scale

Generate a compact type-scale reference using the actual document styles.

```ts
interface DocumentTypeScale {
  display: TextStyleSpec;
  h1: TextStyleSpec;
  h2: TextStyleSpec;
  body: TextStyleSpec;
  caption: TextStyleSpec;
  technical: TextStyleSpec;
}

interface TextStyleSpec {
  font: FontName;
  size: number;
  lineHeight: number | 'AUTO';
  letterSpacing: number;
  textCase: 'ORIGINAL' | 'UPPER' | 'LOWER' | 'TITLE';
}
```

Show no more than six styles. The page is a handover document, not a full design-system specification.

## 10.6 Specimen content

Display specimen:

```text
Aa
ABCDEFGHIJKLM
NOPQRSTUVWXYZ
0123456789
```

Body specimen:

```text
A clear identity system helps a brand remain recognisable across every context.
```

Do not use pangrams by default.

## 10.7 Visual direction

- Use generous scale contrast.
- Balance the full frame; avoid large accidental empty zones.
- Let the typeface itself become the primary visual asset.
- Use rules, baseline alignment and spacing to create structure.
- Avoid small card grids.
- Avoid presenting the page like a settings screen.
- Keep samples large enough to judge shape, rhythm and texture.
- Use the extracted brand palette sparingly for dividers, page numbers or specimen highlights.
- Maintain presentation-mode contrast requirements.

## 10.8 Availability note

For local or organisation fonts, place a small note at the bottom:

> Editing this document may require access to the selected font.

Do not let the note dominate the page.

## 10.9 No fake brand guidance

Do not automatically generate:

- Font-pairing rationale
- Tone claims
- Brand personality claims
- Recommended web fallbacks
- Licensing claims
- CSS stacks

# 11. Document architecture

For one logo entry:

1. Cover — includes the quiet Logo Kit creator credit
2. Identity Overview
3. Logo Treatments
4. Clear Space
5. Minimum Size
6. Background Use
7. Recommended Usage
8. Brand Palette
9. Typography
10. Colour Scales
11. Accessibility & Pairings
12. Gradients
13. Notes — includes the closing Logo Kit creator credit

Omit pages without valid content.

Typography remains present unless font discovery fails entirely.

When font discovery fails:

- Use an internal fallback
- Generate the document
- Omit the Typography page
- Record `W_FONT_DISCOVERY_FAILED`

---

# 12. Typography and presentation modes

## 12.1 Light mode

Prefer:

- Strong dark display text
- Restrained body typography
- High whitespace
- Display family may be serif or sans

## 12.2 Dark mode

Prefer:

- Display weights with sufficient stroke strength
- Avoid very thin styles
- Ensure body rendering remains legible
- Use light text that passes contrast

## 12.3 Brand mode

Check selected type against brand surfaces.

When a chosen style becomes unreadable:

- Use another generated colour scale step
- Do not silently change the selected font
- Only change the colour treatment

---

# 13. Manifest changes

```ts
interface GenerationManifest {
  schemaVersion: 10;
  typography: TypographySelection;
  colours: NamedColour[];
  attribution: ProductAttribution;
  // existing v9 fields remain
}
```

Persist:

- Display family and style
- Body family and style
- Mono family and style
- Source classifications
- Portability classifications
- Actual loaded fallbacks
- Use-one-family state

---

# 14. Re-run behaviour

On re-run:

- Preselect the previous fonts when still available
- Preserve the previous source labels
- Warn when a font is no longer available
- Offer the nearest style in the same family
- Fall back only after user confirmation during the Typography step

Do not silently replace a missing selected family before the user sees the change.

---

# 15. Privacy

The plugin may inspect the list of fonts available to the current Figma environment.

It must not:

- Upload font names
- Upload font availability
- Upload organisation font metadata
- Read font files
- Export font files
- Share local font paths
- Download fonts
- Send typography analytics

---

# 16. Errors and warnings

## Blocking errors

```text
E_NO_FONTS_AVAILABLE
E_TYPOGRAPHY_SELECTION_INVALID
```

These should rarely block the full plugin.

Where possible, fall back and proceed.

## Warnings

```text
W_FONT_FALLBACK
W_FONT_DISCOVERY_FAILED
W_FONT_SOURCE_UNKNOWN
W_FONT_NOT_PORTABLE
W_SELECTED_FONT_MISSING
W_STYLE_SUBSTITUTED
```

## User-facing language

Avoid:

- Unsafe font
- Unsupported font
- Bad choice
- Incompatible font

Prefer:

- Available on this device
- Available in this workspace
- Availability may vary
- Another style was used
- Another editor may need access to this font

---

# 17. Visual acceptance criteria

## Typography step

1. The picker appears as one continuous experience.
2. There are no Safe/All tabs.
3. There is no segmented mode control.
4. Portable fonts appear naturally near the top.
5. Local and organisation fonts remain searchable.
6. Source notes remain visually secondary.
7. The live preview updates without reopening the picker.
8. A local-font warning does not interrupt selection.

## Typography page

9. The page looks like a premium type specimen.
10. It does not use a grid of small font cards.
11. Display typography is the dominant visual element.
12. Body typography remains easy to assess.
13. Availability notes remain secondary.
14. The page does not claim the fonts were extracted from the logo.
15. The page adapts to same-family and paired-family selections.

## Whole document

16. Selected typography is applied consistently.
17. Mono values remain visually distinct.
18. Missing font styles do not break generation.
19. Fallback usage appears in Notes.
20. The Typography page fits the same art direction as the rest of the document.
21. The Cover contains the Logo Kit creator credit without competing with the client logo.
22. The Notes page contains the closing creator credit.
23. The attribution remains readable in Light, Dark and Brand presentation modes.
24. Typography pages use the full frame intentionally and do not leave an accidental empty half.
25. Typography pages include a practical hierarchy specimen and compact type scale.
26. Every palette colour includes an editable human-readable name.
27. Generated colour names remain secondary to exact numeric values.

---

# 18. Functional acceptance criteria

1. `listAvailableFontsAsync()` is called once per plugin launch unless refresh is requested.
2. Search covers family and style.
3. Display and Body fonts can come from any available source.
4. Portable and non-portable fonts appear in one list.
5. The user can browse all fonts without switching mode.
6. Selected fonts are loaded before text nodes are created.
7. Missing styles use deterministic fallback.
8. Source logo text is never restyled.
9. Local-font use produces a portability note.
10. Organisation-font use produces a workspace-access note.
11. Font metadata never leaves Figma.
12. Re-runs restore available previous selections.
13. Missing previous fonts are surfaced before generation.
14. Typography page can be omitted safely when discovery fails.
15. The rest of the v9 document continues to generate correctly.
16. The plugin UI exposes the persistent creator link without interrupting the flow.
17. Activating the creator link opens https://x.com/shugardadddy externally.
18. The Cover and Notes attribution ranges contain valid URL hyperlinks.
19. Attribution generation does not introduce a network dependency.
20. Colour naming runs locally from a bundled dataset.
21. Nearest-name matching uses a perceptual colour-distance method.
22. Users can edit suggested colour names without changing the underlying colour.
23. Duplicate colour names are disambiguated deterministically.

---

# 19. Definition of done

Logo Kit v10 is complete when:

- Users can select both portable and locally available fonts
- The picker does not feel split into separate modes
- Portable fonts are prioritised without hiding the full library
- Local and organisation fonts carry subtle, accurate availability notes
- Selected fonts shape the entire handover document
- The Typography page feels like a premium brand-document page
- The plugin never claims to identify the logo’s font
- Missing fonts fail gracefully
- No font files are downloaded, embedded or exposed
- No network traffic occurs
- Source artwork remains unchanged
- The plugin UI includes a persistent, secondary credit for Leslie Williams and Ovalay Studios
- Every generated document includes linked creator credits on the Cover and Notes pages
- The creator link resolves to https://x.com/shugardadddy
- Every extracted colour receives an editable, locally generated name
- The Typography page feels complete, editorial and intentionally composed rather than sparse
