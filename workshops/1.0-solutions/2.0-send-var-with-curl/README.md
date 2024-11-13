[Previous Exercise](../2.0-send-var-with-curl) - [Next Exercise](../3.0-config-eda-send-extra-variables)

# 2.0 - How to capture the Event Driven playload via command line.

Access the bastion node via ssh:

```bash
  ssh -l sysadmin 172.25.5.1 or ssh sysadmin@172.25.5.1 
```

Create the files below:

   ```bash
    mkdir 02-lab
    cd 02-lab/
    touch  inventory.yml hello-rh1.yml webhook-example.yml
   ```

 - inventory.yml

   Edit the file inventory.yml
   ```bash
    vim inventory.yml
   ```
   Add the Content below:
   To add press i

```yml
all:
  hosts:
    localhost:
      ansible_connection: local
```

  Save file: esq + :x

  Do the same for the files: hello-rh1.yml e webhook-example.yml

 - hello-rh1.yml
```yml
---
- name: create env
  hosts: localhost
  tasks:
    - name: Add hosts
      ansible.builtin.add_host:
        name: "{{ hosts_update }}"
        groups: centrify
      with_items: '{{ hosts_update.split() }}'

- name: Install centrifyDC Client
  hosts: centrify
  become: yes
  remote_user: root
  tasks:
    - name: Uptime host test
      ansible.builtin.shell: uptime

```

 - webhook-example.yml
```yml
---
- name: Listen for events on a webhook
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000
  rules:
    - name: Hello RH1
      condition: event.payload.event_name == "Hello"
      action:
        run_playbook:
          name: hello-rh1.yml
          extra_vars:
            hosts_update: "{{ event.payload.host_host }}"

```

In the terminal where you created the files. Run ansible-rulebook:

```bash
  ansible-rulebook --rulebook webhook-example.yml -i inventory.yml --verbose  --print-events

ex:

[sysadmin@bastion-rhel8 ansible-var-eda]$ ansible-rulebook --rulebook webhook-example.yml -i inventory.yml --verbose  --print-events
2024-10-20 22:57:10,866 - ansible_rulebook.app - INFO - Starting sources
2024-10-20 22:57:10,866 - ansible_rulebook.app - INFO - Starting rules
2024-10-20 22:57:10,866 - ansible_rulebook.engine - INFO - run_ruleset
2024-10-20 22:57:10,867 - drools.ruleset - INFO - Using jar: /usr/local/lib/python3.9/site-packages/drools/jars/drools-ansible-rulebook-integration-runtime-1.0.2-SNAPSHOT.jar
2024-10-20 22:57:11 160 [main] INFO org.drools.ansible.rulebook.integration.api.rulesengine.AbstractRulesEvaluator - Start automatic pseudo clock with a tick every 100 milliseconds
2024-10-20 22:57:11,162 - ansible_rulebook.engine - INFO - ruleset define: {"name": "Listen for events on a webhook", "hosts": ["all"], "sources": [{"EventSource": {"name": "ansible.eda.webhook", "source_name": "ansible.eda.webhook", "source_args": {"host": "0.0.0.0", "port": 5000}, "source_filters": []}}], "rules": [{"Rule": {"name": "Hello RH1", "condition": {"AllCondition": [{"EqualsExpression": {"lhs": {"Event": "payload.event_name"}, "rhs": {"String": "Hello"}}}]}, "actions": [{"Action": {"action": "run_playbook", "action_args": {"name": "hello-rh1.yml"}}}], "enabled": true}}]}
2024-10-20 22:57:11,169 - ansible_rulebook.engine - INFO - load source
2024-10-20 22:57:11,358 - ansible_rulebook.engine - INFO - load source filters
2024-10-20 22:57:11,358 - ansible_rulebook.engine - INFO - loading eda.builtin.insert_meta_info
2024-10-20 22:57:11,535 - ansible_rulebook.engine - INFO - Calling main in ansible.eda.webhook
2024-10-20 22:57:11,536 - ansible_rulebook.engine - INFO - Waiting for all ruleset tasks to end
2024-10-20 22:57:11,536 - ansible_rulebook.rule_set_runner - INFO - Waiting for actions on events from Listen for events on a webhook
2024-10-20 22:57:11,536 - ansible_rulebook.rule_set_runner - INFO - Waiting for events, ruleset: Listen for events on a webhook
2024-10-20 22:57:11 536 [drools-async-evaluator-thread] INFO org.drools.ansible.rulebook.integration.api.io.RuleExecutorChannel - Async channel connected


```

