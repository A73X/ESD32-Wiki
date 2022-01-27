==========
BLOODHOUND
==========

**Source** : https://www.hackingarticles.in/active-directory-enumeration-bloodhound/

Introduction
============

BloodHound est programmé pour générer des graphiques qui révèlent les secrets et les relations au sein d'un réseau Active Directory. 
BloodHound prend également en charge Azure. 
BloodHound permet aux attaquants d'identifier des chemins d'attaque complexes qu'il serait impossible d'identifier autrement. 
L'équipe bleue peut utiliser BloodHound pour identifier et corriger ces mêmes schémas d'attaque.

Linux Installation
==================

Quelques guides et méthodes multiples peuvent vous aider à configurer et installer Bloodhound sur votre machine hôte. 
Nous allons suivre les documents officiels de BloodHound qui peuvent être trouvés sur leur `GitHub`_ mais en affinant le processus. Comme toujours avant d'installer un outil sur votre machine Linux, il est recommandé d'effectuer une mise à jour et de mettre à niveau vos paquets logiciels. 
De plus, si **Java** n'est pas installé sur votre machine, installez-le pour continuer. 
Nous ne procéderons pas à l'installation de **Java** car nous travaillons sur Kali Linux qui est préinstallé avec Java. 
La configuration de Bloodhound est un processus en 3 étapes. BloodHound a une interface graphique, un scrapper de données et une base de données neo4j. 
Cela signifie que nous devons les configurer individuellement.  
Nous commençons par l'interface graphique de Bloodhound qui peut être installée directement en utilisant la commande ``apt``.

.. _Github: https://github.com/BloodHoundAD/BloodHound


.. code-block:: bash
  
  apt install bloodhound


Ensuite, nous devons configurer le service **neo4j** qui contiendra les données qui peuvent être représentées sous forme de graphique. Lorsque nous avons exécuté ``apt install bloodhound``, il a installé **neo4j** avec lui. 
Si ce n'est pas le cas dans votre cas, vous pouvez toujours le télécharger en exécutant ``apt install neo4j``. 
Maintenant, nous devons configurer l'authentification et d'autres paramètres sur le service neo4j. 
Pour ce faire, nous lançons l'instance de la console neo4j. 
Elle hébergera l'interface distante à laquelle on peut accéder à l'aide d'un navigateur Web. 
Par défaut, elle est hébergée sur le port **7474**.

.. code-block:: bash
  
  neo4j console

En entrant l'URL : http://localhost:7474 nous avons accès à l'interface distante.
Elle comporte quelques valeurs pré-remplies et quelques champs noirs. 
Ici, entrez un nom d'utilisateur, nous choisissons le nom d'utilisateur **neo4j** et entrez un mot de passe. 
Après avoir entré les informations suivantes, vous serez en mesure de vous connecter à la base de données neo4j.

.. note:: Avant de se connecter, il vous demandera de changer le mot de passe car il s'agit de votre première connexion. Entrez le mot de passe de votre choix et déplacez-vous pour connecter l'interface à distance neo4j.

Maintenant que le service **neo4j** est opérationnel, nous pouvons lancer l'interface graphique de Bloodhound. 
Pour l'exécuter, il suffit de taper bloodhound dans votre terminal et d'appuyer sur la touche Entrée. 
Vous pouvez également essayer de chercher bloodhound dans la liste des applications installées dans le menu de Kali Linux et l'exécuter directement à partir de là.

Dès que l'interface graphique de BloodHound démarre, elle demande un ensemble d'informations d'identification que nous venons de définir dans la configuration de neo4j. 
Utilisez le même ensemble d'informations d'identification et vous pourrez vous connecter à cette interface. 

.. important:: Vous pouvez enregistrer vos informations d'identification afin de ne pas avoir à vous connecter à chaque fois que vous souhaitez utiliser Bloodhound.

Après s'être connecté à l'interface graphique de BloodHound, un écran blanc s'ouvre avec quelques boutons d'interaction sur le côté droit et une boîte de recherche sur le côté gauche avec quelques modules qui y sont attachés. 
C'est là que se termine la configuration de l'interface graphique. Comme nous l'avons vu dans l'introduction, le Bloodhound représente les données sous forme de jolis graphiques et recherche les chemins possibles. 
Pour tracer les graphiques, il a besoin des données du domaine. 
Ces données peuvent être extraites en utilisant un scrapper de données que nous devons maintenant installer.

.. important:: Cela peut prêter à confusion mais pour que les choses soient claires une fois de plus. Nous avons installé BloodHound GUI dans les étapes précédentes qui tracent des graphiques basés sur les données. Maintenant, nous installons l'outil qui va extraire les données du domaine.

