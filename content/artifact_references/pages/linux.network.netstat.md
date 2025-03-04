---
title: Linux.Network.Netstat
hidden: true
tags: [Client Artifact]
---

This artifact will parse /proc and reveal information
about current network connections.


```yaml
name: Linux.Network.Netstat
description: |
   This artifact will parse /proc and reveal information
   about current network connections.

type: CLIENT

parameters:
   - name: StateRegex
     type: regex
     default: "Listening|Established"
     description: Only show these states

sources:
  - precondition:
      SELECT OS From info() where OS = 'linux'

    query: |
        -- Break down the address of the form 0100007F:22B9
        LET _X(x) = parse_string_with_regex(string=addr,
          regex="(..)(..)(..)(..):(....)")

        -- Unroll hex encoded IPv4 address into more usual form.
        LET ParseAddress(addr) = dict(
            IP=format(format="%d.%d.%d.%d", args=[
               int(int="0x" + _X(x=addr).g4),
               int(int="0x" + _X(x=addr).g3),
               int(int="0x" + _X(x=addr).g2),
               int(int="0x" + _X(x=addr).g1)]),
            Port=int(int="0x" + _X(x=addr).g5)
        )

        -- https://elixir.bootlin.com/linux/latest/source/include/net/tcp_states.h#L14
        LET StateLookup <= dict(
           `01`="Established",
           `02`="Syn Sent",
           `06`="Time Wait", -- No owner process
           `0A`="Listening"
        )

        -- Enumerate all the sockets and cache them in memory for
        -- reverse lookup. The following is basically lsof.
        LET X = SELECT split(string=FullPath, sep="/")[2] AS Pid,
               Data.Link AS Filename,
               parse_string_with_regex(
                  string=Data.Link,
                  regex="(?P<Type>socket|pipe):\\[(?P<inode>[0-9]+)\\]") AS Details
        FROM glob(globs="/proc/*/fd/*")

        LET AllSockets <= SELECT atoi(string=Pid) AS Pid,
               read_file(filename="/proc/" + Pid + "/comm") AS Command,
               read_file(filename="/proc/" + Pid + "/cmdline") AS CommandLine,
               Filename,
               Details.Type AS Type,
               Details.inode AS Inode
        FROM X
        WHERE Type =~ "socket"

        -- Parse the TCP table and refer back to the socket
        -- so we can print process info.
        SELECT inode, get(item=StateLookup, field=st) AS State, uid, {
                  SELECT * FROM AllSockets
                  WHERE Inode=inode
                  LIMIT 1
               } AS ProcessInfo,
               ParseAddress(addr=local_address) AS LocalAddr,
               ParseAddress(addr=rem_address) AS RemoteAddr
        FROM split_records(
             columns=["_", "sl","local_address", "rem_address", "st", "queues", "tr_tm_when",
                      "retransmit", "uid", "timeout", "inode"],
             filenames="/proc/net/tcp",
             regex=" +")
        WHERE sl =~ ":"  -- Remove header row
          AND State =~ StateRegex

```
