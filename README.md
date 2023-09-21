Ansible_Proj
What is Ansible.
What is ansible: Ansible is powerful automation tool best suited for configuring and managing existin infrastructure, like installing packages on server, modifying configuration files, adding and deleting users to a server, starting and stopping services on packages like httpd, mysql etc. It is a configuration management tool.
Where are the configuration files of ansible
When ansible is downloaded to our servers, there is a default configuration file at /etc/ansible/, this is where where we have the ansible.cfg, hosts file, inventory file. But we can create our configuration file in the present working directory by just running nano ansible.cfg. Inside the created the config file we can make all the configurations we want for our use case, like the inventory file, the path to the .pem key like this:
[defaults] inventory = inventory private_key_file = <path to .pem_key> host_key_checking = false (optional) deprecation_warning - false (optional) <and anythings else of importance to our use_case)
inventory
[group_name] private_ip address
when we specify a particular group name or ip address or group_name when running our playbook as opposed to "all" it means want the playbook to run on specific nodes.
How does Ansible talk to the target machines.
Ansible connects to target machines using passwordless authentication (using ssh protocol), it can use the .pem method or by creating a keypair in the ansible controller and sending the public key to the target machines.
1: log out of your VM into the folder where your .pem file is stored
2: In my case, it is my download folder. From there run : eval 'ssh-agent -s'
6 : From that same folder run : ssh-add testkey.pem ( to add the pem key to our ssh agent)
7: Then connect back to the vm using the command : ssh -A ubuntu@public_ip
8: Then add the .pem key to the server with the command : ssh-add -l
9: eval 'ssh-agent -s' ssh-add testkey.pem ssh -A ubuntu@public_ip ssh-add -l ssh <name_of_user>@private_ip ( to test if the control_node can connect to the target machines)
10: you can then connect to other servers using the same .pem key usin the command:
ssh <name_of_user>@private_ip of the target server. Note: Name of user is only necessary when the ansible controller and the target machine have diffreent users.
Testing connectivity
11: To test if the ansible controller can connect with target nodes run the command: ansible -m <name_of_server_or_group_name> ping.
##Ad-Hoc Command These are single ansible commands use to perform ansible tasks(automation) at runtime. Ad-hoc commands are usually single commands and use certain arguments. to run. an example of an ad-hoc command to copy a file from the ansible controller to one of the client nodes:
ansible <group_name, all or single_node> -m copy -a 'src=/home/ubuntu/Ansible_Proj/cal.txt dest=/tmp'
group_name = The name of the specific agent_node on which ansible will run the task
-m = the is the flag that will be used to pass the module to be used (the copy module)
-a = this is the argument used to pass the in the required argument
src= this is the source-where the file to be copied resides in, give the full path. If the file is in the same folder
dest= this is where we are copying the file to, in this case the location is /tmp.
2: Ansible ad-hoc command to install apache on a client node.
ansible <group_name, all or single_node> -m yum -a 'name=httpd state=present' -become -u <the user, ubuntu or ec2-user>
group_name = The name of the specific agent_node on which ansible will run the task
-m = the is the flag that will be used to pass the module to be used (the yum module)
-a = this is the argument used to pass the in the required argument
name=httpd: This is the name of the package, in this case it is httpd.
state=present: this is the state the package has to be in, in a ready state.
-become= this flag tells ansible that the task has to be run as sudo user
-u: this flag allows us to pass the user argument.
3: Another ansible ad-hoc command
ansible <group_name, all or single_node> -m shell -a 'ls -la' -u <the user, ubuntu or ec2-user>
group_name = The name of the specific agent_node on which ansible will run the task
-m = the is the flag that will be used to pass the module to be used (the shell module)
-a = this is the argument used to pass the in the required argument
'ls -la' = This is the command ansible will run in the target node.
-u = -u: this flag allows us to pass the user argument.
In this case it is ubuntu.
Ansible uses modules to perform different tasks, there are specific modules with specific names for different tasks and they can all be found in the ansible documentation.
Ansible playbooks: these are a set tasks arranged using the yaml/yml format in a file used to execute multiple tasks simultaneously.
1:The components of a playbook include Variables, tasks, files, module, template, handlers. 2:Variables provide the much needed the flexibility in the playbooks, templates and inventory file. 3: Variables are of 2 types, system defined variables (built-in variables) like "hostvars, groups, group_names". These variables are not defined by the user, they are pre-defined by ansible. 4: The other type of variable are the user=defined variable of the custom variable like "myvar, var, tune_01 etc" The user define these kind of variables according to his requirement. 5: Variable names must always start with letters, it can have numbers,letters, numbers and underscore but no spaces or hyphen. 6: Ansible variables are defined by using the vars keyword in the playbook file.



