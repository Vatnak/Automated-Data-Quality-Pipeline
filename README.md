# Automated Data Quality Pipeline

**Status:** Version 1 is planned. The commands and examples below describe the intended interface.

This Python command-line tool will inspect CSV and Excel data for common quality issues and optionally validate it against a reusable JSON rule profile. It is designed for people who receive tables with different columns or need to check the same kind of file regularly.

## What it will do

- Read one UTF-8, comma-delimited CSV or one worksheet from an Excel `.xlsx` workbook. The first row is the header. Excel defaults to the first worksheet unless you select one.
- Report row and column counts, likely column types, blank cells, identical rows, and numeric observations such as negative values.
- Optionally check rules for required values, data types, number ranges, and date formats.
- Print a terminal summary and write a detailed JSON report. The source file is opened read-only.

## Architecture and stack

The CLI passes the selected file to a reader. Both readers produce a pandas DataFrame, so the same profiler can inspect a survey, a sales export, or a student record sheet. If you supply a rule profile, the validator checks it and adds its results to the report.

| Component | Technology | Responsibility |
| --- | --- | --- |
| CLI | Python and `argparse` | Accept the input path, sheet, rules, and output path. |
| File readers | `pandas` and `openpyxl` | Load CSV and Excel into a shared table format. |
| Profiler and validator | Python and pandas | Produce general findings and check dataset-specific rules. |
| Reporting | Python's `json` and `pathlib` | Write the report and manage file paths. |
| Tests | Python's `unittest` | Verify equivalent CSV and Excel inputs and common error cases. |

## Planned usage

After the CLI is implemented, run these commands from the repository root.

```powershell
python pipeline.py sales.csv --output sales_report.json
python pipeline.py students.xlsx --sheet "Class A" --rules students_rules.json --output students_report.json
```

Without `--rules`, the run reports general observations. With `--rules`, it checks that the input contains the profile's expected headers before validating values. An incompatible profile produces a configuration error and the general profiling results.

### Example rules

A proposed `students_rules.json` profile could look like this.

```json
{
  "columns": {
    "Student ID": { "type": "text", "required": true },
    "Age": { "type": "integer", "min": 0, "max": 120 }
  }
}
```

This example expects both headers, requires a value in `Student ID`, and checks any supplied `Age` value. The same profile can be used with CSV and Excel files that have the same headers and column meanings. A negative age violates this profile. A negative sales amount may represent a refund and can appear as a general observation.

### Example report

The JSON report is expected to include the source filename, sheet if applicable, dataset dimensions, column profiles, observations, rule violations, and configuration errors. A summary might identify a blank `Amount` in data row 2 and a row 4 that repeats row 1. Data row numbers start at 1 after the header.

## Delivery plan

1. Build the CSV and Excel readers and validate headers.
2. Add general profiling, terminal summaries, and the JSON report.
3. Add manually selected JSON profiles and validate the input against them.
4. Verify both formats with equivalent fixtures and test two datasets with different headers.

Later milestones include approved cleaning rules and export, scheduled folder processing, and automatic rule matching for recurring data.

[Project plan and architecture diagram](https://docs.google.com/document/d/1Lrdn6-PaKy1botq-L3WtjsTnjI9XErnzhKG28Ge2YzA/edit)