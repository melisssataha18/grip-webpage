---
title: "Getting Started Guide"
date: 2021-03-05T17:12:57-08:00
draft: false
---
# This is where the web platform will live!

## Creating the env
```bash
conda create -n venv_tess python=3.8
conda activate venv_tess
pip install -r requirements.txt
```

## Ensure you install mysql - first time only!
```bash
brew install mysql
brew services start mysql
mysqladmin -u root password 'password'
brew services stop mysql
```

## Checking out the DB
```bash
brew services start mysql
# I use TablePlus, so at this point, launch tableplus and add in the parameters you created above
# namely, localhost, username = root and password. You can leave the DB name blank for now

# Alternatively, you can use the command line
mysql -u root -p 'your root password'

# Then, create the tess_user, something like:
# 👇 this user and password is for the development config
# CREATE USER 'tess_user'@'localhost' IDENTIFIED BY 'tess_db_password_local';
# FLUSH PRIVILEGES;
# Make sure to grant the appropriate perms for this user.
# You can follow instructions here if you're unsure: https://tableplus.com/blog/2019/08/how-to-manage-user-mysql-tableplus-gui.html

# Lastly, create the DB - name it tess
```

## Almost there...
```bash
# set the flask environmental variable
export FLASK_APP=web

# the TESS DB now exists but its schema is still in the ether.
# to ensure that our python models are reflected as tables, run
flask db upgrade  

# you only need to do this when first creating the DB or whenever
# there are new models or changes to existing models!!
```

## Our frontend is currently leveraging react and mdc to generate the whole experience
Everything gets bundled via webpack, however, for sake of speed (since I don't want to deal with routing on the web client) this is not a SPA, so we need to manually register every file to bundle in the webpack config. Anyways...
```bash
# make sure you have node/nvm installed
nvm use  # this will ensure that the node verison is loaded to the pegged version in the .nvmrc
# make sure you have yarn installed, https://classic.yarnpkg.com/en/
yarn install # install all the dependencies in the package.json
yarn build   # bundle and deploy all the assets referenced in the application
```

## Starting the Redis Server
```bash
redis-server
```

## Running the project
```bash
flask run
```
... navigate to `localhost:5000`


## Simulating a more production style application run:
```bash
# green unicorn is in the requirements.txt, so it should be installed already
# if not, pip install gunicorn
gunicorn web:app
> Starting gunicorn 20.0.4
> Listening at: http://127.0.0.1:8000 (30581)
> Using worker: sync
> Booting worker with pid: 30583
# navigate to locahost:8000 and everything should work as normal
```

----

Some things worth noting and some resources
- Our frontend is leveraging the React Wrapper for the Material Design Components (MDC) from Google: https://rmwc.io/ and https://material.io/develop/web/

----

## Setting up the AMI and getting the application running in EC2
These are the steps I took
```bash
sudo yum install python36 python36-pip
sudo yum install git -y
git clone https://github.com/slacgismo/TESS.git
sudo yum install python36-devel

# sudo yum install python36-setuptools
# curl -O https://bootstrap.pypa.io/get-pip.py

python3 get-pip.py
python3 -m pip install Flask
python3 -m pip install -r requirements.txt
curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
curl -sL https://rpm.nodesource.com/setup_12.x | sudo bash -
sudo yum install -y nodejs
sudo yum install yarn
yarn install
yarn build
sudo yum install epel-release
sudo yum install nginx
sudo service nginx start
python3 -m pip install supervisor
supervisord
cd /etc/nginx
sudo mkdir sites-available
sudo mkdir sites-enabled
cd sites-available/
sudo touch tess
sudo nginx -t
sudo service nginx restart

# exported vars
export FLASK_ENV="production"
export FLASK_DEBUG="0"
export DB_PASSWORD="add-the-actual-password-here!!!this is not it"
export DB_SERVER="tess-dev.cudndiboutru.us-west-1.rds.amazonaws.com"
export DB_USER="admin"

# restart the process
supervisorctl restart tess

# TROUBLESHOOTING
# did you pip install new dependencies?
# did you yarn install new dependencies?
# did you check the logs in /tmp/tess*?
```
To update the code with the latest changes:
- SSH into the above machine
- git pull --rebase the latest from master
- install the new python/js dependencies if needed
- run the new migrations if needed
- supervisorctl restart tess
