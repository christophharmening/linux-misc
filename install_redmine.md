# Install Redmine Ticket System on Debian 12

Install required packages
```
apt-get install apt-transport-https ca-certificates dirmngr gnupg2 make gcc linux-headers-$(uname -r) apache2 apache2-dev libapache2-mod-passenger mariadb-server mariadb-client build-essential ruby-dev libxslt1-dev libmariadb-dev libxml2-dev zlib1g-dev imagemagick libmagickwand-dev curl -y
```

Setup a root password for mysql
```
mysql_secure_installation
```

Login into mysql
```
mysql -u root -p
```

Create a database for redmine
```
CREATE DATABASE redminedb CHARACTER SET utf8mb4;
```

and setup a DB user for redmine
```
GRANT ALL PRIVILEGES ON redminedb.* TO 'redmineuser'@'localhost' IDENTIFIED BY 'password';
```
```
FLUSH PRIVILEGES;
```
```
quit;
```


Add Linux redmine user
```
useradd -r -m -d /opt/redmine -s /usr/bin/bash redmine
```
get into webgroup
```
usermod -aG www-data redmine
```

Download redmine and extract 
```
mkdir /opt/remine
```
```
tar -xvzf redmine-VERSION.tar.gz -C /opt/redmine/ --strip-components=1
```
Change owner
```
chown -R redmine: /opt/redmine
```

Go into directory as redmine user
```
su redmine
```
```
cd /opt/redmine
```

Next copy the default config files
```
cp /opt/redmine/config/configuration.yml{.example,}
```
```
cp /opt/redmine/public/dispatch.fcgi{.example,}
```
```
cp /opt/redmine/config/database.yml{.example,}
```

Connect redmine with your database
```
nano /opt/redmine/config/database.yml
```
```
production:
  adapter: mysql2
  database: redminedb
  host: localhost
  username: redmineuser
  password: "password"
  ...
  # transaction_isolation: "READ-COMMITTED"
  tx_isolation: "READ-COMMITTED" 
```

Redmine reqire bundler
```
gem install bundler
```
or with proxy
```
gem install --http-proxy http://PROXYUSER:PROXYPW@ADDRESS:80 bundler
```
Next install from **redmine main folder as user redmine** 
```
bundle install --without development test --path vendor/bundle
```
or if you use a proxy you must setup the http_proxy and https_proxy system variables
```
export http_proxy=http://PROXYUSER:PROXYPW@ADDRESS:80
```
```
export https_proxy=http://PROXYUSER:PROXYPW@ADDRESS:80
```

Next, generate a secret token with the following command:
```
bundle exec rake generate_secret_token
```

Next, create a Rails database structure and insert default configuration data into the database with the following command:
```
RAILS_ENV=production bundle exec rake db:migrate
```
```
RAILS_ENV=production REDMINE_LANG=en bundle exec rake redmine:load_default_data
```

Now create some required directorys and files
```
for i in tmp tmp/pdf public/plugin_assets; do [ -d $i ] || mkdir -p $i; done
```
```
chmod -R 755 /opt/redmine
```

## Setup apache

First create a Certificate like here 

