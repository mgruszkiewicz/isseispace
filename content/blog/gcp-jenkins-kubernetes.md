---
title: "How to fix issue with Jenkins pipeline on Kubernetes"
date: 2022-08-13T00:24:56+02:00
draft: true
tags: ['gcp', 'ci/cd', 'en']
---
gcp

# Issues

## `Jenkins` doens't have label `sample-app`

## All nodes of label `sample-app` are offline


## jnlp Connection refused

```
- jnlp -- terminated (255)
-----Logs-------------
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
Oct 16, 2022 11:55:58 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
Oct 16, 2022 11:55:58 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://127.0.0.1:8080/]
Oct 16, 2022 11:55:58 AM hudson.remoting.jnlp.Main$CuiListener error
SEVERE: Failed to connect to http://127.0.0.1:8080/tcpSlaveAgentListener/: Connection refused (Connection refused)
java.io.IOException: Failed to connect to http://127.0.0.1:8080/tcpSlaveAgentListener/: Connection refused (Connection refused)
	at org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver.resolve(JnlpAgentEndpointResolver.java:214)
	at hudson.remoting.Engine.innerRun(Engine.java:724)
	at hudson.remoting.Engine.run(Engine.java:540)
Caused by: java.net.ConnectException: Connection refused (Connection refused)
	at java.base/java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.base/java.net.AbstractPlainSocketImpl.doConnect(Unknown Source)
	at java.base/java.net.AbstractPlainSocketImpl.connectToAddress(Unknown Source)
	at java.base/java.net.AbstractPlainSocketImpl.connect(Unknown Source)
	at java.base/java.net.Socket.connect(Unknown Source)
	at java.base/sun.net.NetworkClient.doConnect(Unknown Source)
	at java.base/sun.net.www.http.HttpClient.openServer(Unknown Source)
	at java.base/sun.net.www.http.HttpClient.openServer(Unknown Source)
	at java.base/sun.net.www.http.HttpClient.<init>(Unknown Source)
	at java.base/sun.net.www.http.HttpClient.New(Unknown Source)
	at java.base/sun.net.www.http.HttpClient.New(Unknown Source)
	at java.base/sun.net.www.protocol.http.HttpURLConnection.getNewHttpClient(Unknown Source)
	at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect0(Unknown Source)
	at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect(Unknown Source)
	at java.base/sun.net.www.protocol.http.HttpURLConnection.connect(Unknown Source)
	at org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver.resolve(JnlpAgentEndpointResolver.java:211)
	... 2 more
```

You need to set proper Jenkins URL in settings.
Get pod IP using `kubectl get pod -o wide`