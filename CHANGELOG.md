# Changelog

All notable changes to the Model Health Extractor project will be documented in this file.

## [2026-06-16]
### Final Polish Phase
- **UI Save File Dialog Integration:** Replaced the hardcoded filepath input with a Windows Forms `SaveFileDialog` UI popup, allowing users to select the save destination interactively when the script runs.
- **Extraction Note UI & Column:** Added a second UI prompt that appears after the Save dialog to ask for an "Extraction Note" (e.g., phase name like '50% DD'). This Note is now inserted as a new column between "Project Number" and "File Size" across all sheets for easy phase comparison.
- **Node Input Simplification:** Reduced the number of Dynamo `IN` ports from 9 down to 1 (`IN[0]`), which now acts as a master toggle to enable/disable the script. The script now intelligently appends to existing files to maintain history, or creates new ones.
- **File Size Detection Fix:** Upgraded `get_file_size_mb` to correctly detect when a model is "Unsaved", on "Revit Server" (`RSN://`), or "Cloud Model", instead of outputting `0` when the file path is inaccessible.
- **Refactored Codebase:** Completely rewrote the Python script (`dynamo_extractor_node_final.py`) into modular functions (`extract_warnings()`, `extract_groups()`, etc.) for vastly improved readability and future maintenance.
- **HTML User Manual:** Authored a comprehensive HTML User Manual (`Manuals\ModelHealth_Extractor_Manual.html`) utilizing MTC's UI guidelines and CSS styling, featuring embedded HTML mock simulation tables.
- **Mock Data Generation:** Developed a Python script to automatically generate `Mock_ModelHealth_Report.xlsx`.

### Added
- 'Date Extracted', 'Project Name', 'Project Number' columns prepended to all datasets as a Stamp/Group.
- 'openpyxl' automatic row grouping (`ws.row_dimensions.group`) to make appended data collapsable.
- Added `Creator` (Worksharing User) tracking to Views & Sheets, and CAD Links datasets.
- Added **Revit Links** dataset tracking loaded status, attachment type, and pinned status.
- Added **Model / Detail Groups** dataset tracking placement counts and member counts.
- Added **Line Styles & Fill Patterns** dataset to detect suspicious junk styles (e.g. from AutoCAD).
- Added **Loadable Families** dataset tracking usage count, sorted automatically by Category then Name.
- Added grouping by Creator for Views & Sheets, CAD Links, and Line Styles.
- Added Summary counts for the new datasets.

### Fixed
- **GroupSet Indexing TypeError Bug:**
  - *Symptom:* The "Groups" dataset was completely empty (0 rows) even though the model had many groups.
  - *Root Cause:* The Revit API's `GroupSet` object does not support integer indexing (e.g., `placements[0]`). Attempting this threw a `TypeError`. In addition, some groups in Revit may not have certain properties available, triggering silent failures in the CPython3 environment.
  - *Fix:* Replaced `placements[0]` with `placements.GetEnumerator()` to safely extract the first instance. Also implemented "Bulletproof" logic: split every single property access (`Name`, `Category`, `Creator`, etc.) into its own `try/except` block. Modified the exceptions to catch and output the exact exception string directly into the Excel cells.

- **Python.NET `property cannot be read` Bug:**
  - *Symptom:* After deploying the bulletproof logic, the Group Name column outputted `property cannot be read` for every single group, while other columns like Category and Placement Count worked fine.
  - *Root Cause:* This is a known bug in the `pythonnet` package (used by CPython3 in Dynamo/Revit) where accessing a property defined in a base class (like `Element.Name`) on a derived type wrapper (like `GroupType`) fails to resolve the property getter correctly.
  - *Fix:* Bypassed the bug by calling the descriptor directly on the base class: `Element.Name.__get__(gtype)`. Added a secondary fallback to use `gtype.get_Parameter(BuiltInParameter.SYMBOL_NAME_PARAM).AsString()`.