Example: 

 hosts: all 
 vars:
   salutations: Hello Everyone!
 tasks:
  - name: Ansible variable basic usage
    debug:
      msg: "{{ salutations }}"




This example shows a playbook with the "vars" keyword used and the correct indentation (yml format).
In this case, the variable is salutations, which is under the vars keyword indicating that it is a variable, and the salutations keyword has a value (Hello Everyone), it is put inside double curly bracket to show that it it a variable with a value, and when used the value will be displayed.

  ### DYNAMIC INVENTORY

The plugin method is the recommended method by Ansible as opposed to the python script.

1:  The first thing to do is to install

pip. pip is Python’s package manager used to install all Python packages  

In this case we need to install boto3 and botocore with pip.


1: Install pip with this command in ubuntu: sudo apt install python3-pip

2: Install boto3 and botocore:
Now, you can use pip to install boto3 and botocore. Run the following command:                          pip3 install boto3 botocore  


3:  You can verify that boto3 and botocore have been installed successfully by running the following command, which will display the installed versions:     pip3 show boto3 botocore

4: Since we will be connecting to aws from our command line to fetch the ip addresses we need to configure awscli in our instance, to do we use the following commands: 

5: sudo apt update

6:  sudo apt install awscli

7:  pip install --upgrade awscli botocore

8:  run : aws —version to ensure awscli has been installed 

9 :   Then configure awscli with the command: aws configure then input your access key and secret access keys

10:  we will then configure the aws plugin in ansible to enable ansible talk to aws.
In the ansible.cfg (I prefer to create my own custom ansible.cfg file in my present working directory)
add this code to the existing configuration there:      enable_plugins = aws_ec2 

11:  Still in our present working directory we will create the plugin configuration file with the extension  aws_ec2.yaml. so if we chose inventory as the plugin configuration file the full name will be inventory_aws_ec2.yaml.

the configuration is : 

—
plugin: aws_ec2
region:
   - eu-west-2

This tells ansible what region to fetch the ip addresses from, aws_ec2 is the plugin name which is the file extension where the plugin configuration is.

note: we can have more than one region configured in the plugin config file.

To test the configuration without running any playbook, we run this command: ansible-inventory -i inventory_aws_ec2.yaml --list

This command will list/show us all the attributes in this specified region 

To get only the the ip addresses without any other detail run: 

ansible-inventory -i inventory_aws_ec2.yaml —graph 

When running the playbook, we run ansible-playbook -i  <dynamic_inventory_file> playbook

The reason we pass the -i argument is that we are passing a different inventory file from what is specified in the default inventory in the ansible.cfg, the inventory file passed in the command line using the -i argument overrides the whatever is specified in the ansible.cfg file.



12: Using Filters to target specific instances: We can target specific servers using the various attributes, one of which is the tags. The configuration is as follows: 

—
plugin: aws_ec2
region:
   - eu-west-2
filters: 
   tag:Name:  prod* ( in this case, the playbook will be executed in all the servers tagged prod only)

servers can also be selected using other parameters like " instance-state-name: running ", in this case all the servers that are in the running state will be selected, there are other parameters which we will find in the dynamic inventory section of the ansible documentation.

We can go to ansible documentation to get different parameters for filtering.

