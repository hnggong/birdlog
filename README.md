# birdlog converter

A browser-based tool that converts birdlog notes, recorded via iPhone Shortcuts, into a CSV file ready for direct import into [eBird](https://ebird.org).

## Overview

Birdlog is a two-part system:

1. **[iPhone Shortcut]https://www.icloud.com/shortcuts/92799c91914e4def878b1b4804599278**: captures a voice recording of a bird observation and appends a structured 4-line entry to a Notes file, including timestamp, the voice-dictated species name, and locality string with GPS coordinates.

2. **Birdlog Converter**: a single HTML file that runs entirely in the browser. Paste your birdlog notes into the input box, click Convert, get a live preview of the parsed observations, and download a CSV in eBird Record Format (Extended) for eBird import.

## Input Format

Each observation in the notes file is a 4-line block:

```
DD_MM_YYYY_HH_MM
<species name>
<locality string with coordinates>
---
```

**Example:**
```
18_03_2026_14_32
American Robin
Coyote Lake County Park, San Martin US-CA (37.0828,-121.5321)
---
18_03_2026_14_45
3 CAGO
Delaware and Raritan Canal Towpath, Titusville US-NJ (40.2785,-74.8538)
---
```

- **Line 1** — timestamp in `DD_MM_YYYY_HH_MM` format, underscore-separated
- **Line 2** — captured via voice dictation (species name, count + species, a command, etc.)
- **Line 3** — locality string, ending with GPS coordinates in `(lat,lon)` format
- **Line 4** — `---` separator (blank lines between entries are also accepted)

## Spoken Part Processing

The spoken part (line 2) goes through a series of transformations before becoming a CSV row.

### Count extraction

Counts can be expressed two ways:

**As digits directly before the species name:**
```
3 robin       →  count: 3,  species: robin
12CAGO        →  count: 12, species: CAGO
```

**As spoken number homophones** (words that sound like numbers but are spelled differently by voice recognition):

| Spoken word | Interpreted as |
|-------------|---------------|
| `one`       | 1             |
| `to`        | 2             |
| `free`      | 3             |
| `for`       | 4             |
| `ate`       | 8             |

Example: `free robin` → count: 3, species: robin

If no count is found, the count defaults to 1.

### 4-letter code uppercasing

If the species name — after removing any spaces — is exactly 4 alphabetic characters, it is converted to uppercase. This handles standard eBird banding/alpha codes.

```
deju   →  DEJU
amro   →  AMRO
```

### Skip logic

The following voice entry text are ignored and produce no CSV row:

- Empty voice entry
- Voice entry that is purely numeric (e.g. `3`)
- Voice entry that is exactly `test` (case-insensitive) 

### The `remove` command

If the voice entry is `remove`, the most recently added observation is deleted from the output. 

## Plus syntax — multiple species per record

A single voice entry can contain multiple species separated by a plus sign. Both forms are accepted:

- The word `plus` surrounded by spaces: `amro plus deju`
- The symbol `+` with optional surrounding spaces: `amro + Mdeju` or `amro+deju`

Multiple pluses are supported: `amro plus deju plus cago`

Each species token is processed independently (count extraction, 4-letter uppercasing, skip logic all apply per token). All resulting CSV rows share the same date, time, and location as the original record.

**`remove` after a plus record** removes all rows produced by that record. If `amro plus deju` was the previous record, a subsequent `remove` deletes both the AMRO and MALL rows.

## Output

The converter produces a table preview and a downloadable CSV in eBird Record Format (Extended). Each row contains:

| Column | Value |
|--------|-------|
| Species | Processed species name |
| Count | Numeric count (default 1) |
| Location | Full locality string with rounded coordinates |
| Lat | Latitude rounded to 4 decimal places |
| Long | Longitude rounded to 4 decimal places |
| Date | MM/DD/YYYY |
| Time | HH:MM |
| Protocol | `incidental` (fixed) |

Warnings are shown for entries with malformed timestamps or missing coordinates — these entries are skipped but all others are still processed.

## Usage

1. Paste birdlog notes into the input box on the left
2. Click **Convert** to preview parsed observations on the right
3. Click **Download CSV** to save the file, then import it into eBird via [eBird Imports](https://ebird.org/import/upload.form)
