---
title: "How to kill ollama server"
source: "https://waylonwalker.com/how-to-kill-ollama-server/"
author:
  - "[[@_waylonwalker]]"
published:
created: 2025-02-02
description: "I recently updated [ollama](https://ollama.com/), and it now installs a systemdservice that I was not expecting.  Seems like a great option, but I hadn&#x27;t"
tags:
  - "clippings"
---
I recently updated [ollama](https://ollama.com/), and it now installs a systemd service that I was not expecting. Seems like a great option, but I hadn't expeted this and I was able to kill it previously. It was using up gpu, and I do other things on my machine with a gpu. I tried pkill, kill, and everything, it was still coming back.

> No matter what it comes back

```
        
# stop it
systemctl stop ollama.service

# disable it if you want
systemctl disable ollama.service

# confirm its status
systemctl status ollama.service
```

You can confirm this with the following command.

```
        
# checking running processes
ps aux | grep ollama
pgrep ollama

# checking gpu processes
gpustat --show-cmd --show-pid
```

Next time you want to start you can do it as before with `ollama serve`.