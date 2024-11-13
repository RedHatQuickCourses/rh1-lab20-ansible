[Previous Exercise](../0.0-apresentation) - [Next Exercise](../2.0-send-var-with-curl)

# 1.0 - Show how to use CURL with Extra Variables for Event Driven.

Access the AAP node via ssh:

```bash
  ssh -l sysadmin 172.25.5.1 or ssh sysadmin@172.25.5.1 
```

```bash
  subscription-manager repos --enable ansible-automation-platform-2.5-for-rhel-9-x86_64-rpms
  dnf install ansible-lint ansible-rulebook ansible-builder ansible-navigator molecule socat wireshark -y
  
```


Create the files below:

   ```bash
    mkdir 01-lab
    cd 01-lab/
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
- hosts: localhost
  connection: local
  tasks:
    - debug:
        msg: "Hello RH1"

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
```

In the terminal where you created the files. Run ansible-rulebook:

```bash
  ansible-galaxy collection install ansible.eda
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
  curl -H 'Content-Type: application/json' -d '{"event_name": "Hello" }' localhost:5000/endpoint
```

In the terminal where you executed the rulebook command. You will get this output:


```bash
2024-10-20 22:50:01,957 - aiohttp.access - INFO - 127.0.0.1 [21/Oct/2024:01:50:01 +0000] "POST /endpoint HTTP/1.1" 200 158 "-" "curl/7.61.1"
{   'meta': {   'endpoint': 'endpoint',
                'headers': {   'Accept': '*/*',
                               'Content-Length': '24',
                               'Content-Type': 'application/json',
                               'Host': 'localhost:5000',
                               'User-Agent': 'curl/7.61.1'},
                'received_at': '2024-10-21T01:50:01.956477Z',
                'source': {   'name': 'ansible.eda.webhook',
                              'type': 'ansible.eda.webhook'},
                'uuid': '16fd293e-342a-4fd8-9203-196ff207239b'},
    'payload': {'event_name': 'Hello'}}
2024-10-20 22:50:01 980 [main] INFO org.drools.ansible.rulebook.integration.api.rulesengine.RegisterOnlyAgendaFilter - Activation of effective rule "Hello RH1" with facts: {m={payload={event_name=Hello}, meta={headers={Accept=*/*, User-Agent=curl/7.61.1, Host=localhost:5000, Content-Length=24, Content-Type=application/json}, endpoint=endpoint, received_at=2024-10-21T01:50:01.956477Z, source={name=ansible.eda.webhook, type=ansible.eda.webhook}, uuid=16fd293e-342a-4fd8-9203-196ff207239b}}}
2024-10-20 22:50:01,983 - ansible_rulebook.rule_generator - INFO - calling Hello RH1
2024-10-20 22:50:01,984 - ansible_rulebook.rule_set_runner - INFO - call_action run_playbook
2024-10-20 22:50:01,984 - ansible_rulebook.rule_set_runner - INFO - substitute_variables [{'name': 'hello-rh1.yml'}] [{'event': {'payload': {'event_name': 'Hello'}, 'meta': {'headers': {'Accept': '*/*', 'User-Agent': 'curl/7.61.1', 'Host': 'localhost:5000', 'Content-Length': '24', 'Content-Type': 'application/json'}, 'endpoint': 'endpoint', 'received_at': '2024-10-21T01:50:01.956477Z', 'source': {'name': 'ansible.eda.webhook', 'type': 'ansible.eda.webhook'}, 'uuid': '16fd293e-342a-4fd8-9203-196ff207239b'}}}]
2024-10-20 22:50:01,984 - ansible_rulebook.rule_set_runner - INFO - action args: {'name': 'hello-rh1.yml'}
2024-10-20 22:50:01,984 - ansible_rulebook.builtin - INFO - running Ansible playbook: hello-rh1.yml
2024-10-20 22:50:01,985 - ansible_rulebook.builtin - INFO - ruleset: Listen for events on a webhook, rule: Hello RH1
2024-10-20 22:50:01,985 - ansible_rulebook.builtin - INFO - Calling Ansible runner

PLAY [localhost] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [debug] *******************************************************************
ok: [localhost] => {
    "msg": "Hello RH1"
}

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
2024-10-20 22:50:13,309 - ansible_rulebook.builtin - INFO - Ansible Runner Queue task cancelled
2024-10-20 22:50:13,311 - ansible_rulebook.builtin - INFO - Playbook rc: 0, status: successful
2024-10-20 22:50:13,312 - ansible_rulebook.rule_set_runner - INFO - Task action::run_playbook::Listen for events on a webhook::Hello RH1 finished, active actions 0

```

[Previous Exercise](../0.0-apresentation) - [Next Exercise](../2.0-send-var-with-curl)