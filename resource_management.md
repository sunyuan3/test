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
