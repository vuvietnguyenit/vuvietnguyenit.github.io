---
layout: post
title: GPU tracing and monitoring at low-level
subtitle: The intelligent way to collect metrics and monitor GPU workloads if you’re an SRE.
tags: [sre, system, ebpf, linux, tracing, gpu]
author: VuNguyen
---

Recently, I tried to find some ways to monitor workload of GPU in Linux server to help detect issues when training AI models. I went to internet and find some opensource to help me do that because I don’t want to take a lot of time to build something from scratch 🙂. I’m lazy and always want to use my brain that think about anything beautiful, colorful in the world, not from computer 😌

But almost I can’t find any opensource this can fit with my case, I found an exporter that might help me a little bit: [https://github.com/utkuozdemir/nvidia_gpu_exporter](https://github.com/utkuozdemir/nvidia_gpu_exporter). I read this opensource very carefully and realized that the implementation of code is basic (call cmd of nvidia-smi, get result and parse the result in Go code), that implementation works fine but not native and it can’t help me in trace problems in low-level function (malloc(), free() call, performance of train processes, bottlenecks or something like that).

Because of all this, I very curious with how tools like nvidia-smi, nvtop can trace process’s resource usage. And when I checked it, both of them mapped with a shared library libnvidia-ml.so

```bash
root@gpu1:~# ps aux | grep nvidia-sm                                                            
root      623505  4.2  0.0 313864 18320 pts/75   Sl+  10:34   0:12 nvidia-smi pmon -s u                                                                                                          
root      625262  0.0  0.0   9276  1964 pts/35   S+   10:39   0:00 grep --color=auto nvidia-sm                                                                                                   
root@gpu1:~# ps aux | grep nvtop                                                                                                                                                                 
root      625267  0.0  0.0   9144  2064 pts/35   S+   10:39   0:00 grep --color=auto nvtop                                                                                                       
root     3667564  2.0  0.0 185120 22220 pts/69   Sl+  Aug07  51:54 nvtop
root@gpu1:~# cat /proc/623505/maps | grep libnvidia
7b4a27200000-7b4a273e7000 r-xp 00000000 00:1c 356260                     /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.570.133.07 # here
root@gpu1:~# cat /proc/3667564/maps | grep libnvidia
7eb296600000-7eb2967e7000 r-xp 00000000 00:1c 356260                     /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.570.133.07
```

When tools like nvidia-smi be use, it load [libnvidia-ml.so] first and then make the calls to API functions written by C to query and control NVIDIA GPUs via this shared library under the hood.

We can check the functions provided by shared library

```bash
root@gpu1:~# nm -D --defined-only /usr/lib/x86_64-linux-gnu/libnvidia-ml.so | head
0000000000069dc0 T nvmlComputeInstanceDestroy
000000000006a390 T nvmlComputeInstanceGetInfo
000000000006a5a0 T nvmlComputeInstanceGetInfo_v2
0000000000085c20 T nvmlDeviceClearAccountingPids
00000000000515f0 T nvmlDeviceClearCpuAffinity
000000000007b260 T nvmlDeviceClearEccErrorCounts
0000000000064240 T nvmlDeviceClearFieldValues
00000000000683f0 T nvmlDeviceCreateGpuInstance
0000000000068620 T nvmlDeviceCreateGpuInstanceWithPlacement
0000000000063e60 T nvmlDeviceDiscoverGpus
```

And reference it with its documentation [https://docs.nvidia.com/deploy/nvml-api/](https://docs.nvidia.com/deploy/nvml-api/) to see how the function is used.

Additionally, when jobs run on the GPU server, they make calls to the CUDA Runtime/Driver API to use GPU resources (memory, CPU, etc.) via the shared libraries `libcuda.so` and `libcudart.so`. You can list the functions defined in the library using `nm` as shown above, and refer to its documentation on the internet.

## eBPF comes into play

I was thinking about write an eBPF program that can help me do everything above like this:

```c
SEC("uprobe/cuMemAlloc")
int trace_cu_mem_alloc_entry(struct pt_regs *ctx) {
  __u64 pid_tgid = bpf_get_current_pid_tgid();
  __u32 pid = pid_tgid >> 32;
  CUdeviceptr *dptr_ptr = (CUdeviceptr *)PT_REGS_PARM1(ctx);
  __u64 size = PT_REGS_PARM2(ctx);
  if (bpf_map_lookup_elem(&inflight, &pid_tgid)) {
    bpf_printk("cuMemAlloc entry: pid_tgid=%llu already has inflight alloc",
               pid_tgid);
    return 0;
  }
  bpf_map_update_elem(
      &inflight, &pid_tgid,
      &(struct alloc_info_t){.size = size, .dptr_addr = (CUdeviceptr)dptr_ptr},
      BPF_ANY);
  bpf_printk("cuMemAlloc entry: pid_tgid=%llu, size=%llu, ptr=0x%llx", pid_tgid,
             size, dptr_ptr);
  return 0;
}
```

This implementation helps me trace anything at the low-level function level and lets me gather all the metrics I need with minimal resource usage on the server. However, it requires C for the eBPF program and another “frontend” language (Go, Python), which is a big challenge. But it can give me everything operation I want. As far as I know, there’s no open-source project that does this yet, so I want to contribute it as the first program to achieve that 😄.

That’s genius!
