# openc2-python-slpf-iptables
This is an Apache and Python3 implementation of the OpenC2 SLPF for an Amazon Linux AMI server.
For more information on OpenC2 see https://www.oasis-open.org/committees/openc2

## Getting Started
You can test my already running implementation in AWS EC2 by making the following request
```
curl -k -vvv https://ec2-34-220-41-4.us-west-2.compute.amazonaws.com/openc2 -H 'X-Correlation-ID: shq5x2dmgayf' -H 'Content-Type: application/openc2+json;version=1.0' -d '{"id":"0bc6dc48-0eaa-40a8-802f-0acb73e3fa88","action": "deny","target": {"ip_connection": {"src_addr": "2.2.2.6"}},"actuator": {"slpf": {"actuator_id": "1"}}}'
```
You should get the following response
```
HTTP/1.1 200 OK
< Date: Thu, 13 Sep 2018 17:51:11 GMT
< Server: Apache/2.4.34 (Amazon) OpenSSL/1.0.2k-fips
< X-Correlation-ID: shq5x2dmgayf
< Transfer-Encoding: chunked
< Content-Type: application/openc2+json;version=1.0
< 
{
    "status": 200,
    "status_text": "ok"
}

```
Note that this requests blocks 2.2.2.6, you could test with your own IP address.
Every 30 minutes a cron job runs and it will clear out iptables so that I dont have a bunch of odd rules installed
### Prerequisites

An EC-2 host running Amazon Linux AMI along with the public IP address where it can be reached.

### Installing
Configure EC2 Security group
```
Type,Protocol,Port Range, Source, Description
HTTP,TCP,80,0.0.0.0/0,web
SSH,TCP,22,0.0.0.0/0,remote access
HTTPS,TCP,443,0.0.0.0/0,ssl
```

Log into the EC2 host via SSH and update packages
```
ssh -i awskey.key ec2-user@ec2-34-220-x-y.us-west-2.compute.amazonaws.com
sudo yum update
```

Install and Start Apache2
```
sudo yum install httpd24.x86_64
sudo chkconfig httpd on
sudo service httpd start
```

Install Python3

```
sudo yum install python36.x86_64
sudo ln -s /usr/bin/python3 /usr/bin/python
```


Edit or Replace Apache2 config  [httpd.conf](httpd.conf) file, esp. lines 120-173 
```
sudo vi /etc/httpd/conf/httpd.conf
sudo service httpd restart
```

Optionally configure TLS/SSL and edit ssl.conf
```
sudo sudo yum install mod24_ssl.x86_64
sudo openssl req -new -x509 -nodes -out server.crt -keyout server.key
sudo vi /etc/httpd/conf.d/ssl.conf 
```

Edit or Replace Apache2 config  [sudoers](sudoers) file, esp. line 92
```
sudo visudo
```

If SELinux is enforcing, take care of that
```
exercise left for the reader
```

Place [openc2](openc2) file in /var/www/html or other documentroot and set appropriate permissions for apache user
```
cd /var/www/html
sudo vi openc2
```
## Running the tests

Explain how to run the automated tests for this system

## Comments

One should perform input/output sanitization, esp. when working with untrusted input

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
* All partipants of the OASIS OpenC2 Actuator Subcommittee
