# openc2-python-slpf-iptables
A python/apache implementation of the OpenC2 SLPF that can be ran on an Amazon Linux host

## Getting Started

### Prerequisites

An EC-2 host running Amazon-Linux along with the public address where it can be reached.

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

End with an example of getting some data out of the system or using it for a little demo

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

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Joe Brule
* All partipants of the Oasis OpenC2 Actuator Subcommittee
