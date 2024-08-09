---
title: Deadline
description: Setting up Deadline
icon: material/toy-brick
slug: deadline
tags:
  - deadline
publish: false
---

## Web Service

 <font color=4287ff><b>CMD</b></font>
 
``` shell
"%DEADLINE_PATH%\deadlinewebservice.exe"
```

 <font color=4287ff><b>PowerShell</b></font>
 
```powershell
& "$env:DEADLINE_PATH\deadlinewebservice.exe"
```

Change the listening port with the flag `-port`:

<font color=4287ff><b>CMD</b></font>

```shell
"%DEADLINE_PATH%\deadlinewebservice.exe" -port 8082
```

<font color=4287ff><b>PowerShell</b></font>

``` powershell
& "$env:DEADLINE_PATH\deadlinewebservice.exe" - port 8082
```

Check in your browser that webservice is running:

```
http://<host IP>:<port>
``` 

```
http://192.168.0.104:8081
``` 

> [!warning]
> Watch out for `http` instead of `https`.

## AYON

> [!error]
> **Do not** add a slash at the end of the Deadline addon webservice configuration.

