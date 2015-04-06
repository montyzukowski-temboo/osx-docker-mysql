#osx-docker-mysql, a.k.a dgraziotin/mysql

osx-docker-mysql, which is known as 
[dgraziotin/mysql](https://registry.hub.docker.com/u/dgraziotin/mysql/) 
in the Docker Hub, is a reduced fork of 
[dgraziotin/osx-docker-lamp](https://github.com/dgraziotin/osx-docker-lamp), 
which is an "Out-of-the-box LAMP image (PHP+MySQL) for Docker". 

osx-docker-mysql is instead an:

	Out-of-the-box MySQL Docker image that *just works* on Mac OS X.

However, it has also been tested for Docker running under GNU/Linux (Ubuntu 14.10).

Some info about osx-docker-mysql:

- It is based on [phusion/baseimage:latest](http://phusion.github.io/baseimage-docker/)
  instead of ubuntu:trusty.
- It fixes OS X related [write permission errors for MySQL](https://github.com/boot2docker/boot2docker/issues/581)
- It lets you mount OS X folders *with write support* as volumes for
  - The database
- It creates a default database and user with permissions to that database
- It is documented for less advanced users (like me)


##Usage

If you need to create a custom image `youruser/mysql`, 
execute the following command from the `osx-docker-mysql` source folder:

	docker build -t youruser/mysql .

If you wish, you can push your new image to the registry:

	docker push youruser/mysql

Otherwise, you are free to use dgraziotin/mysql as it is provided. Remember first
to pull it from the Docker Hub:

    docker pull dgraziotin/mysql


###Running your LAMP docker image

If you start the image without supplying your code, e.g.,

	docker run -t -i -p 3306:3306 --name db dgraziotin/mysql

At [boot2docker ip] you should be able to connect to MySQL.

###Loading your custom MySQL files

If you wish to mount a MySQL folder locally, so that MySQL files are saved on your
OS X machine, run the following instead:

	docker run -i -t -p "3306:3306" -v ${PWD}/mysql:/mysql --name db dgraziotin/mysql

The MySQL database will thus become persistent at each subsequent run of your image.

##Environment description

###The /mysql folder

MySQL is configured to serve the files from the `/mysql` folder, which is a symbolic
link to `/var/lib/mysql`. In osx-docker-mysql, the MySQL user `mysql` 
has full write permissions to the `mysql` folder.

###MySQL

MySQL runs as user `mysql` and group `staff`.

####The three MySQL users

The bundled MySQL server has three  users, that are `root`, `admin`, and `user`. 

The `root` account comes with an empty password, and it is for local connections
(e.g., using some code). The `root` user cannot remotely access the database 
(and the container).

However, the first time that you run your container, a new user `admin` 
with all root privileges  will be created in MySQL with a random password. 

To get the password, check the logs of the container by running:

	docker logs [name or id, e.g., mywebsite]

You will see an output like the following:

	========================================================================
	You can now connect to this MySQL Server using:

	    mysql -uadmin -p47nnf4FweaKu -h<host> -P<port>

	Please remember to change the above password as soon as possible!
	MySQL user 'root' has no password but only allows local connections
	========================================================================

In this case, `47nnf4FweaKu` is the password allocated to the `admin` user.

Finally, a user called `user` with password `password` is created for your convenience.
The `user` user has full privileges on a database called `db`, which is also created
for your convenience.

##Environment variables

- MYSQL_ADMIN_PASS="mypass" will use your given MySQL password for the `admin`
user instead of the random one.
- MYSQL_USER_NAME="daniel" will use your given MySQL username instead of `user`
- MYSQL_USER_DB="supercooldb" will use your given database name instead of `db`
- MYSQL_USER_PASS="supersecretpassword" will use your given password  instead of `password`

Set these variables using the `-e` flag when invoking the `docker` client.

	docker run -i -t -p "3306:3306" -v ${PWD}/mysql:/mysql -e MYSQL_ADMIN_PASS="mypass" --name db dgraziotin/mysql

Please note that the MySQL variables will not work if an existing MySQL volume is supplied.