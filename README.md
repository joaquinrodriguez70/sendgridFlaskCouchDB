# How to receive Sendgrid webhooks events using amazon, python and  couchdb 

1.- Create a  amazon AMI (t2.medium) with 2 cpus and 4 gb of memory


2.-Do an update 

`sudo su
yum update -y
yum upgrade -y
yum install -y gcc-c++
yum install -y libxml2-python libxml2-devel
yum install -y python-devel
easy_install 	
easy_install Flask
yum install -y nginx`

3.-Change NGINX configuration file for proxy.
As root 

`vi /etc/nginx/nginx.conf`
Find the location / section, and change it to as follows:

`location / {
    include uwsgi_params;
    uwsgi_pass 127.0.0.1:10080;
}`


4.-Start NginX

`service nginx start
chkconfig nginx on`
	
	
5.- Note that user directories are on  /usr/share/nginx/  but we're going to use a python script in  the "ec2-user" folder



6.-Install Python 

`sudo yum install python34
sudo yum install python34-devel`

7.-Download a script to install Flask  as ec2-user

`mkdir ~/pythonsetup

cd ~/pythonsetup

wget https://bootstrap.pypa.io/get-pip.py

sudo python3  get-pip.py

sudo /usr/local/bin/pip3 install flask

sudo /usr/local/bin/pip3 install uwsgi

sudo /usr/local/bin/pip3 install couchdb

sudo /usr/local/bin/pip3 install request`

8.- Copy pysendgrid.py to ~/pysendgrid

9.-In ~/pysendgrid create app.yaml 

vi app.yaml

uwsgi:
  socket: 127.0.0.1:10080
  master: 1
  workers: 1
  chmod-socket: 666
  auto-procname: 1
  python-path: .
  pidfile: /tmp/uwsgi.pid
  daemonize: /var/log/uwsgi.log
  module: pysendgrid:app


10.-Change permissions

sudo touch /var/log/uwsgi.log 
sudo chown ec2-user /var/log/uwsgi.log                             


11.-Start the app 

uwsgi --yaml app.yaml



12.-create ~/mozilla

mkdir ~/mozilla

13.-Get sources and compile mozilla javascript engine

cd ~/mozilla
wget http://ftp.mozilla.org/pub/mozilla.org/js/js185-1.0.0.tar.gz
tar zxvfl js185-1.0.0.tar.gz
cd js-1.8.5/js/src/
/usr/local/autoconf-2.13/bin/autoconf
sudo ./configure
sudo make
sudo make install 


14.- Modify /etc/yum.repos.d/epel.repo. Under the section marked [epel] , change enabled=0 to enabled=1.

15.-install requisites for couchdb

sudo yum install gcc gcc-c++ libtool curl-devel ruby-rdoc zlib-devel openssl-devel make automake rubygems perl git-core

sudo yum groupinstall "Development Tools"
sudo yum install perl-Test-Harness
sudo yum install erlang-erts
sudo yum install erlang-os_mon

sudo yum install erlang-tools
sudo yum install erlang-eunit
sudo yum install libicu-devel
sudo yum install autoconf-archive 
sudo yum install curl-devel
sudo yum install erlang-etap
sudo yum install libicu-devel
sudo yum install erlang-eunit
sudo yum install erlang-sasl
sudo yum install erlang-os_mon
sudo yum install erlang-asn1
sudo yum install erlang-xmerl

16.-Download couchdb and extract

17.-Compile couchdb

sudo ./configure --with-erlang=/usr/lib64/erlang/usr/include/
sudo make 
sudo make install

18.-Create users
For security reasons, create a dedicated couch user and correct files ownership and permissions:

sudo adduser -r --home /usr/local/var/lib/couchdb -M --shell /bin/bash --comment "CouchDB Administrator" couchdb
sudo mkdir /usr/local/etc/couchdb
sudo mkdir /usr/local/var
sudo mkdir /usr/local/var/lib
sudo mkdir /usr/local/var/log
sudo mkdir /usr/local/var/run
sudo mkdir /usr/local/var/lib/couchdb
sudo mkdir /usr/local/var/log/couchdb
sudo mkdir /usr/local/var/run/couchdb
sudo chown -R couchdb:couchdb /usr/local/etc/couchdb
sudo chown -R couchdb:couchdb /usr/local/var/lib/couchdb
sudo chown -R couchdb:couchdb /usr/local/var/log/couchdb
sudo chown -R couchdb:couchdb /usr/local/var/run/couchdb
sudo chmod 0770 /usr/local/etc/couchdb
sudo chmod 0770 /usr/local/var/lib/couchdb
sudo chmod 0770 /usr/local/var/log/couchdb
sudo chmod 0770 /usr/local/var/run/couchdb
chown couchdb /usr/local/etc/couchdb/local.ini

19.-run couchdb
sudo -i -u couchdb /usr/local/bin/couchdb -b

20.- create a database
HOST="http://127.0.0.1:5984"
curl -X PUT $HOST/correos
curl -X PUT $HOST/_config/admins/**COUCHDBUSER** -d '"**COUCHDBPASS**"' ""
HOST="http://**COUCHDBUSER**:**COUCHDBPASS**@127.0.0.1:5984"
curl $HOST

21-create a ssh tunnel
ssh -L5984:127.0.0.1:5984 -i **YOUREC2KEY**.pem ec2-user@**YOUREC2HOST*

connect to futon

22.http://127.0.0.1:5984/_utils/
