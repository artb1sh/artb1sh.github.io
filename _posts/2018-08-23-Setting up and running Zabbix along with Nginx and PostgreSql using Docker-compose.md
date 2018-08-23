# Setting up and running Zabbix along with Nginx and PostgreSql using Docker-compose

Hi everyone, i am going to share some of my experience about setting up Zabbix in Docker-compose. So before starting let talk about Zabbix. 

### Zabbix

Zabbix is an open source monitoring software that let you monitor your servers for resources like ram usage, memory usage, swap memory usage, disk I/O and more. You can also setup some notification so you recieve alerts as soon as resources reach critical limits like ram is used up to max limit and you need to expand else it will crash. You can also monitor network device, jmx applications and much more.

Out of the box jabber, sms and mail media is configured. You can add others by custom scripts. I will add this part later . Now going back to Zabbix setup.

### List of Docker Images we require

1. [Postgres](https://hub.docker.com/_/postgres/)
2. [Zabbix-server-pgsql](https://hub.docker.com/r/zabbix/zabbix-server-pgsql/)
3. [Zabbix-web-nginx-pgsql](https://hub.docker.com/r/zabbix/zabbix-web-nginx-pgsql/)
4. [Zabbix-agent](https://hub.docker.com/r/zabbix/zabbix-agent/~/dockerfile/)
5. [Adminer](https://hub.docker.com/_/adminer/) (optional)
6. [Graffana](https://hub.docker.com/r/grafana/grafana/) (optional)

### Docker-compose configuration(YML)

```
version: '3.1'
services:
  postgres-server:    # The Postgres Database Service
    image: postgres:latest
    restart: always
    environment:   # Username, password and database name variables
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbixNew
      PG_DATA: /var/lib/postgresql/data/pgdata #data storage
  zabbix-server:     # The main Zabbix Server Software Service
    image: zabbix/zabbix-server-pgsql:ubuntu-latest
    restart: always
    environment:   # The Postgres database value variable
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbixNew
      ZBX_HISTORYSTORAGETYPES: log,text #Zabbix configuration variables
      ZBX_DEBUGLEVEL: 1
      ZBX_HOUSEKEEPINGFREQUENCY: 1
      ZBX_MAXHOUSEKEEPERDELETE: 5000
    depends_on:
      - postgres-server
    volumes:  # Volumes for scripts and related files you can add
      - /usr/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts

  zabbix-web:    # The main Zabbix web UI or interface 
    image: zabbix/zabbix-web-nginx-pgsql:ubuntu-latest
    restart: always
    environment:  # Postgre database variables
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbixNew
      ZBX_SERVER_HOST: zabbix-server  # Zabbix related and Php variables
      ZBX_POSTMAXSIZE: 64M
      PHP_TZ: "Europe/Moscow"  
      ZBX_MAXEXECUTIONTIME: 500
    depends_on:
      - postgres-server
      - zabbix-server
    ports:    # Port where Zabbix UI is available
      - 8090:80
  zabbix-agent:   # Zabbix agent service that tracks usage and send to zabbix server
    image: zabbix/zabbix-agent:latest
    privileged: true   #access mode for allowing resource access
    network_mode: "host"
    restart: unless-stopped
    environment:
      - ZBX_SERVER_HOST=127.0.0.1 #the IP/Dns of Zabbix server
  grafana-xxl:  #optional more functional and creative UI 
    image: monitoringartist/grafana-xxl:latest
    ports:
     - 3000:3000
```

### Steps for running

1. Put the configuration in a file and save with name docker-compose.yml. To keep separate create folder with name you want and put the compose file in that folder. This folder you can use for running docker-compose.

2. Open Terminal and cd to folder you created and then run command

```
docker-compose up -d
```

or if you prefer logs to be shown, then

```
docker-compose up
```

3. Now access Zabbix UI at port 8090(replace this port with your port if you have changed in above compose file). Open this url in browser

```
http://localhost:8090
```

4. Now login by entering username , password for default Admin user with credentials.

username : `Admin`

password: `zabbix`

5. Now you are setup and will reach dashboard with various tiles for information.

6. So now you can do monitoring with zabbix.

   I hope this one I shared with you will be worthy, I will share later topics in new post later. Feel free to comment your opinion about my post.
