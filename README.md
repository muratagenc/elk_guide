# elk_guide
Complete Guide to ElasticSearch, Logstash and Kibana installation

************************************************************************
new Ubuntu installation on VBox

Raw installation
Download Ubuntu from ubuntu.com/download (choose Ubuntu Server 15)
create a new VM on VBox, 8GB disk, 1GB memory, using the ISO just downloaded.
do not select any server applications during the installation
login to the server using your username and password
run sudo apt-get update to update the packages

************************************************************************
Running SSH server and using SSH keys

Generate new SSH keys on your local computer, which will be the client to connect to the server
type ssh-keygen on linux terminal, 
use default filename or give another name for the private key
enter a passphrase to protect your private key (in case someone steals your key)
now, we will convert the *public* key to a pem file later, because MS Azure can accept only pem files, not the one we just created.

-rw-------. 1 mgenc mgenc 1766 Nov 10 15:44 murat2ndprivate_id_rsa
-rw-r--r--. 1 mgenc mgenc  409 Nov 10 15:44 murat2ndprivate_id_rsa.pub

type the following command to convert your public key to pem file, for Azure.

#ssh-keygen -f murat2ndprivate_id_rsa.pub -e -m pem

copy and paste the output into a pem file.
change permissions to 700

chmod 700 murat2ndprivate_id_rsa_pub.pem

-rwx------. 1 mgenc mgenc  427 Nov 10 15:47 murat2ndprivate_id_rsa_pub.pem

enable SSH on the server
sudo apt-get install openssh-server
take a backup of the ssh server configuration file;
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
make the backup copy read only
sudo chmod a-w /etc/ssh/sshd_config.backup

edit the sshd_config file with vi (or any other text editor you like)
sudo vi /etc/ssh/sshd_config
PermitRootLogin no

re-start the SSH server to load the new configuration file;
sudo systemctl restart ssh

install nmap to check open ports on the server (optional)
sudo apt-get install nmap

run;
#nmap localhost

you should see;
PORT STATE SERVICE
22/tcp open ssh

add a new network interface to VBox client Settings for Ubuntu server, as Host Adapter, and also set VBox-File-Settings, Network, Host Adapters DHCP configurations. once you get the host and guest talking, you can use ssh.

test the system without ssh keys first from your client computer.
type ifconfig to get the IP address of the server, and type;

ssh muratagenc@192.168.56.101 (which is the IP of the Server's host adapter)

you should enter your password, and go in with no problem.

Last login: Tue Nov 10 16:31:07 2015
muratagenc@dpsubuntuazure:~$ 


copy your public key to the server;
since you can ssh to the server now, you can easily copy your identity (publi key) from the client machine;

***
[mgenc@localhost .ssh]$ ssh-copy-id muratagenc@192.168.56.101
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
muratagenc@192.168.56.101's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'muratagenc@192.168.56.101'"
and check to make sure that only the key(s) you wanted were added.
***

now, when you ssh to the server, it will ask you the enter the passphrase, but not password for the username.



** do not install any Java application (e.g. ElasticSearch) before you install Java since the application gets confused with the paths and it would be pretty time consuming to fix path problems after all. **


************************************************************************
Install OpenJDK on the Ubuntu server
https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get

install OpenJDK 7 JRE
sudo apt-get install openjdk-7-jre

install Jave OpenJDK 7
sudo apt-get install openjdk-7-jdk

to make sure or select the right Java environment;
sudo update-alternatives --config java

after this command, you should see that there is only one path to Java;
/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java

for JAVA_HOME variable, edit the file /etc/environment with vi, add;
JAVA_HOME="/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java"

muratagenc@dpsubuntuazure:~$ sudo vi /etc/environment 

and reload the environment file;
muratagenc@dpsubuntuazure:~$ source /etc/environment 

test the new settings;
muratagenc@dpsubuntuazure:~$ echo $JAVA_HOME
/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java

test Java by typing compiling the HelloTeam1.java code below;
public class HelloTeam1{
	public static void main(String args[]){
		System.out.println("Hello Team 1!");
	}
}
if you can compile and run the application, all is good
************************************************************************

install and test ElasticSearch

https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html

first;
sudo apt-get update

then get the public key of ElasticSearch
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
you should receive OK message back.

add repo information to your server;
echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list

now you can install ElasticSearch using apt-get
sudo apt-get install elasticsearch

take a backup and edit ElasticSearch configuration file for localhost
sudo cp /etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml.backup
sudo vi /etc/elasticsearch/elasticsearch.yml
unmark network.host and set it to localhost

make sure elasticsearch runs as a service and it is active;

muratagenc@dpsubuntuazure:~$ sudo service elasticsearch restart

muratagenc@dpsubuntuazure:/usr/share/elasticsearch/bin$ sudo service elasticsearch status
● elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2015-11-10 19:18:36 EST; 4s ago
     Docs: http://www.elastic.co
  Process: 2669 ExecStartPre=/usr/share/elasticsearch/bin/elasticsearch-systemd-pre-exec (code=exited, status=0/SUCCESS)
 Main PID: 2672 (java)
   CGroup: /system.slice/elasticsearch.service
           └─2672 /usr/bin/java -Xms256m -Xmx1g -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOcc...

Nov 10 19:18:36 dpsubuntuazure systemd[1]: Stopped Elasticsearch.
Nov 10 19:18:36 dpsubuntuazure systemd[1]: Starting Elasticsearch...
Nov 10 19:18:36 dpsubuntuazure systemd[1]: Started Elasticsearch.


now, make the service starts on boot;
sudo update-rc.d elasticsearch defaults 95 10


test elasticsearch with the following command.
muratagenc@dpsubuntuazure:/var/log/elasticsearch$ curl -X GET 'http://127.0.0.1:9200'
and the output:
{
  "name" : "Silver Samurai",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.0.0",
    "build_hash" : "de54438d6af8f9340d50c5c786151783ce7d6be5",
    "build_timestamp" : "2015-10-22T08:09:48Z",
    "build_snapshot" : false,
    "lucene_version" : "5.2.1"
  },
  "tagline" : "You Know, for Search"
}


