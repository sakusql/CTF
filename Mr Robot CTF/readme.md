Voici un guide pas à pas de la façon dont j’ai résolu le défi CTF Mr. Robot que vous pouvez télécharger sur https://www.vulnhub.com/entry/mr-robot-1,151/

**Identification de l'adresse IP de la machine virtuelle**

Je fais tourner Mr. Robot sur une machine virtuelle. Après l'installation et le démarrage de la VM, je suis accueilli par un écran de connexion, et c'est tout. En examinant les paramètres de ma VM, sous "Description", je trouve mon premier indice : la machine semble utiliser **wordpress-4.3.1-0-ubuntu-14.04**.

Ma première tâche est d’identifier l’adresse IP de la VM. Pour cela, je dois d'abord trouver l'adresse IP de mon réseau local avec la commande suivante :
```
ip address show
```
l'adresse IP est donc 192.168.1.105/24.

Je peux maintenant utiliser **nmap** pour scanner cette plage et identifier l'IP de la VM avec la commande suivante :
```
nmap –sT 192.168.1.00/24
```
J'ai pu déterminer que ma VM utilise l'adresse IP **192.168.1.56** et que les ports 80 et 443 sont ouverts.

**Consulter le fichier robots.txt**

Maintenant que je connais l'IP, je consulte le fichier **robots.txt** en utilisant la commande :
```
curl 192.168.1.56/robots.txt
```
Je trouve deux fichiers intéressants : le fichier de la première clé et une liste de mots. J'utilise **curl** pour lire le fichier de la clé et **wget** pour télécharger le fichier de dictionnaire sur mon ordinateur.

**WPScan**

Étant donné que le site fonctionne sous WordPress, j'utilise **WPScan** pour recueillir des informations sur les vulnérabilités potentielles et essayer d'énumérer les noms d'utilisateurs.
```
wpscan –u 192.168.1.56 —enumerate users
```
Le scan m'a identifié plusieurs vulnérabilités et m'a fourni des liens de référence pour chacune d'elles. J'ai décidé de ne pas inclure les résultats du scan ici pour ne pas trop aider.

**Brute Force**

Je décide d'essayer une attaque par force brute sur le nom d'utilisateur et le mot de passe en utilisant le fichier de dictionnaire trouvé. Après quelques essais, je devine que le nom d'utilisateur pourrait être **elliot** (le personnage principal de la série Mr. Robot). Je confirme que le nom d'utilisateur est valide et utilise **wpscan** pour tenter une attaque brute force sur le mot de passe.
```
wpscan –u 192.168.1.56 —username elliot —wordlist fsocity.txt
```
Après environ 10 minutes, j'obtiens le mot de passe : **ER28–0652**. Maintenant, je peux me connecter !

**Ouvrir un shell**

Après m'être connecté au site, je tente de d'ouvrir un shell, mais je reçois un message d'erreur. J'utilise alors le plugin **WP Add Mime Types** pour permettre le téléchargement de fichiers PHP. Ensuite, je prépare un **reverse shell** que j'upload dans la bibliothèque multimédia de WordPress. Malgré l'erreur, le shell est bien ouvert.

**Configurer Netcat**

Pour utiliser le reverse shell, j'utilise **Netcat** sur ma machine :
```
nc –l –v –n –p 1234
```
Ensuite, je navigue vers l'URL du shell et je constate que j'ai établi une connexion. En parcourant les répertoires, je trouve le fichier **key-2-of-3.txt** dans le dossier **/home/robot**, mais je n’ai pas les permissions pour le lire. Cependant, je trouve un fichier **password.raw-md5** contenant le nom d’utilisateur **robot** et un hash md5.

Je parviens à cracker le hash en utilisant l'outil en ligne CrackStation, ce qui me donne le mot de passe **abcdefghijklmnopqrstuvwxyz**. Je l’utilise pour me connecter en tant que **robot** et je peux maintenant lire la deuxième clé.

**NMAP**

Je continue à explorer le système et je tombe sur une copie de **nmap** dans le répertoire **/usr/local**. En utilisant le mode interactif de **nmap**, je découvre que je peux exécuter des commandes shell. En utilisant **!sh**, je deviens **root**. Je change de répertoire vers **/root** et trouve la troisième clé.
```
cat key-3-of-3.txt
```
Et voilà ! J'ai terminé le défi !
