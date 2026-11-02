# Home lab scope

| Component | Status |
|---|---|
| Splunk on an Ubuntu machine | User-reported; OS/version not independently inspected |
| Splunk web access | Verified through the existing signed-in local browser session |
| Splunk version | Search page title displayed 9.4.0 |
| BOTS index | `botsv1`, verified by completed searches |
| Relevant telemetry | `stream:http`, `XmlWinEventLog`, `WinEventLog`, plus inventory results for IIS, Suricata, firewall and other Stream sources |
| GitHub Desktop installed | User-reported; not needed for the local analysis |

```mermaid
flowchart LR
  BOTS[Splunk BOTS v1 training data] --> INDEX[botsv1 index]
  INDEX --> SPL[Search and Reporting]
  SPL --> ANALYSIS[HTTP and endpoint correlation]
  ANALYSIS --> REPORT[Evidence, timeline and analyst report]
```

This is a logical analysis flow, not a verified physical network diagram. Imported BOTS endpoint logs do not imply that those Windows systems are live machines in the home lab. The Splunk server's private address is omitted from the public-facing documentation. No claims are made about forwarder deployment, segmentation, alerting, or ingestion automation.
