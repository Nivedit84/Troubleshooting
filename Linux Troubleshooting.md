Linux Troubleshooting

1. Disk space is full
-- If the disk space on a server is full, follow the below steps:
    1. run df -h, shows disk filesystem details like used space, available space and total space on a filesystem in human readable format
    2. Can also list which directories are largest, use 
        $du -ahx | sort -rh | head -n 20 --> du shows all files/directories in human readable format and for one file system, use this to find large files/directories on your server
    3. Can also use,
        $lsof | grep deleted --> list of files which were deleted, deleted files can also be a reason of disk full
    This was about finding the cause, now comes the resolution:
        1. Check for logs, sometimes logs can be accumulated, to clear either create a script to rotate logs or run:
            $logrotate /etc/logrotate.conf --> this basically rotates the logs, compresses them or remove them, can create a cronjob for this
        2. Can also clear systemd logs, to clear run:
            $journalctl --vacuum-size=500M

2. Disk space is free, but still can't write
-- By running df -h check whether disk space is free or not, if free but still can't create files:
    1. This usually happends when inodes are full for a filesystem, each filesystem has its own capacity to give inodes, to check inodes:
        $df -i --> shows used, total and available inodes for particular filesystem
    2. For this, use this for loop and create a script for this,
        for i in /*;do
            echo "$i: $(find $i -xdev -type f 2>/dev/null | wc -l) files"
        done
        In this, we are finding all the directories in root and without going into other filesystems and handling permissions errors we get the file count for each directory

    3. Whichever directory has high file count you can start looking there, as it might be the reason of inode exhaustion
    4. Mostly these would be /var/log or directories related to this.


3. /var partition is full due to logs and temp files, no logs are getting recorded.
-- To identify the cause, run du command on /var directory to see which sub-directory is consuming more space
    1. Run:
        $du -h --max-depth=1 /var | sort -rh --> only tells the size of sub directories under /var and presents in a reverse order
    2. Files are identified, then go the most consuming directory and compress files or remove unncessaryfiles, can use logrotate for this