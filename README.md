# pbi-measure-lineage

**Interactive measure dependency explorer for Power BI models.** Visualize which measures depend on which other measures and columns — entirely in the browser, no installs required.

→ **[Launch the tool](https://YOUR_USERNAME.github.io/pbi-measure-lineage/)**

---

## What it does

Drop in your Power BI model's dependency data and instantly get:

- **Focus View** — Select any measure from a searchable sidebar. See its upstream dependencies (what it depends on) and downstream dependents (what uses it) in a clean left-to-right PBI lineage-style layout. Measures and columns are grouped separately.
- **Graph View** — Full force-directed network of all measure relationships. Click any node to highlight its complete dependency chain. Filter by measures/columns. Search, zoom, pan.

Columns appear only as dependencies of measures — they don't clutter the sidebar or dominate the graph.

## Why this exists

Power BI has no built-in way to see measure-level lineage. The workspace lineage view stops at the dataset/report level. When you have 300+ measures and need to understand what feeds what — especially during optimization, refactoring, or onboarding — you need something better. This tool fills that gap.

## Getting started

### Option 1: DAX Query View (recommended — zero installs)

This works in any Power BI Desktop file. No external tools needed.

**Step 1.** Open your `.pbix` → go to the **DAX Query View** tab → paste and run:

```dax
EVALUATE
VAR _deps = INFO.CALCDEPENDENCY()
VAR _filtered =
    FILTER(
        _deps,
        [OBJECT_TYPE] = "MEASURE"
            && NOT CONTAINSSTRING([TABLE], "LocalDateTable_")
            && NOT CONTAINSSTRING([TABLE], "DateTableTemplate_")
            && NOT CONTAINSSTRING([REFERENCED_TABLE], "LocalDateTable_")
            && NOT CONTAINSSTRING([REFERENCED_TABLE], "DateTableTemplate_")
    )
RETURN
SELECTCOLUMNS(
    _filtered,
    "FromObject", [OBJECT],
    "FromTable", [TABLE],
    "ToObject", [REFERENCED_OBJECT],
    "ToTable", [REFERENCED_TABLE],
    "DependencyType", [REFERENCED_OBJECT_TYPE]
)
```

**Step 2.** In the results grid: **Ctrl+A** → **Ctrl+C** to copy all rows.

**Step 3.** Open the tool → **Load Data** → paste into the text area → **Load Graph**.

### Option 2: Tabular Editor script (TE2 free or TE3)

Run this C# script in Tabular Editor (works in both the free TE2 and paid TE3):

```csharp
var sb = new System.Text.StringBuilder();
sb.AppendLine("FromObject,FromTable,ToObject,ToTable,DependencyType");

foreach (var m in Model.AllMeasures)
{
    foreach (var dep in m.DependsOn.Measures)
    {
        sb.AppendLine(
            "\"" + m.Name + "\",\"" + m.Table.Name + "\",\"" +
            dep.Name + "\",\"" + dep.Table.Name + "\",\"Measure\""
        );
    }
    foreach (var col in m.DependsOn.Columns)
    {
        sb.AppendLine(
            "\"" + m.Name + "\",\"" + m.Table.Name + "\",\"" +
            col.Name + "\",\"" + col.Table.Name + "\",\"Column\""
        );
    }
}

var filePath = @"C:\temp\measure_lineage.csv";
System.IO.File.WriteAllText(filePath, sb.ToString());
Info("Exported to " + filePath);
```

Then load the CSV file or paste its contents into the tool.

### Option 3: Demo data

Click **Load Data** → **Demo** tab → **Load Graph** to explore the tool with a sample model.

## How to use

### Focus View (default)

1. Select a measure from the left sidebar (use the search to filter).
2. The right panel shows:
   - **Top section**: PBI-style lineage diagram with upstream dependencies on the left, the selected measure in the center, and downstream dependents on the right. Measures and columns are visually grouped.
   - **Bottom section**: Two-column detail list (Depends On / Used By) with type badges and table names.
3. Click any measure card in the diagram or detail list to navigate to it.

### Graph View

1. Switch to **Graph** using the toggle in the header.
2. The full force-directed graph renders all measures and their relationships.
3. Click any node to highlight its full upstream/downstream chain with a slide-out detail panel.
4. Use the **Measures** / **Columns** filter toggles to show or hide node types.
5. Search by name or table. Zoom/pan/drag freely.

## Input formats

The tool auto-detects the format:

| Format | Delimiter | Source |
|---|---|---|
| Tab-separated (TSV) | Tab | DAX Query View paste, DAX Studio export |
| Comma-separated (CSV) | Comma | Tabular Editor script, custom exports |

Expected columns: `FromObject`, `FromTable`, `ToObject`, `ToTable`, `DependencyType`

The `DependencyType` column is optional — if missing, all dependencies default to "Measure". Recognized types: `MEASURE`, `COLUMN_DATA`, `ATTRIBUTE`, `CALC_COLUMN`, `TABLE`.

Auto date/time tables (`LocalDateTable_*`, `DateTableTemplate_*`) are automatically filtered out.

## DAX Query View vs Tabular Editor: what's different?

Both methods produce reliable output. The main differences:

| | DAX Query View | Tabular Editor |
|---|---|---|
| **Installs required** | None (built into PBI Desktop) | TE2 (free) or TE3 (paid) |
| **Accuracy** | Engine-resolved (exact) | Engine-resolved (exact) |
| **Table-scan dependencies** | Returns `TABLE` type (e.g., `COUNTROWS`) | Resolves to individual columns |
| **Coverage** | Slightly more complete for table-scan patterns | May miss measures with only `TABLE` refs |

For most users, DAX Query View is the easiest and most complete option.

## Sharing with your team

This is a single HTML file. To share:

1. Send the `index.html` file along with the exported CSV/TSV data.
2. The recipient opens the HTML in any modern browser, pastes the data, and explores.
3. No logins, no installs, no Power BI license needed to view.

Or point them to the hosted GitHub Pages version and have them paste the data directly.

## Deployment

### GitHub Pages (recommended)

This tool is designed to run as a static GitHub Pages site:

1. Fork or clone this repo.
2. Go to **Settings** → **Pages** → set source to **Deploy from a branch** → select `main` and `/ (root)`.
3. Your tool will be live at `https://YOUR_USERNAME.github.io/pbi-measure-lineage/`.

### Self-hosted

Drop `index.html` on any static file server or open it directly from your filesystem.

## Roadmap

- [ ] VPAX file upload (parse DAX expressions client-side)
- [ ] PBIP / TMDL folder upload (zero-install file-based input)
- [ ] Impact analysis mode (select a column → see every measure that depends on it)
- [ ] Export lineage as image / PDF
- [ ] Depth control (show 1 level, 2 levels, or full chain)

## Contributing

Issues and PRs are welcome. If you find edge cases in the DAX query or parser (unexpected dependency types, models that don't parse correctly), please open an issue with a sample of the problematic rows.

## License

MIT

