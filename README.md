# docker-home
* This docker-compose will automatically set up the `sparcs.org` for domains `sparcs.org` and `www.sparcs.org`.
* This docker-compose will automatically set up the `nugu` service for domain `nugu.sparcs.org`.
* Make sure that the DNS records of `sparcs.org`, `www.sparcs.org`, and `nugu.sparcs.org` to be equal to the IP address of your host before starting the following jobs.
* You should change the password of users `sysop` and `wheel` by executing the following command at your host:
```shell
sudo docker exec -it home-server /bin/bash -c "echo SYSOP && passwd sysop && echo WHEEL && passwd wheel"
sudo docker exec -it nugu-server /bin/bash -c "echo SYSOP && passwd sysop && echo WHEEL && passwd wheel"
```
* You should issue and install the certificate by executing the follwing command at your host:
```shell
sudo docker exec nugu-server /bin/bash -c "\
  a2ensite home && \
  a2dissite 000-default && \
  service apache2 restart"
```

## Setup
```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.23.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
sudo systemctl enable docker
git clone https://github.com/sparcs-kaist/docker-home.git
cd docker-home
mkdir log-db log-home log-nugu data
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport [ip]:/data data
sudo /bin/bash -c \
  "echo [ip]:/data /home/wheel/docker-home/data nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,_netdev,noresvport 0 0 >> /etc/fstab"
```
* The following files should exist in the `./home-db` directory:
  * `mongodb.env`
  * `secret.env`
  * `mongodb.conf`
* The following directory should exist in the `./home-db` directory:
  * `db`
* The following files should exist in the `./home-server` directory:
  * `localconfig.js`
  * `semantic.json`
* The following directory should exist in the `./home-server` directory:
  * `semantic`
* The following files should exist in the `./nugu-server` directory:
  * `settings_local.py`
  * `home.conf`
```shell
sudo docker-compose up -d
```
* After containers are up, you should proxy each domain's request to corresponding container
```shell
sudo apt-get -y install nginx
cp /path/to/sparcs.org /etc/nginx/sites-available/sparcs.org # sparcs.org can be found in repository root
cp /path/to/nugu.sparcs.org /etc/nginx/sites-available/sparcs.org
sudo ln -s /etc/nginx/sites-available/sparcs.org /etc/ngins/sites-enabled/sparcs.org
sudo ln -s /etc/nginx/sites-available/nugu.sparcs.org /etc/ngins/sites-enabled/nugu.sparcs.org
sudo systemctl start nginx
```

* After nginx is up, you should generate ssl certificate to each domains with certbot.
```shell
apt-get install -y software-properties-common python-software-properties
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install -y python-certbot-apache
certbot --nginx -d sparcs.org nugu.sparcs.org
# On prompt, enter
# 1. wheel@sparcs.org
# 2. Agree to terms of service
# 3. Disagree sharing email
# 4. 2: Redirect
```

* Check whether sparcs.org and nugu.sparcs.org are working fine or not.