Comme il est fait en Python, nous pouvons utiliser **pip3** pour installer bloodhound comme ceci :

.. code-block:: bash
  
  pip3 install bloodhound

Extraction de données d'un domaine
==================================

Nous allons exécuter le python Bloodhound que nous venons d'installer en utilisant pip3 et extraire les données du domaine. 
Il est bon de mentionner qu'ici la configuration du domaine est telle que nous avons connecté le contrôleur de domaine, les clients et notre machine attaquante essentiellement dans le même réseau. 
Pour obtenir des données du domaine, n'importe quel utilisateur peut être utilisé. 
Nous utiliserons le compte Administrateur pour extraire un maximum de données pour cette énumération. 
Dans un scénario réaliste, vous vous retrouverez avec un utilisateur normal, puis vous lancerez l'outil Bloodhound et ensuite vous utiliserez les données énumérées pour atteindre l'Administrateur. Nous devons fournir les paramètres suivants pour extraire les données du domaine : nom d'utilisateur, mot de passe, serveur de nom (adresse IP du contrôleur de domaine), domaine et données que nous voulons extraire (nous utilisons "Tout" pour extraire le maximum de données du domaine). 
Les données extraites seront sous la forme de fichiers **.json** qui seront créés sur la base des requêtes qui ont été exécutées sur le domaine à la recherche de chemins et de permissions possibles de divers groupes et utilisateurs.

.. code-block:: bash
  
  bloodhound-python -u administer -p Ignite@987 -ns 192.168.1.172 -d ignite.local -c All

.. note:: Après avoir lancé **bloodhound-python**, vous aurez des fichiers **json** dans votre répertoire courant. Il est possible de les vérifier avec la commande ``ls``. Pour les analyser dans l'interface graphique de BloodHound, vous devez glisser et déposer ces fichiers **json** dans l'interface graphique. Comme on peut l'observer dans l'image ci-dessous, nous avons les fichiers **computers.json**, **domains.json**, **groups.json**, **users.json**.

Maintenant que tous les fichiers **json** ont été téléchargés, l'interface graphique de BloodHound peut commencer à tracer les graphiques. Bloodhound fonctionne de la manière suivante : maintenant qu'il est chargé avec les fichiers de données du domaine, vous pouvez soit entrer des requêtes pour tracer des graphiques, soit utiliser les requêtes pré-construites. 
Dans ce guide, nous allons utiliser les requêtes pré-construites.


Enumération avec BloodHound
===========================

Commençons notre énumération par les requêtes analytiques préétablies. La première d'entre elles que nous utilisons est la requête **Find all Domain Admins**. 
Cette requête va récupérer tous les administrateurs de domaine qu'elle peut trouver dans sa base de données et les représenter sur le graphique comme indiqué dans l'image ci-dessous. 
Puisque notre domaine n'a qu'un seul Admin de domaine, il montre un nœud et ensuite 2 groupes sous cet Admin de domaine.

Le suivant est assez intéressant. 
Elle s'intitule **Find Shortest Paths to Domain Admins**. 
Cela signifie que BloodHound va tracer les Admins de domaine et les utilisateurs qu'il peut trouver et ensuite nous serons en mesure de déduire quel type de chemin nous voulons prendre pour continuer à exploiter afin que nous puissions atteindre l'Admin de domaine avec le moins de résistance. 
Dans notre cas, il y a 4 chemins parmi lesquels deux (nœuds jaunes) sont équidistants. 
Cela signifie que nous pouvons utiliser n'importe lequel d'entre eux pour atteindre les administrateurs de domaine et nous savons qu'il existe une autorisation d'écriture générique que nous pouvons exploiter pour atteindre l'administrateur de domaine. 
C'est ainsi que, dans un environnement de domaine particulièrement compliqué et vaste, l'attaquant peut se frayer un chemin dans le désordre et obtenir l'accès à l'administrateur de domaine.

Une autre requête préconstruite que nous allons utiliser est la requête **Find AS-REP Roastable Users (DontReqPreAuth)**. 
Le roasting **AS-REP** est une technique offensive contre Kerberos qui permet de récupérer les hashs de mots de passe des utilisateurs qui ne nécessitent pas de pré-authentification. 
Si l'utilisateur a activé l'option **Do not use Kerberos pre-authentication**, un attaquant peut récupérer un **AS-REP Kerberos** crypté avec le mot de passe **RC4-HMAC** de l'utilisateur et il peut tenter de craquer ce ticket hors ligne.

