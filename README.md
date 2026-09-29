# Custom Fonts

Place `.ttf`, `.otf`, `.woff`, or `.woff2` font files in this directory.

These fonts will be included in the Document Server image alongside the Microsoft core fonts (Arial, Times New Roman, Calibri, etc.). The default ONLYOFFICE core-fonts bundle is excluded.

After adding or removing fonts, rebuild the DS Docker image. The build runs `documentserver-generate-allfonts.sh` to index them automatically.

## Font catalog (`fonts.json`)

`fonts.json` is the source of truth for which fonts the **portal** offers to users
(the font picker, the catalog API, and per-user font selection). The portal reads
this file at startup from the mounted fonts directory (`/data/fonts` in
production), so updating it and redeploying is enough — no code change required.

Format:

```json
{
  "version": 1,
  "defaults": ["Baskervville", "Cormorant", "..."],
  "fonts": [
    { "name": "Baskervville", "file": "Baskervville-Regular.ttf" },
    { "name": "Cormorant",    "file": "Cormorant-Regular.otf" }
  ]
}
```

- `fonts[].name` — the display name shown in the editor/UI. It must match the
  font family name as indexed by the Document Server (this is the name the editor
  applies to text).
- `fonts[].file` — a font file in **this** directory. It is served at
  `/static-fonts/<file>` and used only for the `@font-face` preview swatches in
  the UI. It does not need to be the same weight/style as every variant shipped
  for that family; pick a representative regular file.
- `defaults` — the font names pre-selected for users who have not customized
  their picker. Each must appear as a `name` in `fonts`.

Not every file in this directory needs a catalog entry. This directory typically
contains many weights/styles per family (and some files that are not meant to be
user-selectable); `fonts.json` is the curated subset exposed in the UI.

If `fonts.json` is missing or invalid, the portal falls back to a built-in
catalog baked into the application, so a bad edit will not take the font picker
offline — but the change simply won't take effect until the file is valid.
