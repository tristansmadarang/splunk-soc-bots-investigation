# SPL query library

Run against authorized BOTS v1 data. All time and no sampling were used. `earliest=0 latest=now` searches historic data; query 05 uses epoch seconds for 21:56:00–21:56:30 UTC on August 10, 2016. `ctime` output follows the Splunk user's display timezone; XML `UtcTime` and HTTP `Z` timestamps are the report's UTC anchors.

| File | Purpose | Validation in this session |
|---|---|---|
| 01-inventory.spl | Discover indexes and sourcetypes | Completed; 22 result rows; first 20 displayed and reviewed |
| 02-http-probes.spl | Review early execution/upload term matches | Completed; first 12 retained; UI raw text capture was bounded, so not every raw event was preserved |
| 03-upload.spl | Isolate successful upload response | Completed; see evidence screenshot and selected transcription |
| 04-webshell-sample.spl | Inspect early mentions of agent.php | Completed; 12-row sample |
| 05-execution-window-http.spl | Correlate HTTP with process start | Completed; eight events |
| 06-sysmon-raw.spl | Obtain process-creation XML | Completed; five events preserved |
| 07-process-lineage.spl | Extract PID/parent lineage | Completed; five rows, matching preserved XML |
| 08-security-corroboration.spl | Inspect mixed Windows matches | Completed; first five events include Security 4688 records and Sysmon module loads |
| 09-follow-up-network.spl | Hunt executable network connections | Proposed follow-up; not run and not a finding |

The single-quoted XML attribute patterns in query 07 reflect the actual stored XML in this dataset. Adapt them if your ingestion changes XML quoting or casing. A match on `3791.exe` includes events where it is a parent process; five matching Event ID 1 rows do not mean five executions of the executable itself.

Query 03 extracts the two known filename strings from the raw event, then removes duplicate matches. The original multipart `content_disposition` values were separately inspected to establish that these strings were uploaded filenames. This is a confirmation query for this case, not a generic upload detector.

For a broader hunt, remove `head` only when you intend to review/export the full result set. Do not infer aggregate counts from a sampled table. Export private evidence with `_raw` and preserve unredacted originals locally. Publish only reviewed excerpts. No query here changes Splunk configuration or executes the suspect files.
