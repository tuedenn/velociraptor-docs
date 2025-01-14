---
title: Windows.System.SVCHost
hidden: true
tags: [Client Artifact]
---

Typically a windows system will have many svchost.exe
processes. Sometimes attackers name their processes svchost.exe to
try to hide. Typically svchost.exe is spawned by services.exe.

This artifact lists all the processes named svchost.exe and their
parents if the parent is not also named services.exe.


```yaml
name: Windows.System.SVCHost
description: |
  Typically a windows system will have many svchost.exe
  processes. Sometimes attackers name their processes svchost.exe to
  try to hide. Typically svchost.exe is spawned by services.exe.

  This artifact lists all the processes named svchost.exe and their
  parents if the parent is not also named services.exe.

sources:
  - precondition: |
      SELECT OS From info() where OS = 'windows'

    query: |
        // Cache the pslist output in memory.
        LET processes <= SELECT Pid, Ppid, Name, Exe FROM pslist()

        // Get the pids of all procecesses named services.exe
        LET services <= SELECT Pid FROM processes where Name =~ "services.exe"

        // The interesting processes are those which are not spawned by services.exe
        LET suspicious = SELECT Pid As SVCHostPid,
            Ppid As SVCHostPpid,
            Exe as SVCHostExe,
            CommandLine as SVCHostCommandLine
        FROM processes
        WHERE Name =~ "svchost" AND NOT Ppid in services.Pid

        // Now for each such process we display its actual parent.
        SELECT * from foreach(
           row=suspicious,
           query={
              SELECT SVCHostPid, SVCHostPpid, SVCHostExe,
                     SVCHostCommandLine, Name as ParentName,
                     Exe As ParentExe
              FROM processes
              WHERE Pid=SVCHostPpid
          })

```