** you may need to apt-get purge/remove the elasticsearch and reinstall. you should see the service active AND running ** 

you can also check the elasticsearch cluster status;

muratagenc@dpsubuntuazure:~$ curl -XGET 'http://localhost:9200/_cluster/health?pretty=true'
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 16,
  "active_shards" : 16,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 16,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}

status should read "yellow".

** you may need to start elasticsearch manually; #sudo service elasticsearch start | sudo service elasticsearch status



************************************************************************
install and test LogStash

https://www.elastic.co/guide/en/logstash/current/package-repositories.html

add the key for logstash
wget -qO - https://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -

add repo information
echo "deb http://packages.elasticsearch.org/logstash/2.0/debian stable main" | sudo tee -a /etc/apt/sources.list

run sudo apt-get update 
* do this before installing anything...

now you can install logstash with apt-get
sudo apt-get install logstash

also read this article:
https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-14-04

https://www.elastic.co/guide/en/logstash/current/configuration.html
create a simple configuration file named logstash_test.conf with the following content;
location: /opt/logstash/bin/

input { stdin { } }
output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}

if you are receiving the following error;
muratagenc@dpsubuntuazure:/opt/logstash/bin$ ./logstash -f logstash_test.conf 
Could not find any executable java binary. Please install java in your PATH or set JAVA_HOME.

do this;
muratagenc@dpsubuntuazure:/opt/logstash/bin$ which java
/usr/bin/java
muratagenc@dpsubuntuazure:/opt/logstash/bin$ export JAVACMD='/usr/bin/java'

then re-run;
muratagenc@dpsubuntuazure:/opt/logstash/bin$ ./logstash -f logstash_test.conf 
output:
Default settings used: Filter workers: 1
Logstash startup completed

meaning, it works. (Ctrl-C to stop)

let's try another example;
https://www.elastic.co/guide/en/logstash/current/config-examples.html

create another test configuration file named logstash_test2.conf under the same folder, with the content;
input { stdin { } }

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}


while the cursor blinking on the screen, copy and paste this line;
127.0.0.1 - - [11/Dec/2013:00:01:45 -0800] "GET /xampp/status.php HTTP/1.1" 200 3891 "http://cadenza/xampp/navi.php" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0"

the output;
muratagenc@dpsubuntuazure:/opt/logstash/bin$ ./logstash -f logstash_test2.conf 
Default settings used: Filter workers: 1
Logstash startup completed
127.0.0.1 - - [11/Dec/2013:00:01:45 -0800] "GET /xampp/status.php HTTP/1.1" 200 3891 "http://cadenza/xampp/navi.php" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0"
{
        "message" => "127.0.0.1 - - [11/Dec/2013:00:01:45 -0800] \"GET /xampp/status.php HTTP/1.1\" 200 3891 \"http://cadenza/xampp/navi.php\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0\"",
       "@version" => "1",
     "@timestamp" => "2013-12-11T08:01:45.000Z",
           "host" => "dpsubuntuazure",
       "clientip" => "127.0.0.1",
          "ident" => "-",
           "auth" => "-",
      "timestamp" => "11/Dec/2013:00:01:45 -0800",
           "verb" => "GET",
        "request" => "/xampp/status.php",
    "httpversion" => "1.1",
       "response" => "200",
          "bytes" => "3891",
       "referrer" => "\"http://cadenza/xampp/navi.php\"",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0\""
}


