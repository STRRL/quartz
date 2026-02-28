---
title: "Latency Profiling and Optimization"
tags: [reference]
created: 2026-02-17
---

https://x.com/vivekgalatage/status/2010905882968932738

https://www.youtube.com/watch?v=lv03NAT4Mwc&feature=youtu.be

45 min video from Dmitry Vyukov @ Google

### Takeaway


- Understand the problem
- `perf` tool
	- active cpu (throughput)
	- latency / wall clock 
	- parallelism
- CPU time sampling vs Wall clock sampling
	- focus: throughput vs latency
- parallelism histogram
	- find parallelization opportunities
- ![[Pasted image 20260228140851.png]]
- 