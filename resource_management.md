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

enables flexible sharing of memory. Under normal circumstances, control groups are allowed to use as much of the memory as needed, constrained only by their hard limits set with the memory.limit_in_bytes parameter. However, when the system detects memory contention or low memory, control groups are forced to restrict their consumption to their soft limits. To set the soft limit for example to 256 MB, execute:
| common cgroup interface | description | the corresponding docker interface |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| cgroup/memory/memory.limit_in_bytes | sets the maximum amount of user memory, in bytes. And it is possible to use suffixes to represent larger units -- k or K for kilobytes, m or M for megebytes, and g or G for gigabytes. | -m, --memory="" |
| cgroup/memory/memory.memsw.limit_in_bytes | sets the maximum amount of memory and swap space, in bytes. You can prevent a run out of swap partition by setting this value. | --memory-swap="" |
| cgroup/memory/memory.soft_limit_in_bytes |  | --memory-reservation="" |
| cgroup/memory/memory.kmem.limit_in_bytes |设定内核内存上限。| --kernel-memory="" |
| cgroup/memory/memory.oom_control |如果设置为0，那么在内存使用量超过上限时，系统不会杀死进程，而是阻塞进程直到有内存被释放可供使用时，另一方面，系统会向用户态发送事件通知，用户态的监控程序可以根据该事件来做相应的处理，例如提高内存上限等。| --oom-kill-disable="" |
| cgroup/memory/memory.swappiness |控制内核使用交换分区的倾向。取值范围是0至100之间的整数（包含0和100）。值越小，越倾向使用物理内存。| --memory-swappiness="" |

2.2 cpu -- 这个子系统使用调度程序提供对 CPU 的 cgroup 任务访问。<br>

| 子系统常用cgroups接口 | 描述 | 对应的docker接口 |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| cgroup/cpu/cpu.shares | 负责CPU比重分配的接口。假设我们在cgroupfs的根目录下创建了两个cgroup（C1和C2），并且将cpu.shares分别配置为512和1024，那么当C1和C2争用CPU时，C2将会比C1得到多一倍的CPU占用率。要注意的是，只有当它们争用CPU时CPU share才会起作用，如果C2是空闲的，那么C1可以得到全部的CPU资源。 | -c, --cpu-shares="" |
| cgroup/cpu/cpu.cfs_period_us | 负责CPU带宽限制，需要与cpu.cfs_quota_us搭配使用。我们可以将period设置为1秒，将quota设置为0.5秒，那么cgroup中的进程在1秒内最多只能运行0.5秒，然后就会被强制睡眠，直到下一个1秒才能继续运行。 | --cpu-period="" |
| cgroup/cpu/cpu.cfs_quota_us | 负责CPU带宽限制，需要与cpu.cfs_period_us搭配使用。 | --cpu-quota="" |

2.3 cpuset -- 这个子系统为 cgroup 中的任务分配独立 CPU（在多核系统）和内存节点。<br>

| 子系统常用cgroups接口 | 描述 | 对应的docker接口 |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| cgroup/cpuset/cpuset.cpus | 允许进程使用的CPU列表（例如：0-4,9）。 | --cpuset-cpus="" |
| cgroup/cpuset/cpuset.mems | 允许进程使用的内存节点列表（例如：0-1）。 | --cpuset-mems="" |

2.4 blkio -- 这个子系统为块设备设定输入/输出限制，比如物理设备（磁盘、固态硬盘、USB等）。<br>

| 子系统常用cgroups接口 | 描述 | 对应的docker接口 |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| cgroup/blkio/blkio.weight | 设置权重值，取值范围是10至1000之间的整数（包含10和1000）。这跟cpu.shares类似，是比重分配，而不是绝对带宽的限制，因此只有当不同的cgroup在争用同一个块设备的带宽时，才会起作用。 | --blkio-weight="" |
| cgroup/blkio/blkio.weight_device | 对具体的设备设置权重值，这个值会覆盖上述的blkio.weight。 | --blkio-weight-device=""  |
| cgroup/blkio/blkio.throttle.read_bps_device | 对具体的设备，设置每秒读块设备的带宽上限。 | --device-read-bps="" |
| cgroup/blkio/blkio.throttle.write_bps_device | 设置每秒写块设备的带宽上限。同样需要指定设备。 | --device-write-bps="" |
| cgroup/blkio/blkio.throttle.read_iops_device | 设置每秒读块设备的IO次数的上限。同样需要指定设备。 | --device-read-iops="" |
| cgroup/blkio/blkio.throttle.write_iops_device | 设置每秒写块设备的IO次数的上限。同样需要指定设备。 | --device-write-iops="" |
