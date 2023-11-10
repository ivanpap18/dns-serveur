
# Configuration de BIND pour `example.com` avec Docker

Ce référentiel contient la configuration de BIND (Berkeley Internet Name Domain) pour le domaine example.com. La configuration est divisée en deux fichiers principaux :

### Fichier `named.conf.options`

Le fichier `named.conf.options` spécifie les options de configuration globales pour le serveur BIND. Les serveurs de transfert de requêtes DNS sont configurés pour utiliser les adresses IP 8.8.8.8 et 9.9.9.9.

```bind
options {
    forwarders {
        8.8.8.8;
        9.9.9.9;
    };
};
```

### Fichier named.conf.local
Le fichier named.conf.local contient la configuration spécifique aux zones, indiquant le type de zone et le chemin du fichier de zone pour le domaine example.com.

```
zone "example.com" IN {
    type master;
    file "/etc/bind/example-com.zone";
};
```

### Fichier de zone example-com.zone
Le fichier de zone pour le domaine example.com (/etc/bind/example-com.zone) contient les enregistrements de ressources nécessaires, tels que les enregistrements SOA, NS, A, SRV et TXT pour les différents sous-domaines. Les enregistrements DNS pour un Replica Set consistent principalement en des enregistrements de type SRV et TXT. Voici comment vous pouvez les ajouter :

```
$ORIGIN example.com.

@               IN      SOA     ns.example.com. info.example.com. (
                            20231109    ; serial
                            12h         ; refresh
                            15m         ; retry
                            3w          ; expire
                            2h          ; minimum ttl
                            )

            IN      NS      ns.example.com.

ns          IN      A       10.5.0.5

mongodb1    IN      A       10.5.0.6
mongodb2    IN      A       10.5.0.7
mongodb3    IN      A       10.5.0.8
srv-test    IN      A       10.5.0.9

_mongodb._tcp.server.example.com.   IN      SRV     0 5 27017 mongodb1.example.com.
_mongodb._tcp.server.example.com.   IN      SRV     0 5 27017 mongodb2.example.com.
_mongodb._tcp.server.example.com.   IN      SRV     0 5 27017 mongodb3.example.com.

server.example.com. IN      TXT     "replicaSet=dbrs"
```

Dans cet exemple :

* Les adresses IP pour les nœuds MongoDB (mongodb1, mongodb2, mongodb3) sont définies en tant qu'enregistrements de type A.
* Les enregistrements SRV spécifient les détails de connexion pour chaque nœud du Replica Set.
* Les enregistrements TXT sont utilisés pour stocker des informations supplémentaires, comme le nom du Replica Set.

### Configuration Docker avec docker-compose.yml
Utilisez le fichier docker-compose.yml fourni pour déployer le conteneur BIND9 avec Docker. Assurez-vous d'adapter les volumes et d'autres paramètres en fonction de vos besoins spécifiques.
```
services:
  bind9:
    container_name: dns-serveur
    image: ubuntu/bind9:latest
    environment:
      - BIND9_USER=root
      - TZ=Europe/Paris
    # ports:
    #   - "53:53/tcp"
    #   - "53:53/udp"
    volumes:
      - ./config:/etc/bind
    networks:
      network:
        ipv4_address: 10.5.0.5
    restart: unless-stopped

networks:
  network:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.5
```

### Comment utiliser ce référentiel

1. Clonez ce référentiel vers votre machine hôte.
```
git clone https://github.com/votre-utilisateur/bind-example-config.git
```
2. Modifiez le fichier docker-compose.yml pour correspondre à votre configuration spécifique, en particulier les volumes et les paramètres réseau.

3. Lancez le conteneur BIND9 avec Docker Compose.
```
docker-compose up -d
```

### Exemple de comment vous pourriez tester la résolution DNS avec nslookup :

1. Assurez-vous que votre serveur DNS BIND9 est en cours d'exécution dans le conteneur Docker avec l'adresse IP 10.5.0.5.
2. Utilisez la commande nslookup sur votre machine locale pour interroger ce serveur DNS. Supposons que vous souhaitiez interroger l'enregistrement A pour le domaine server.example.com. Assurez-vous d'ajuster ces informations en fonction de votre configuration.
```
nslookup server.example.com 10.5.0.5
```
Cela devrait retourner les informations de résolution DNS pour le domaine server.example.com à partir du serveur DNS BIND9 que vous avez configuré.