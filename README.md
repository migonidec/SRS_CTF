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
Il nous suffit donc du l'uploader sur le serveur pour avoir accès au serveur. Nous obtenons un shell avec les droits de l'application web. Ces droits nous permettent de mener nos investigations sur l'ensemble du serveur et notament dans le repertoire `/root` où nous trouverons le premier flag [screenshot](https://raw.githubusercontent.com/migonidec/SRS_CTF/master/images/web/babyWeb/babyWeb_4.PNG). 
```
> cat /root/flag.txt
advens{easy_baby_web}
```