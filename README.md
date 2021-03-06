# aws-deployments

**UPDATE** *(08/26/2016)*: Now that we have cloud-init on BIG-IP starting v12.0.0, this project has been de-emphasized in favor of CloudFormation approach. Please see [https://github.com/F5Networks/f5-aws-cloudformation](https://github.com/F5Networks/f5-aws-cloudformation) instead.
If interested in Ansible integration, please see [https://github.com/f5networks/f5-ansible](https://github.com/f5networks/f5-ansible)

## Deployment examples for F5's BIG-IP platform in AWS

#### About

f5aws is a tool for deploying F5 BIG-IP in various possible architectures within an AWS EC2 Virtual Private Cloud.

As of now, these deployment models are:<br>
-single-standalone (single big-ip and application server in one availability zone) <br>
-single-cluster (cluster of big-ips and application server in one availability zone) <br>
-standalone-per-zone (big-ips in multiple availability zones, fronted by a gtm in each AZ, application hosts in each AZ, and a host in the external subnet for traffic generation)<br>
-cluster-per-zone (big-ip clusters in multiple availability zones, fronted by gtm in each AZ, application hosts in each AZ, and a host in the external subnet for traffic generation)<br>

For more information, see the PDF in /docs.

#### Support

This code is provided as is and should be used as a reference only.  It is not provided as a production-ready tool and F5 support will not field requests for this work.  


### Install/Setup:
1) Install Virtual Box and Vagrant 
Install virtual box (tested using 4.3.26)<br>
https://www.virtualbox.org/wiki/Downloads

Install vagrant (tested using 1.7.2)<br>
http://docs.vagrantup.com/v2/installation/

2) Clone this code to your desktop:<br>
```git clone https://github.com/F5Networks/aws-deployments.git```

3) Setup the virtualbox host with vagrant: <br>
```cd aws-deployments/vagrant```<br>
```vagrant up```

4) When prompted by the vagrant, choose network interface attached to the internet.

5) Once the virtual box has started, login to the machine:<br>
```vagrant ssh```

6) Edit/Copy (manually with VIM/Nano or SCP) your credentials and environment variables over to your vagrant guest:

- ~/.ssh/<your_aws_ssh_key>
- ~/.aws/credentials
- ~/.f5aws

An example of copying your your AWS SSH private key over to vagrant guest:


user1@desktop:demo $scp -P 2222 ~/.ssh/AWS-SSH-KEY.pem vagrant@127.0.0.1:~/.ssh/AWS-SSH-KEY.pem
Warning: Permanently added '[127.0.0.1]:2222' (RSA) to the list of known hosts.
vagrant@127.0.0.1's password:
AWS-SSH-KEY.pem            100% 1696     1.7KB/s   00:00

IMPORTANT: Make sure the ssh key has correct permissions after creation or upload [chmod 400 your-key].

7)  Depending on the deployment, you will need to go to the AWS Console and accept the EULAs for each of the AMIs in use. 

- F5 BIG-IP Virtual Edition 25 Mbps - Better: https://aws.amazon.com/marketplace/pp/B00JL3Q2VI <br>

- Client: Ubuntu 14.04 LTS: https://aws.amazon.com/marketplace/pp/B00JV9JBDS<br>


### Usage:

1) To create a new environment, use the 'init' command.
This will initialize the set of ansible variables necessary for deployment (known as an 'inventory'. After execution of this playbook, inspect '~/vars/f5aws/env/<b>env_name</b>'.

```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "single-standalone", "region": "us-east-1", "zone": "us-east-1b"}'```

```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "single-cluster", "region": "us-east-1", "zone": "us-east-1b" }'```


You can also try out a more complex deployment ( complete with GTMs (up to 2 - one in each AZ) and a jmeter client to generate traffic) that can leverage multiple AZs. You must choose the availability zones in which you want to deploy. 
 
 ```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "standalone-per-zone", "region": "us-east-1", "zones": ["us-east-1b"]}'```

 ```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "standalone-per-zone", "region": "us-east-1", "zones": ["us-east-1b","us-east-1c"]}'```

 ```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "standalone-per-zone", "region": "us-east-1", "zones": ["us-east-1b","us-east-1c","us-east-1d"]}'```

or using clusters:

 ```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "cluster-per-zone", "region": "us-east-1", "zones": ["us-east-1b","us-east-1c"]}' ```

You can add splunk to any deployment model by using the "deploy_analytics" flag:
 ```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "standalone-per-zone", "deploy_analytics": "true", "region": "us-east-1", "zones": ["us-east-1b"] }'```

Finally, to deploy an ASM profile to protect your application (i.e. web application FW), set the deployment_type to "lb_and_waf".  This deploys a generic linux, mysql, pache, FW policy for basic injections and vulnerabilities. 

 ```./bin/f5aws init <your env> --extra-vars '{"deployment_model": "standalone-per-zone", "deployment_type": "lb_and_waf", "region": "us-east-1", "zones": ["us-east-1b"] }'```



2) Deploy and manage the environment you instantiated in step 1.  This creates all the resources associated with environment, including AWS EC2 hosts, a VPC, configuration objects on BIG-IP and GTM, and docker containers.  

```./bin/f5aws deploy <your env>```

3) When you are done, just teardown the environment:

```./bin/f5aws teardown <your env>```

4) At any time, you can list all the deployments which are under management:

```./bin/f5aws list```

5) List additional details about an environment via the info command, which has three subcommands:

- display login information for hosts deployed in ec2<br>
```./bin/f5aws info login <your env>```

- print the ansible inventory (dynamic inventory groups like bigips, apphosts, gtms, etc are not printed)<br>
```./bin/f5aws info inventory <your env>```

- print the status of deployed infrastructure and output from cloudformation stacks<br>
```./bin/f5aws info resources <your env>```

Please be aware of the following conditions:

- The more complex deployment models require additional account resources (EIPs + CFTs) so you may need to increase your limits ahead of time by working with AWS support. For more information, see the PDF in /docs.

- Due to AMI availability, currently working regions are:
 - us-east-1 , us-west-1, us-west-2, eu-west-1, ap-southeast-2, ap-northeast-1

See PDF in /docs for more details.