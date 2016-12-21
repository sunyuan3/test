# Resource management of Docker - Cgroups feature supporting Docker

With Docker technology has been accepted by more and more individuals and enterprises, Docker is more widely applied in many aspects. Docker resource management includes the limitation of CPU, memory and IO resources, but for the most part of Docker users, they often just know how without knowing why when they are using Docker resource management interface. This article introduces the cgroups feature which is supporting Docker resource management, and lists Docker resource management interfaces and the corresponding cgroups interfaces, so that the readers can not only know how but also know why about Docker resource management.

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
Cgroups is the abbreviation of control groups, which is a linux feature that limits，accounts for, and isolates the physical resource usage (CPU, memory and disk I/O, etc.) of process groups. Engineers at Google started the work on this feature, and then cgroups functionality was merged into the linux kernel mainline. The allocation and management of resources are implemented by cgroups subsystems. There are seven cgroups systems, respectively are cpuset, cpu, cpuacct, blkio, devices, freezer, and memory. The following describes four cgroups subsystems which are related to Docker resource management interfaces.

2.1 memory -- This subsystem is to control the maximum amount of memory in a cgroup.<br>

| memory cgroup interface | description | the corresponding docker interface |
| ---------------------------------------- | --------------------------------------------------------------------------- | ---------------------------------------- |
| cgroup/memory/memory.limit_in_bytes | sets the maximum amount of user memory, in bytes. And it is possible to use suffixes to represent larger units -- k or K for kilobytes, m or M for megebytes, and g or G for gigabytes. | -m, --memory="" |
| cgroup/memory/memory.memsw.limit_in_bytes | sets the maximum amount of memory and swap space, in bytes. You can prevent a run out of swap partition by setting this value. | --memory-swap="" |
| cgroup/memory/memory.soft_limit_in_bytes | sets soft limit of memory usage. This limitation won't stop processes using excess memory. however, when the system detects memory contention or low memory, cgroups is forced to restrict its consumption to the soft limits. | --memory-reservation="" |
| cgroup/memory/memory.kmem.limit_in_bytes | sets hard limit for kernel memory. | --kernel-memory="" |
| cgroup/memory/memory.oom_control | contains a flag that enables or disables the Out of Memory killer for a cgroup. If disabled(0), tasks that attempt to consume excess memory will not be killed, but be paused until additional memory is freed. In addition, the system will prompt send event notification to user mode, the monitoring program in user mode is able to process it accordingly, such as raising the memory upper limit. | --oom-kill-disable="" |
| cgroup/memory/memory.swappiness | sets the tendency of the kernel to use the swap partition. The value is between 0 and 100(include 0 and 100), and lower value increases the kernel's tendency to use physical memory. | --memory-swappiness="" |

2.2 cpu -- This subsystem uses the scheduler to provide access to cpu cgroup tasks. <br>

| cpu cgroup interface | description | the corresponding docker interface |
| ---------------------------------------- | --------------------------------------------------------------------------- | ---------------------------------------- |
| cgroup/cpu/cpu.shares | specifies a relative share of CPU time avaliable to the tasks in a cgroup. Supposed that we have created two cgroups(C1 and C2) under the root directory of cgroupfs, and set the cpu.shares values to 512 and 1024 respectively. Then when C1 and C2 are vying for CPU, C2 will receive twice the CPU time of C1. It is noteworthy that cpu.shares only works when CPU is being competed, if C2 is idle, then C1 can receive the whole CPU time. | -c, --cpu-shares="" |
| cgroup/cpu/cpu.cfs_period_us | specifies the CPU bandwidth limit, and should collocate with cpu.cfs_quota_us. We can set the period to 1s, and the quota to 0.5s, so that the task in the cgroup can work at most 0.5s within 1s, and then the task will be forced to sleep until the next 1s comes. | --cpu-period="" |
| cgroup/cpu/cpu.cfs_quota_us | specifies the CPU bandwidth limit, and should collocate with cpu.cfs_period_us | --cpu-quota="" |

2.3 cpuset -- This subsystem assigns individual CPUs and memory modes to cgroups.<br>

| cpuset cgroup interface | description | the corresponding docker interface |
| ---------------------------------------- | ---------------------------------------------------------------------- | ---------------------------------------- |
| cgroup/cpuset/cpuset.cpus | specifies the CPUs that tasks in this cgroup are permitted to access(0-4, 9, etc.). | --cpuset-cpus="" |
| cgroup/cpuset/cpuset.mems | specifies the memory nodes that tasks in this cgroup are permitted to access(0-1, etc.) | --cpuset-mems="" |

2.4 blkio -- This subsystem controls and monitors access to I/O on block devices, such as physical devices(disk, solid-state drives, USB, etc.).<br>

| blkio cgroup interface | description | the corresponding docker interface |
| ---------------------------------------- | ----------------------------------------------------------------------- | ---------------------------------------- |
| cgroup/blkio/blkio.weight | specifies the relative proportion(weight) of block I/O access, in the range from 10 to 1000(include 10 and 1000). It is similar to cpu.shares, is proportion allocation rather than absolute bandwidth constraint. So it only works when different cgroups are competing to use the bandwidth of the same block device.  | --blkio-weight="" |
| cgroup/blkio/blkio.weight_device | specifies the relative proportion(weight) of I/O access on specific devices, and this value will override the blkio.weight parameter for specific devices. | --blkio-weight-device=""  |
| cgroup/blkio/blkio.throttle.read_bps_device | specifies the upper bandwidth limit of read operations for specific devices. | --device-read-bps="" |
| cgroup/blkio/blkio.throttle.write_bps_device | specifies the upper bandwidth limit of write operations for specific devices. | --device-write-bps="" |
| cgroup/blkio/blkio.throttle.read_iops_device | specifies the upper limit on the I/O number of read operations for specific devices. | --device-read-iops="" |
| cgroup/blkio/blkio.throttle.write_iops_device | specifies the upper limit on the I/O number of write operations for specific devices. | --device-write-iops="" |
