1. run vagrant up, to bring up all the servers

2. ssh into the ansible-control (vagrant ssh ansible-control) - this is where all the rest of the machines will be contriled from

3. copy the host_file from vagrant to /etc/hosts ($ sudo cp /vagrant/host_file /etc/hosts)

3.1. You only do this if there is a name resolution on the ip addresses, if not there will be no need to to copy to etc/hosts

eg. 
192.168.56.100 ansible-control
192.168.56.101 db01
192.168.56.102 web01
192.168.56.103 web02
192.168.56.104 loadbalancer

4. cat etc/hosts to check its content

5. now you can ping all the server to see if there are working.

6. install ansible
      $ sudo apt update
      $ sudo apt install ansible -y


7. To run commands against out servers, we have to create an inventory file whcih will contain all the servers we want to target
      eg. - inventory.ini or anyName 

[control] #sets group for the fargeted servers. only targets servers under it
ansible-control

[proxy] 
loadbalancer

[webservers] 
web01
web02

[database] 
db01

[webstack:children] #group of groups
proxy 
webservers 
database


8. To run any activity against the inventory file, cd to were the file is located before you run any commands.

9. run commands in the webstack server 
                  ansible webstack -i hosts -m command -a hostname (this will fail cuz we have not set ssh key to log into the servers to do so ...., run ssh-keygen to generate ssh keys)


10. ssh-keygen

11. now that we have generated the key, we have to copy it to local host
            use command 
                  ssh-copy-id localhost

                  to check if everything worked
                              ssh localhost

      Now copy to all the servers
                  ssh-copy-id web01 && ssh-copy-id web02 &&  ssh-copy-id web01 && ssh-copy-id web02 && ssh-copy-id db01 && ssh-copy-id loadbalancer && ssh-copy-id ansible-control
12. Try to see if it worked using some ad-hoc commands
            ansible webstack -i hosts -m command -a hostname
            ansible webstack -i hosts -m command -a date
            ansible webstack -i hosts -m command -a cal


13. To get the full functionality of Ansible, we have to install some packages to all out servers
            ansible all -i hosts -m command -a 'sudo apt-get -y install python-simplejson'

14. Try your hands on some ah-hoc commands

                  ansible database -i hosts --become -m apt -a 'name=mysql-server state=present'
                  ansible all -i hosts --become -m apt -a 'update_cache=yes'
                  ansible database -i hosts -m service -a 'name=mysql-server state=started'
                  ansible database -i hosts --become -m service -a 'name=mysql-server state=restarted'
                  ansible webstack -i hosts -m command -a 'reboot --reboot'

15. Ansible playbook

      write a simple ansible playbook    with the name playbbok.yml 

  - name: install apache on webserver
  ```yml
    hosts: webserver  
    become: true
    vars:
      http_port: 8000
      https_port: 4443
      html_welcome_msg: "hello world kenneth"

    tasks:
      - name: ensure apache is install at the latest version
        apt:
         name: apache2
         state: latest

```

to run it cd into where the inventory file is and run the following
            $ ansible-playbook -i hosts -k playbook.yml


PROJECT
      use playbook to install and start nginx and view on web browser



16. to clean up the playbook file, we can create differnt files and use the (include_tasks: PATH to file) to add files into our play book. we can also use (import_tasks)

17. Roles

      to get the folder structure in the terminal (run tree command) id tree is not installed use 'apt install tree'


      To get the correct folder strudtre for ansible roles 'run ansible-galaxy init roles/apache2'
                        NOTE: the roles name is apache2, this can be anything you want.

            it adaviceable to create roles for differnt tasks eg. webservers, database servers, loadbalancers etc.



18. Tags
      With the use odf tags, we can spscify which play to run and which need not to run.
            => to check for the tags you have in yoyr playbook
                        '--list-tags'
            => To use the tag
                        '--tags tageNamehere 

19. Variables
      types of variables
            - system variables
                  ansible -i hosts proxy - m setup >> ansible_facts.json
            - User created variables                             