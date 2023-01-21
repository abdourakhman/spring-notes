# SERVICES SPRING CLOUD

## SERVICE CONFIGURATION
---------------------

--> Côté serveur: installer la dépendance config serveur de spring
                -activer  la configuration avec @EnableConfigServer dans le fichier principale
                -configurer le service dans son fichier application.properties:
                -configurer le port avec server.port=8888
                -specifier  l'adresse ou se trouve le fichier de configuraion dossier loacale/Git
                spring.cloud.config.server.git.uri= si c'est un dossier local : file://${user.home}/cloud-conf
                                                    si c'est un repository distant urlRepo ex: https://github.com/abdourakhman/cloud-config
                                                            
                Dans ce fichier de configuration, sera contenu tous les fichiers de configurations  des micro-services ex: custumer-sevice.properties 
                Les configurations communes aux micro- services seront stockés dans le fichier application.properties' 
                

-->Côté client: installer la dépendance config client de spring
                -renommer le fichier application.properties en bootstrap.properties pour indiquer à spring de démarrer sur ce fichier de conf
                -Dans son fichier bootstrap.properties, on configure le nom du micro service en lui donnant le même nom que le fichier de configuration du micro-service
                ex: si le fichier de configuration exposé sur git ou le dossier local est company.properties alors spring.application.name=company

                -Une fois que le micro service démarre on lui spécifie ou il doit recupérer le reste de  sa configuraion
                spring.cloud.config.uri=http://localhost:8888  (serveur de configuration)

                - la configuration du micro-service peut changer mais le micro service n'est pas au courant, C'est la qu'intervient le service Actuator
                - 'Installer la dépendance Actuator de spring joignable sur localhost/actuator 
                   pour actualiser il s'uffit de lancer une requete post avec un corps vide en json sur le endpoint localhost/actuator/refresh'


## SERVICE D'ENREGISTREMENT
---------------------