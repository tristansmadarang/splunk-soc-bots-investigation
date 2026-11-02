# Evidence register and provenance

Evidence was read from completed searches in the user's live Splunk interface on November 2, 2026. Publication material contains selected training-data excerpts. It excludes session cookies, credentials, full HTTP headers, malware binaries, and unrelated browser content.

| ID | Artifact | Source and use |
|---|---|---|
| E01 | [selected-http-observations.csv](selected-http-observations.csv) | Selected probe observation from `stream:http`; 404 and Acunetix header were inspected in the source event |
| E02 | Same CSV | Early `agent.php` match sample; installer submission and later GET requests |
| E03 | [upload-observation.txt](upload-observation.txt), [upload screenshot](../screenshots/02-upload-response.png) | Selected upload metadata and success response, plus subsequent directory query |
| E04 | [execution-http-observations.csv](execution-http-observations.csv) | Eight displayed HTTP events in an explicit 30-second UTC window |
| E05 | [sysmon-process-events.xml](sysmon-process-events.xml), [process-lineage.csv](process-lineage.csv), [lineage screenshot](../screenshots/01-process-lineage.png) | Five complete XML Event ID 1 records recovered from displayed UI text; CSV parsed from those XML records |
| E06 | [security-4688-excerpts.txt](security-4688-excerpts.txt) | Selected Windows Security process-creation records 28447 and 28449 |

The CSVs for HTTP are manually transcribed selections, not native Splunk exports. Empty values mean the field was absent from the displayed result. XML formatting is preserved from UI text; these files do not constitute acquisition of the original Windows EVTX files. Screenshot 01 shows the first four rows, with the last partially visible; the CSV/XML preserve all five records. Screenshots were cropped to remove browser chrome and unrelated tabs; result pixels were not altered.

The working UI captures are kept outside this repository. SHA-256 checksums in [SHA256SUMS.txt](SHA256SUMS.txt) provide an integrity baseline for the packaged artifacts after preparation, not proof of original-source authenticity or chain of custody.

Run the queries listed in [queries/README.md](../queries/README.md) to independently reproduce results. For stronger preservation, use Splunk's export control after each completed search to export all results as XML/JSON or CSV including `_raw`, and record the query, time bounds, timezone, search job ID, row count, collection time, and file hash. Keep unredacted exports private.
