---
language: ja
title: Compute Engineの Diskを上げる
published: 2022/01/29 
description: ""
slug: "compute-engine-disk"
---

## 背景 / Background

[https://redash.io/help/open-source/setup](https://redash.io/help/open-source/setup "https://redash.io/help/open-source/setup") こちらにそって、re-dashをたち上げていいたら、dockerが立ち上がらなくなっていた。

よく調べると、以下のようなエラーを吐き、diskの容量が満パンで、動かないという感じだった。

I was faced with an error that compute engine, which is setting up redash, would not start.

When I checked it out, it threw up the following error and felt that the disk was full of space and would not work.

## エラーログ / Error log

    root@re-dash:/opt/redash# systemctl status docker.service
    ● docker.service - Docker Application Container Engine
       Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
       Active: failed (Result: exit-code) since Sun 2022/01-30 14:21:36 UTC; 6s ago
         Docs: <https://docs.docker.com>
      Process: 2070 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
     Main PID: 2070 (code=exited, status=1/FAILURE)

    Jan 30 14:21:34 re-dash systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
    Jan 30 14:21:34 re-dash systemd[1]: docker.service: Failed with result 'exit-code'.
    Jan 30 14:21:34 re-dash systemd[1]: Failed to start Docker Application Container Engine.
    Jan 30 14:21:36 re-dash systemd[1]: docker.service: Service hold-off time over, scheduling restart.
    Jan 30 14:21:36 re-dash systemd[1]: docker.service: Scheduled restart job, restart counter is at 3.
    Jan 30 14:21:36 re-dash systemd[1]: Stopped Docker Application Container Engine.
    Jan 30 14:21:36 re-dash systemd[1]: docker.service: Start request repeated too quickly.
    Jan 30 14:21:36 re-dash systemd[1]: docker.service: Failed with result 'exit-code'.
    Jan 30 14:21:36 re-dash systemd[1]: Failed to start Docker Application Container Engine.


    root@re-dash:/opt/redash# journalctl -xe
    Journal file /var/log/journal/8e85609a0aa2b1c3a15d333477b1820e/system.journal is truncated, ignoring file.
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: file '7' write error: No space left on device [v8.32.0 try <http://www.rsyslog.com/e/2027> ]
    Jan 30 14:22:18 re-dash rsyslogd[896]: action 'action 3' (module 'builtin:omfile') message lost, could not be processed. Check for additional error messages before this one. [v8.32.0 try <http://www.rsyslog.com/e/2027> ]

## 解決方法 / Solution

- 以下、2点 URLを確認する。
- 後者の方の手順をやる

[Troubleshooting full disks and disk resizing | Compute Engine Documentation | Google Cloud](https://cloud.google.com/compute/docs/troubleshooting/troubleshooting-disk-full-resize)

[Resize a persistent disk | Compute Engine Documentation | Google Cloud](https://cloud.google.com/compute/docs/disks/resize-persistent-disk)
