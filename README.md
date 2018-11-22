
# SRS Capture The Flag
Dans le cadre de l'UV SRS à l'IMT Lille Douai, Advens nous a proposé de participé à un CTF (Capture The Flag) afin de nous former aux techniques de ethical hacking.
Ce document nous permettra d'expliquer les méthodes que nous avons employé pour résoudre les différents challenges de ce CTF. 


## Web

### Baby Web
Ce challenge commence sur une page d'authentification comprenant 2 champs : login/password. Afin de forcer le passage de cette page, nous avons utilisé une **injection SQL** en complétant l'un des champ avec le code suivant : `'OR 1=1'`.
> En effet, en supposant que la requête SQL executée du coté serveur est du type `SELECT * from admins WHERE login='$login' AND password='$password'`. Notre injection permettra d'avoir le code suivant exécuté du coté serveur et donc de forcer l'accès : `SELECT * from admins WHERE login=''OR 1=1'' AND password=whatever`.

Cela nous donne accès à une page d'upload de fichier sur le serveur. Afin de gagner un accès au serveur, nous allons injecter un shell via ce service d'upload. Nous utiliserons le webshell de  [b374k](https://github.com/b374k/b374k).
On génère un webshell grâce aux commandes suivantes : 
```
> git clone  https://github.com/b374k/b374k
> cd b374k
> php  -f  index.php  --  -o  shell.php  -p  password123
```
Il nous suffit donc du l'uploader sur le serveur pour avoir accès au serveur. Nous obtenons un shell avec les droits de l'application web. Ces droits nous permettent de mener nos investigations sur l'ensemble du serveur et notament dans le repertoire `/root` où nous trouverons le premier flag [[screenshot]](https://raw.githubusercontent.com/migonidec/SRS_CTF/master/images/web/babyWeb/babyWeb_4.PNG). 
```
> cat /root/flag.txt
advens{easy_baby_web}
```

### Pentester Adventures
Ce challenge nous amène sur un site bloqué par la NSA et donc non-indexé. L'indexation d'un site est géré par le fichier `robots.txt` qui est situé à la racine du site [doc](http://robots-txt.com/). Lors de la consultation de ce document, on voit que le site comprend 2 pages [[screenshot]](https://raw.githubusercontent.com/migonidec/SRS_CTF/master/images/web/pentesterAdventures/pentesterAdventures_1.PNG) : 
 - `/dict.txt` : une suite d'identifiants et de mots de passe au format texte
 - `/nsa-secure-scripts.php` : une page d'identification avec un couple login:mdp. Nous allons tenter d'utiliser le fichier `dict.txt` pour brute-forcer l'accès à cette page.

La page d'identification renvois une erreur *wrong login* lorsque le login ne correspond pas à un utilisateur connu. Nous pouvons donc supposer qu'en testant tous les identifiants, nous arrivrons à un message du type *wrong password*, et donc une taille de réponse différente.
Avec le logiciel Burp, nous allons intercepter une requête d'identification et faire varier uniquement le login en se servant du fichier `dict.txt`. On compare ensuite les tailles des réponses pour identifier le bon login [[screenshot]](https://raw.githubusercontent.com/migonidec/SRS_CTF/master/images/web/pentesterAdventures/pentesterAdventures_5.PNG). De la même manière, on peux identifier le mot de passe de la page pour arriver au coupe `qwertyui:security` [[screenshot]](https://raw.githubusercontent.com/migonidec/SRS_CTF/master/images/web/pentesterAdventures/pentesterAdventures_7.PNG). 

On accède enfin à une page qui liste 2 liens : une capture d'un ping local et un fichier de configuration php à l'adresse suivante `/nsa-secure-scripts.php?script=scripts/phpinfos.php`
//TODO

## Forensic

### Simple HTTP
Cette épreuve nous met à disposition une capture de trames où 2 machines échanges des données. 
On suppose que l'une des trame doit contenir un mot de passe ou un flag. En filtrant les trames par protocole, il est facile de repérer un paquet `HTTP/POST`. En inspectant le contenu de cette trame, on trouve rapidement un fichier `flag.txt` encapsulé un paquet `MIME`.  On trouve donc le flag en clair dans cette partie de la trame [[screenshot]](https://raw.githubusercontent.com/migonidec/SRS_CTF/master/images/forensic/simpleHTTP/simpleHTTP_1.PNG).
