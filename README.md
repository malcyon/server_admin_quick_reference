# Server Administration Quick Reference

## Introduction

This is an assorted list of potentially useful Unix commands. These are commands I've found useful in the past that I don't want to forget.


## Filesystem Commands

### Zeroing out a file

    cat/dev/null > <filename>



### Grep multiple files across multiple directories

    find . -type f -print | xargs fgrep -i -l "text_to_grep_for"


### Find files larger than 1000000 bytes

    ll -R <directoryname> | awk '{if ($5 > 1000000) print $5 "\t" $9}'

    find . -xdev -size +1000000c

On Linux:

    find . -xdev -size +1000000c -exec ls -al {} \; | sort -k 5n

On HP-UX:

    find . -xdev -size +1000000c -exec ls -al {} \; | sort -r +4n


### Find files modified 3 or fewer days ago

On Linux:

    find . -xdev -type f -mtime 3 -exec ls -al {} \; | sort -rk +5n

On HP-UX:

    find . -xdev -type f -mtime 3 -exec ls -al {} \; | sort -r +4n


### Delete object files and print what's deleted

    find . -name "*.o" -exec echo 'rm -f {}' \; -exec rm -f {} \;


### Show list of directories and their sizes in kilobytes

On Linux:

    du -xhk | sort -k 1n

On HP-UX:

    du -xk | sort +0n


### Show how much space a directory is taking up in kilobytes

    du -xks


Alternate version (only shows one level):

    du -shx



### Determine what process has a file open

    fuser -u /path/to/file

    psg <pid>



### Monitor a directory for open filehandles. Repeat command every second forever.

    while true; do lsof +d /tmp; sleep 1; done



### Show space on drives

    df -k


### Querying file locks max value on HP-UX

    kcusage | grep nflocks


### Create a file of a given size

    dd if=/dev/zero of=don.out bs=1024 count=10240



### Delete file by inode

    ls -li

    find . inem <inode number> -exec rm -l {} \;



### Search/Replace in a file

    sed -ri 's/(test1|test2)/value3/g' app.cfg



### Delete logfiles without an open file handle

    find /tmp -type f -exec bash -c "fuser {} || rm {}" \;



### Determine if it is an HDD or SSD (0 means SSD, 1 means HDD)

    cat /sys/block/sdc/queue/rotational  



### Manually formatting and mounting a block device

    lsblk  
    file -s /dev/xvdb  
    mkfs -t ext4 /dev/xvdb  
    mkdir /mnt/jenkins  
    mount /dev/xvdb /mnt/jenkins  
    ll /mnt/jenkins  
    df -h  
