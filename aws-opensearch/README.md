# AWS Deployment for Alfresco with Search Enterprise 3 (Elasticsearch) using OpenSearch

Local Deployment of ACS Stack using an OpenSearch AWS managed service for Alfresco Search Enterprise 3

## Docker Compose

Docker Compose template includes following files:

```
.
├── .env-RemovePostFix
├── docker-compose.yml
├── alfresco
│   └── Dockerfile
├── keystores
├── license
└── scripts
```

* `.env-RemovePostFix` includes common values and service versions to be used by Docker Compose. A sample is provided (.env-RemovePostFix), save it as `.env` after values for your environment has been provided.
* `docker-compose.yml` is a regular ACS Docker Compose, including Elasticsearch Connector and the endpoints provided by OpenSearch Service, RDS and MQ in AWS.
* `Dockerfile` has the build instructions to customize default Alfresco Repository Docker Image.
* `keystores` folder is empty by default, copy your `truststore.pkcs12` file produced with `scripts/build-truststore.sh` tool
* `license` folder is empty be default, copy your `alfresco.lic` file in this folder
* `scripts` folder includes a set of bash scripts to configure the Alfresco EC2 Instance


## Creating an OpenSearch service in AWS

Before starting Docker Compose deployment, OpenSearch service must be available in AWS.

[Amazon OpenSearch Service](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/gsg.html) can be deployed using the AWS Web Console. Following settings are recommended to be used with this ACS Docker Compose for testing purposes.

| Property                           | Value                                |
|------------------------------------|--------------------------------------|
| Name                               | acs-elasticsearch                    |
| Deployment type                    | Development and testing              |
| Version                            | 1.3                                  |
| Auto-Tune                          | True                              |
| Availability Zones                 | 1-AZ                                 |
| Instance type                      | r6g.large.search                     |
| Number of nodes                    | 1                                    |
| Storage type                       | EBS                                  |
| EBS volume type                    | General purpose (SSD)                |
| EBS storage size per node          | 20                                   |
| Network                            | VPC access                        |
| Enable fine-grained access control | True                                 |
| Create master user                 | True                                 |
| Master username                    | <OPENSEARCH_MASTER_USERNAME>                           |
| Master password                    | <OPENSEARCH_MASTER_PASSWORD>                           |
| Domain access policy               | Only use fine-grained access control |

Once the Domain is available, the hostname (for instance `search-acs-elasticsearch-qpmpsomtkszxgrrryehiwxkhfm.eu-west-1.es.amazonaws.com`) will be available using HTTPs in port 443 and you should be able to connect using a browser with Master credentials.

Add the values for your environment in Docker Compose `.env` file

```
ELASTICSEARCH_SERVER_NAME=<OPENSEARCH_SERVER_NAME>
ELASTICSEARCH_USER=<OPENSEARCH_MASTER_USERNAME>
ELASTICSEARCH_PASS=<OPENSEARCH_MASTER_PASSWORD>
```

## Preparing the Alfresco EC2 Instance

Before deploying Docker Compose templates, follow this steps to configure the EC2 Instance using the scripts available in `scripts` folder.

```
cd scripts
```

**Install Docker and Docker Compose**

```
./install-docker.sh
```

After this step, rebooting the instance is required (use `sudo reboot` command)

**Login quay.io**

```
./login-quay.sh
```

Credentials required to download Alfresco Docker Images from quay.io. Contact Alfresco Support to get them.

**Mount Filesystem**

```
./mount-efs.sh <EFS_HOSTNAME>
```

`<EFS_HOSTNAME>` is the EFS DNS Name created using the Cloud Formation Template, for instance `fs-06919b0d9d232efe3.efs.eu-west-1.amazonaws.com`

**Create Database**

```
./create-database <DB_HOSTNAME>
```

`<DB_HOSTNAME>` is the DB DNS Name created using the Cloud Formation Template, for instance `acs-opensearch-opensearchalfresco.cijbca5yttz2.eu-west-1.rds.amazonaws.com`

**Create TrustStore from OpenSearch**

```
./build-truststore.sh <OPENSEARCH_HOSTNAME>
```

`<OPENSEARCH_HOSTNAME>` is the OpenSearch DNS Name created using the Cloud Formation Template, for instance `vpc-acs-opensearch-kwu4eqb2qmj745h7fdcoa5jp5e.eu-west-1.es.amazonaws.com`

Once this script has been executed successfully, copy the produced `truststore/truststore.pkcs12` file to `keystores` folder

**Adding Alfresco License**

Copy your `alfresco.lic` file to `license` folder. Contact Alfresco Support to get a valid license file for ACS 7.2.


## Using

```
$ docker-compose up --build --force-recreate
```

## Service URLs

http://localhost:8080/workspace

ADW
* user: admin
* password: admin

http://localhost:8080/alfresco

Alfresco Repository
* user: admin
* password: admin
