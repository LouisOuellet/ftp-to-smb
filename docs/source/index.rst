FTP to SMB Documentation
========================

FTP to SMB file transfer script. Including email reporting. I wrote this script because I needed a way to transfer files from various ftp accounts to network shares on a Windows network. Since ADDC does not allow to map a ftp drive at logon, this was my next solution so I could deploy those files to many users centrally. My other solution was to go at each desk and setup the FTP on each machine locally.

Details
-------
The script works as followed. The FTP and SMB shares are mounted. A rsync transfer is performed from the FTP share to the SMB share. Once the rsync transfer has completed, the script loops through each transferred files to validate their md5sum. This allows the script to confirm the integrity of each files that were transferred. Once the script has validated a file, it will move the original files on the FTP share to an archives sub-directory and moves the transferred file into a validated sub-directory. Once the transfers have completed, the report is generated and sent via email if requested. To ensure that the credentials are not retrieved by someone looking into the log file, the script will automatically change any references of any of the settings from the logfile.

.. toctree::
   :maxdepth: 2
   :caption: Table of Contents

   usage
   configure
   license
