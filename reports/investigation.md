# Investigation report: Joomla-hosted code execution

**Case:** BOTS-v1-Joomla-RCE · **Analysis date:** November 2, 2026 · **Environment:** training dataset in a home lab

## Executive assessment

Execution of `C:\inetpub\wwwroot\joomla\3791.exe` on `we1149srv` is confirmed by Sysmon process creation and corroborating Windows Security events. At 21:56:18.142 UTC on August 10, 2016, `php-cgi.exe` spawned `cmd.exe`, which launched that executable under `NT AUTHORITY\IUSR`. HTTP telemetry records a successful upload response for `3791.exe` and `agent.php` approximately three and a half minutes earlier. Requests from `23.22.63.114` to `/joomla/agent.php` occurred immediately before execution.

These observations strongly support web-mediated remote code execution. They do not independently identify the operator, establish a particular CVE, or reveal the exact command-bearing request payload. Proposed severity for triage is **High** because a web-facing application process launched an uploaded executable. No production impact is claimed.

## Scope and method

The investigation used the actual `botsv1` index in the user's Splunk 9.4.0 web interface. Discovery covered all indexed time, then narrowed to the August 10 activity. Primary sources were `stream:http`, `XmlWinEventLog`, and `WinEventLog`. Other sourcetypes were inventoried but are not treated as corroboration without event-level review.

Completed searches were inspected through the Windows browser interface. Evidence in this repository consists of selected, manually preserved UI excerpts and CSV tables derived from displayed source events. These are not full native Splunk exports or a forensic image. Raw local working captures were excluded from the publication package because some HTTP events contain session cookies.

## Timeline

All times below are UTC on **2016-08-10**. HTTP `_time` generally tracks the transaction end in the reviewed examples; the upload's explicit `timestamp` and `endtime` are shown separately. Sysmon times use the XML `UtcTime`, not a timezone assumption about the browser.

| Time UTC | Observation | Interpretation / evidence |
|---|---|---|
| 21:37:01.591894–21:37:01.592646 | `40.80.148.42` sent an encoded traversal request for `winnt/system32/cmd.exe` with `/c dir` to `192.168.250.70`; response 404. Headers identify Acunetix WVS/10.0. | Command-execution probe; this request does not demonstrate successful execution. E01 |
| 21:50:31.324 | HTTP POST from `40.80.148.42` to Joomla installer endpoint; form begins with URL-encoded PHP in `install_package`; response 303. | Suspicious package submission; the redirect is not proof that the package ran. E02 |
| 21:52:45.437445–21:52:47.035555 | Upload transaction with `c_ip=40.80.148.42`, destination `192.168.250.70`, filenames `3791.exe` and `agent.php`; response body reports `Upload successful!` and `success:true`. | The application reported successful upload. The source field is `c_ip` because `src_ip` was absent in this event. E03 |
| 21:52:48.889 | POST from `40.80.148.42` to `/joomla/administrator/index.php`, `option=com_extplorer`, `action=getdircontents`; status 200. | Directory-listing activity following upload. E03 |
| 21:55:22.673 | First displayed GET in the reviewed early-request sample from `23.22.63.114` to `/joomla/agent.php`; status 200. | Access to the uploaded PHP path; not an assertion of the first request across every possible source. E02 |
| 21:56:17.028625–21:56:18.138512 | Eight HTTP events in the 21:56:00–21:56:30 window mention `agent.php`; all display source `23.22.63.114` and destination `192.168.250.70`. Seven display status 200; the final event has no displayed status. | Tight temporal correlation with execution; absent status is unknown, not success or failure. E04 |
| 21:56:18.142 | Sysmon record 428881: PHP PID 1208 → command shell PID 2896, command `cmd.exe /c "3791.exe 2>&1"`. | Application-originated shell execution. E05 |
| 21:56:18.142 | Sysmon record 428908: command shell PID 2896 → uploaded executable PID 3880, user `NT AUTHORITY\IUSR`. | Confirmed executable launch. Security 4688 records 28447 and 28449 corroborate shell/file creation. E05, E06 |
| 21:58:23.896 | Sysmon record 431508: `3791.exe` PID 3880 → `cmd.exe` PID 3620. | Subsequent shell launched by the executable. E05 |
| 22:08:13.902 | Sysmon record 443590: same executable PID 3880 → `cmd.exe` PID 3424. | Further shell activity. E05 |

## Process evidence

