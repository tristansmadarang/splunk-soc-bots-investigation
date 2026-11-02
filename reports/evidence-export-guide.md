# Reproduce and export the evidence

The live Splunk instance was accessible during this investigation. These instructions support independent review and acquisition of native exports; they are not a claim that native exports were collected here.

1. Open Search & Reporting in your authorized lab. Choose **All time** and **No Event Sampling**.
2. Run each of the searches below from the corresponding `.spl` file. Wait for **Complete**. Use the export/download control above the result table, choose CSV for tables or JSON/XML for `_raw`, and export **all results**, not only the displayed page. Labels may vary by Splunk version.
3. Save unredacted exports in a private folder outside the Git repository. Record the exact query, time bounds, search job ID, completion status, result count, export time, display timezone and SHA-256 checksum.

| Search | Suggested export name | Expected result in the reviewed lab |
|---|---|---|
| `queries/01-inventory.spl` | `01-inventory.csv` | 22 sourcetype/index rows |
| `queries/03-upload.spl` | `03-upload.csv` | One successful upload transaction |
| `queries/04-webshell-sample.spl` | `04-webshell-sample.csv` | 12-row sample; `head 12` intentionally limits it |
| `queries/05-execution-window-http.spl` | `05-execution-window-http.csv` | Eight HTTP events |
| `queries/06-sysmon-raw.spl` | `06-sysmon-raw.json` | Five matching Sysmon Event ID 1 records |
| `queries/07-process-lineage.spl` | `07-process-lineage.csv` | Five rows |
| `queries/08-security-corroboration.spl` | `08-security-corroboration.json` | Five-row mixed Windows sample |

For full upload source preservation, run this additional search and export JSON with `_raw`:

```spl
index=botsv1 earliest=0 latest=now sourcetype=stream:http "3791.exe"
| table _time host source sourcetype _raw
```

This full-source export query is provided for follow-up acquisition and was not run in this exact form during preparation. Two HTTP records containing the filename were inspected through other searches.

Capture screenshots with the complete query, time range and finished results visible. Remove unrelated tabs, the lab address, session cookies and personal information from the publication copy. Keep the original screenshot privately. If the full result table does not fit, capture a second frame and retain the CSV as the authoritative tabular copy.

Check that JSON/XML field values match your table before using them in a report. A blank HTTP status is unknown. A successful upload response supports an upload finding; executable launch requires endpoint corroboration. Review every proposed public file before publication.
