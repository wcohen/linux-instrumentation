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

Below is an example output of the script with using the
docker/whalesay from the
[docker tutorial](https://docs.docker.com/engine/getstarted/step_three/).

    $ ./container_check.stp -c  "docker run docker/whalesay cowsay boo"
    starting container_check.stp. monitoring 8204
     _____ 
    < boo >
     ----- 
        \
         \
          \     
                        ##        .            
                  ## ## ##       ==            
               ## ## ## ##      ===            
           /""""""""""""""""___/ ===        
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
           \______ o          __/            
            \    \        __/             
              \____\______/   
    
    
          executable:      prob capability
    
    
    
          executable,              syscall (       capability ) :            count
    
    
          executable,              syscall:            count
    
    
          executable,              syscall =            errno:            count
              docker,                fcntl =            EBADF:                1
              docker,               access =           ENOENT:                1
              docker,                ioctl =           ENOTTY:                1
      docker-current,                futex =                 :                4
      docker-current,                 read =           EAGAIN:                7
      docker-current,                futex =                 :                1
      docker-current,           epoll_wait =            EINTR:                1
      docker-current,               select =                 :                1
      docker-current,                 stat =           ENOENT:               11
      docker-current,                futex =           EAGAIN:                1
      docker-current,              connect =           ENOENT:                2
      docker-current,               access =           ENOENT:                1
              stapio,               execve =           ENOENT:                5
              stapio,         rt_sigreturn =            EINTR:                1
    WARNING: Number of errors: 0, skipped probes: 7412

