**Linux Performance Optimization**

A collection of 140 little known Linux optimization commands for optimizing Linux VPS's and general Gaming Performance. Gaming devices such as the Steam Deck/Linux laptops and Desktops should have improved FPS, Battery life and network performance.

Formatting is a WIP due to how many optimizations there are, commands will be bolded soon. For now they are space separated with a blank line between the description of what the command does and the actual command.

# Linux Optimization Commands

## Table of Contents
1. [System Configuration](#system-configuration)
2. [Memory Management](#memory-management)
3. [CPU Optimization](#cpu-optimization)
4. [Disk I/O Optimization](#disk-io-optimization)
5. [Network Optimization](#network-optimization)
6. [File System Optimization](#file-system-optimization)
7. [Process Management](#process-management)
8. [Kernel Tuning](#kernel-tuning)
9. [Database Optimization](#database-optimization)
10. [Miscellaneous Optimizations](#miscellaneous-optimizations)

## System Configuration

### Disable Transparent Huge Pages (THP)
THP can cause performance issues for databases. [More info](https://www.pingcap.com/blog/transparent-huge-pages-why-we-disable-it-for-databases/)

### Use Tuned-ADM
Tuned-ADM is a tool for tuning system performance. [Manual](https://manpages.ubuntu.com/manpages/jammy/man7/tuned-profiles.7.html)

### Change Init Level (For VPS hosting WordPress sites only)
```bash
init 3
```
[Instructions for changing run levels](https://www.cyberciti.biz/howto/question/linux/changing-run-levels-3-5.php)

### Use Stacer
Stacer is a system optimizer and monitoring tool. [More info](https://www.site24x7.com/learn/linux/linux-performance-optimization.html)

### Disable Unnecessary Linux Services
```bash
systemctl list-unit-files --type=service
sudo systemctl disable SERVICE_NAME
```

### Optimize Boot Parameters (GRUB only)
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash noapic noacpi nosplash irqpoll"
```

### Tune Systemd for Performance
```
DefaultTimeoutStartSec=10s
DefaultTimeoutStopSec=10s
```

## Memory Management

### Enable Z-Swap
```bash
sudo perl -pi -e 's/^(GRUB_CMDLINE_LINUX_DEFAULT.*)"$/$1 zswap.enabled=1 zswap.compressor=lz4 zswap.zpool=z3fold zswap.max_pool_percent=25"/' /etc/default/grub
sudo update-grub
sudo echo -e 'lz4\nlz4_compress\nz3fold' >> /etc/initramfs-tools/modules
sudo update-initramfs -u
sudo reboot
```

### Enable MGLRU (Multi-Gen. Least Recently Used)
```bash
cat << EOF | sudo tee /etc/tmpfiles.d/mglru.conf
w /sys/kernel/mm/lru_gen/enabled - - - - 7
w /sys/kernel/mm/lru_gen/min_ttl_ms - - - - 0
EOF
```

### Unlock Memory Lock
```bash
cat << EOF | sudo tee /etc/security/limits.d/memlock.conf
* hard memlock 2147484
* soft memlock 2147484
EOF
```

### Optimize Virtual Memory Settings
```bash
sysctl -w vm.dirty_background_ratio=5
sysctl -w vm.dirty_ratio=10
```

### Use HugeTLB Pages
```bash
echo 'vm.nr_hugepages=128' >> /etc/sysctl.conf
```

### Enable and Configure zram
```bash
apt-get install zram-config
```

### Adjust Swappiness
```bash
sysctl -w vm.swappiness=10
```

## CPU Optimization

### Switch to Performance CPU Governor
```bash
cat << EOF | sudo tee /etc/systemd/system/cpu_performance.service
[Unit]
Description=CPU performance governor
[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower frequency-set -g performance
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable cpu_performance.service
```

### Use CPU Isolation
```bash
isolcpus=1,2
```

### Optimize Kernel Scheduler
```bash
echo 1 > /proc/sys/kernel/sched_autogroup_enabled
```

### Use schedtool for Advanced CPU Scheduling
```bash
apt-get install schedtool
schedtool -R -p 20 -e myapp
```

## Disk I/O Optimization

### Change I/O Scheduler
```bash
cat << EOF | sudo tee /etc/udev/rules.d/64-ioschedulers.rules
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="kyber"
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="kyber"
EOF
```

### Enable Writeback Caching
```bash
hdparm -W1 /dev/sda
```

### Use RAM Disks for Temporary Storage
```bash
mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk
```

### Optimize Swap Settings with zswap
```bash
echo 1 > /sys/module/zswap/parameters/enabled
echo lz4 > /sys/module/zswap/parameters/compressor
echo 20 > /sys/module/zswap/parameters/max_pool_percent
```

### Use blktrace for Block Layer Tracing
```bash
apt-get install blktrace
blktrace -d /dev/sda
```

### Increase Read-Ahead Buffer
```bash
blockdev --setra 2048 /dev/sda
```

## Network Optimization

### Enable Jumbo Frames
```bash
ifconfig eth0 mtu 9000
```
or
```bash
ip link set dev eth0 mtu 9000
```

### Enable TCP BBR
```bash
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

### Optimize TCP Network Stack
```bash
sysctl -w net.ipv4.tcp_keepalive_time=600
sysctl -w net.ipv4.tcp_keepalive_probes=5
sysctl -w net.ipv4.tcp_keepalive_intvl=60
sysctl -w net.ipv4.tcp_window_scaling=1
sysctl -w net.ipv4.tcp_sack=1
sysctl -w net.ipv4.tcp_fin_timeout=30
sysctl -w net.ipv4.tcp_max_syn_backlog=2048
sysctl -w net.ipv4.tcp_max_tw_buckets=400000
sysctl -w net.ipv4.tcp_fastopen=3
```

### Enable Offload Processing
```bash
ethtool -K eth0 gro on
ethtool -K eth0 gso on
ethtool -K eth0 tso on
```

### Optimize DNS Resolution
```bash
apt-get install dnsmasq
```

### Implement TCP MSS Clamping
```bash
iptables -t mangle -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
```

### Enable TCP Window Scaling
```bash
sysctl -w net.ipv4.tcp_window_scaling=1
```

### Optimize Network Buffer Sizes
```bash
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
```

### Implement Network Interface Teaming
```bash
apt-get install ifenslave
```

### Enable IPv6
```bash
sysctl -w net.ipv6.conf.all.disable_ipv6=0
```

### Optimize TCP Reassembly Settings
```bash
sysctl -w net.ipv4.tcp_reordering=3
```

### Adjust Network Queue Length
```bash
ifconfig eth0 txqueuelen 10000
```

## File System Optimization

### Use xfs_fsr for XFS File System Defragmentation
```bash
xfs_fsr /dev/sda1
```

### Use fstrim for SSDs
```bash
fstrim -v /
```

### Implement ZFS
```bash
apt-get install zfsutils-linux
zpool create mypool /dev/sda
```

### Optimize File System Access Time
```bash
mount -o remount,noatime,nodiratime /
```

## Process Management

### Increase Open File Limits
```bash
echo "fs.file-max = 500000" >> /etc/sysctl.conf
ulimit -n 500000
```

### Adjust Out-of-Memory (OOM) Killer Settings
```bash
echo '-1000' > /proc/$(pidof critical_process)/oom_score_adj
```

### Use cgroups for Resource Management
```bash
cgcreate -g cpu,memory:mygroup
cgset -r cpu.shares=512 mygroup
cgset -r memory.limit_in_bytes=512M mygroup
```

### Use pm2 for Node.js Process Management (VPS's using Node.js only)
```bash
npm install pm2@latest -g
pm2 start app.js
```

### Use cpulimit to Limit CPU Usage of Processes
```bash
apt-get install cpulimit
cpulimit -e myapp -l 50
```

## Kernel Tuning

### Optimize Kernel Parameters
```bash
sysctl -w net.core.somaxconn=1024
sysctl -w net.core.netdev_max_backlog=5000
sysctl -w net.ipv4.tcp_syncookies=1
sysctl -w fs.inotify.max_user_watches=524288
sysctl -w kernel.panic=10
sysctl -w kernel.panic_on_oops=1
sysctl -w net.ipv4.tcp_rfc1337=1
sysctl -w net.ipv4.tcp_mtu_probing=1
sysctl -w net.ipv4.tcp_base_mss=536
sysctl -w fs.file-max=2097152
sysctl -w net.ipv4.tcp_low_latency=1
```

### Adjust Kernel Timer Frequency
```bash
echo 1000 > /proc/sys/kernel/timer_frequency
```

### Enable Kernel Samepage Merging (KSM)
```bash
echo 1 > /sys/kernel/mm/ksm/run
```

### Tune Kernel Semaphore Parameters
```bash
sysctl -w kernel.sem="250 32000 100 128"
```

### Optimize Inode Cache
```bash
sysctl -w fs.inode-nr
sysctl -w fs.inode-state
```

### Tune Kernel Preemption Model
```bash
echo 0 > /proc/sys/kernel/preempt
```

## Database Optimization

### Enable Asynchronous I/O (AIO) for Databases
```
innodb_use_native_aio = 1
```

### Enable and Configure Query Caching (MySQL/MariaDB)
```
query_cache_size = 64M
query_cache_type = ON
```

## Miscellaneous Optimizations

### Use Custom Kernel
[More info](https://www.fosslinux.com/111937/tips-and-tricks-for-optimizing-linux-device-performance.htm)

### Store Temporary Files in Memory
```bash
mount -t tmpfs -o size=512M tmpfs /tmp
```

### Use jemalloc for Memory Allocation (VPS's only)
```bash
LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2
```

### Implement Squid for Caching and Proxying (VPS's only)
```bash
apt-get install squid
```

### Use Lightweight Logging
```bash
sed -i 's/^#LogLevel=info/LogLevel=error/' /etc/systemd/journald.conf
```

### Prevent Superfluous File Access Time Book-keeping
```bash
sudo sed -i -e '/home/s/\bdefaults\b/&,noatime/' /etc/fstab
```

Note: Always test these optimizations in a controlled environment before applying them to production systems. Some optimizations may not be suitable for all use cases or may require fine-tuning based on your specific hardware and workload.
