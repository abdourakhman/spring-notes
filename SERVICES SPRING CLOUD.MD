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


## SERVICE DE ROUTAGE
---------------------
Il s'agit d'un pare-feu applicatif qui permet de gérer les routages et les politiques de sécurité pour les services micro-services. Il peut être utilisé pour créer un point d'entrée unique pour l'ensemble de votre architecture de services, pour effectuer des opérations de routage basées sur les critères de métadonnées de service, pour gérer les politiques de sécurité et pour effectuer des opérations de transformation de requête et de réponse. En gros, il permet de centraliser la gestion des routes d'une application micro-services.

-->Côté serveur: installer la dépendance gateway server de spring cloud 
                 -ensuite allez dans le fichier de  configuration application .properties, donnez un nom et un port au service;
                 -puis définissez des routes. La définition de route peut être statique ou dynamique.
                 #### ROUTE STATIQUE:
                 exemple de configuration:
                  spring.application.name=GATEWAY-SERVICE
                  server.port=8008
                  spring.cloud.gateway.routes[0].id=r1
                  spring.cloud.gateway.routes[0].uri=http://localhost:8000/api/custumers
                  spring.cloud.gateway.routes[0].predicates[0]=Path=/api/custumer**
                  Ces propriétés définissent une route dans Spring Gateway. La propriété spring.cloud.gateway.routes[0].id définit l'ID de la route, qui peut être utilisé pour la référencer plus tard. La propriété spring.cloud.gateway.routes[0].uri définit l'URI de destination pour la route, c'est-à-dire l'adresse où les demandes seront redirigées. La propriété spring.cloud.gateway.routes[0].predicates[0] définit une prédicat pour la route, qui spécifie une condition pour que la route soit utilisée. Dans ce cas, la condition est que le chemin de la demande doit commencer par "/api/customer". Si cette condition est remplie, la demande sera redirigée vers l'URI de destination spécifié.
                  Pour véerifier, faire une requête http://localhhost:{port-gateway}/api/custumers

                  On peut aussi creer une classe de configuration RouteConfig, et dans cette classe on fait appel à l'annotation @Configuration.
                  Ensuite on crée une méthode qui retourne une RouteLocatot avec comme paramètre:
                  -un RouteLocatorBuilder et dans cette méthode on crée nos route. Par exemple
                  
                  @Configuration
                  public class RouteConfig(){
                     @Bean
                     public RouteLocator createRoute(RouteLocatorBulder rlb){
                        return rlb.routes().
                                       .route(r-> r.path("/api/custumers**").uri("http://localhost:8000"))
                                       .route(r-> r.path("/api/bills**").uri("http://localhost:8002"))
                                       .build(); 
                     }
                  }

                  Toutefois il faut aussi gérer le loadbalancing et pour cela il faut combiner le gateway avec Eureka server. Pour cela il ne faut plus mettre les adresses des URI en dur mais préciser le loadbalancer à utiliser avec lb suivi du nom du loadbalancer fourni par Eureka. ex: http://localhost:8000 --> lb://CUSTUMER-SERVICE
                  et pour finir il faut lier le gateway avec eureka a partir du fichier de configuration .properties avec:
                  eureka.client.service-url.default-zone=http://localhost:8761/eureka/
                  spring.cloud.gateway.discovery.locator.enabled=true
                  et ajouter @EnableDiscoveryClient sur le fichier principale
                  #### ROUTE DYNAMIQUE:
                  pour ce qui est des routes dynamiques, il faut creer une classe de configuration RouteConfig, et dans cette classe on fait appel à l'annotation @Configuration.
                  Ensuite on crée une méthode avec l'annotation bean. Ce bean va aller recuperer tous les clients eureka en utilisant ReactiveDiscoveryClient et avec l'objet
                  DiscoverLocatorPoperties on recupère tous les proprietés du client y compris les adresses ip. Donc à chaque fois qu'un micros service s'enregistre une route lui aussitôt réservée.  

                  @Bean
                   public DiscoveryClientRouteDefinitionLocator routeConfig(
                   ReactiveDiscoveryClient rdc, DiscoveryLocatorProperties dlp)
                  {
                     return new DiscoveryClientRouteDefinitionLocator(rdc, dlp);
                  }

                  url de test: http://localhost:8008/CUSTUMER-SERVICE/custumers