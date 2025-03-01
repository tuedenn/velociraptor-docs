---
title: Windows.Analysis.EvidenceOfExecution
hidden: true
tags: [Client Artifact]
---

In many investigations it is useful to find evidence of program execution.

This artifact combines the findings of several other collectors into
an overview of all program execution artifacts. The associated
report walks the user through the analysis of the findings.


```yaml
name: Windows.Analysis.EvidenceOfExecution
description: |
  In many investigations it is useful to find evidence of program execution.

  This artifact combines the findings of several other collectors into
  an overview of all program execution artifacts. The associated
  report walks the user through the analysis of the findings.

sources:
  - name: UserAssist
    query: |
      SELECT * FROM Artifact.Windows.Registry.UserAssist()

  - name: Amcache
    query: |
      SELECT * FROM Artifact.Windows.Detection.Amcache()

  - name: Timeline
    query: |
      SELECT * FROM Artifact.Windows.Forensics.Timeline()

  - name: ShimCache
    query: |
      SELECT * FROM Artifact.Windows.Registry.AppCompatCache()

  - name: Prefetch
    query: |
      SELECT * FROM Artifact.Windows.Forensics.Prefetch()

  - name: Recent Apps
    query: |
      SELECT * FROM Artifact.Windows.Forensics.RecentApps()

```
