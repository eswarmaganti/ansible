# Event Driven Ansible

## What is Event-Driven Ansible?
- Event-Driven Ansible is a new way of working with Ansible based on events. When a specific event occurs a corresponding action is triggered.
- This allows for immediate and automated response to issues or unexpected occurrences.

## How does it work?
- Event-Driven Ansible connects sources of events with corresponding actions via rules.
- Ansible Rulebooks define the event source and explain -- in the form of conditional "if-this-then-that" instructions -- the action to take when the event occurs.
- Based on the rulebook, Event-Driven Ansible recognizes the specified event, matches it with the appropriate action, and automatically executes it.
- Action's can optionally include executing existing ansible playbooks, templates, or modules to extend value from your trusted automation.

- The `Ansible Rulebook` is a YAML file which includes the following fields
  - `sources`:
    - This is a list of possible sources from which events can be gathered. At the moment there are already a few sources available. 
    - For example the webhook source plugin provides a `webhook` which can be triggered from any application. The `kafka` plugin allows to receive events via a `kafka` topic. You can find the current list of supported sources [here](https://docs.ansible.com/projects/rulebook/en/latest/sources.html)
  - `rules`:
    - Rules describe what actions should be taken depending on specific events. Some of the possible actions are: `run_playbook`, `run_job_template`, `run_workflow_tempalte`, and many more. You can find the list of supported actions [here](https://docs.ansible.com/projects/rulebook/en/latest/actions.html)
- A rulebook is started with the `ansible-rulebook` CLI tool, available through `pip`. Alternatively, for customers of the AAP, there is also the possibility of installing the EDA controller: a web UI for Event-Driven Ansible.

- Let's consider an example:
  - Your observability tool - the event source - is watching network routers and discovers that a routing is not responding, recognizing this as an event.
  - Event-Driven Ansible receives this event, finds the corresponding Ansible Rulebook, and matches the event with the desired action - which could be re-applying a configuration, resetting the router, or creating a service ticket.
  - Event-Driven Ansible triggers the instructions in the rulebook and the router is reset , restoring it to normal function.

## Example
- Let's have a look at minimum example which demonstartes how Event-Driven Ansible works. We image a situation in which we have a webserver running and want to monitor it. We can do that with the following rulebook.

    ```yaml
    ---
    - name: Check webserver
      hosts: all
      sources:
        - ansible.eda.url_check:
            urls:
              - http://<webserver_fqdn>
            delay: 30
      rules:
        - name: Restart Nginx
          condition: event.url_check.status == "down"
          action:
            run_playbook:
              name: restart_nginx.yml
    ```
- This rulebook uses the `url_check` plugin to query the webpage at `http://<webserver_fqdn> every 30 seconds.
- There is only one rule. When the URL check returns a status of `down`, and Ansible Playbook is automatically started.

    ```yaml
    # Playbook to restart the nginx server
    - name: Restart Nginx
      hosts: localhost
      gather_facts: yes
      become: true
      tasks:
        - name: Get the nginx process details
          ansible.builtin.systemd_service:
            name: nginx
          register: nginx_info
    
        - name: Log the process info
          debug:
            msg: "{{ nginx_info }}"
    
        - name: Restart Nginx
          ansible.builtin.systemd_service:
            name: nginx
            state: started
          register: output
    
        - name: Log the output
          debug:
            msg: "{{ output }}"
    ```
- We can start monitoring the webserver using the below ansible-rulebook command

    ```commandline
    ansible-rulebook --rulebook rulebook.yml -i inventory.yml --v
    ```
- The above command runs in foreground and listens for events.
- When it hasn't received an event yet, the output looks like this:

```commandline
2025-12-10 08:32:39,796 - ansible_rulebook.engine - INFO - load source ansible.eda.url_check
2025-12-10 08:32:40,141 - ansible_rulebook.engine - INFO - loading source filter eda.builtin.insert_meta_info
2025-12-10 08:32:40,393 - ansible_rulebook.engine - INFO - Waiting for all ruleset tasks to end
2025-12-10 08:32:40,393 - ansible_rulebook.rule_set_runner - INFO - Waiting for actions on events from Check the Nginx Server
```
- When the webserver is stopped, the rulebook will receive the event and trigger the action playbook

```commandline
2025-12-10 08:32:40,393 - ansible_rulebook.rule_set_runner - INFO - Waiting for events, ruleset: Check the Nginx Server
2025-12-10 08:32:40 394 [drools-async-evaluator-thread] INFO org.drools.ansible.rulebook.integration.api.io.RuleExecutorChannel - Async channel connected
PLAY [Restart Nginx] ***********************************************************

```