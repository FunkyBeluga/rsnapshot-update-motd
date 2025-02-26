# Simple rsnapshot status to show upon ssh login

I have an Ubuntu machine dedicated to backing up my other devices over ssh using rsnapshot. I found that I kept logging in over ssh, listing files in the rsnapshot base directory, and tailing the rsnapshot log to see if there had been any errors recently. No problem, but all I really care about is when the last daily/weekly/monthly backups were completed, and what the most recent rsnapshot completion status was. If the latest daily was "this morning," and the latest completion status is "completed successfully," I feel safe assuming that previous backups have gone on without issue. Why not automate those commands using the update-motd framework and just show the info on login?

### Creating the script:

The scripts that update-motd runs upon ssh login are found in ```/etc/update-motd.d/```. Start by creating the rsnapshot script:

```
$ sudo nano /etc/update-motd.d/55-rsnapshot
```

Add the following, save, and exit:

```
#!/bin/sh

lastDaily=$(ls -l /mnt/backup/rsnapshot/ | grep daily.0 | cut -d ' ' -f 6-8)
lastWeekly=$(ls -l /mnt/backup/rsnapshot/ | grep weekly.0 | cut -d ' ' -f 6-8)
lastMonthly=$(ls -l /mnt/backup/rsnapshot/ | grep monthly.0 | cut -d ' ' -f 6-8)
lastCompleted=$(cat /var/log/rsnapshot.log | grep completed | tail -1 | cut -d ' ' -f 4-100 | fold -w 68 -s)

echo " "
echo "-------------------------------------------------------------------"
echo "LATEST RSNAPSHOT STATUS:"
echo "$lastCompleted"
echo " "
echo "LATEST DAILY:            LATEST WEEKLY:            LATEST MONTHLY:"
echo "$lastDaily             $lastWeekly              $lastMonthly"
echo "-------------------------------------------------------------------"
```

Next, make it executable:

```
$ sudo chmod +x /etc/update-motd.d/55-rsnapshot
```

### Check it:

Exit the ssh session, and log back in over ssh. The new rsnapshot message should appear alongside the other motd messages:

```
bobbydiggs@thinkpad ~> ssh backup
bobbydiggs@192.168.1.11's password:
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-53-generic x86_64)

 System information as of Tue Feb 25 10:11:31 PM EST 2025

  System load:  0.0                Temperature:           41.9 C
  Usage of /:   0.8% of 912.81GB   Processes:             182
  Memory usage: 21%                Users logged in:       0
  Swap usage:   0%                 IPv4 address for eno1: 192.168.1.11

  Disk             Total      Used       Unused     Percent
  /mnt/backup      26T        15T        9.9T       60%

-------------------------------------------------------------------
LATEST RSNAPSHOT STATUS:
completed successfully

LATEST DAILY:            LATEST WEEKLY:            LATEST MONTHLY:
Feb 25 03:22             Feb 18 03:14              Dec 29 15:26
-------------------------------------------------------------------

Last login: Tue Feb 25 21:58:02 2025 from 192.168.1.80
```

Awesome!

Obviously this technique can be used to display anything you can possibly output from a script, for example a list of running containers, disk usage, log messages, etc.
