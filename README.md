# FTP to SMB
FTP to SMB file transfer script. Including email reporting. I wrote this script because I needed a way to transfer files from various ftp accounts to network shares on a Windows network. Since ADDC does not allow to map a ftp drive at logon, this was my next solution so I could deploy those files to many users centrally. My other solution was to go at each desk and setup the FTP on each machine locally.

## Details
The script works as followed. The FTP and SMB shares are mounted. A rsync transfer is performed from the FTP share to the SMB share. Once the rsync transfer has completed, the script loops through each transferred files to validate their md5sum. This allows the script to confirm the integrity of each files that were transferred. Once the script has validated a file, it will move the original files on the FTP share to an archives sub-directory and moves the transferred file into a validated sub-directory. Once the transfers have completed, the report is generated and sent via email if requested. To ensure that the credentials are not retrieved by someone looking into the log file, the script will automatically change any references of any of the settings from the logfile.

## Build docs
``` bash
git clone https://github.com/LouisOuellet/ftp-to-smb.git
cd ftp-to-smb/docs
make html
```

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

## Configure getfiles service
Enabling the service will allow getfiles to constantly sync the 2 shares together.

```BASH
sudo ln -s /opt/php-pdf/init /etc/init.d/getfiles
sudo systemctl daemon-reload
sudo systemctl enable getfiles
sudo systemctl start getfiles
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
