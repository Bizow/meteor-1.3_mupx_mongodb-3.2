### mupx Deployment on AWS EC2

> Process for deploying multiple meteor 1.3 apps to a single aws ec2 
instance pointing at the same mongo db at version 3.2.

#### Ubuntu Server 14.04 LTS

##### Configure instance security group
It is important to configure your security group with the following entry
which allows for the instance to connect to mongodb as we will turn OFF
the automatically created db by mupx.

| Type              | Protocol      | Port Range   |   Source    |
| :----------------:|:-------------:| :-----------:| :--------:  |
| Custom TCP Rule   | TCP           | 27017        | hostname    |


##### Connect to instance
    ssh -i ~/.ssh/pemfilename.pem ubuntu@hostname


##### Install and configure nginx

    sudo apt-get update
    sudo apt-get install nginx
    sudo vi /etc/nginx/sites-enabled/default
sample config for default

    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''     close;
    }

    server {
            listen 80;
            server_name yourdomain.com;
            location / {
                    proxy_pass http://127.0.0.1:3000;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header Host $host;
    
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection $connection_upgrade;
            }
    }

or if you have ssl

    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''     close;
    }
    
    server {
        listen 80;
        server_name yourdomain.com;
        return 301 https://$server_name$request_uri;
    }
    
    server {
            listen 443 ssl;
            server_name yourdomain.com;
            ssl_certificate     /path/to/name.chained.crt;
            ssl_certificate_key /path/to/name.key;
            
            location / {
                    proxy_pass http://127.0.0.1:3000;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header Host $host;
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection $connection_upgrade;
            }
    }


reload nginx after saving default

    sudo nginx -s reload
    
##### Install and configure MongoDB

[Follow official MongoDB Instructions](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)

    sudo vi /etc/mongod.conf

comment out the following line

    bind_ip = 127.0.0.1
    
then restart mongo

    sudo service mongod restart
    
connect to mongo and create user
    
    mongo
    
    use myappname
    
    db.createUser({
        user: "username",
        pwd: "password",
        roles: [
            { role: "readWrite", db: "myappname" }
        ]
      })
      
##### Install and configure mupx

[Follow official mupx Instructions](https://github.com/arunoda/meteor-up/tree/mupx)

See sample mup.json file in project.






