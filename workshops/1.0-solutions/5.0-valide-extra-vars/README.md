[Previous Exercise](../4.0-valide-playload-eda-zabbix) - [Next Exercise](../6.0-ansible-builder)
#5.0 - Validate in AAP how Extra Variables are arriving and how to configure the Job Template to use Extra Variables. 


Let's create a project called: rh1-ansible-eda-vars-zabbix:

Let's access it in GitLab:

Let's click on New project
![alt text](files/images/image-1.png)

Create blank project

![alt text](files/images/image-2.png)

Let's name it: rh1-ansible-eda-vars-zabbix

Check box:  "Initialize repository with a README"

![alt text](files/images/image-3.png)

Let's click Clone me and copy clone with SSH to use on the host.

![alt text](files/images/image-4.png)


Let's clone the repository and create the files:

   ```bash
   git clone git@gitlab.lnx.example.local:root/rh1-ansible-eda-vars-zabbix.git
   ``` 

   ```bash
   mkdir rulebooks/
   touch rulebooks/webhook-zabbix.yml
   vi rulebooks/webhook-zabbix.yml
   ``` 


   ```bash
   ---
   - name: Listen for events on a webhook
     hosts: all
     sources:
       - ansible.eda.webhook:
           host: 0.0.0.0
           port: 5000
     rules:
       - name: Zabbix Apache 
         condition: event.payload.event_name is regex("Apache.*Service is down", ignorecase=true)
         action:
           run_job_template:
             name: projeto-http
             organization: Default
             job_args:
               extra_vars:
                 hosts_update: "{{ event.payload.host_host }}"
   ``` 

In this project we will create the project-http folder

   ```bash
   mkdir projeto-http/
   touch projeto-http/playbook.yml
   vi projeto-http/playbook.yml
   ``` 

   ```bash
   ---
   - name: Create env
     hosts: localhost
     tasks:
       - name: Add hosts
         ansible.builtin.add_host:
           name: "{{ hosts_update }}"
           groups: rh1
         with_items: '{{ hosts_update.split() }}'
   
   - name: RH1
     hosts: rh1
     become: true
     remote_user: root
     tasks:
       - name: install httpd
         ansible.builtin.yum:
           name: httpd
           state: latest
       - name: start
         ansible.builtin.service:
           name: httpd
           state: started
           enabled: yes
       - name: Create index.html
         ansible.builtin.copy:
           content: "{{ ansible_facts.fqdn }}"
           dest: /var/www/html/index.html
   ``` 

Let's commit and push the files:

   ```bash
   git add . ; git commit -m add ; git push
   ``` 

In Ansible Automation Platform we will create the rh1-ansible-eda-vars-zabbix project in Automation Decisions:


![alt text](files/images/image-5.png)

In Ansible Automation Platform we will create the rh1-ansible-eda-vars-zabbix project in Automation Execution:

Let's click on Create Project
![alt text](files/images/image-6.png)

Let's add the Name: rh1-ansible-eda-vars-zabbix
Make sure the check box is checked: Clean, Update revision on launch and Delete

![alt text](files/images/image-8.png)

Let's make sure the project Success syncs

![alt text](files/images/image-9.png)

Let's create the rh1-ansible-eda-vars-zabbix rulebook.
Click on Create rulebook activation:

![alt text](files/images/image-10.png)

Name: rh1-ansible-eda-vars-zabbix
Organization: Default
Project: rh1-ansible-eda-vars-zabbix
Rulebook: webhook-zabbix.yml
Credential: AAP
Decision Enviroment: Default Decision Enviroment
Log Level: Debug

Now click on Create rulebook activation

![alt text](files/images/image-11.png)

Validate if the rulebook is running:

![alt text](files/images/image-12.png)

Let's create the job_template:

Let's go to the Automation Execution section in Templates > Create Template > Create job Template:

![alt text](files/images/image-13.png)

Let's create the project-http inventory containing only the host localhost:

![alt text](files/images/image-17.png)


Let's click on Create Host:


![alt text](files/images/image-18.png)

Now let's add the host localhost:

![alt text](files/images/image-19.png)

Remember to put the host in disabled mode inside the inventory:

![alt text](files/images/image-22.png)

Em Name: projecto-http
Project: rh1-ansible-eda-vars-zabbix
Playbbok: projeto-http/playbook.yml

![alt text](files/images/image-14.png)

After creating the job_template, let's check the Automation Decisions logs:
Click on Rulebook Activations > rh1-ansible-eda-vars-zabbix > History

Let's stop the server's httpd:

   ```bash
   systemctl stop httpd
   ``` 

Now Zabbix will send the notification to Ansible Event Driven:


![alt text](files/images/image-21.png)

Now we can see the logs in Ansible Event Driven

![alt text](files/images/image-15.png)

And Check if the job ran successfully:

![alt text](files/images/image-16.png)



#EXTRA: 
Validate the playback using tcpdump + wireshark:

Let's use tcpdump to get the packets from the Event Driven host:


   ```bash
   tcpdump -i any port 5000 -w eda.cap
   ``` 
Let's create the FollowStream.sh file and add the line below:


   ```bash
   vi FollowStream.sh
   #!/bin/bash
   echo "Connection Test for TCP Port 5000"
   filename=$1
   if [[ -z "$filename" ]]; then
     echo "Usage: ./FollowStream.sh [FILE.PCAP]"
     exit
   fi
   stream=$(tshark -r $filename -Y tcp.port==5000 -T fields -e tcp.stream | sort -n | uniq | head -1)
   tshark -q -r $filename -z follow,tcp,ascii,$stream

   ``` 
Add execute permission:
   ```bash
   chmod -x FollowStream.sh
   ``` 
![alt text](files/images/image-23.png)



[Previous Exercise](../4.0-valide-playload-eda-zabbix) - [Next Exercise](../6.0-ansible-builder)