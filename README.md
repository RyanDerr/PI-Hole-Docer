# Wordpress Docker Install 
## What is Wordpress?
WordPress is a free and open-source content management system written in PHP and paired with a MySQL or MariaDB database
## Installation Guide based on [HowtoForge's Guide](https://www.howtoforge.com/tutorial/how-to-install-wordpress-with-docker-on-ubuntu/)
**Assumption for setup include a stable internet connection and an already existing Debian based distribution (Ubuntu 20.04 for me) to setup the PI-Hole**
1. First update the Linux distro and install docker 
    - The commands to execute this are as follows:
     ```
     sudo apt-get update
     sudo apt upgrade
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" |  sudo   tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io
    sudo usermod -aG docker [user]
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
   ```
2. Next we'll need to start docker 
    - Issue the command `sudo systemctl start docker`
## Install with Docker-Compose 
1. First create a directory for wordpress, I used `mkdir ~/wordpress`
2. Create a yml file within that directory called `docker-compose.yml`
3. Edit the yml file just created to contain the following information 
    ```
    version: "3.9"
    services:
    db:
        image: mysql:5.7
        volumes:
          - db_data:/var/lib/mysql
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: somewordpress
          MYSQL_DATABASE: wordpress
          MYSQL_USER: wordpress
          MYSQL_PASSWORD: wordpress
   wordpress:
        depends_on:
          - db
        image: wordpress:latest
        volumes:
          - wordpress_data:/var/www/html
        ports:
          - "8000:80"
        restart: always
        environment:
          WORDPRESS_DB_HOST: db:3306
          WORDPRESS_DB_USER: wordpress
          WORDPRESS_DB_PASSWORD: wordpress
          WORDPRESS_DB_NAME: wordpress
    volumes:
       db_data: {}
       wordpress_data: {}
    ```
4. Run `sudo docker-compose up -d` 
    - This builds and deploys the container you've edited 
5. Within a web bowser search either `http://localhost:8000` or enter `https://[IP]:8000`
    - Once issued it will take you to an installation site from your docker to install WordPress and will request you fill out your information and submit 
    - Once submitted login, it should now take you to your WordPress installation with your user information looking as such ![Wordpress Installed](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/final.png)
    - User installed should look like this in addition ![Wordpress User](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/user.png)
    - ***After this you have completed the docker installation for WordPress***
## Install Without Docker Compose 
1. Setup database container, specifically MariaDB
    - To install the container issue the command `sudo docker pull mariadb`
    - Next begin creating directories for data storage as such 
    ```
    mkdir ~/wordpress
    mkdir -p ~/wordpress/database
    mkdir -p ~/wordpress/html
    ```
    - Issue the command to create a new container for MariaDB 
    ```
    sudo docker run -e MYSQL_ROOT_PASSWORD=password -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpass -e MYSQL_DATABASE=wordpress_db -v /root/wordpress/database:/var/lib/mysql --name wordpressdb -d mariadb
    ```
      - Note that you can change the passwords and user to whatever is desired on your end, I did this as an example 
   - You can then inspect the container with the command `sudo docker inspect -f '{{ .NetworkSettings.IPAddress }}' wordpressdb`
   - To log in to check databased issue the commands
   ```
   sudo mysql -u mysqluser -h 172.17.0.2 -p 
   show databases;
   Exit;
   ```
   - The result should look as such ![DBImage](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/mysql.png)
2. Begin WordPress container setup 
    - Fetch the docker install for WordPress with the command `sudo docker pull wordpress:latest`
    - Create a wpcontainer with the following command 
    ```
    docker run -e WORDPRESS_DB_USER=wpuser -e WORDPRESS_DB_PASSWORD=pass -e WORDPRESS_DB_NAME=wordpress_db -p 8081:80 -v /root/wordpress/html:/var/www/html --link wordpressdb:mysql --name wpcontainer -d wordpress
    ```
   - Again, note that the user, password, and DB_Name can be changed in addition with the name of the container 
   - Find you IP using the command `ip addr show` 
   - With that IP found issue the command `curl -I [IP]:8081` to check the state of the WordPress container 
    - The result should look like ![docker status](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/status.png) 
3. Final Installation 
    - Issue the command `curl -I [IP]:8081` once more
    - Once issued it will take you to an installation site from your docker to install WordPress and will request you fill out your information and submit 
    - Once submitted login, it should now take you to your WordPress installation with your user information looking as such ![Wordpress Installed](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/final.png)
    - User installed should look like this in addition ![Wordpress User](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/user.png)
    - ***After this you have completed the docker installation for WordPress***
