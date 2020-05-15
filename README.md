# macstat
Bash shell script that collects macOS system metrics for [Devo Data Analytics Platform](http://devo.com).

It collects almost 30 system metrics in a log file ready to be sent to Devo using a third-party syslog application such as syslog-ng. [Dstat](https://github.com/dstat-real/dstat) and [Devo Monitor](https://docs.devo.com/confluence/ndt/sending-data-to-devo/event-sources/unix-like-machines/configuration-packages-for-nix) inspired this project.


## Prerequisites

#### 1. Disable DTrace restrictions. 
   macstat uses `iotop` to get disk stats. This command is, in fact, a DTrace script. [DTrace](http://www.brendangregg.com/dtrace.html) (Dynamic Tracing) is a powerful debugging framework. `iotop` is just one of the numerous DTrace scripts shipped by default on macOS. To run it, you need __root privileges__ and tell Apple's [System Integrity Protection (SIP)](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html) to let DTrace do its job. To do so, boot your Mac into recovery mode and run:
   ```sh
   csrutil enable --without dtrace
   ```
   Then restart your machine. Oh and ignore the scary output message. Trust me, it’s better to have an unsupported configuration than disabling SIP altogether.
   
   > Note: If you don't want or can't disable SIP, then use `iostat` instead. You'll need additional calculations, but to get you started I included a sample command in the comments to get data transferred in bytes/sec.

#### 2. Compile `networkStats.c` file.
   Execute the following command to compile this C file: 
   ```sh
   cc networkStats.c -DWEBVIEW_COCOA=1 -ObjC -framework Cocoa -framework WebKit -o networkStats
   ```
   This creates a binary file called `networkStats` that macstat will use to get network data sent and received in bytes/sec.
   
> Note: I'm not the author of this C file. I just found it on the Internet, but I don't remember the source.
   
#### 3. Move `iotop` script to `/usr/local/bin`.
This is a custom `iotop` script. I added improved settings recommended by `iotop`'s author in his book to address dynamic variable drops errors. As mentioned in step 1, `iotop` requires DTrace enabled and __root privileges__. Using the default `/usr/bin/iotop` command will show inaccurate results. 
> Note: If you are using `iostat`, skip this step.

#### 4. Edit the USER SETTINGS section.
Replace the variables in the USER SETTINGS section with your own values accordingly. 


## Testing
```sh
USAGE: sudo ./macstat [options]

  -a      list all system metrics
  -h      show help 
  -V      display version info
```

For testing, run macstat on itself: 
```sh
sudo ./macstat
```
If it was successful, you should get this message: 
```sh
macstat finished successfully
Log: /Users/roberto/Library/Logs/Devo/macstat.log
```


## Run it!
Add it to the __root__ crontab to be scheduled to run every minute:
```sh
* * * * * /Users/roberto/macstat
```
> Note: If you are not using `iotop`, then add it to your own user crontab.


## Sending macstat log file to Devo
In the `extras` directory, I provided the config files `syslog-ng.conf` and `syslog-ng.conf.relay` to send `macstat.log` to Devo using [syslog-ng](https://github.com/syslog-ng/syslog-ng) depending on your environment. Also, I included the config file `devo.conf`, which uses the default log file rotation utility in macOS: [newsyslog](https://www.freebsd.org/cgi/man.cgi?query=newsyslog.conf&sektion=5&manpath=FreeBSD+12.1-RELEASE+and+Ports). For more information on how to implement these files, review my [Medium](https://medium.com/@rcmelendez/macos-monitoring-with-devo-422e502937bd) guide.


## License
macstat is licensed under the terms of the MIT License. See the `LICENSE` file for details.


## Contact
Find me as __rcmelendez__ on [LinkedIn](https://www.linkedin.com/in/rcmelendez/), [Medium](https://medium.com/@rcmelendez), and of course [GitHub](https://github.com/rcmelendez/).

