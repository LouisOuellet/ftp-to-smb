# FTP to SMB
FTP to SMB file transfer script. Including email reporting. I wrote this script because I needed a way to transfer files from various ftp accounts to network shares on a Windows network. Since ADDC does not allow to map a ftp drive at logon, this was my next solution so I could deploy those files to many users centrally. My other solution was to go at each desk and setup the FTP on each machine locally.

## Details
The script works as followed. The FTP and SMB shares are mounted. A cp transfer is performed from the FTP share to the SMB share. 2 modes are available for the script. Default mode is sync. This will always transfer all files from the FTP to the SMB share. This is useful if you need all the files of the FTP and you don't need to keep track of the files you have handle. The second mode is archive. In this mode, getfiles will start by copying the files from the FTP to the SMB. And then, it will mv the files in the FTP to a subdirectory "archives". This allows someone to keep track of what file they have already handled without loosing the original file. If you enable the email reporting, getfiles will send an email notification every time it has run. getfiles can be run manually or using the included service file. To sync multiple FTP, simply clone ftp-to-smb in multiple directories. Don't forget to use unique names for each service.

## Changelog

 - Added support for FTPS transfer
 - A timeout was added to both mount command to prevent the script from getting stuck if an error occur during mounting.
 - Fixed the service file to be more dynamic. Specially when running multiple instances of the script.
 - Added the PIDs to the tmp folder instead of the /var/run folder.
 - Added an install script to make it easier to setup as a service on debian based distro.

## Usage
``` bash
$ ./getfiles
Unable to retrieve settings.json.
File not found.

Usage: ./getfiles [options]

Options:

-a                     => Enable Archive Mode
                          This mode moves the transferred file in an archive folder
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

## Configure getfiles service
Enabling the service will allow getfiles to constantly sync the 2 shares together.

```BASH
./getfiles -i
```

## Configuring getfiles settings
To support email reporting, you will need to setup a smtp account. So in the directory of getfiles create a file called settings.json and insert the following with the proper changes.

``` json
{
    "smtp":{
        "host": "fqdn.domain.com",
        "port": "465",
        "username": "username@domain.com",
        "password": "Password123"
    },
    "report":{
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
To enable archive mode on the service

``` json
{
    "archive":true
}
```
