# SSH reverse tunneling  

Au cégep, nous avons un ordinateur qui est derrière un par feu qui a accès à l'extérieur, mais qu'on ne peut pas rejoindre de l'extérieur. Si l'on peut établir une connexion de notre ordinateur du cégep vers notre ordinateur de la maison (local), le tunnel SSH inversé (reverse ssh tunnel) vous permet d'utiliser cette connexion établie pour configurer une nouvelle connexion de votre ordinateur local vers l'ordinateur du cégep.

1. On fait un port forwarding sur notre routeur de maison du port 22 extérieur vers notre machine au port 22 : \*:22 -> 192.168.1.100:22  
2. On test a connexion à partir de l'ordinateur du cégep (seulement le port 22 fonctionnait) : <code>ssh clroy@votremaison.ddns.net</code> (voir noip.com)  
3. Il faut pour commencer ouvrir le tunnel de l'ordinateur du cégep vers la maison en reverse ssh tunneling : <code>ssh -R 43022:localhost:22 clroy@votremaison.ddns.net</code>  
4. Une fois le tunnel établi, on peut se connecter sur l'ordinateur du cégep à partir de la maison. De l'ordinateur de la maison, on ouvre un shell et on se connecte avec : <code>ssh user01@localhost -p 43022</code>  
5. On peut automatiser la connexion avec un script et en copiant la clé ssh de votre ordinateur à celui du cégep.  

### Références

[https://www.howtogeek.com/428413/what-is-reverse-ssh-tunneling-and-how-to-use-it/](https://www.howtogeek.com/428413/what-is-reverse-ssh-tunneling-and-how-to-use-it/)  

[https://www.pugetsystems.com/labs/hpc/How-To-Use-SSH-Client-and-Server-on-Windows-10-1470/](https://www.pugetsystems.com/labs/hpc/How-To-Use-SSH-Client-and-Server-on-Windows-10-1470/)  

[https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh\\_keymanagement](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)