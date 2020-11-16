Usage
=====
If no FTP and/or SMB is provided, the script will request the information by GUI/CLI depending on your setup.

.. code-block:: bash

   $ ./getfiles
   Unable to retrieve settings.json.
   File not found.

   Usage: ./getfiles [options]

   Options:

   -v                     => Enable Reporting Mode
                             Input commands sent are stored in
   -e                     => Compile errors and warnings after execution
   -d                     => Set report destination email
   -r                     => Send report via email
   -c                     => Disable all formatting
   -f                     => Specify a FTP ex: "ftp.domain.com username password"
   -s                     => Specify a SMB ex: "0.0.0.0 shareName destinationDirectory username password"
