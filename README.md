# imixs-workflow

The image imixs/workflow provides a container to run [Imixs-Workflow](http://www.imixs.org) as a microservice on the Wildfly application server.
The image provides the following managed volume mount points:

* /opt/wildfly/standalone/configuration/
* /opt/wildfly/standalone/deployments/
* /opt/wildfly/standalone/log/

The volumes can be externalized on the docker host to provide more flexibility in runtime configuration.


## How to Run

To run Imixs-workflow in a Docker container, the container need to be linked to a postgreSQL database container. The database connection is configured in the Wildfly standalone.xml file and can be customized to any other database system. 

### Starting a Postgress Container
To start a postgreSQL container run the following command:
	
	docker run --name imixs-workflow-db -e POSTGRES_DB=workflow-db -e POSTGRES_PASSWORD=adminadmin postgres:9.5.2
 
This command will start a [Postgres container](https://hub.docker.com/_/postgres/) with a database named 'workflow-db'. This container can be liked to the Imixs-Workflow Container.
To start, stop and remove the Postgres container run:

    docker start imixs-workflow-db
    docker stop imixs-workflow-db
    docker rm -v imixs-workflow-db 
    
Read details about the official postgres image [here](https://hub.docker.com/_/postgres/).

 
### Starting Imixs-Workflow

After the postgres database container with the database 'workflow-db' was started, you can run the imixs/workflow container with a link to the postgres container using the following command:    

	docker run --name="imixs-workflow" -d -p 8080:8080 -p 9990:9990 \
             -e WILDFLY_PASS="adminadmin" \
             --link imixs-workflow-db:postgres \
             imixs/imixs-workflow

The link to the postgres container allows the wildfly server to access the postgress database via the host name 'postgres' which is mapped by the --link parameter.  This host name is used for the data-pool configuration in the standalone.xml file of wildfly.  

You can access Imixs-Microservice from you web browser at the following url:

http://localhost:8080/imixs-microservice

To start, stop and remove the imixs/workflow container run:

    docker start imixs-workflow
    docker stop imixs-workflow
    docker rm -v imixs-workflow 
    
More details about the imixs/wildfly image, which is the base image for Imixs-Workflow, can be found [here](https://hub.docker.com/r/imixs/wildfly/).



# docker-compose
You can simplify the start process of Imixs-Workflow by using 'docker-compose'. 
The following example shows a docker-compose.yml file for imixs-workflow:

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

Run start imixs-wokflow with docker-compose run:

	docker-compose up


 
 
# Monitoring

Use the following docker command watch the wildfly log file:

	docker logs -f imixs-workflow

# Configuration

The imixs/workflow image provides a standalne.xml configuration file for wildfly located in /opt/jboss/wildfly/standalone/configuration/. This configuration can be mapped to a external volume to customize or add additional configuration settings. 
    
    docker run -it -p 8080:8080 -p 9990:9990 \
    	-v ~/git/docker-imixs-workflow/src/docker/configuration/standalone.xml:/opt/jboss/wildfly/standalone/configuration/standalone.xml \
    	imixs/imixs-workflow
    	
# Contribute
General information about Imixs-Workflow can be found the the [project home](http://www.imixs.org). The sources for this docker image are available on [Github](https://github.com/imixs-docker/imixs-workflow). Please report any issues.

To build the image from the Dockerfile manually checkout the sources and run:

	docker build --tag=imixs/imixs-workflow .
