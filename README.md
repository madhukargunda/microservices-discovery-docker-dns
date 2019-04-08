# microservices-discovery-docker-dns
Service discovery for microservices using built-in Docker DNS


### How Docker DNS Works

- Docker Container has inbuilt DNS which automatically resolves IP to container names in user-defined networks.
- Docker default docker0 bridge driver doesnot contains DNS , hence DNS will not work inbuilt docker0 bridge driver.

- When docker creates the container it copies the following three files from the host machine
```
/etc/hostname – This file maps the container IP address to a name.
/etc/hosts – This file maps other container’s IP addresses to names.
/etc/resolv.conf – This file contains the IP addresses of other DNS servers to refer to if the 
 container cannot resolve a name to IP address.

This file contains the Docker DNS Server ip addreass

root@4410aa54c605:/etc# cat resolv.conf 
nameserver 127.0.0.11
options ndots:0
root@4410aa54c605:/etc# 
```
## Specify a service discovery method for external clients connecting to a swarm.

Specify a service discovery method for external clients connecting to a swarm.

Version 3.3 only.
```
endpoint_mode: vip - Docker assigns the service a virtual IP (VIP) that acts as the “front end” for clients to reach the
service on a network. Docker routes requests between the client and available worker nodes for the service, without client
knowledge of how many nodes are participating in the service or their IP addresses or ports. (This is the default.)

endpoint_mode: dnsrr - DNS round-robin (DNSRR) service discovery does not use a single virtual IP. Docker sets up DNS entries
for the service such that a DNS query for the service name returns a list of IP addresses, and the client connects directly to
one of these. DNS round-robin is useful in cases where you want to use your own load balancer, or for Hybrid Windows and Linux
applications.
```

```
version: "3.3"

services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    networks:
      - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip

  mysql:
    image: mysql
    volumes:
       - db-data:/var/lib/mysql/data
    networks:
       - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

volumes:
  db-data:

networks:
  overlay:


```

Referrence

https://blog.hook.sh/en/docker/compose-deploy/
