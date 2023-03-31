


### Lancer cette commande dans le dossier init

```bash
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgres > initdb.sql
```

###  docker compose file

```yaml
version: '2.0'

# networks
# create a network 'guacnetwork_compose' in mode 'bridged'
networks:
  guacnetwork_compose:
    driver: bridge

# services
services:
  # guacd
  guacd:
    container_name: dev-guacd_compose
    image: guacamole/guacd
    networks:
      guacnetwork_compose:
    environment:
      GUACD_LOG_LEVEL: debug
    restart: unless-stopped
    volumes:
    - ./drive:/drive:rw
    - ./record:/record:rw
  # postgres
  postgres:
    container_name: dev-postgres_guacamole_compose
    environment:
      PGDATA: /var/lib/postgresql/data/guacamole
      POSTGRES_DB: guacamole_db
      POSTGRES_PASSWORD: 'ChooseYourOwnPasswordHere1234'
      POSTGRES_USER: guacamole_user
    image: postgres:13.4-buster
    networks:
      guacnetwork_compose:
    restart: unless-stopped
    volumes:
    - ./init:/docker-entrypoint-initdb.d:z
    - ./data:/var/lib/postgresql/data:Z

  # guacamole
  guacamole:
    container_name: dev-guacamole_compose
    depends_on:
    - guacd
    - postgres
    volumes:
    - ./GUACAMOLE_HOME:/etc/guacamole
    environment:
      GUACAMOLE_HOME: /etc/guacamole
      GUACD_HOSTNAME: guacd
      POSTGRES_DATABASE: guacamole_db
      POSTGRES_HOSTNAME: postgres
      POSTGRES_PASSWORD: 'ChooseYourOwnPasswordHere1234'
      POSTGRES_USER: guacamole_user
    image: guacamole/guacamole
    links:
    - guacd
    networks:
      guacnetwork_compose:
    ports:
## enable next line if not using nginx
    - 8080:8080/tcp # Guacamole is on :8080/guacamole, not /.
    restart: unless-stopped
```

Lancer cette commande après avoir créer le fichier compose

```bash
docker compose up -d
```



### guacamole.properties

> [!warning] 
> Il ne faut pas oublier de mettre des retour à la ligne à la fin du fichier de config

```
openid-issuer: http://192.168.4.114:8090/realms/master
openid-authorization-endpoint: http://192.168.4.114:8090/realms/master/protocol/openid-connect/auth
openid-jwks-endpoint: http://192.168.4.114:8090/realms/master/protocol/openid-connect/certs
openid-client-id: guacamole
openid-redirect-uri: http://192.168.4.114:8080/guacamole/

openid-scope: openid profile email
openid-groups-claim-type: groups
openid-username-claim-type: preferred_username
extension-priority: openid

# Don't forget to space out EOF
# Ne pas oublier de mettre des retour à la ligne


```

Dans le cas ou l'on oublie ce fameux retour à la ligne, le fichier guacamole.properties dans le container se retrouve avec une ligne concatenée, ce qui rend les connexions aux serveurs distants impossible. L'authentification fonctionnera correctement toutefois.

En inspectant notre container on peut voir que nous obtenons alors une fin de fichier comme ci-dessous :
```
openid-username-claim-type: preferred_username
extension-priority: openidguacd-hostname: guacd
guacd-port: 4822
postgresql-username: guacamole_user
postgresql-password: ChooseYourOwnPasswordHere1234
postgresql-database: guacamole_db
postgresql-hostname: postgres
postgresql-port: 5432
```

> [!error] 
> `extension-priority: openidguacd-hostname: guacd` 


Quand on rajoute bien un retour à la ligne à la fin de notre guacamole.properties nous obtenons au final :
```
openid-username-claim-type: preferred_username
extension-priority: openid

guacd-hostname: guacd
guacd-port: 4822
postgresql-username: guacamole_user
postgresql-password: ChooseYourOwnPasswordHere1234
postgresql-database: guacamole_db
postgresql-hostname: postgres
postgresql-port: 5432
```

> [!success] 
> Les lignes sont bien espacées
> `openid-username-claim-type: preferred_username`
`extension-priority: openid`

