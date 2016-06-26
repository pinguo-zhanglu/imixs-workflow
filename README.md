# imixs-microservice

The image imixs/microservice provides a container to run Imixs-Workflow as a microservice on the wildfly application server.
The image provides the following managed volume mount points:

* /opt/wildfly/standalone/configuration/
* /opt/wildfly/standalone/deployments/
* /opt/wildfly/standalone/log/

The volumes can be externalized on the docker host to provide more flexibility in runtime configuration.


## How to Run

To run Imixs-Microservice the container need to be linked to a postgreSQL database container

### Starting a Postgress Database
To start a postgreSQL container run the following command:
	
	docker run --name imixs-workflow-db -e POSTGRES_DB=workflow-db -e POSTGRES_PASSWORD=adminadmin postgres:9.5.2
 
Data database 'workflow-db' will be persisted in the container file system.
To start, stop and remove the container run:

    docker start imixs-workflow-db
    docker stop imixs-workflow-db
    docker rm -v imixs-workflow-db 
    
Read details about the official postgres image [here](https://hub.docker.com/_/postgres/).

 
### Starting Imixs-Workflow

After a postgreSQL database container with the database 'office' was started you can run the imixs/office-workflow container with a link to the postgreSQL container:    

	docker run --name="imixs-workflow" -d -p 8080:8080 -p 9990:9990 \
             -e WILDFLY_PASS="adminadmin" \
             --link imixs-workflow-db:postgres \
             imixs/imixs-workflow

The link to the postgres container allows the wildfly server to access the postgress database via the host name 'postgres' which is mapped by the --link parameter.  This host name need to be used for the data-pool configuration in the standalone.xml file.  

You can access Imixs-Microservice from you web browser at the following url http://localhost:8080/imixs-microservice

To start, stop and remove the imixs/microservice container run:

    docker start imixs-workflow
    docker stop imixs-workflow
    docker rm -v imixs-workflow 
    
Read details about the the imixs/wildfly image [here](https://hub.docker.com/r/imixs/wildfly/).



# How to run with docker-compose
You can run Imixs-Offie-Workflow on docker-compose to simplify the startup. 
The following example shows a docker-compose.yml for imixs-office-workflow:

	postgres:
	  image: postgres
	  environment:
	    POSTGRES_PASSWORD: adminadmin
	    POSTGRES_DB: workflow-db
	
	imixsworkflow:
	  image: imixs/workflow
	  environment:
	    WILDFLY_PASS: adminadmin
	  ports:
	    - "8080:8080"
	    - "9990:9990"
	  links: 
	    - postgres:postgres
 
Take care about the link to the postgres container. The host 'postgres' name need to be used in the standalone.xml configuration file in wildfly to access the postgres server.

Run docker-compose up, wait for it to initialize completely, and visit http://localhost:8080/microservice or http://host-ip:8080/microsservice

 
 
# Monitoring

Use the following docker command watch the wildfly log file:

	docker logs -f imixs-workflow

# Configuration

The imixs/microservice image provides a standalne.xml configuration file for wildfly located in /opt/jboss/wildfly/standalone/configuration/. This configuration can be mapped to a external volume to customize or add additional configuration settings. 
    
    docker run -it -p 8080:8080 -p 9990:9990 \
    	-v ~/git/docker-imixs-workflow/src/docker/configuration/standalone.xml:/opt/jboss/wildfly/standalone/configuration/standalone.xml \
    	imixs/imixs-workflow
    	
# Contribute
The source is available on Github. Please report any issues.

To build the image from the Dockerfile run:

	docker build --tag=imixs/imixs-workflow .
