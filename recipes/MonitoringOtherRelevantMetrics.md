* Monitoring other relevant metrics of arango

## Problem

[Collectd offers a pleathora of plugins](https://collectd.org/wiki/index.php/Table_of_Plugins) here is a tiny abstract of others that may be interesting to use with ArangoDB


###Ingedients

For this receipie you need to install the following tools:

  * [collectd](https://collectd.org/) The metrics aggregation Daemon
  * we base on [Monitoring with Collecd recipy](MonitoringWithCollectd.html) for understanding the basics about collectd.

###Diskusage
You may want to monitor ArangoDB doesn't run out of disk space. The [df Plugin](https://collectd.org/wiki/index.php/Plugin:DF) can aggregate these values for you.

First we need to find out, which disks are used by your arango; by default you need to find /var/lib/arango in the mountpoints; Since nowadays many virtual filesystems are also mounted on a typical *nix system we want to sort the output of mount:
    mount | sort
    /dev/sda3 on /local/home type ext4 (rw,relatime,data=ordered)
    /dev/sda4 on / type ext4 (rw,relatime,data=ordered)
    /dev/sdb1 on /mnt type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=utf8,shortname=mixed,errors=remount-ro)
    binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
    cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
    ....
    udev on /dev type devtmpfs (rw,relatime,size=10240k,nr_inodes=1022123,mode=755)

So here we can see the mountpoints are **/** **/local/home** **/mnt/** so **/var/lib/** can befound on the root partition (**/**) **/dev/sda3** here. A production setup may be different so the OS doesn't interfere with the services.

The collectd configuration /etc/collectd/collectd.conf.d/diskusage.conf is like this:

    LoadPlugin df
    <Plugin df>
      Device "/dev/sda3"
      #  Device "192.168.0.2:/mnt/nfs"
      #  MountPoint "/home"
      #  FSType "ext4"
      #  ignore rootfs; else, the root file-system would appear twice, causing
      #  one of the updates to fail and spam the log
      FSType rootfs
      # ignore the usual virtual / temporary file-systems
      FSType sysfs
      FSType proc
      FSType devtmpfs
      FSType devpts
      FSType tmpfs
      FSType fusectl
      FSType cgroup
      IgnoreSelected true
      #  ReportByDevice false
      #  ReportReserved false
      #  ReportInodes false
      #  ValuesAbsolute true
      #  ValuesPercentage false
    </Plugin>

###Disk I/O Usage
Another interesting metric is the amount of data read/written to disk - its an estimate how busy your ArangoDB or the whole system currently is.
The [Disk plugin](https://collectd.org/wiki/index.php/Plugin:Disk) aggregates these values.

Our configuration /etc/collectd/collectd.conf.d/disk_io.conf looks like this:

    LoadPlugin disk
    <Plugin disk>
    	Disk "hda"
    	Disk "/sda[23]/"
    	IgnoreSelected false
    </Plugin>


###CPU Usage
While the arangodb self monitoring already offers some overview of the running threads etc. you can get a deeper view using the [Process Plugin](https://collectd.org/wiki/index.php/Plugin:Processes).

If you're running a single arango instance, a simple match by process name is sufficient, /etc/collectd/collectd.conf.d/arango_process.conf looks like this:

    LoadPlugin processes
    <Plugin processes>
      Process "name"
    </Plugin>

If you're running a cluster, you can match the specific instances by commandline parameters, /etc/collectd/collectd.conf.d/arango_cluster.conf looks like this:

    LoadPlugin processes
    <Plugin processes>
      ProcessMatch "Claus" "/usr/bin/arangod .*--cluster.my-id Claus.*"
      ProcessMatch "Pavel" "/usr/bin/arangod .*--cluster.my-id Pavel.*"
      ProcessMatch "Perry" "/usr/bin/arangod .*--cluster.my-id Perry.*"
      Process "etcd-arango"
    </Plugin>

### More Plugins
As mentioned above, the list of available plugins is huge; Here are some more one could consider interesting:
* Use the [CPU Plugin](https://collectd.org/wiki/index.php/CPU) to monitor the overal cpu utilisation
* Use the [Memory Plugin](https://collectd.org/wiki/index.php/Plugin:Memory) to monitor main memory availability
* use the [Swap Plugin](https://collectd.org/documentation/manpages/collectd.conf.5.shtml#plugin_swap) to see whether excess RAM usage forces the system to page and thus slow down
* [Ethernet Statistics](https://collectd.org/wiki/index.php/Plugin:Ethstat) with whats going on at your Network cards to get a more broad overview of network traffic
* you may [Tail logfiles](https://collectd.org/wiki/index.php/Plugin:Tail) like an apache requset log and pick specific requests by regular expressions
* [Parse tabular files](https://collectd.org/wiki/index.php/Plugin:Table) in the /proc filesystem
* you can use [filters](https://collectd.org/documentation/manpages/collectd.conf.5.shtml#filter_configuration) to reduce the amount of data created by plugins (i.e. if you have 32 CPU cores, you may want the combined result) or decide where to route data to which writer plugin.
* While you may have seen that metrics are stored at a fixed rate or frequency, your metrics (i.e. the durations of webrequests) may come in a random & higher frequency. Thus you want to burn them down to a fixed frequency, and know Min/Max/Average/Median. So you want to  [Aggregate values using the statsd pattern](https://collectd.org/wiki/index.php/Plugin:StatsD).
* You may start rolling your own in [Python](https://collectd.org/wiki/index.php/Plugin:Python), [java](https://collectd.org/wiki/index.php/Plugin:Java), [Perl](https://collectd.org/wiki/index.php/Plugin:Perl) or for shure in [C](https://collectd.org/wiki/index.php/Plugin_architecture), the Language collectd is implemented in.

Finaly while kcollectd is nice to get a quick success at inspecting your collected metrics during working your way into collectd, its not as sufficient for operating a production site. Since collectds default storage RRD is already widespread in system monitoring, there are [many webfrontents](https://collectd.org/wiki/index.php/List_of_front-ends) to choose for the visualisation. Some of them replace the RRD storage by simply adding a writer plugin, most prominent the [Graphite graphing framework](http://graphite.wikidot.com/screen-shots) with the [Graphite writer](https://collectd.org/wiki/index.php/Plugin:Write_Graphite) which allows you to combine random metrics in single graphs - to find coincidences in your data [you never dreamed of](http://metrics20.org/media/). 

If you already run [Nagios](http://www.nagios.org) you can use the [Nagios tool](https://collectd.org/documentation/manpages/collectd-nagios.1.shtml) to submit values.

As usual its a good idea to browse the [Fine Manual](https://collectd.org/documentation.shtml).
