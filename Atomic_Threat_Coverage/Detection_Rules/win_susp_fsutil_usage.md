| Title                | Fsutil suspicious invocation                                                                                                                                                 |
|:---------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Description          | Detects suspicious parameters of fsutil (deleting USN journal, configuring it with small size..). Might be used by ransomwares during the attack (seen by NotPetya and others)                                                                                                                                           |
| ATT&amp;CK Tactic    |   This Detection Rule wasn't mapped to ATT&amp;CK Tactic yet  |
| ATT&amp;CK Technique |  This Detection Rule wasn't mapped to ATT&amp;CK Technique yet  |
| Data Needed          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| Enrichment           |  Data for this Detection Rule doesn't require any Enrichments.  |
| Trigger              |  There is no documented Trigger for this Detection Rule yet  |
| Severity Level       | high |
| False Positives      | <ul><li>Admin activity</li><li>Scripts and administrative tools used in the monitored environment</li></ul>  |
| Development Status   |  Development Status wasn't defined for this Detection Rule yet  |
| References           | <ul><li>[https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil-usn](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil-usn)</li></ul>  |
| Author               | Ecco |


## Detection Rules

### Sigma rule

```
title: Fsutil suspicious invocation
id: add64136-62e5-48ea-807e-88638d02df1e
description: Detects suspicious parameters of fsutil (deleting USN journal, configuring it with small size..). Might be used by ransomwares during the attack (seen
    by NotPetya and others)
author: Ecco
date: 2019/09/26
level: high
references:
    - https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil-usn
logsource:
    category: process_creation
    product: windows
detection:
    binary_1:
        Image: '*\fsutil.exe'
    binary_2:
        OriginalFileName: 'fsutil.exe'
    selection:
        CommandLine: 
            - '* deletejournal *'  # usn deletejournal ==> generally ransomware or attacker
            - '* createjournal *'  # usn createjournal ==> can modify config to set it to a tiny size
 
    condition: (1 of binary_*) and selection
    
falsepositives:
    - Admin activity
    - Scripts and administrative tools used in the monitored environment

```





### splunk
    
```
((Image="*\\\\fsutil.exe" OR OriginalFileName="fsutil.exe") (CommandLine="* deletejournal *" OR CommandLine="* createjournal *"))
```






### Saved Search for Splunk

```
Generated with Sigma2SplunkAlert
[Fsutil suspicious invocation]
action.email = 1
action.email.subject.alert = Splunk Alert: $name$
action.email.to = test@test.de
action.email.message.alert = Splunk Alert $name$ triggered \
List of interesting fields:   \
title: Fsutil suspicious invocation status:  \
description: Detects suspicious parameters of fsutil (deleting USN journal, configuring it with small size..). Might be used by ransomwares during the attack (seen by NotPetya and others) \
references: ['https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil-usn'] \
tags:  \
author: Ecco \
date:  \
falsepositives: ['Admin activity', 'Scripts and administrative tools used in the monitored environment'] \
level: high
action.email.useNSSubject = 1
alert.severity = 1
alert.suppress = 0
alert.track = 1
alert.expires = 24h
counttype = number of events
cron_schedule = */10 * * * *
allow_skew = 50%
schedule_window = auto
description = Detects suspicious parameters of fsutil (deleting USN journal, configuring it with small size..). Might be used by ransomwares during the attack (seen by NotPetya and others)
dispatch.earliest_time = -10m
dispatch.latest_time = now
enableSched = 1
quantity = 0
relation = greater than
request.ui_dispatch_app = sigma_hunting_app
request.ui_dispatch_view = search
search = ((Image="*\\fsutil.exe" OR OriginalFileName="fsutil.exe") (CommandLine="* deletejournal *" OR CommandLine="* createjournal *")) | stats values(*) AS * by _time | search NOT [| inputlookup Fsutil_suspicious_invocation_whitelist.csv]
```