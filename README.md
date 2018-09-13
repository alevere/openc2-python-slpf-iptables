# openc2-python-slpf-iptables
This is an Apache and Python3 implementation of the OpenC2 SLPF for an Amazon Linux AMI server

## Getting Started

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
myusername$ ssh -i awskey.key ec2-user@ec2-34-220-x-y.us-west-2.compute.amazonaws.com
[ec2-user@ip-172-31-x-y html]$ sudo yum update
```

Install and Start Apache2
```
myusername$ sudo yum install httpd24.x86_64
myusername$ sudo chkconfig httpd on
myusername$ sudo service httpd start
```

Install Python3

```
myusername$ sudo yum install python36.x86_64
myusername$ sudo ln -s /usr/bin/python3 /usr/bin/python
```

Edit or Replace Apache2 config 
See httpd.conf file, esp. lines 120-170
```
myusername$ sudo vi /etc/httpd/conf/httpd.conf
```

Optionally configure TLS/SSL
```
myusername$ sudo sudo yum install mod24_ssl.x86_64
myusername$ sudo openssl req -new -x509 -nodes -out server.crt -keyout server.key
```

Edit sudoers file
```
myusername$ sudo visudo
```

See httpd.conf file, esp. lines 120-170
```
myusername$ sudo vi /etc/httpd/conf/httpd.conf
```

## Running the tests

Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

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
* All partipants of the Oasis OpenC2 Actuator Subcommittee
