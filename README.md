# setup-vps-digital-ocean

This project is documantation for setup VPS Digital Ocean.

# Sign in into Digital Ocean
![image](https://user-images.githubusercontent.com/22932211/125054880-dcf81f80-e0d0-11eb-83c1-c2e4033824fc.png)

# Create New Droplets
- Click Create -> Droplets
  ![image](https://user-images.githubusercontent.com/22932211/125057900-ea62d900-e0d3-11eb-8504-6ac8cc18d0c6.png)
- Choose your OS, i'm using Ubuntu 18.04 LTS
- Choose a plan, i'm using `Premium Intel with NVMe SSD -> 4gb/2 intel cpu, 80gb NVMe SSD, 4tb transfer`
- Choose region, i'm using Singapore
- Add Authentication, i'm using password
- Create root password
- Choose a hostname, this is the name for your Droplet
- Create droplet

# Add Domain
- Click Add Domain
  ![image](https://user-images.githubusercontent.com/22932211/125057564-92c46d80-e0d3-11eb-9412-098b17b50e41.png)
- Type your domain
- Click button `Add Domain`

# Get Name Server and Add A Record for pointing domain
- Click your domain, and check your `NS`. And then register it into your platform website where you bought the domain.
  ![image](https://user-images.githubusercontent.com/22932211/125057172-25b0d800-e0d3-11eb-830f-758a32cccefd.png)
- Add A record to your domain, and register it again into your platform website where you bought the domain.

# Setup Firewall Digital Ocean
- Open Manage -> Networking 
- Open tab Firewalls
- Set your Inbounds Rules like :
  - Port 22 - All IPv4 - All IPv6
  - Port 80 - All IPv4 - All IPv6
  - Port 443 - All IPv4 - All IPv6
  - Port 3306 - All IPv4 - All IPv6
  - Port 3000 - All IPv4 - All IPv6
- Set your Outbounds Rules like :
  - TCP All Ports
  - UDP All Ports
 - Set the droplest, and add your droplet

# SSH into VPS
- I'm using `WinSCP` and `Putty`
- Open your `WinSCP`
- Click `new site` and set :
  - File protocol `SFTP`
  - Host name is `Your Droplet IP Address`
  - Port number `22`
  - Username `root`
  - Password `Your password root Droplet` 
- Login

# Fresh install
- Open your `putty session` in `winscp`
- Login it with your password
- run `apt-get update`
- run `apt-get upgrade`
- run `apt-get install nginx -y` for using nginx as webserver
- run `service nginx start` for running service nginx
- install php 7.2
  - run `apt-cache pkgnames | grep php7.2`
  - run `apt-get install php -y`
  - run `apt-get install php-{bcmath,bz2,intl,gd,mbstring,mysql,zip,fpm} -y`
  - run  `service nginx restart`
- install certbot for ssl
  - run `add-apt-repository ppa:certbot/certbot`
  - run `apt install python-certbot-nginx`
- run `apt-get install git`
- run `apt-get install build-essential`
- run `apt-get install curl openssl libssl-dev`
- run `apt-get install nodejs`
- run `apt-get install npm`
- run `npm install pm2 -g`

# Setup Virtual Host - Project PHP
- Create project in directory `/var/www/yourdomain.com/htdocs/index.php`, please check your permission directory
- open directory `/etc/nginx/sites-available`
  - create file `yourdomain.com.conf`
  - Copy and edit this code
    ```
    server {
        server_name yourdomain.com;
        root /var/www/yourdomain.com/htdocs;
        index index.php index.pl index.cgi index.html index.xhtml index.htm;
        underscores_in_headers on;

        location / {
          try_files $uri $uri/ /index.php?$args;
        }
        location ~ \.php$ {
              include snippets/fastcgi-php.conf;
              fastcgi_pass unix:/run/php/php7.2-fpm.sock;
        }
        location ~ /\.ht {
              deny all;
        }
        location ~* \.(eot|ttf|woff|woff2|jpg|jpeg|png|pdf)$ {
              add_header Access-Control-Allow-Origin *;
        }

        location ~ /\.git {
            return 404;
        }
      }
    ```
  - save file
- open directory `/etc/nginx/sites-enables`
  - create link `yourdomain.com.conf`
  - point link set into directory `/etc/nginx/sites-available/yourdomain.com.conf`
- run `service nginx restart`
- add ssl
  - run certbot --nginx -d yourdomain.com
  - input email and answer another questions
  - choose `2 - Redirect - Make all requests redirect to secure HTTPS access`
- open your domain in browser

# Setup Virtual Host - Project Node
- Create project in directory `/var/www/yourdomain.com/htdocs/`, please check your permission directory
- Open the directory project and run `pm2 start "npm run start" --name "web-reactjs"`, please check your port project
- open directory `/etc/nginx/sites-available`
  - create file `yourdomain.com.conf`
  - Copy and edit this code
    ```
    upstream web_reactjs {
        server 127.0.0.1:3000;
    }

    server {
        # listen on both hosts
        server_name yourdomain.com;

        location / {
            try_files $uri @proxy;
        }

        location @proxy {
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Url-Scheme $scheme;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_pass  http://web_reactjs;
        }
    }
    ```
  - save file
- open directory `/etc/nginx/sites-enables`
  - create link `yourdomain.com.conf`
  - point link set into directory `/etc/nginx/sites-available/yourdomain.com.conf`
- run `service nginx restart`
- add ssl
  - run certbot --nginx -d yourdomain.com
  - input email and answer another questions
  - choose `2 - Redirect - Make all requests redirect to secure HTTPS access`
- open your domain in browser
