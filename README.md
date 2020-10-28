# openc2-python-slpf-iptables
This is a partial Apache and Python3 implementation of the OpenC2 SLPF Committee Specification version 1.1 for an Amazon Web Services (AWS) Linux 2 AMI or Google Cloud Platform (GCP) Ubuntu web server. Successful commands will instantiate stateful or stateless rules via iptables or ip6tables.
For more information on OpenC2 see openc2.org

## Overview
[ client aka producer]  ----HTTPS----->  [server aka consumer/actuator]

## Getting Started

One can test my running compute engine in GCP using this request:
```
curl -k -vvv https://www.iptablestest.tk./openc2 -H 'X-Request-ID: 0bc6dc48-0eaa-42a8-802f-0acbb3e3fa00' -H 'Content-Type: application/openc2-cmd+json;version=1.0' -d '{"action": "query","target": {"features":["pairs"]}}'
```
Or, you can test my already running implementation in AWS EC2 by making the following request
```
curl -k -vvv https://ec2-34-207-226-118.compute-1.amazonaws.com/openc2 -H 'X-Request-ID: 0bc6dc48-0eaa-42a8-802f-0acbb3e3fa00' -H 'Content-Type: application/openc2-cmd+json;version=1.0' -d '{"action": "query","target": {"features":["pairs"]}}'
```
You should get the following response
```
HTTP/1.1 200 OK
Date: Sat, 25 Jan 2020 16:48:55 GMT
Server: Apache/2.4.41 () OpenSSL/1.0.2k-fips
Upgrade: h2,h2c
Connection: Upgrade
X-Request-ID: 0bc6dc48-0eaa-42a8-802f-0acbb3e3fa00
Cache-Control: no-cache
Transfer-Encoding: chunked
Content-Type: application/openc2-rsp+json;version=1.0
 
{
    "results": {
        "pairs": {
            "allow": [
                "ipv4_net"
            ],
            "deny": [
                "ipv4_net"
            ],
            "query": [
                "features"
            ]
        }
    },
    "status": 200
}

```
For allow/deny actions use your own IP address.
Every half hour a cron job runs and it will clear out iptables so that I dont end up with a bunch of odd rules installed.

### Prerequisites

An EC2 host running Amazon Linux 2 AMI along with the public IP address where it can be reached.

### Installing
Configure the following EC2 Security Group
```
Type,Protocol,Port Range, Source, Description
SSH,TCP,22,0.0.0.0/0,remote access
HTTPS,TCP,443,0.0.0.0/0,ssl
```

Log into the EC2 host via SSH and update packages
```
ssh -i awskey.key ec2-user@ec2-34-220-x-y.us-west-2.compute.amazonaws.com
sudo yum update
#sudo apt-get update for GCP
```

Install and Start Apache2
```
sudo yum install httpd.x86_64
sudo systemctl enable httpd.service
sudo systemctl start httpd.service
#sudo apt-get install apache2 for GCP
```

Install Python3

```
sudo yum install python3.x86_64
#already present on GCP instance
```


Edit or Replace Apache2 config  [httpd.conf](httpd.conf) file, including lines 117-170 in repo
```
sudo vi /etc/httpd/conf/httpd.conf
#sudo vi /etc/apache2/apache2.conf for GCP
sudo systemctl restart httpd.service
#sudo a2enmod rewrite for GCP to enable ReWrite Engine
```

Optionally configure TLS/SSL and edit ssl.conf
```
sudo yum install mod_ssl.x86_64
sudo openssl req -new -x509 -nodes -out /etc/pki/tls/certs/localhost.crt -keyout /etc/pki/tls/private/localhost.key
```

Edit or Replace Apache2 config  [sudoers](sudoers) file, esp. line 92
```
sudo visudo
```

If SELinux is enforcing, take care of that
```
exercise left for the reader
```

Place [openc2](openc2) file in /var/www/html (or other Document Root) and set appropriate permissions for apache user
```
cd /var/www/html
sudo vi openc2
//paste in file contents from repo
sudo chgrp apache openc2
sudo chmod 750
sudo systemctl restart httpd.service
```
## Running the tests

First, make sure the webserver is running and listening on 80 and 443 if that is what you configured
```
sudo ps -ef | grep -i http
```
```
root     16877     1  0 15:31 ?        00:00:00 /usr/sbin/httpd
apache   16879 16877  0 15:31 ?        00:00:01 /usr/sbin/httpd
apache   16880 16877  0 15:31 ?        00:00:01 /usr/sbin/httpd
apache   16881 16877  0 15:31 ?        00:00:01 /usr/sbin/httpd
apache   16882 16877  0 15:31 ?        00:00:01 /usr/sbin/httpd
apache   16883 16877  0 15:31 ?        00:00:01 /usr/sbin/httpd
apache   17025 16877  0 16:33 ?        00:00:00 /usr/sbin/httpd
ec2-user 17247 17124  0 18:00 pts/0    00:00:00 grep --color=auto -i http
```
and
```
sudo netstat -tulpn | grep -i http
```
```
tcp        0      0 :::80                       :::*                        LISTEN      16877/httpd         
tcp        0      0 :::443                      :::*                        LISTEN      16877/httpd 
```

Test that you can connect remotely
```
curl -vvv http://[insert IP or name here]
```

Check to see what iptables currently has configured. In the response below it is allowing all packets.
```
sudo iptables --list
```
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

Test that python3 works as expected,
```
[ec2-user@ip-172-31-45-108 ~]$ python3 -V
Python 3.7.9
[ec2-user@ip-172-31-45-108 ~]$ python3 /var/www/html/openc2
```
This should result in
```
HTTP/1.1 500
Content-Type: application/openc2+json;version=1.0

[
    "response",
    {
        "status": 500,
        "status_text": "server error"
    }
]

Traceback (most recent call last):
  File "openc2", line 109, in <module>
    main()
  File "openc2", line 38, in main
    request_handler()
  File "openc2", line 51, in request_handler
    request=sys.stdin.read()
  File "openc2", line 44, in handler
    raise IOError("Couldn't open device!")
OSError: Couldn't open device!
```

## Comments
Presently, this code only allows/blocks the source IP given a properly formatted request via POST and responds to some query commands.
One should perform input/output sanitization, esp. when working with untrusted input
I do not recommend using this in production or on a server exposed to the Internet

## Built With

* [EC2](http://aws.amazon.com/) - The server used
* [Python3](https://www.python.org/) - Programming language used
* [Apache2](https://www.apache.org) - The web server used


## Authors
Alex Everett

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgments

* Joe Brule
* All partipants of the OASIS OpenC2 Actuator Profile Subcommittee