La pré-authentification est l'étape initiale de l'authentification Kerberos, qui est gérée par le serveur d'authentification **KDC** et est destinée à empêcher les attaques par **force brute**.

.. note:: La conclusion à laquelle nous sommes arrivés d'après notre dénombrement avec BloodHound est que l'utilisateur X est vulnérable à AS-REP Roasting. Cette affirmation peut être vérifiée en parcourant les utilisateurs et les ordinateurs d'Active Directory, puis en descendant dans les propriétés de l'utilisateur X. Dans la fenêtre des propriétés de l'utilisateur X, il y a un onglet Compte. Dans l'onglet Compte, nous pouvons voir que l'utilisateur X ne nécessite pas de préauthentification Kerberos.

En savoir plus : `AS-REP Roasting`_

.. _AS-REP Roasting: https://www.hackingarticles.in/as-rep-roasting/

Une autre attaque que nous pouvons énumérer à l'aide du BloodHound est l'attaque DC Sync. Cette attaque permet à un attaquant de reproduire le comportement d'un contrôleur de domaine (DC). En général, il se fait passer pour un contrôleur de domaine et demande à d'autres DC des données d'identification des utilisateurs via GetNCChanges. Mais le compte compromis doit être un membre des administrateurs, Admin de domaine ou Admin d'entreprise pour récupérer les hachages de mots de passe des autres contrôleurs de domaine.

A partir du graphique BloodHound, nous pouvons voir que l'utilisateur X est vulnérable à cette attaque.

.. note:: La conclusion à laquelle nous sommes arrivés d'après notre énumération avec BloodHound est que l'utilisateur X est vulnérable à l'attaque DCSync. Cette affirmation peut être vérifiée en parcourant les utilisateurs et les ordinateurs de l'Active Directory et en descendant ensuite dans les propriétés de l'utilisateur X. Dans la fenêtre des propriétés de l'utilisateur X, il y a un onglet Member Of. Dans l'onglet Member Of, nous pouvons voir que l'utilisateur X fait partie de Domain Admins, ce qui le rend vulnérable à une attaque DC Sync.

En savoir plus : `DCSync Attack`_

.. _DCSync Attack: https://www.hackingarticles.in/credential-dumping-dcsync-attack/

La prochaine énumération que nous allons effectuer en utilisant BloodHound est la liste de tous les comptes **Kerberoastable**. Le **Kerberoasting** est une technique qui permet à un attaquant de voler le ticket **KRB_TGS**, qui est crypté avec **RC4**, pour forcer brutalement le hachage des services applicatifs afin d'en extraire le mot de passe. D'après le graphique tracé par le BloodHound, on peut dire que **KRBTGT** et **SVC_SQLSERVICE** sont les deux utilisateurs qui sont vulnérables à cette attaque.

Il existe un grand nombre de requêtes personnalisées et de requêtes intégrées qui peuvent être utilisées pour effectuer des recensements avec BloodHound. Une fois l'énumération et l'analyse terminées, vous pouvez effacer les valeurs de la base de données et ajouter de nouveaux fichiers **json** de valeurs différentes en parcourant l'onglet Database Info de l'interface graphique de BloodHound et en cliquant sur le bouton **Clear Database** (effacer la base de données).

BloodHound sur Windows
======================

Il est également possible d'analyser et de dénombrer BloodHound directement à partir d'une machine Windows. Cela peut être utile dans les environnements qui limitent le déploiement de Kali Linux et d'autres outils pour attaquants. Le processus reste plus ou moins le même. Nous devons configurer l'ingestion de données, l'interface graphique de BloodHound et la base de données **neo4j** sur une machine Windows, comme nous l'avons fait précédemment pour Linux. Pour commencer, nous allons installer l'injecteur de données pour Windows qui s'appelle Sharphound. La différence entre l'investisseur Linux et l'investisseur Windows est qu'au lieu de créer des fichiers **json** , le SharpHound crée un fichier compressé qui inclut des fichiers CSV. La méthode de collecte des données reste la même.

SharpHound peut être téléchargé sur `GitHub.`_

.. _GitHub.: https://github.com/BloodHoundAD/SharpHound3

Extraction de données depuis un domaine
=======================================

Lorsque l'attaquant exécute le SharpHound sur la machine connectée à Domain, il a créé un fichier compressé avec le nom BloodHound.

.. code-block:: bash

  sharphound.exe
  dir

Windows Installation
====================

De la configuration Linux, nous nous souvenons que BloodHound nécessite le service **neo4j**. Il peut être téléchargé pour Windows, puis exécuté à l'aide d'un fichier batch fourni avec le paquet d'installation. Ce service fonctionne également sur le **port 7474**.

