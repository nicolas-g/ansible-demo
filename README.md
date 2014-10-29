 LAMP Stack + HAProxy.
--------------------------------------------------------------------------------

`ec2quick_launch.yml` to launch all resourses you need in AWS (ec2 instances/Security Groups, Route 53)
`site.yml` to configure and deploy the LAMP stack.
`rolling_update.yml` for rolling upgrades on the Web Tier

Tested with
-  Ansible 1.7.1
-  CentOS release 6.5



#### Initial Launch: run `ansible-playbook ec2quick_launch.yml` 
* this will provision all the required AWS resources that you will use to deploy your application in the next step. The `ec2quick_launch.yml` playbook will configure Route53 records, Security Groups and the bellow 4 ec2 instances defined in the `hosts` inventory file grouped by their purpose:

		[webservers]
		web1.demo.cogmatch.net
		web2.demo.cogmatch.net
		
		[dbservers]
		db1.demo.cogmatch.net
		
		[lbservers]
		lb1.demo.cogmatch.net

#### Configure the servers : run `ansible-playbook -i hosts site.yml --private-key=~/.ssh/rs-dev -u root`

This example is an extension of the simple LAMP deployment. Here we'll install
and configure a web server with an HAProxy load balancer in front, and deploy
an application to the web servers. This set of playbooks also have the
capability to dynamically add and remove web server nodes from the deployment.
It also includes examples to do a rolling update of a stack without affecting
the service.

* when you have configured the servers open on your browser the address http://lb1.demo.cogmatch.net:8888/ to see the page checkout from git 
* run site.yml using the `release` tag
* commit a change in demo-app.git repository and run again `site.yml` using the `release` tag
* visit http://lb1.demo.cogmatch.net:8005/ for HA_Proxy statistics
* remove 1 web node from the `hosts` file
* run again the site.yml
* open http://lb1.demo.cogmatch.net:8005/ and confirm the web node was removed 

### Removing and Adding a Node

Removal and addition of nodes to the cluster is as simple as editing the
hosts inventory and re-running:

        ansible-playbook -i hosts site.yml

### Rolling Update

Rolling updates are the preferred way to update the web server software or
deployed application, since the load balancer can be dynamically configured
to take the hosts to be updated out of the pool. This will keep the service
running on other servers so that the users are not interrupted.

In this example the hosts are updated in serial fashion, which means that
only one server will be updated at one time. If you have a lot of web server
hosts, this behaviour can be changed by setting the 'serial' keyword in
webservers.yml file.

Once the code has been updated in the source repository for your application
which can be defined in the group_vars/all file, execute the following
command:

	 ansible-playbook -i hosts rolling_update.yml

You can optionally pass: -e webapp_version=xxx to the rolling_update
playbook to specify a specific version of the example webapp to deploy.
