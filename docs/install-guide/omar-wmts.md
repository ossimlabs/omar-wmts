# OMAR WMTS

## Dockerfile
```
FROM omar-ossim-base
EXPOSE 8080
RUN mkdir /usr/share/omar
COPY omar-mensa-app-1.0.0-SNAPSHOT.jar /usr/share/omar
RUN chown -R 1001:0 /usr/share/omar
RUN chown 1001:0 /usr/share/omar
RUN chmod -R g+rw /usr/share/omar
RUN find $HOME -type d -exec chmod g+x {} +
USER 1001
WORKDIR /usr/share/omar
CMD java -server -Xms256m -Xmx1024m -Djava.awt.headless=true -XX:+CMSClassUnloadingEnabled -XX:+UseGCOverheadLimit -Djava.security.egd=file:/dev/./urandom -jar omar-wmts-app-1.0.0-SNAPSHOT.jar
```
Ref: [omar-base](../../../omar-ossim-base/docs/install-guide/omar-base/)

## JAR

Browse the SNAPSHOT and RELEASE jar files here:
[omar-wmts-app](http://artifacts.radiantbluecloud.com/artifactory/webapp/#/artifacts/browse/tree/General/omar-local/io/ossim/omar/apps/omar-wmts-app)


##Configuration

The [Dockerfile](#Dockerfile) section shows how to run the container and jar if you have configured a config server to m,anage all your configuration scripts.  Here we will show you the config properties required by the wmts service and how to specify the configuration directly when running the JAR service.  Create a file called applicaiton.yml with contents:

```
server:
  contextPath: /omar-wmts

serverName: "localhost"
serverProtocol: "http"

omarDb:
  host: 
  port: 5432
  name: omardb-prod
  url: jdbc:postgresql://${omarDb.host}:${omarDb.port}/${omarDb.name}
  driver: org.postgresql.Driver
  username: 
  password:
   
environments:
  production:
    dataSource:
      pooled: true
      jmxExport: true
      driverClassName: ${omarDb.driver}
      url:      ${omarDb.url}
      username: ${omarDb.username}
      password: ${omarDb.password}

endpoints:
  enabled: true
  health:
    enabled: true

omar:
  wmts:
    wfsUrl: http://omar-wfs-app:8080/omar-wfs/wfs
    wmsUrl: http://omar-wms-app:8080/omar-wms/wms
    oldmarWmsFlag: false
    footprints:
      url: ${serverProtocol}://${serverName}/omar-wms/footprints/getFootprints
      layers: "omar:raster_entry"
      styles: "byFileType"

```

where:

 * **server.contextPath** adds a context to the beginning of all endpoints
 * **serverName** is a Variable used throughout the application.yml file to specify the server name.  In this example we assume localhost
 * **serverProtocol** is a variable used to specify the protocol used.  Should be either http or https.
 * **omarDb** is a variable used throughout the file
  * **host** defines the host where the database server resides.  Thisis the DNS portion of the location without http or https
  * **port** typically 5432 for Postgres.
  * **name** name you want to use for the database you created on the database server.
  * **url** the JDBC connection string url.
  * **driver** the driver name used for connecting to the database.
  * **username** the username for the database
  * **password** the password for the database
 * **environments.production.datasource** This is for profile production.  This is the default profile when using the java -jar execution. This section will hold the database connection values specific for the **driverClassName**
  * **pooled**  
  * **jmxExport**
  * **driverClassName** 
    

We can then have a command to start the service with the added config file

`java -server -Xms256m -Xmx1024m -Djava.awt.headless=true -Dspring.config.location=./application.yml -XX:+CMSClassUnloadingEnabled -XX:+UseGCOverheadLimit -Djava.security.egd=file:/dev/./urandom -jar omar-wmts-app-1.0.0-SNAPSHOT.jar`

When building the container just add the **application.yml** to the home directory and then modify the startup command to run with the application.yml.

