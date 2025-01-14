---
title: Linux.Network.NetstatEnriched
hidden: true
tags: [Client Artifact]
---

Report network connections, and enrich with process information.


```yaml
name: Linux.Network.NetstatEnriched
description: |
  Report network connections, and enrich with process information.

type: CLIENT

precondition:
  SELECT OS From info() where OS = 'linux'

parameters:
  - name: IPRegex
    description: "regex search over IP address fields."
    default:  "."
    type: regex
  - name: PortRegex
    description: "regex search over port fields."
    default: "."
    type: regex
  - name: ProcessNameRegex
    description: "regex search over source process name"
    default: "."
    type: regex
  - name: UsernameRegex
    description: "regex search over source process user context"
    default: "."
    type: regex
  - name: ConnectionStatusRegex
    description: "regex search over connection status"
    default: "LISTEN|ESTABLISHED"
    type: regex
  - name: ProcessPathRegex
    description: "regex search over source process path"
    default: "."
    type: regex
  - name: CommandLineRegex
    description: "regex search over source process commandline"
    default: "."
    type: regex
  - name: CallChainRegex
    description: "regex search over the process callchain"
    default: "."
    type: regex


sources:
  - query: |
      SELECT Laddr.IP AS Laddr,
             Laddr.Port AS Lport,
             Raddr.IP AS Raddr,
             Raddr.Port AS Rport,
             Pid,
             Status,
             process_tracker_get(id=Pid).Data AS ProcInfo,
             join(array=process_tracker_callchain(id=Pid).Data.Name, sep=" -> ") AS CallChain,
             process_tracker_tree(id=Pid) AS ChildrenTree
      FROM connections()
      WHERE Status =~ ConnectionStatusRegex
       AND  Raddr =~ IPRegex
       AND  ( Lport =~ PortRegex OR Rport =~ PortRegex )
       AND ProcInfo.Name =~ ProcessNameRegex
       AND ProcInfo.Username =~ UsernameRegex
       AND ProcInfo.Exe =~ ProcessPathRegex
       AND ProcInfo.CommandLine =~ CommandLineRegex
       AND CallChain =~ CallChainRegex

column_types:
  - name: ChildrenTree
    type: tree

```
