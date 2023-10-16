## Laravel Framework Deployed on http://138.68.138.233/


## Angular Framework deployed on http://209.97.128.58/


### The Angular folder contains the angular file and the DevOps framework contains the laravel file

## Deploy and Run  the latest version of Laravel and Angular Frameworks on DigitalOcean 

To deploy and run the latest version of Laravel and Angular frameworks, we need to install dependencies such as: 

We will need a DigitalOcean Account

- Laravel: PHP, Composer
- Angular: Node.js, NPM
- Server: Nginx/Apache
- DigitalOcean: Droplets with ubuntu 20.04 recomended, select droplet plan, and data center region. 
- Create your public key in your terminal, add it to your DigitalOcean keys and use it to SSH into your root ip


## Install LEMP Stack (Linux, Ngninx, PHP, MySQL)

```
sudo apt update
sudo apt install nginx
```

Install MySQL

```
sudo apt install mysql-server
```


Install PHP and Composer

```
sudo apt install php php-fpm php-mbstring php-xml composer
```

### Create laravel project 

Navigate to the `/var/www` directory and create a new Laravel project using Composer. 

- composer create-project --prefer-dist laravel/laravel devops ; `devops` is the name of my laravel project

- Navigate to the project directory using `cd devops`


### Configure .env File 

Copy `.env.example`to `.env` and modify the database. 

```
cp .env.example .env

nano .env
```

```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

<other configs/env vars>
```

### Generate Application Key using `php artisan key:generate`


Create your database using `CREATE DATABASE devops;`

Create a User

`CREATE USER 'devops'@'localhost' IDENTIFIED BY 'devops';`

Grant permissions to the User

`GRANT ALL PRIVILEGES ON devops.* TO 'devops'@'localhost';`


### Set Permissions 

- Change the owner and group: `chown -R www-data:www-data /var/www/devops`
- chmod -R 755 /var/www/devops/storage

### Configure Nginx for Laravel

Navigate to your Ngnix sites-available directory:

```
cd /etc/nginx/sites-available
```

server {
    listen 80;
    server_name 142.93.34.71;

    root /var/www/devops/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

- Enable the Nginx site configuration by creating a symbolic link and test the configuration:

```
ln -s /etc/nginx/sites-available/[Your_Project_Name] /etc/nginx/sites-enabled/

nginx -t
```

You should see an output that says: 

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Install Angular on the Droplet

- curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -

- sudo apt-get install -y nodejs

- node -v

- npm -v


### Install Angular CLI

Install Angular CLI globally using NPM:

- sudo npm install -g @angular/cli


### Create new Angular APP

- cd /var/www

- ng new angular


### Build the Angular App

- cd angular 

- ng build --prod

### Configure Nginx to Serve Angular App 

```
server {
    listen 80;
    server_name 142.93.34.71;

    # Serve Laravel
    location /api {
        alias /var/www/devops/public;
        index index.php;
        
        try_files $uri $uri/ /index.php?$query_string;

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }

    # Serve Angular
    location / {
        root /var/www/angular/dist/angular;
        index index.html;

        try_files $uri $uri/ /index.html;

        location ~ /\.ht {
            deny all;
        }
    }
}
```




