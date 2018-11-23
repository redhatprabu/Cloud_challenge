#scenario 1:

Docker volumes :
======================

Instructions & Tasks :

For your final task, create a new Docker volume named mission-status on your host machine. Once the volume is created, use the docker cli to display the volume mountpoint on your server. Once you have found the mountpoint, drop to root to move contents found under content-dockerquest-spacebones/volumes/ to local vol directory.

Once this is done, exit root, then start a new container running on port :80 using the base httpd image found in DockerHub, then visit the site in your browser!


###Results :
=======
docker volume create missionstatus
docker volume inspect missionstatus  - copy the mount point
copy your content :      ```cp -r content-dockerquest-spacebones/volumes/* /var/lib/docker/volumes/missionstatus/_data```


#Scenario 2:

Container Logging :
=====================

Instructions & Tasks
Log in to your live environment and sudo to root. Edit the syslog config file and uncomment the two lines under Provides UDP syslog reception. Then, start the syslog service.

Configure Docker to use syslog as the default log driver and then start the Docker service.

Create two new containers using the httpd image. The first one will be called syslog-logging and will use syslog for the log driver. The second will be called json-logging and will use the JSON file for the log driver.

Verify that the syslog-logging container is sending its logs to syslog by running tail on the message log file. Then, verify that the json-logging container is sending its logs to the JSON file. Good luck!


###Answer :

 1. Edit /etc/rsyslog.conf  - > enable UDP settings
 2. Create a file under /etc/docker /daemon.json and add
 {
  "log-driver: "syslog",
   "log-opts": {
       "udp://<privateIP:514"
	 }
 }

 3. reload syslog and docker services
 4. create and run container #docker container run -d --name syslog-logging httpd  
 5. check logs getting updated in /var/log/messages and you can't see logs in docker logs syslog-logging.
 6. create another container with different log driver
	#docker container create -d --name json-logging --log-driver json.file httpd

	now check logs getting updated for json-logging container under #docker logs json-logging not under /var/log/messages


===============================================================================================================================


#WatchTower :
=============
Instructions & Tasks
Welcome to the second learning activity! In this learning activity, you will be using Watchtower to monitor containers for updates. In order to complete this learning activity, you will need a Docker Hub account.

Log in to your live environment and sudo to root.

###Create the Dockerfile
	1.The base image should be node.
	2.Using the RUN instruction, make a directory called /var/node
	3.Use the ADD instruction to add the contents of the code directory into /var/node.
	4.Make /var/node the working directory.
	5.Execute an npm install.
	6.Set ./bin/www as the command.
	7.From the command line, log in to Docker Hub.
	8.Build your image using <USERNAME>/express.
	9.Push the image to Docker Hub.
###Create the demo app

	1.Create a Docker container called demo-app.
	2.The port mapping should be port 80 on the host, mapping to 3000 on the container.
	3.The restart policy should be set to always.
	4.Use the image that you created, <USERNAME>/express.
###Create the Watchtower container

	1.Create a Docker container called watchtower.
	2.The restart policy should be set to always.
	3.Use the -v flag to set /var/run/docker.sock:/var/run/docker.sock.
	4.Use the v2tec/watchtower followed by the -i flag to set the iteration to 30 seconds.
Update the Docker image


	1.Add an instruction to create /var/test. This should be done after creating /var/node.
	2.Rebuild your image.
	3.Push the image to Docker Hub.

	Watchtower will update demo-app

	The Watchtower interval is set to 30 seconds.

	After about 30 seconds, check to see if the container has been updated by executing docker ps.


#Adding Metadata and Labels
## Scenario :
For the last six months, the Acme Anvil Corporation has been migrating some of their bare metal infrastructure to Docker containers. After the initial implementation, you mention to the team that labels can be used for storing metadata about images and containers. This is useful for keeping track of image and container attributes like the build date and application version.

Your team thinks this is a great idea, and you’ve been tasked with creating a quick demo to show how useful labels can be. You created a small weather app a few months ago while learning Node.JS, and it’s perfect for a quick demo. On your Docker workstation, you will create a Dockerfile that contains four labels: the maintainer, build date, application name, and application version. You will then build the image and push it to Docker Hub. On your Docker server, you will create a new container using the weather-app image. Once the container is running, you will update the branch of your application, rebuild the image, and push the changes to Docker Hub. On the Docker server, Watchtower will update the running container with the new version of the image

## Try :  
Instructions & Tasks
In this learning activity, you will deploy a container that uses labels. In order to complete this learning activity, you will need a Docker Hub account.

Log in to your Docker workstation and Docker server, and sudo to root. You will create the Dockerfile and image on the Docker workstation. The weather-app container will be run on your Docker server.

Create the Dockerfile
  1.  Add three arguments: BUILD_VERSION, BUILD_DATE, and APPLICATION_NAME.
  2.  Add three labels:
      org.label-schema.build-date (will be set using the BUILD_VERSION argument)
      org.label-schema.application (will be set using the APPLICATION_NAME argument)
      org.label-schema.version (will be set using the BUILD_VERSION argument)
  3.  Using the RUN instruction, make a directory called /var/node.
  4.  Use the ADD instruction to add the contents of the code directory into /var/node.
  5.  Make /var/node the working directory.
  6.  From the command line, log in to Docker Hub.
  7.  Build your image using <USERNAME>/weather-app, and supply the following three build arguments:
        BUILD_DATE should be set to $(date -u +'%Y-%m-%dT%H:%M:%SZ').
        APPLICATION_NAME should be set to weather-app.
        BUILD_VERSION should be set to v1.0.
  8. Push the image to Docker Hub.
## Create the weather app
  1. Create a Docker container called weather-app.
  2.  The port mapping should be port 80 on the host, mapping to 3000 on the container.
  3.  The restart policy should be set to always.
  4.  Use the image that you created, <USERNAME>/weather-app.
  5.  Use the docker inspect command with the filter flag to filter output for {{.Config.Labels}}.
## Update the weather app to version 1.1

  1.  Change directories to /root/weather-app.
  2.  Check out branch v1.1.
  3.  Change directories back to /root.
  4.  Rebuild your image with the following build arguments:
      BUILD_DATE should be set to $(date -u +'%Y-%m-%dT%H:%M:%SZ').
      APPLICATION_NAME should be set to weather-app.
      BUILD_VERSION should be set to v1.1.
  5.  Push the image to Docker Hub.

## Watchtower will update weather-app

  1.  The Watchtower interval is set to 5 seconds. After about 10 or 15 seconds, check to see if weather-app has been updated by executing    

        docker ps.
  2.  Use the docker inspect command with the filter flag to filter output for {{.Config.Labels}} to see if the version is set to 1.1.
