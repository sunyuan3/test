## 1. Overview of Docker resource management interfaces  
| Option                     |  Description                                                                                                                                    |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `-m`, `--memory=""`        | Memory limit (format: `<number>[<unit>]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`. Minimum is 4M.               |
| `--memory-swap=""`         | Total memory limit (memory + swap, format: `<number>[<unit>]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`.         |
| `--memory-reservation=""`  | Memory soft limit (format: `<number>[<unit>]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`.                         |
| `--kernel-memory=""`       | Kernel memory limit (format: `<number>[<unit>]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`. Minimum is 4M.        |
| `--oom-kill-disable=false` | Whether to disable OOM Killer for the container or not.                                                                                         |
| `--memory-swappiness=""`   | Tune a container's memory swappiness behavior. Accepts an integer between 0 and 100.                                                            |
| `-c`, `--cpu-shares=0`     | CPU shares (relative weight)                                                                                                                    |
| `--cpu-period=0`           | Limit the CPU CFS (Completely Fair Scheduler) period                                                                                            |
| `--cpu-quota=0`            | Limit the CPU CFS (Completely Fair Scheduler) quota                                                                                             |
| `--cpuset-cpus=""`         | CPUs in which to allow execution (0-3, 0,1)                                                                                                     |
| `--cpuset-mems=""`         | Memory nodes (MEMs) in which to allow execution (0-3, 0,1). Only effective on NUMA systems.                                                     |
| `--blkio-weight=0`         | Block IO weight (relative weight) accepts a weight value between 10 and 1000.                                                                   |
| `--blkio-weight-device=""` | Block IO weight (relative device weight, format: `DEVICE_NAME:WEIGHT`)                                                                          |
| `--device-read-bps=""`     | Limit read rate from a device (format: `<device-path>:<number>[<unit>]`). Number is a positive integer. Unit can be one of `kb`, `mb`, or `gb`. |
| `--device-write-bps=""`    | Limit write rate to a device (format: `<device-path>:<number>[<unit>]`). Number is a positive integer. Unit can be one of `kb`, `mb`, or `gb`.  |
| `--device-read-iops="" `   | Limit read rate (IO per second) from a device (format: `<device-path>:<number>`). Number is a positive integer.                                 |
| `--device-write-iops="" `  | Limit write rate (IO per second) to a device (format: `<device-path>:<number>`). Number is a positive integer.                                  |

## 2. Introduction of Docker resource management principle——Cgroups subsystems
Cgroups is the abbreviation of control groups, which is a machanism that the linux kernel provides to limit，record and isolate the physical resources(eg. CPU, memory and IO) used by process groups. Cgroups is originally brought forward by google engineers, and then be integrated into the linux kernel. The allocation and management of resources are implemented by cgroups subsystems. There are seven cgroups systems, respectively are cpuset, cpu, cpuacct, blkio, devices, freezer, memory. The following describes four cgroups subsystems which are related to Docker resource management interfaces.

2.1 memory -- This subsystem is to limit the memory upper limit.<br>

| 子系统常用cgroups接口 | 描述 | 对应的docker接口 |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| cgroup/memory/memory.limit_in_bytes | 设定内存上限，单位是字节，也可以使用k/K、m/M或者g/G表示要设置数值的单位。| -m, --memory="" |
| cgroup/memory/memory.memsw.limit_in_bytes |设定内存加上交换分区的使用总量。通过设置这个值，可以防止进程把交换分区用光。| --memory-swap="" |
| cgroup/memory/memory.soft_limit_in_bytes |设定内存限制，但这个限制并不会阻止进程使用超过限额的内存，只是在系统内存不足时，会优先回收超过限额的进程占用的内存，使之向限定值靠拢。| --memory-reservation="" |
| cgroup/memory/memory.kmem.limit_in_bytes |设定内核内存上限。| --kernel-memory="" |
| cgroup/memory/memory.oom_control |如果设置为0，那么在内存使用量超过上限时，系统不会杀死进程，而是阻塞进程直到有内存被释放可供使用时，另一方面，系统会向用户态发送事件通知，用户态的监控程序可以根据该事件来做相应的处理，例如提高内存上限等。| --oom-kill-disable="" |
| cgroup/memory/memory.swappiness |控制内核使用交换分区的倾向。取值范围是0至100之间的整数（包含0和100）。值越小，越倾向使用物理内存。| --memory-swappiness="" |