if you are receiving this output, everything is okay.

************************************************************************

Install and test Kibana
references;
https://www.elastic.co/guide/en/kibana/current/setup.html
http://www.unixmen.com/install-kibana-ubuntu-14-04/

we will install kibana as a service. it will be done via an init script. 

download kibana
wget https://download.elastic.co/kibana/kibana/kibana-4.2.0-linux-x64.tar.gz

extract the files;
tar -xvf kibana-4.2.0-linux-x64.tar.gz

edit kibana.yml configuration file under config folder, set server.host to localhost
server.host="localhost"

make a directory under opt, move kibana files under
sudo mkdir /opt/kibana
sudo cp -Rrvf kibana-4.2.0-linux-x64/* /opt/kibana/

download init.d script to run kibana as a service;
cd /etc/init.d/
wget https://gist.githubusercontent.com/thisismitch/8b15ac909aed214ad04a/raw/bce61d85643c2dcdfbc2728c55a41dab444dca20/kibana4

make kibana4 script executable
sudo chmod +x kibana4

make the script runnable at boot as a service
sudo update-rc.d kibana4 defaults 96 9

run the service and check status (without rebooting)
muratagenc@dpsubuntuazure:/etc/init.d$ sudo /etc/init.d/kibana4 restart
[ ok ] Restarting kibana4 (via systemctl): kibana4.service.
muratagenc@dpsubuntuazure:/etc/init.d$ sudo /etc/init.d/kibana4 status
● kibana4.service - LSB: Starts kibana4
   Loaded: loaded (/etc/init.d/kibana4)
   Active: active (running) since Tue 2015-11-10 20:34:41 EST; 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 3324 ExecStart=/etc/init.d/kibana4 start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/kibana4.service
           └─3330 /opt/kibana/bin/../node/bin/node /opt/kibana/bin/../src/cli

Nov 10 20:34:41 dpsubuntuazure systemd[1]: Stopped LSB: Starts kibana4.
Nov 10 20:34:41 dpsubuntuazure systemd[1]: Starting LSB: Starts kibana4...
Nov 10 20:34:41 dpsubuntuazure kibana4[3324]: * Starting Kibana4
Nov 10 20:34:41 dpsubuntuazure kibana4[3324]: ...done.
Nov 10 20:34:41 dpsubuntuazure systemd[1]: Started LSB: Starts kibana4.

test kibana 
muratagenc@dpsubuntuazure:/opt/kibana/config$ curl -X GET 'http://127.0.0.1:5601'

you should receive the following output;

<script>var hashRoute = '/app/kibana';
var defaultRoute = '/app/kibana';

var hash = window.location.hash;
if (hash.length) {
  window.location = hashRoute + hash;
} else {
  window.location = defaultRoute;
}</script>


make kibana web service accesible on the network.
to do this, we need to configure Ubuntu firewall.
sudo ufw allow 5601/tcp

and test;
muratagenc@dpsubuntuazure:/opt/kibana/config$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
5601/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
5601/tcp (v6)              ALLOW IN    Anywhere (v6)

test from your client browser, if you don't see Kibana welcome page, then set server.host to the public IP of the server.
in my case;
server.host="192.168.56.101"

TODO: create a second Ubuntu server, cluster name for both servers is DPS804, node names are dpsnode1, dpsnode2 (elasticsearch nodes).

a useful reference: http://www.slideshare.net/prajalkulkarni/attack-monitoring-using-elasticsearch-logstash-and-kibana

**********************************************************************************************
OPTIONAL:
if you want your VBox machine Kibana service available on the Internet;
1. enable port forwarding on your router, forward port 5601 to your host machine
2. enable port forwarding on your host machine, forward port 5601 to the guest machine

on the host
sudo sysctl net.ipv4.ip_forward=1
sudo iptables -t nat -A PREROUTING -p tcp -d 192.168.0.2 --dport 5601 -j DNAT --to-destination 192.168.56.101:5601
sudo iptables -t nat -A POSTROUTING -j MASQUERADE

192.168.0.2: host machine
192.168.56.101: VBox guest machine (Host adapter IP)
**************************************************************************************

this is a draft document...
muratagenc@gmail.com
