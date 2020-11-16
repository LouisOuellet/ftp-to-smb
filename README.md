# FTP to SMB
FTP to SMB file transfer script. Including email reporting. I wrote this script because I needed a way to transfer files from various ftp accounts to network shares on a Windows network. Since ADDC does not allow to map a ftp drive at logon, this was my next solution so I could deploy those files to many users centrally. My other solution was to go at each desk and setup the FTP on each machine locally.

## Details
The script works as followed. The FTP and SMB shares are mounted. A rsync transfer is performed from the FTP share to the SMB share. Once the rsync transfer has completed, the script loops through each transferred files to validate their md5sum. This allows the script to confirm the integrity of each files that were transferred. Once the script has validated a file, it will move the original files on the FTP share to an archives sub-directory and moves the transferred file into a validated sub-directory. Once the transfers have completed, the report is generated and sent via email if requested. To ensure that the credentials are not retrieved by someone looking into the log file, the script will automatically change any references of any of the settings from the logfile.

## Usage
``` bash
$ ./getfiles
Unable to retrieve settings.json.
File not found.

Usage: ./getfiles [options]

Options:

-a                     => Enable Archive Mode
                          This mode moves the transferred file in an archive folder instead of deleting it
-v                     => Enable Reporting Mode
                          Input commands sent are stored in
-e                     => Compile errors and warnings after execution
-d                     => Set report destination email
-r                     => Send report via email
-c                     => Disable all formatting
-f                     => Specify a FTP ex: "ftp.domain.com username password"
-s                     => Specify a SMB ex: "0.0.0.0 shareName destinationDirectory username password"
```
If no FTP and/or SMB is provided, the script will request the information by GUI/CLI depending on your setup.

## Requirements (non-root user)
For the script to work with a non-root user, you will need to add some parameters to your sudoers file.

``` bash
echo "username ALL= NOPASSWD: /bin/umount" | sudo tee -a /etc/sudoers
echo "username ALL= NOPASSWD: /bin/mount" | sudo tee -a /etc/sudoers
echo "username ALL= NOPASSWD: /bin/curlftpfs" | sudo tee -a /etc/sudoers
echo "username ALL= NOPASSWD: /sbin/reboot" | sudo tee -a /etc/sudoers
```

These are required so that the script can mount both SMB and FTP shares as the local user to process the file transfer.

## Configuring getfiles
To support email reporting, you will need to setup a smtp account. So in the directory of getfiles create a file called settings.json and insert the following with the proper changes.

``` json
{
    "smtp":{
        "host": "fqdn.domain.com",
        "port": "465",
        "username": "username@domain.com",
        "password": "Password123"
    },
    "send":{
        "name": "Monitoring",
        "from": "monitoring@domain.com",
        "to": "destination@domain.com"
    },
    "logs":{
        "directory": "/tmp/"
    }
}
```
Additionnally you can set the default ftp and smb settings

``` json
{
    "ftp":{
        "host": "fqdn.domain.com",
        "port": "465",
        "username": "username",
        "password": "Password123"
    },
    "smb":{
        "host": "fqdn.domain.com",
        "share": "share1",
        "destination": "/subdirectory/",
        "username": "username",
        "password": "Password123",
        "version": "1.0"
    }
}
```


## Configuring a cron for multiple transfers
To run as a cron, you have multiple options. If you only have one transfer to process, I would suggest you add them directly to your crontab. But if you have multiple transfers to do, I recommend creating a script like this to process them.

``` bash
#!/bin/bash

# Verify if the file-system is currently in READ-ONLY state
FILE=/tmp/test.txt;touch ${FILE};if [ -f ${FILE} ]; then
  cd /home/${USER}/bin/getfiles # Location of getfiles
  bash getfiles -vcrd destination@domain.com -f "host1.domain.com 21 username password" -s "host share1 destination username password"
  bash getfiles -a -f "host2.domain.com 21 username password" -s "host share2 destination username password"
else
  # Reboot the host if the file-system is READ-ONLY
  sudo reboot
fi
```
