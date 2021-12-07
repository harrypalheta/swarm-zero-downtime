# Swarm Zero Downtime

Load tests with Docker Swarm and Jmeter for Zero Downtime

## Swarm

### init

```
docker swarm init --advertise-addr YOUR_IP
```

### up

```
docker network create -d overlay nginx-swarm
```

```
docker stack  deploy -c docker-compose.yml NGINX
```
## make tests with jmeter

Install Jmeter: 

```
wget -c http://www.gtlib.gatech.edu/pub/apache/jmeter/binaries/apache-jmeter-5.2.1.zip -O jmeter.zip && \
unzip jmeter.zip && \
mv apache-jmeter-5.2.1 apache-jmeter
```

Open terminals:

**terminal 1:** `watch docker ps`

**terminal 2:** `docker service logs NGINX_nginx -f -n 100`

**terminal 3:** `rm testsResults.jtl && ./apache-jmeter/bin/jmeter.sh -n -t tests.jmx -p user.properties -l testsResults.jtl`

> You'll be 1 minutos for tests bellow.

Change image nginx:

**terminal 4:** `sed -i 's/nginx:alpine/nginx:stable-alpine/g' docker-compose.yml && docker stack deploy -c docker-compose.yml NGINX`
> `sed -i 's/nginx:stable-alpine/nginx:alpine/g' docker-compose.yml && docker stack deploy -c docker-compose.yml NGINX`

## generate report

```
rm -rf results/ && ./apache-jmeter/bin/jmeter.sh -g testsResults.jtl -o results
```

> open in your browser `ex: firefox file://$(pwd)/results/index.html`

## execute rollback

```
rollbackWithTime(){
    SECONDS=0
    docker service rollback NGINX_nginx
    duration=$SECONDS
    echo "$(($duration / 60)) minutes and $(($duration % 60)) seconds elapsed."
}
rollbackWithTime
```