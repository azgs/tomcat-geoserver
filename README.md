### AZGS custom clutered Geoserver deployment scheme and automation tool
Documentation on how our Geoserver cluster works and a BASH tool for automated deployment of new Geoservers.

The Geoserver cluster lives on `Harzburgite` at `/home/geoserver/tomcat-cluster`.

Tomcat binaries live in `~/tomcat-cluster/apache-tomcat-7.0.42` and we're currently running Apache-Tomcat v7.0.42 (http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.42/bin/apache-tomcat-7.0.42.tar.gz)

We run each instance of Geoserver in it's own thread of the Tomcat binaries just described.  The threads all
live in `~/tomcat-cluster/cluster` and follow the naming convention: `tomcat-XXXX` where `XXXX` is the 
`connector port` respective of the thread.  So `~/tomcat-cluster/cluster/tomcat-8081` is a Tomcat thread
being served out of port 8081.

### Build a new thread
Use `install.sh` to build a new thread and Geoserver instance.  If the installation script is not recognized as an executable file, make it one:

    $ sudo chmod u+x install.sh

The following variables should be changed before running `install.sh`:

    # The port that this Tomcat thread will run on
    CONNECTOR_PORT
    # The port that this Tomcat thread will shutdown on.  Must be an open port.
    SHUTDOWN_PORT
    # An alpha-numeric name for the Geoserver instance.  Must start with a letter.
    GEOSERVER_NAME

After running the installation script, the new Geoserver should be running on:

    http://localhost:{CONNECTOR_PORT}/{GEOSERVER_NAME}

To run it under a reverse proxy with Nginx, a new location directive needs to be added into `/home/geoserver/config/nginx.config`:

    location /my_location_directive/ {
    	proxy_pass http://127.0.0.1:{CONNECTOR_PORT}/{GEOSERVER_NAME};
    }

And then reload Nginx:

    $ sudo service nginx reload

### JSONP
We're deploying custom Tomcat/Geoserver threads with JSONP enabled.  Some of our applications (like the AZ Hazard Viewer) have a JSONP dependency.  The configuration has already been made, so this isn't something to worry about when deploying fresh instances.  But just important to keep in mind for the next time we migrate our data.

When deploying Geoserver with Tomcat, we enable JSONP like this:

    # Edit tomcat/bin/setenv.sh, create if it doesn't exist
    $ sudo nano ~/tomcat-cluster/apache-tomcat-7.0.42/bin/setenv.sh
    
    # Add the following text and save
    $ export ENABLE_JSONP=true and outFormat=text/javascript

And then restart the Tomcat server/thread(s)