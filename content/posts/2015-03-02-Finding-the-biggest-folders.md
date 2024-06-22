---
title: "Finding the biggest folders"
description: "This will helps you, finding the biggest folder, even if it's nested somewhere in your filesystem"
date: 2015-03-02
---

If you have on a server a folder which is to big, and you want to know, which file is the reason for a problem, use `du`
```bash
du -m | sort -n | tail -n 10
```
for finding the 10 biggest folders.    
(repeat this step in the biggest folder again to find the “problem folders”)