```text
php-cgi.exe    PID 1208
└── cmd.exe    PID 2896  [Sysmon record 428881]
    ├── conhost.exe PID 3680 [record 428894]
    └── 3791.exe PID 3880    [record 428908]
        ├── cmd.exe PID 3620 [record 431508]
        └── cmd.exe PID 3424 [record 443590]
```

The command shell's ProcessGuid `{E500B0EA-A302-57AB-0000-00102E63C301}` matches the executable's ParentProcessGuid. The executable's ProcessGuid `{E500B0EA-A302-57AB-0000-00108D65C301}` appears as the parent GUID of the later shells. This is stronger linkage than PID proximity alone.

Executable SHA-256 from **Event ID 1, record 428908**:

`EC78C938D8453739CA2A370B9C275971EC46CAF6E479DE2B2D04E97CC47FA45D`

Executable MD5: `AAE3F5A29935E6ABCC2C2754D12A9AF0`.

The hashes in Sysmon Event ID 7 describe loaded modules. They were deliberately not used as the executable hash. Likewise, the `High` integrity value and Security token boilerplate are not sufficient evidence of a separate privilege-escalation step.

## Findings and confidence

| Finding | Confidence | Basis and qualification |
|---|---|---|
| Uploaded executable ran on `we1149srv` | High | Sysmon Event ID 1 plus Security 4688, matching path, time, and PIDs |
| Execution originated through PHP | High | `php-cgi.exe` parent of the launching shell; matching parent/child process GUIDs |
| Upload and execution form one compromise sequence | High | Matching filename, Joomla directory, successful upload response, and endpoint execution |
| `agent.php` was involved in the remote execution path | Moderate to high | Uploaded PHP file, requests immediately before launch, and PHP ancestry; exact command payload not recovered |
| Both source IPs were operated by one actor | Unresolved | Two addresses appear in related activity, but no operator identity evidence was collected |
| Persistence, exfiltration, lateral movement, or a named malware family | Not established | Outside the evidence reviewed; no negative assurance is implied |

## Indicators for retrospective hunting

| Indicator | Observed role | Handling |
|---|---|---|
| `40.80.148.42` | Probe, installer submission, upload client | Historical BOTS indicator; do not assume present-day malicious ownership |
| `23.22.63.114` | Requests to the uploaded PHP path near execution | Same historical-use limitation |
| `192.168.250.70` | HTTP destination in the dataset | Training victim address, not the user's Splunk server |
| `we1149srv` | Endpoint recording execution | Correlated with web traffic through the observed application/file path and timing; no independent asset inventory was obtained |
| `imreallynotbatman[.]com` | Host header of reviewed probes | Defanged training indicator |
| `/joomla/agent.php` | Uploaded PHP path subsequently requested | Investigate content and execution context before classifying on filename alone |
| `C:\inetpub\wwwroot\joomla\3791.exe` | Executed uploaded file | Correlate with the Event ID 1 hash above |

## Analyst handoff and proposed response

In a production setting, escalate the confirmed application-originated executable launch to incident response. Preserve the web root, application logs, process telemetry, relevant network captures, and memory if operationally appropriate. Coordinate isolation of the affected server and restrict the exposed application while maintaining evidence. Investigate Joomla and extension access, authentication history, file changes, and credentials available to the application identity. Scope the process GUID and hash across endpoints and network telemetry. Restore from a trusted state and correct the validated entry condition before returning the service.

These are recommendations. No isolation, deletion, blocking, password change, or remediation was performed in this lab.

## Limitations and next questions

- The exact command-bearing web request, any encoding key, and the contents of `agent.php` were not reconstructed. A POST-only search of the PHP URI returned zero events; the observed requests were GETs. That zero result does not rule out headers, cookies, or other command channels.
- Initial credential acquisition and a particular exploitation vulnerability were not established. Do not label this as a specific CVE or successful brute force without further evidence.
- The early HTTP search used `head 12`; it is a sample. The execution-time search returned eight events in its explicitly bounded window. The process-creation search returned five matching records.
- XML UTC timestamps align with the HTTP UTC timestamps. The Security raw rendered time is 02:56:18 PM while indexed `_time` is 21:56:18; the report uses XML UTC for chronology and does not claim the Security rendering was UTC.
- The Ubuntu OS and lab setup were reported by the user. Host OS version, ingestion procedure, installed add-ons, and dataset checksum were not independently verified.
- A seed filename (`3791.exe`) was tested as a hypothesis during assisted investigation; it was then verified in actual upload and process records. This was not a fully blind discovery exercise.
