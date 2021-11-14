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
3. Setup database container, specifically MariaDB
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
4. Begin WordPress container setup 
    - Fetch the docker install for WordPress with the command `sudo docker pull wordpress:latest`
    - Create a wpcontainer with the following command 
    ```
    docker run -e WORDPRESS_DB_USER=wpuser -e WORDPRESS_DB_PASSWORD=pass -e WORDPRESS_DB_NAME=wordpress_db -p 8081:80 -v /root/wordpress/html:/var/www/html --link wordpressdb:mysql --name wpcontainer -d wordpress
    ```
   - Again, note that the user, password, and DB_Name can be changed in addition with the name of the container 
   - Find you IP using the command `ip addr show` 
   - With that IP found issue the command `curl -I [IP]:8081` to check the state of the WordPress container 
    - The result should look like ![docker status](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/status.png) 
5. Install Nginx web server and configure 
  - Install nginx with the command `sudo apt-get install nginx`
  - Next we'll need to go to the nginx director and add a new virtual host for Wordpress docker container 
    - Issue these commands 
    ```
    cd /etc/nginx/sites-available/
    sudo nano wordpress
    ```
    - Once in the file add these lines then save and exit 
   ```
   server {
  listen 80;
  server_name wp-docker.co www.wp-docker.co;
 
  location / {
    proxy_pass http://localhost:8081;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }
  ```
  - Note the server_name can be set to whatever you want 
  - Lastly we need to activate the new Wordpress vitual host and remove the default config, issue the following commands
  ```
  ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
  rm -f /etc/nginx/sites-available/default
  rm -f /etc/nginx/sites-enabled/default
  ```
  - Then reset the nginx server `systemctl restart nginx`
6. Final Installation 
    - Issue the command `curl -I [IP]:8081` once more
    - Once issued it will take you to an installation site from your docker to install WordPress and will request you fill out your information and submit 
    - Once submitted login, it should now take you to your WordPress installation with your user information looking as such ![Wordpress Installed](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/final.png)
    - User installed should look like this in addition ![Wordpress User](https://github.com/RyanDerr/Wordpress-Docker/blob/main/Images/user.png)
    - ***After this you have completed the docker installation for WordPress***
