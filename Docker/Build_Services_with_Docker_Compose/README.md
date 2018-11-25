
# Instructions & Tasks

In this learning activity, you will create a Ghost Blog service using Docker Compose.

  1.  Log in to your live environment and sudo to root.
  2.  Create a Docker Compose file in the root directory of your live environment. You will create two services: a Ghost Blog service and a MySQL service.
  3.  Set the Compose version to 3.
  4.  Create your first service called ghost.
  5.  Use the ghost:1-alpine image.
  6.  Call the container ghost-blog.
  7.  You will use five environment variables:
        Set database__client to mysql.
        Set database__connection__host to mysql.
        Set database__connection__user to root.
        Set database__connection__password to P4sSw0rd0!
        Set database__connection__database to ghost.
  8.    Create a volume called ghost-volume and map it to /var/lib/ghost.
  9.  Map port 80 on the host to port 2368 on the container.
  10. The ghost_blog container will be dependent on the mysql container.
  11. Make sure that the container always restarts.
  12. Create a second service called mysql.
  13. Use the mysql:5.7 image.
  14. Name the container ghost-db.
  15. You will add the following environment variable:
          Set MYSQL_ROOT_PASSWORD to P4sSw0rd0!
  16. Create a volume called mysql-volume and map it to /var/lib/mysql.
  17. Make sure that the container always restarts.
  18. Make sure that the two volumes are called ghost-volume and mysql-volume.
  19. Execute a compose up and make sure to use the build and detached flags.
  20. Verify that your app is up and running.

## Eg :

'''
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
07f1a2fe8484        ghost:1-alpine      "docker-entrypoint.s…"   7 minutes ago       Up 7 minutes        0.0.0.0:80->2368/tcp   ghost-blog
bf330a0a6523        mysql:5.7           "docker-entrypoint.s…"   7 minutes ago       Up 7 minutes        3306/tcp, 33060/tcp    ghost-db
[root@ip-10-0-1-195 ~]#
'''
