# Zabbix Cheat Sheet

 A cheat sheet for Zabbix

## Install Zabbix in docker (for test)

```yaml
version: '3.8'

services:
  # Database PostgreSQL
  postgres:
    image: postgres:17-alpine
    container_name: zabbix-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: zabbix #databese create name
      POSTGRES_USER: zabbix # user database
      POSTGRES_PASSWORD: zabbix_password # passworddatabase
    volumes:
      - postgres-data:/var/lib/postgresql/data # database 
    healthcheck: 
      test: ["CMD", "pg_isready", "-U", "zabbix"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - zabbix-net

  # Zabbix Server
  zabbix-server:
    image: zabbix/zabbix-server-pgsql:alpine-7.4-latest
    container_name: zabbix-server
    restart: unless-stopped
    environment:
      DB_SERVER_HOST: postgres # name service Postgress in docker network
      POSTGRES_DB: zabbix # name databese
      POSTGRES_USER: zabbix # user database
      POSTGRES_PASSWORD: zabbix_password # password database
      ZBX_ENABLE_SNMP_TRAPS: "true" # SNMP traps on
    ports:
      - "10051:10051"
    volumes:
      - /opt/zabbix-server-data:/var/lib/zabbix # cash, hystory etc.
      - /opt/zabbix/config/server:/etc/zabbix:ro # configs
      - /opt/zabbix/modules:/usr/lib/zabbix/modules:ro # modules
      - /opt/zabbix/scripts:/usr/lib/zabbix/alertscripts:ro # allert scripts
    depends_on:
      postgres:
        condition: service_healthy # Run else healthcheck good
    networks:
      - zabbix-net

  # Zabbix Web Interface
  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:alpine-7.4-latest
    container_name: zabbix-web
    restart: unless-stopped
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_DB: zabbix
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix_password
      ZBX_SERVER_HOST: zabbix-server # name zabbix-server  in docker network
      PHP_TZ: Europe/Riga # time zone
    ports:
      - "8080:8080"
    volumes:
      - zabbix-web-conf:/etc/zabbix/web/conf # config web interface
    depends_on:
      postgres:
        condition: service_healthy # start else service_healthy check and server
      zabbix-server:
        condition: service_started 
    networks:
      - zabbix-net

  # Zabbix Agent 
  zabbix-agent:
    image: zabbix/zabbix-agent:alpine-7.4-latest
    container_name: zabbix-agent
    restart: unless-stopped
    environment:
      ZBX_HOSTNAME: "Zabbix server" # name zabix-agent
      ZBX_SERVER_HOST: zabbix-server # server zabbix for active check
      ZBX_PASSIVESERVERS: zabbix-server # server zabbix for passive check
      ZBX_STARTAGENTS: 3
    ports:
      - "10050:10050"
    volumes:
      - /opt/zabbix/config/agent:/etc/zabbix:ro # config agent
    networks:
      - zabbix-net

networks:
  zabbix-net:
    name: zabbix-net
    driver: bridge

volumes:
  postgres-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /opt/zabbix/postgres
  zabbix-server-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /opt/zabbix/server
  zabbix-web-conf:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /opt/zabbix/web-conf
```
PostgreSQL runs  as the user:
```bash
# UID=999(postgres)
# GID=999(postgres)

sudo chown -R 999:999 /opt/zabbix/postgres
sudo chmod 700 /opt/zabbix/postgres
# ❌ If you leave root:root → the container does not start
```
Zabbix-server runs  as the user:

```bash
# UID=1997(zabbix)
# GID=1997(zabbix)

sudo chown -R 1997:1997 /opt//zabbix/server
sudo chmod 750 /opt//zabbix/server
# ❌ If you leave root:root → the container does not start
```

zabbix/zabbix-web-nginx-pgsql runs  as the user:

```bash
# UID=1000(nginx/php-fpm)
# GID=1000(nginx/php-fpm

sudo chown -R 1000:1000 /opt/zabbix/web-conf
sudo chmod 750 /opt//zabbix/web-conf

```

/zabbix/config/server write root

```bash
sudo chown -R root:root /opt/zabbix/config/server
sudo chmod -R 755 /opt//zabbix/config/server

```


zabbix/config/agent write root


```bash
sudo chown -R root:root /opt/zabbix/config/agent
sudo chmod -R 755 /opt//zabbix/config/agent

```

/zabbix/scripts write root

```bash
sudo chown -R root:root /opt/zabbix/scripts
sudo chmod -R 750 /opt//zabbix/scripts

```
zabbix/modules write root

```bash
sudo chown -R root:root /opt/zabbix/modules
sudo chmod -R 755 /opt/zabbix/modules

```

| Path          | Who write | UID:GID   | Права     |
| ------------- | --------- | --------- | --------- |
| postgres      | postgres  | 999:999   | 700       |
| server        | zabbix    | 1997:1997 | 750       |
| web-conf      | web       | 1000:1000 | 750 → 550 |
| config/server | никто     | root      | 755       |
| config/agent  | никто     | root      | 755       |
| scripts       | никто     | root      | 750       |
| modules       | никто     | root      | 755       |


Check:
```bash
ls -ld /home/buktop/zabbix/*
docker exec zabbix-postgres id
docker exec zabbix-server id
docker exec zabbix-web id
```

## Zabbix-get

### Instal reposetary
```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
apt update
```
### Install zabbix-get
```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
apt update
apt install zabbix_get
```
Example:
```bash
# An example of running Zabbix get under UNIX to get the processor load value from the agent:
zabbix_get -s 127.0.0.1 -p 10050 -k "system.cpu.load[all,avg1]" 

# Another example of running Zabbix get for capturing a string from a website:
zabbix_get -s 192.168.1.1 -p 10050 -k "web.page.regexp[www.example.com,,,\"USA: ([a-zA-Z0-9.-]+)\",,\1]"

 -s --host <host name or IP>      Specify host name or IP address of a host.
  -p --port <port number>          Specify port number of agent running on the host. Default is 10050.
  -I --source-address <IP address> Specify source IP address.
  -k --key <item key>              Specify key of item to retrieve value of.
  -h --help                        Give this help.
  -V --version                     Display version number.

```
 ### Userparametrs
You may write a command that retrieves the data you need and include it in 
the user parameter in the agent configuration file ('UserParameter' configuration parameter).

```bash
UserParameter=<key>,<command>
```
Example:
```bash
#/etc/zabbix/agent/zabbix_agentd.d/test
UserParameter=test2[*],echo 1

# zabbix server 
sudo zabbix_get -s zabbix-agent.local -p 10050 -k test2
```