.. code-block:: bash

  Download neo4j Windows
  dir
  neo4j.bat console

Nous prenons l'URL de la console **neo4j** et l'ouvrons dans notre navigateur Web. Si nous nous souvenons bien, nous devons à ce stade configurer les informations d'identification qui seront également utilisées pour le BloodHound.

.. note:: Après la configuration du mot de passe, nous essayons de nous connecter pour la première fois dans le service neo4j. Pour cela, nous devons redonner le mot de passe comme indiqué dans la capture d'écran ci-dessous.

Après avoir réinitialisé le mot de passe et nous être connectés au service **neo4j** que nous avons configuré sur notre appareil Windows, nous pouvons accéder au service à partir de notre navigateur Web. Le panneau nous indique que nous sommes connectés avec succès au service **neo4j**.

Maintenant que nous avons installé **l'Ingestor SharpHound** et le service **Neo4j** sur notre appareil Windows, il nous reste à installer l'interface graphique de BloodHound. C'est assez simple puisque nous avons un exécutable pour cela. Nous utilisons l'invite de commande Windows pour exécuter l'interface graphique comme le montre l'image ci-dessous.

Télécharger `BloodHound GUI Windows`_

.. _BloodHound GUI Windows: https://github.com/BloodHoundAD/BloodHound/releases

.. code-block:: bash

  dir
  BloodHound.exe

L'interface graphique de BloodHound est exécutée et nous avons un panneau de connexion comme le montre l'image ci-dessous. Nous utilisons les informations d'identification que nous avons définies dans la configuration de **neo4j** pour nous connecter à BloodHound GUI.

Enumération avec BloodHound
===========================

À partir de là, le processus d'analyse et de dénombrement sur BloodHound est le même que celui dont nous avons parlé ci-dessus. Grâce à cet ensemble d'instructions, vous êtes en mesure d'exécuter BloodHound sur un appareil Windows.

SharpHound avec PowerShell
==========================

Dans le prolongement du processus d'énumération de BloodHound sur Windows, nous souhaitons également démontrer le processus qui peut être suivi par les professionnels de la sécurité lorsqu'ils souhaitent utiliser le SharpHound sur Windows par le biais de PowerShell. Cela peut être fait car nous disposons des scripts PowerShell pour le SharpHound Ingestor. Après avoir contourné la restriction des scripts sur PowerShell, nous importons les modules du script PowerShell de SharpHound. Il contient un cmdlet du nom de Invoke BloodHound. Il peut être utilisé pour collecter des données sur la machine cible. Ceci est utile dans le scénario où il n'est pas possible d'exécuter un exécutable sur la machine cible. 

Télécharger le script `PowerShell SharpHound`_

.. _PowerShell SharpHound: https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors

.. code-block:: bash

  powershell -ep bypass
  Import-Module .\SharpHound.ps1
  Invoke-BloodHound -CollectMethod All

SharpHound avec PowerShell Empire
=================================

Le script SharpHound que nous avons utilisé précédemment sur PowerShell peut également être trouvé dans Kali Linux. Il est situé dans l'Empire PowerShell. Après avoir réussi à prendre pied sur un appareil faisant partie d'un domaine, l'attaquant peut utiliser directement l'Empire pour exécuter SharpHound et en extraire les données. Pour ce faire, nous devons utiliser le module Bloodhound dans les modules Situational Awareness de l'Empire. Après son exécution, il indique à l'attaquant l'emplacement où se trouvent les fichiers csv de données.

.. code-block:: bash

  usemodule situtationa_awareness/network/bloodhound
  execute

Un attaquant peut utiliser la commande download sur PowerShell Empire pour transférer les fichiers csv vers la machine hôte, c'est-à-dire Kali Linux. Nous pouvons utiliser les multiples fichiers csv de la même manière que nous avons utilisé les fichiers **json** précédemment pour tracer des graphiques et énumérer un Active Directory.

.. code-block:: bash

  ls
  download group_memberships.csv
  download local_admins.csv
  download trusts.csv
  download user_sessions.csv

Conclusion
==========

Ce guide a été créé par nos soins afin que les professionnels de la sécurité, qu'ils appartiennent à l'équipe rouge ou à l'équipe bleue, puissent déployer, configurer et utiliser BloodHound pour recenser les déploiements Active Directory. Il s'agit d'un outil très utile qui peut être utilisé pour comprendre les mécanismes d'un réseau Active Directory et ensuite utiliser ces informations pour élever les privilèges ou exploiter le réseau.

Author: Pavandeep Singh is a Technical Writer, Researcher, and Penetration Tester.