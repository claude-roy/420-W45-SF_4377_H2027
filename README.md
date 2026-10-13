# Installation des serveurs et des services 420-W45-SF
## UBR4375, hiver 2026  
### PROFESSEURS
Emmanuel Garreau <egarreau@csfoy.ca>  
Claude Roy <clroy@csfoy.ca>  

### CALENDRIER

Ce calendrier est donné à titre indicatif et peut être modifié en fonction des circonstances. Dans le cas des examens, les étudiants seront toujours avisés dans un délai raisonnable.  

|Cours	|Date cours |Contenu|Exercices <br> Évaluations|  Remise exercice   |
|----------:|:-------------:|----------------|:------:|------:|
|1|	10 mars<br>P-409<br>Emmanuel |Présentation du plan de cours <br> Présentation des concepts de bases <br> Exercice 1 - Installation Ubuntu|[Ex1](Exercices/Exercice01_InstallationClient.md) **(ER)**||
|2|	13 mars<br>P-408<br>Emmanuel|Installation d'un serveur Linux avec LVM et configuration réseau	 |[Ex2](Exercices/Exercice02_InstallationServeur.md) et [Ex3](Exercices/Exercice03_GestionLVM.md)||
|3| 17 mars<br>P-408<br>Emmanuel  |Accès distant, les protocoles SSH/SCP |[Ex4](Exercices/Exercice04_PriseEnMainSrv.md)||
|4| 20 mars<br>P-408<br>Emmanuel |Admin système les droits et accès |[Ex5](Exercices/Exercice05_AdminSysLinux.md) **(ER)**||
|5|	 24 mars<br>P-408<br>Emmanuel |Admin système Noyau, processus et logs |[Ex6](Exercices/Exercice06_InstallationEnvTest.md)||
|6|	31 mars<br>P-409<br>Emmanuel  |**TP 1 - Procédure d'installation d'un serveur** |**TP1**||
|7|	10 avril<br>P-408<br>Emmanuel  |Docker	 |[Ex7](Exercices/Exercice07_PriseEnMainConteneur.md) **(ER)**||
|8|	14 avril<br>P-408<br>Emmanuel  |Docker file	 |[Ex8](Exercices/Exercice08_DockerImage.md) ||
|9|	 17 avril<br>P-408<br>Emmanuel  |Docker Réseau et volume|[Ex9](Exercices/Exercice09_DockerRzEtVolume.md) ||
|10| 21 avril<br>P-408<br>Emmanuel |**TP 2 - Docker** |**TP2**||
|11| 24 avril<br>P-408<br>Emmanuel |**TP 2 - Docker** | ||
|12| 1 mai<br>P-408<br>Emmanuel |Docker compose |[Ex10](Exercices/Exercice10_DockerCompose.md)||
|13| 5 mai<br>P-408<br>Emmanuel |Présentation du service de résolution de nom DNS |[Ex11](Exercices/Exercice11_DNS.md)||
|14| 8 mai<br>P-408<br>Emmanuel|Révision	 |||
|15| 12 mai<br>P-408<br>Emmanuel |**Examen partiel** |**Examen 1**||  
|16| 13 mai<br>P-408<br>Claude |Apache, installation, config de base|[Ex12](Exercices/Exercice12_Apache.md) ||
|17| 14 mai<br>P-409<br>Claude |Apache, modules et SSL	 |[Ex13](Exercices/Exercice13_Apache_modules-SSL.md) **(ER)**||
|18|  21 mai<br>**Teams**<br>Claude|Apache, installation d’un site Web et site virtuel|[Ex14](Exercices/Exercice14_Apache_SiteVirtuel.md) ||
|19| 22 mai<br>P-409<br>Claude |Nginx, installation, config de base|[Ex15](Exercices/Exercice15_nginx.md) ||
|20| 25 mai<br>P-408<br>Claude |Nginx, configurations avancées (reverse proxy et load balancer) |[Ex16](Exercices/Exercice16_nginx_LB.md) ||  
|21| 27 mai<br>P-408<br>Claude |**TP3 - Mise en place d’un service Web (Docker Compose)**|[TP3](TPs/TravailPratique03.md) ||
|22|  2 juin<br>**Teams**<br>Claude|**TP3 - Mise en place d’un service Web (Docker Compose)**|||
|23| 5 juin<br>P-408<br>Claude  |**TP3 - Mise en place d’un service Web (Docker Compose)**|||
|24|  9 juin<br>**Teams**<br>Claude|Automatisation et Ansible|[Ex17](Exercices/Exercice17_AnsibleMiseEnPlace.md) ||
|25|  11 juin<br>**Teams**<br>Claude|Automatisation mode AdHoc|[Ex18](Exercices/Exercice18_AnsibleModeAdHoc.md) ||
|26|  12 juin<br>**Teams**<br>Claude|Travail sur les exercices|||
|27|  2 juillet<br>**Teams**<br>Claude|Automatisation PlayBook|[Ex19](Exercices/Exercice19_AnsiblePlaybook.md) **(ER)**||
|28|  8 juillet<br>**Teams**<br>Claude |Travail sur les exercices|||
|29|  9 juillet<br>**Teams**<br>Claude|**Évaluation finale à caractère synthèse** |[EFCS](TPs/EFCS.md)||
|30| 13 juillet<br>P-408<br>Claude |**Évaluation finale à caractère synthèse**|||

### Récapitulatif de la pondération des évaluations sommatives

|Évaluation | Pondération (en %) |
|:-------------|:------:|
|TP1 – Procédure d'installation d'un serveur| 10|
|TP2 – Docker	|15|
|TP3 - Mise en place d’un service Web (Docker compose)| 15|
|Examen partiel	| 20|
|Évaluation finale à caractère synthèse	 |30|
|Exercices à remettre	|10|
