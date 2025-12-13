# Ansible Project Lab Setup

- We will set up ubuntu based lab environment to test our playbooks.
- The environment will have a three instances of ssh enabled ubuntu bases containers, where we can test out playbooks.

## Building Docker Image
- under `/project/docker` you can find a docker file with filename `Dockerfile`
- Run the following command to build an image out of our `Dockerfile`. 
  ```
  # make sure you are in project dir
  cd project
  
  # build the docker image
  docker build -t ubuntu-ssh -f ./docker/Dockerfile ./docker
  
  # verify the docker image
  docker image ls | grep ubuntu-ssh
  ```
- Run the docker container using docker compose
    ```
    # change to docker dir
    cd project/docker
    
    # run the compose up to spin the container
    docker compose up
    ```

## Ansible Inventory and Test the connectivity

- Crated an inventory file under `project/inventory/inventory.yml`.
    ```yaml
    ---
    all:
      children:
        ubuntulab:
          hosts:
            ubuntulab0101:
              ansible_host: 127.0.0.1
              ansible_port: 2221
            ubuntulab0102:
              ansible_host: 127.0.0.1
              ansible_port: 2222
            ubuntulab0103:
              ansible_host: 127.0.0.1
              ansible_port: 2223
    ```
- Added the ansible user and password information under `/project/inventory/group_vars/all.yml` file
    ```yaml
    ---
    ansible_user: ansible
    ansible_password: "{{ lookup('env', 'ansible_password') }}"
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
    ansible_python_interpreter: "/usr/bin/python3.12"
    ```
  
- To test the connectivity to the lab environment container using the inventory, run the below command

    ```yaml
    $export ansible_password=p@ssw0rd
    $pwd
    /Users/eswarmaganti/Developer/Projects/DevOps/ansible
    
    $ansible -m ping -i project/inventory/inventory.yml ubuntulab
      ubuntulab0103 | SUCCESS => {
    "changed": false,
    "ping": "pong"
    }
      ubuntulab0102 | SUCCESS => {
    "changed": false,
    "ping": "pong"
    }
      ubuntulab0101 | SUCCESS => {
    "changed": false,
    "ping": "pong"
    }
    ```