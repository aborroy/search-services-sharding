# Alfresco Search Services with Sharding

Deployment template based in official [Docker Composition](https://github.com/Alfresco/acs-community-deployment/tree/master/docker-compose) using SOLR Sharding with two nodes routing the content by DB_ID.

>> DBID (DB_ID): This method is available in Alfresco Search Services 1.0 and later versions and is the default sharding option in Solr 6. Nodes are evenly distributed over the shards at random based on the murmur hash of the DBID. The access control information is duplicated in each shard. The distribution of nodes over each shard is very even and shards grow at the same rate.

This configuration provides two SOLR Shards, identified as "0" and "1". Despite this feature is available for both Enterprise and Community versions, documentation is available at [SOLR Sharding](https://docs.alfresco.com/search-enterprise/concepts/solr-shard-overview.html).

Multiple shard instances have the following advantages:

* It provides high availability in case a shard/node fails.
* It allows you to scale out your search throughput because searches can be executed on all the instances in parallel.
* It increases performance: search requests are handled by the multiple shard instances.

You should review volumes, configuration, modules & tuning parameters before using this composition in **Production** environments.

## Source Images

* [alfresco-content-repository-community:6.1.2-ga](https://hub.docker.com/r/alfresco/alfresco-content-repository-community)
* [alfresco-share:6.1.0](https://hub.docker.com/r/alfresco/alfresco-share)
* [alfresco-search-services:1.3.0.4](https://hub.docker.com/r/alfresco/alfresco-search-services)
* [postgres:10.1](https://hub.docker.com/_/postgres)
* [nginx:stable-alpine](https://hub.docker.com/_/nginx)

## Project structure

```
.
├── .env
├── config
│   ├── nginx.conf
│   └── nginx.htpasswd
├── docker-compose.yml
└── search
    └── Dockerfile
```

* `.env` includes Docker environment variables to set Docker Image release numbers
* `config` folder includes configuration for HTTP Proxy based in NGINX
* `docker-compose.yml` is a Docker Compose template to use ACS Community with SOLR Replication
* `search` folder includes a customised Dockerfile to configure SOLR Docker Image with Sharding

## SOLR Considerations

Alfresco SOLR API has been protected to be accessed from outside Docker network, as using HTTP allows unauthenticated requests.

```
    # Protect access to SOLR APIs
    location ~ ^(/.*/service/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/s/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/wcservice/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/wcs/api/solr/.*)$ {return 403;}

    location ~ ^(/.*/proxy/alfresco/api/solr/.*)$ {return 403 ;}
    location ~ ^(/.*/-default-/proxy/alfresco/api/.*)$ {return 403;}
```

SOLR Web Console access has been protected with username/password (admin/admin).


# How to use this composition

## Start Docker

Start docker and check the ports are correctly bound.

```bash
$ docker-compose up -d
$ docker ps --format '{{.Names}}\t{{.Image}}\t{{.Ports}}'
sharding_share_1        alfresco/alfresco-share:6.1.0                            8000/tcp, 8080/tcp
sharding_alfresco_1     alfresco/alfresco-content-repository-community:6.1.2-ga  8080/tcp
sharding_proxy_1        nginx:stable-alpine                                      0.0.0.0:80->80/tcp
sharding_postgres_1     postgres:10.1                                            0.0.0.0:5432->5432/tcp
sharding_solr6shard2_1  search-services-sharding_solr6shard2                     8983/tcp
sharding_solr6shard1_1  search-services-sharding_solr6shard1                     8983/tcp
```

### Viewing System Logs

You can view the system logs by issuing the following.

```bash
$ docker-compose logs -f
```

Logs for every service are also available at `logs` folder.

## Access

Use the following username/password combination to login.

 - User: admin
 - Password: admin

Alfresco and related web applications can be accessed from the below URIs when the servers have started.

```
http://localhost/alfresco      - Alfresco Repository
http://localhost/solrshard1    - Alfresco Search Services (Shard 0)
http://localhost/solrshard2    - Alfresco Search Services (Shard 1)
http://localhost/share         - Alfresco Share
```

## Notes

Some errors will be logged when starting the composition.

```
ERROR (SolrTrackingPool-alfresco-MetadataTracker-7) [   ] o.a.s.SolrInformationServer SolrInformationServer problem
org.alfresco.solr.TrackerStateException: 05140000 The trackers work was rolled back by another tracker error
```

Despite these errors are not relevant for regular use of the service, they have been fixed in Search Services 1.4.x
