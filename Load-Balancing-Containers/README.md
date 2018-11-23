#Instructions & Tasks
Acme Anvil Corporation - Load Balancing Containers
In this learning activity, you will load balance containers using two methods. First, you will use Nginx to load balance traffic to three weather-app containers. Next, you will use Docker Swarm to load-balance a pair of Nginx containers.

Log in to Swarm Server 1 and Swarm Server 2 and sudo to root.

##Part 1
We will be using Docker Compose to set up the load balancer and containers.

  1.  On Swarm Server 1, in the root directory, navigate to lb-challenge and create the Docker Compose file.
  2.  Set the Compose version to 3.2.
  3.  Create three weather-app services:
  weather-app1
  weather-app2
  weather-app3
  4.  The services should build the Dockerfile that is in the weather-app directory.
  5.  All three should have tty set to true.
  6.  All three containers should be using the frontend network.
  7.  Create a service called loadbalancer.
  9.  It should use the Dockerfile located in the load-balancer directory.
  10. Set tty to true.
  11. The port mapping should be set to 80 on the host and 80 on the container.
  12. The load balancer should be using the frontend network.
#Create the Frontend Network
  1.  In the load-balancer directory, there will be a file called nginx.conf.
  2.  Add the three weather-app services to the upstream section.
  3.  In the server section, make sure it is listening on port 80.
  4.  Set the server_name to localhost.
  5.  The location should be set to /.
  6.  The location should contain a proxy pass to localhost.
  7.  The proxy set header should be set to $host.
  9.  Execute a compose up and make sure to use the build and detached flags.
  10. Verify that your app is up and running.

##Part 2
  1.  In the root directory, use cat to retrieve the contents of swarm-token.txt.
  2.  Use the docker swarm join --token command from the output of the file to join Swarm Server 2 to the swarm.
  3.  Create a service called nginx-app.
  4.  The published port should be 8080 and the target port should be 80 on the container.
  5.  Make sure there are 2 replicas.
  6.  Use the nginx image.
  7.  Verify that your app is up and running.
