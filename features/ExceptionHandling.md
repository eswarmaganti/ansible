# Handle Exceptions in Ansible Playbooks

## What is a block?
- A block is a logical grouping of tasks within a playbook that can be executed as a single unit. This makes it easy to manage complex playbooks by breaking them down into smaller, more manageable parts.

    ```yaml
    tasks:
      - name: Install, configure, and start apache
        block:
          - name: Install httpd and memcached
            ansible.builtin.yum:
              name:
                - httpd
                - memcached
              state: present
          - name: Apply the httpd.conf file
            ansible.builtin.template:
              src: httpd.conf.j2
              dest: /etc/httpd/conf/httpd.conf
          - name: Start the httpd service
            ansible.builtin.service:
              name: httpd
              state: started
              enabled: true
        when: ansible_facts['distribution'] == 'CentOS'
        become: true
        become_user: root
        ignore_errors: true
    ```
- Notice that the keywords `when`, `become`, `become_user`, and `ignore_errors` are all applied to the block.

# How to use blocks and rescue in Ansible
- Blocks and rescue work together to provide error-handling capabilities in Ansible. Use the rescue keyword in association with a block to define a set of tasks that will be executed if an error occurs in the block.
- You can use the rescue tasks to handle errors, log messages, or take actions to recover from the error.

- High level syntax

    ```yaml
    ---
    hosts: <hosts>
    tasks:
      block:
        - <task1>
        - <task2>
        - <task3>
      rescue:
        - <rescue-task1>
        - <rescue-task2>
        - <rescue-task3>
      always:
        - <always-task>
    ```
- You define tasks under the **block** keyword, which could be as simple as invoking the `ansible.builtin.ping` module, or you can have a combination of multiple tasks and including/importing roles
- The associated **rescue** keyword is where the playbook execution will be sent, for each host, if anything fails along the block.
- Finally, the always section executes for all nodes, no matter if they succeed pr fail.
- Some key ideas from this structure
  - **rescue** and **always** are optional features, which I will use for the specific purpose of demonstrating this "recover and summary" logic
  - When your playbook runs against considerable number of hosts, handling the individual results becomes harder to track.

## Example:
- Consider the below inventory file having two nodes
    ```yaml
    all:
      children:
        nodes:
          hosts:
            vagrantubuntulab0101:
              ansible_host: 192.168.58.101
            vagrantubuntulab0102:
              ansible_host: 192.168.58.102
    ```
- Consider the below playbook
    ```yaml
    ---
    - name: test block/rescue
      hosts: nodes
      gather_facts: true
      tasks:
        - name: Main Block
          block:
            - name: Test the ping
              ansible.builtin.ping:
              failed_when: inventory_hostname == 'vagrantubuntulab0102'
    
            - name: Accumulate success
              ansible.builtin.set_fact:
                _result:
                  host: "{{ inventory_hostname }}"
                  status: "OK"
                  interfaces: "{{ ansible_facts['interfaces'] }}"
    
          rescue:
            - name: Accumulate failure
              ansible.builtin.set_fact:
                _result:
                  host: "{{ inventory_hostname }}"
                  status: "FAIL"
          always:
            - name: Tasks that will run after the main block
              block:
                - name: Collect results
                  ansible.builtin.set_fact:
                    _global_results: "{{ (_global_results | default([])) + [hostvars[item]['_result']]  }}"
                  loop: "{{ ansible_play_hosts }}"
    
                - name: Classify results
                  ansible.builtin.set_fact:
                    _results_ok: "{{ _global_results | selectattr('status', 'equalto', 'OK') | list }}"
                    _results_fail: "{{ _global_results | selectattr('status', 'equalto', 'FAIL') | list }}"
    
                - name: Display results OK
                  ansible.builtin.debug:
                    msg: "{{ _results_ok }}"
                  when: ( _results_ok | length ) > 0
    
                - name: Display results FAIL
                  ansible.builtin.debug:
                    msg: "{{ _results_fail }}"
                  when: ( _results_fail | length ) > 0
              run_once: true
              delegate_to: localhost
    ```
- For this above example playbook, The main block don't do anything special, If a node succeeds, there will be a list of interfaces in the `_result` variable. Otherwise, the status will be set to *FAIL*.
- For each host the playbook is running on:
  - If the actions proceed without errors, the task **Accumulate Success** will execute
  - If the action fails in any of the tasks, the flow goes to the rescue block for each host.
- The **always** section collects the results saved in the variable `_result`. Here is a little breakdown of the logic
  - Up to this point, each host has a variable in its **hostvars** structure, either with a success ot failed status information.
  - In the **Collect results** task, which runs once and is delegated to localhost, it captures the individual results and adds them to the list `_global_results`.
  - The loop is done using the Ansible magic variable `ansible_play_hosts` which is a list of all hosts targeted by this playbook.
  - **Classify results** tasks does some filtering to create a list of all OK and FAILED results. 
- The below will be the results when you target to fail the task in one node.

    ```
    PLAY [test block/rescue] ******************************************************************************************
    
    TASK [Gathering Facts] ********************************************************************************************
    ok: [vagrantubuntulab0102]
    ok: [vagrantubuntulab0101]
    
    TASK [Test the ping] **********************************************************************************************
    [ERROR]: Task failed: Action failed: Unknown error.
    Origin: /Users/eswarmaganti/Developer/Projects/DevOps/ansible/features/playbooks/exception_handling.yml:8:11
    
    6     - name: Main Block
    7       block:
    8         - name: Test the ping
                ^ column 11
    
    fatal: [vagrantubuntulab0102]: FAILED! => {"changed": false, "failed_when_result": true, "msg": "Task failed: Action failed: Unknown error.", "ping": "pong"}
    ok: [vagrantubuntulab0101]
    
    TASK [Accumulate success] *****************************************************************************************
    ok: [vagrantubuntulab0101]
    
    TASK [Accumulate failure] *****************************************************************************************
    ok: [vagrantubuntulab0102]
    
    TASK [Collect results] ********************************************************************************************
    ok: [vagrantubuntulab0101 -> localhost] => (item=vagrantubuntulab0101)
    ok: [vagrantubuntulab0101 -> localhost] => (item=vagrantubuntulab0102)
    
    TASK [Classify results] *******************************************************************************************
    ok: [vagrantubuntulab0101 -> localhost]
    
    TASK [Display results OK] *****************************************************************************************
    ok: [vagrantubuntulab0101 -> localhost] => {
        "msg": [
            {
                "host": "vagrantubuntulab0101",
                "interfaces": [
                    "eth0",
                    "lo",
                    "eth1"
                ],
                "status": "OK"
            }
        ]
    }
    
    TASK [Display results FAIL] ***************************************************************************************
    ok: [vagrantubuntulab0101 -> localhost] => {
        "msg": [
            {
                "host": "vagrantubuntulab0102",
                "status": "FAIL"
            }
        ]
    }
    
    PLAY RECAP ********************************************************************************************************
    vagrantubuntulab0101       : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    vagrantubuntulab0102       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
    
    ```
  