In another terminal, run the curl command:

```bash
  curl -H 'Content-Type: application/json' -d '{"event_name": "Hello", "host_host": "gitlab.lnx.example.local" }' localhost:5000/endpoint
```

In the terminal where you executed the rulebook command. You will get this output:


Now, You have two vars: event_name and host_host.

```bash
2024-10-22 15:05:30,506 - ansible_rulebook.app - INFO - Starting sources
2024-10-22 15:05:30,506 - ansible_rulebook.app - INFO - Starting rules
2024-10-22 15:05:30,506 - ansible_rulebook.engine - INFO - run_ruleset
2024-10-22 15:05:30,506 - drools.ruleset - INFO - Using jar: /usr/local/lib/python3.9/site-packages/drools/jars/drools-ansible-rulebook-integration-runtime-1.0.2-SNAPSHOT.jar
2024-10-22 15:05:30 841 [main] INFO org.drools.ansible.rulebook.integration.api.rulesengine.AbstractRulesEvaluator - Start automatic pseudo clock with a tick every 100 milliseconds
2024-10-22 15:05:30,844 - ansible_rulebook.engine - INFO - ruleset define: {"name": "Listen for events on a webhook", "hosts": ["all"], "sources": [{"EventSource": {"name": "ansible.eda.webhook", "source_name": "ansible.eda.webhook", "source_args": {"host": "0.0.0.0", "port": 5000}, "source_filters": []}}], "rules": [{"Rule": {"name": "Hello RH1", "condition": {"AllCondition": [{"EqualsExpression": {"lhs": {"Event": "payload.event_name"}, "rhs": {"String": "Hello"}}}]}, "actions": [{"Action": {"action": "run_playbook", "action_args": {"name": "hello-rh1.yml", "extra_vars": {"hosts_update": "{{ event.payload.host_host }}"}}}}], "enabled": true}}]}
2024-10-22 15:05:30,852 - ansible_rulebook.engine - INFO - load source
2024-10-22 15:05:31,061 - ansible_rulebook.engine - INFO - load source filters
2024-10-22 15:05:31,061 - ansible_rulebook.engine - INFO - loading eda.builtin.insert_meta_info
2024-10-22 15:05:31,260 - ansible_rulebook.engine - INFO - Calling main in ansible.eda.webhook
2024-10-22 15:05:31,262 - ansible_rulebook.engine - INFO - Waiting for all ruleset tasks to end
2024-10-22 15:05:31,262 - ansible_rulebook.rule_set_runner - INFO - Waiting for actions on events from Listen for events on a webhook
2024-10-22 15:05:31,262 - ansible_rulebook.rule_set_runner - INFO - Waiting for events, ruleset: Listen for events on a webhook
2024-10-22 15:05:31 262 [drools-async-evaluator-thread] INFO org.drools.ansible.rulebook.integration.api.io.RuleExecutorChannel - Async channel connected
2024-10-22 15:07:21,844 - aiohttp.access - INFO - 127.0.0.1 [22/Oct/2024:18:07:21 +0000] "POST /endpoint HTTP/1.1" 200 158 "-" "curl/7.61.1"
{   'meta': {   'endpoint': 'endpoint',
                'headers': {   'Accept': '*/*',
                               'Content-Length': '65',
                               'Content-Type': 'application/json',
                               'Host': 'localhost:5000',
                               'User-Agent': 'curl/7.61.1'},
                'received_at': '2024-10-22T18:07:21.843955Z',
                'source': {   'name': 'ansible.eda.webhook',
                              'type': 'ansible.eda.webhook'},
                'uuid': 'f27e21d6-dd63-4549-bd78-f9b93f454a4c'},
    'payload': {'event_name': 'Hello', 'host_host': 'gitlab.lnx.example.local'}}
2024-10-22 15:07:21 864 [main] INFO org.drools.ansible.rulebook.integration.api.rulesengine.RegisterOnlyAgendaFilter - Activation of effective rule "Hello RH1" with facts: {m={payload={host_host=gitlab.lnx.example.local, event_name=Hello}, meta={headers={Accept=*/*, User-Agent=curl/7.61.1, Host=localhost:5000, Content-Length=65, Content-Type=application/json}, endpoint=endpoint, received_at=2024-10-22T18:07:21.843955Z, source={name=ansible.eda.webhook, type=ansible.eda.webhook}, uuid=f27e21d6-dd63-4549-bd78-f9b93f454a4c}}}
2024-10-22 15:07:21,870 - ansible_rulebook.rule_generator - INFO - calling Hello RH1
2024-10-22 15:07:21,870 - ansible_rulebook.rule_set_runner - INFO - call_action run_playbook
2024-10-22 15:07:21,870 - ansible_rulebook.rule_set_runner - INFO - substitute_variables [{'name': 'hello-rh1.yml', 'extra_vars': {'hosts_update': '{{ event.payload.host_host }}'}}] [{'event': {'payload': {'host_host': 'gitlab.lnx.example.local', 'event_name': 'Hello'}, 'meta': {'headers': {'Accept': '*/*', 'User-Agent': 'curl/7.61.1', 'Host': 'localhost:5000', 'Content-Length': '65', 'Content-Type': 'application/json'}, 'endpoint': 'endpoint', 'received_at': '2024-10-22T18:07:21.843955Z', 'source': {'name': 'ansible.eda.webhook', 'type': 'ansible.eda.webhook'}, 'uuid': 'f27e21d6-dd63-4549-bd78-f9b93f454a4c'}}}]
2024-10-22 15:07:21,873 - ansible_rulebook.rule_set_runner - INFO - action args: {'name': 'hello-rh1.yml', 'extra_vars': {'hosts_update': 'gitlab.lnx.example.local'}}
2024-10-22 15:07:21,873 - ansible_rulebook.builtin - INFO - running Ansible playbook: hello-rh1.yml
2024-10-22 15:07:21,875 - ansible_rulebook.builtin - INFO - ruleset: Listen for events on a webhook, rule: Hello RH1
2024-10-22 15:07:21,875 - ansible_rulebook.builtin - INFO - Calling Ansible runner

PLAY [create env] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Add hosts] ***************************************************************
changed: [localhost] => (item=gitlab.lnx.example.local)

PLAY [Install centrifyDC Client] ***********************************************

TASK [Gathering Facts] *********************************************************
ok: [gitlab.lnx.example.local]

TASK [Uptime host test] ********************************************************
changed: [gitlab.lnx.example.local]

PLAY RECAP *********************************************************************
gitlab.lnx.example.local   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
2024-10-22 15:07:40,528 - ansible_rulebook.builtin - INFO - Ansible Runner Queue task cancelled
2024-10-22 15:07:40,529 - ansible_rulebook.builtin - INFO - Playbook rc: 0, status: successful
2024-10-22 15:07:40,529 - ansible_rulebook.rule_set_runner - INFO - Task action::run_playbook::Listen for events on a webhook::Hello RH1 finished, active actions 0


```

[Previous Exercise](../2.0-send-var-with-curl) - [Next Exercise](../3.0-config-eda-send-extra-variables)