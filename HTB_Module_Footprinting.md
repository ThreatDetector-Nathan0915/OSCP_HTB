# FTP 
File transfer protocol is often used to share files within an org, it operaters on the application layer, similar to HTTP. Runs natively on port 21. TFTP is simpler than FTP (trivial) and it runs on UDP far more insicure than FTP.
```
Commands	Description
connect	Sets the remote host, and optionally the port, for file transfers.
get	Transfers a file or set of files from the remote host to the local host.
put	Transfers a file or set of files from the local host onto the remote host.
quit	Exits tftp.
status	Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on.
verbose	Turns verbose mode, which displays additional information during file transfer, on or off.
```

Default conf path to see what settings are availeble can be found at **/etc/vsftpd.conf**.

Default location where users are defined to be able to user FTP-
**/etc/ftpusers**
**Dangerous Settings**
```
anonymous_enable=YES	Allowing anonymous login?
anon_upload_enable=YES	Allowing anonymous to upload files?
anon_mkdir_write_enable=YES	Allowing anonymous to create new directories?
no_anon_password=YES	Do not ask anonymous for password?
anon_root=/home/username/ftp	Directory for anonymous.
write_enable=YES	Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?
```
if anon login is allowed the following command can be used to dowload all files
```bash
 wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```
# SMB Shares
```bash
└─$ sudo nmap -sC -sV  10.129.202.5 
445/tcp  open  netbios-ssn Samba smbd 4
└─$ smbmap -H 10.129.202.5                   
        sambashare                                              READ ONLY       InFreight SMB v3.1
└─$ smbclient //10.129.202.5/sambashare -N
smb: \> ls
  .                                   D        0  Mon Nov  8 08:43:14 2021
  ..                                  D        0  Mon Nov  8 10:53:19 2021
  .profile                            H      807  Tue Feb 25 07:03:22 2020
  contents                            D        0  Mon Nov  8 08:43:45 2021
  .bash_logout                        H      220  Tue Feb 25 07:03:22 2020
  .bashrc                             H     3771  Tue Feb 25 07:03:22 2020

smb: \contents\> ls
  .                                   D        0  Mon Nov  8 08:43:45 2021
  ..                                  D        0  Mon Nov  8 08:43:14 2021
  flag.txt                            N       38  Mon Nov  8 08:43:45 2021
  smb: \contents\> get flag.txt
getting file \contents\flag.txt of size 38 as flag.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)

└─$ ./enum4linux-ng.py 10.129.202.5 -A 
└─$ rpcclient -U "" -N 10.129.202.5
netname: sambashare
        remark: InFreight SMB v3.1
        path:   C:\home\sambauser\
        password:
rpcclient $> exit
```
