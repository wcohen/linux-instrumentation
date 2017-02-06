# container_check.stp

The script container_check.stp watches for use of
prohibited capabilities, use of prohibited syscalls, and
syscall failures) that would indicate that this application
would not operate properly in a restricted contiainer.

By default this script monitors all systemcalls system-wide.
To limit to limit container_check.stp to monitoring a particular
process and it children use the systemtap -x <pid> option
or -c <command> option.

By default this script lists all capabilities requested.
To limit it to a subset of capabilities use the following
option on the command line with a '-' separated list of
forbidden capabilites:

  -G forbidden_capabilities="badcap1-badcap2"

By default this script allows all syscalls.
To mark syscalls as forbidden use a '-' separate list: 
  
  -G forbidden_syscalls="syscall1-syscall2"

control-c to exit data collection

Below is an example output of the script with showing it follow the
child process of the `sudo` command and then `strace` command that it
is monitoring. The `-DKREACDTIVE=100` is used in this example increase
the number of active return return probes allowed to avoid having the
script miss collecting some data triggering a [WARNING about `skipped
probes` at the
end](https://sourceware.org/systemtap/wiki/TipSkippedProbes).  The
output of the script shows the capablilities used by the ping and sudo
commands and the specific syscalls using those capabilities.

    $ ./container_check.stp -DKRETACTIVE=100 -c  "sudo strace -c -f ping -c 1 people.redhat.com"
    starting container_check.stp. monitoring 30062
    PING people02.pubmisc.prod.ext.phx2.redhat.com (10.5.19.28) 56(84) bytes of data.
    64 bytes from people02.pubmisc.prod.ext.phx2.redhat.com (10.5.19.28): icmp_seq=1 ttl=57 time=46.5 ms
    
    --- people02.pubmisc.prod.ext.phx2.redhat.com ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 46.507/46.507/46.507/0.000 ms
    % time     seconds  usecs/call     calls    errors syscall
    ------ ----------- ----------- --------- --------- ----------------
     37.74    0.000876          97         9         2 socket
     13.66    0.000317          17        19           open
      6.94    0.000161           5        31           mmap
      5.95    0.000138           6        22           mprotect
      5.86    0.000136           7        20           read
      4.57    0.000106           5        20           fstat
      4.27    0.000099           4        24           close
      2.93    0.000068          34         2           sendto
      2.76    0.000064          13         5         2 connect
      1.98    0.000046           8         6           write
      1.77    0.000041          41         1           sendmmsg
      1.59    0.000037           7         5           poll
      1.34    0.000031          10         3           munmap
      1.25    0.000029           4         7           setsockopt
      1.12    0.000026           5         5           ioctl
      1.08    0.000025           4         7           capget
      0.82    0.000019           6         3           recvfrom
      0.56    0.000013           4         3           capset
      0.52    0.000012           3         4           brk
      0.47    0.000011           6         2         2 access
      0.43    0.000010           3         3           rt_sigaction
      0.43    0.000010           5         2           prctl
      0.34    0.000008           8         1           setuid
      0.30    0.000007           4         2           getuid
      0.22    0.000005           5         1           setitimer
      0.22    0.000005           5         1           getsockname
      0.22    0.000005           5         1           getsockopt
      0.17    0.000004           4         1           arch_prctl
      0.13    0.000003           3         1           rt_sigprocmask
      0.13    0.000003           3         1           getpid
      0.13    0.000003           3         1           recvmsg
      0.13    0.000003           3         1           geteuid
      0.00    0.000000           0         1           execve
    ------ ----------- ----------- --------- --------- ----------------
    100.00    0.002321                   215         6 total
    
    
          executable:      prob capability
    
                ping:           cap_setuid
                ping:          cap_net_raw
    
                sudo:           cap_setgid
                sudo:           cap_setuid
                sudo:      cap_audit_write
    
    
    
          executable,              syscall (       capability ) :            count
                ping,               socket ( cap_net_raw ) : 2
                ping,               setuid ( cap_setuid ) : 1
                sudo,            setresuid ( cap_setuid ) : 11
                sudo,               sendto ( cap_audit_write ) : 5
                sudo,               setgid ( cap_setgid ) : 1
                sudo,               setuid ( cap_setuid ) : 1
                sudo,            setgroups ( cap_setgid ) : 5
                sudo,            setresgid ( cap_setgid ) : 10
    
    
          executable,              syscall:            count
    
    
          executable,              syscall =            errno:            count
                ping,               socket =           EACCES:                2
                ping,               access =           ENOENT:                2
                ping,              connect =           ENOENT:                2
              stapio,         rt_sigreturn =            EINTR:                1
              stapio,               execve =           ENOENT:                5
              strace,                wait4 =           ECHILD:                1
              strace,               access =           ENOENT:                1
                sudo,              recvmsg =           EAGAIN:                3
                sudo,               access =           ENOENT:                4
                sudo,         rt_sigreturn =            EINTR:                1
                sudo,                 poll =                 :                1
                sudo,                ioctl =           ENOTTY:                2
                sudo,                fstat =            EBADF:                1
                sudo,                 stat =           ENOENT:                7
                sudo,                 read =           EAGAIN:                1
                sudo,              connect =           ENOENT:               13
                sudo,                 open =           ENOENT:               83