- **Incorrect Creator Grouping Bug:**
  - *Symptom:* The Excel grouping headers for Line Styles, CAD Links, and Views & Sheets showed the Item Name/ID instead of the Creator's name (e.g., `--- 👤 ผู้นำเข้า: 1000mm Horizontal ---`).
  - *Root Cause:* The `category_col_idx` was miscalculated because the 3 prepended stamp columns (`Date`, `Project Name`, `Project Number`) were forgotten during the index counting.
  - *Fix:* Adjusted `category_col_idx` to `8` for ViewsAndSheets and `6` for CADLinks/LineStyles to correctly point to the `Creator` column.

### Fixed
- **Revit API TypeError Issue:**
  - *Symptom:* `TypeError` occurred when attempting to read CAD links.
  - *Root Cause:* Using `hasattr(imp_type, "Name")` in Python.NET can trigger an exception if the internal getter of the Revit element fails (e.g. for deleted or exploded CADs).
  - *Fix:* Replaced `hasattr` checks with explicit `try/except` blocks to safely access properties.
  - *Location:* `dynamo_extractor_node.py` at `cad_name = imp_type.Name`.

- **OpenPyXL Merged Cells Text Visibility Bug:**
  - *Symptom:* The Group Header text (row with gray background) was completely invisible in Excel, despite being present in the file.
  - *Debugging Steps:* Wrote a standalone Python script (`test_openpyxl.py`) to reproduce the bug outside Dynamo. Discovered that merging cells (`ws.merge_cells`) and then applying a background fill to all cells in the range causes Excel rendering issues.
  - *Fix:* Removed `merge_cells` completely. Used `Alignment(horizontal="centerContinuous", vertical="center")` which centers text across multiple cells without physically merging them, bypassing the Excel bug.
  - *Location:* `dynamo_extractor_node.py` (Group Header formatting section).

- **OpenPyXL Excel File Corruption (Formula Bug):**
  - *Symptom:* Excel displays a warning: "We found a problem with some content in 'ModelHealth_Report.xlsx'. Do you want us to try to recover as much as we can?"
  - *Thought Process & Debugging Steps:* 
    1. Initially suspected that the newly added `ws.row_dimensions.group` or `ws.sheet_properties.outlinePr` might be corrupting the file structure when appending data.
    2. Realized that since `.xlsx` files are just ZIP archives of XML files, the best way to find the corruption is to look directly at the underlying XML.
    3. Wrote a quick Python command (`import zipfile... z.read('xl/worksheets/sheet1.xml')`) to unzip and read the raw XML of the generated Excel file.
    4. Upon inspecting the XML, noticed an anomaly in the cell containing the Group Header: `<c r="A2" s="2"><f>== รอบการอัปเดต... ===</f><v></v></c>`.
    5. The `<f>` tag in SpreadsheetML stands for "Formula". This was the "Aha!" moment.
  - *Root Cause:* The string `=== 🕒 รอบการอัปเดต: ... ===` started with `=`. The `openpyxl` library automatically infers that any string starting with `=` is an Excel formula and forcefully writes it as one. Since `== รอบการ...` is invalid Excel syntax, it corrupted the XML and caused Excel to throw a recovery warning.
  - *Fix:* Changed the header prefix from `===` to `---` so it is treated as a plain string value (`<v>`), not a formula (`<f>`).
  - *Location:* `dynamo_extractor_node.py` (`group_header_text`).

- **PermissionError Indentation Syntax Error:**
  - *Symptom:* `IndentationError : ('unexpected unindent', ...)`
  - *Root Cause:* During the addition of the `PermissionError` catch block, the indentation was incorrectly placed at 8 spaces instead of 4, leaving the parent `try` block without an `except`, causing a syntax error.
  - *Fix:* Realigned the `except` block to match the original 4-space indentation.

### Changed
- Improved error handling for `PermissionError` when the output Excel file is currently open in another program, providing a user-friendly message in Thai instead of a traceback.
