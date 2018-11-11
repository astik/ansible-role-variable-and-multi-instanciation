# Variable resolution within multi-instantiated role

Here are some demo relative to a "bug" in Ansible.
This behavior has been encountered with multiple Ansible's versions (at least from up to 2.7.1 on macOS Mojave 10.14)

The idea is to have a role that is used multiple times with different inputs. It works well until you have a dynamic handler that depends on variable. If the variable for handler's name is defined in playbook, it works. If the variable is coming from group_vars, it fails.

As behavior is depending on where variable is set, i found it inconsistent, therefore it seems to be a bug.

The opened issue is : https://github.com/ansible/ansible/issues/48537.

Here are some other issues that seems to be relative :

- 2.0.0.2 : https://github.com/ansible/ansible/issues/14082
- 2.0.2.0 : https://github.com/ansible/ansible/issues/15713
- 2.2.0.0 : https://github.com/ansible/ansible/issues/19898
- 2.3.0 : https://github.com/ansible/ansible/issues/17922 (same observation with 2.4.0 and 2.5.2)

Also, a stackoverflow that seems to be also relative : https://stackoverflow.com/questions/25694249/ansible-using-with-items-with-notify-handler/25704162#25704162

## Demo

Each demo are composed of a naive playbook with a very simple role **test**.
You can test any of those :

    cd xx-my-demo
    ansible-playbook test.yml -i inventories/testInv

### 01-variable-in-playbook\_\_SUCCESS

- variables are defined into playbook directly
- handler has a dynamic name ("handle inst1" and "handle inst2")
- both notifications trigger their own handler

```
PLAY [servers] ****************************************************************

TASK [test : debug] ***********************************************************
ok: [localhost] => {
    "msg": "echo inst1"
}

TASK [test : command] *********************************************************
changed: [localhost]

TASK [test : debug] ***********************************************************
ok: [localhost] => {
    "msg": "echo inst2"
}

TASK [test : command] *********************************************************
changed: [localhost]

RUNNING HANDLER [test : handle inst1] *****************************************
ok: [localhost] => {
    "msg": "inst1"
}

RUNNING HANDLER [test : handle inst2] *****************************************
ok: [localhost] => {
    "msg": "inst2"
}

PLAY RECAP ********************************************************************
localhost                  : ok=6    changed=2    unreachable=0    failed=0
```

### 02-handler-with-dynamic-name\_\_ERROR

- same as demo 01
- variables are moved from playbook to group_vars
- an error occurs when execution ansible-playbook

```
PLAY [servers] ****************************************************************

TASK [test : debug] ***********************************************************
ok: [localhost] => {
    "msg": "echo inst1"
}

TASK [test : command] *********************************************************
ERROR! The requested handler 'handle inst1' was not found in either the main handlers list nor in the listening handlers list
```

### 03-handler-with-static-name\_\_FAIL

- same as demo 02
- handler's name is now static ("handle foobar")
- both notifications are triggered but only one handler is executed

```
PLAY [servers] ****************************************************************

TASK [test : debug] ***********************************************************
ok: [localhost] => {
    "msg": "echo inst1"
}

TASK [test : command] *********************************************************
changed: [localhost]

TASK [test : debug] ***********************************************************
ok: [localhost] => {
    "msg": "echo inst2"
}

TASK [test : command] *********************************************************
changed: [localhost]

RUNNING HANDLER [test : handle foobar] ****************************************
ok: [localhost] => {
    "msg": "inst1"
}

PLAY RECAP ********************************************************************
localhost                  : ok=5    changed=2    unreachable=0    failed=0
```

### 04-handler-with-dynamic-name-take2\_\_SUCCESS

- same as demo 03
- handler's name is back to dynamic but with a variable that is set explicitly in the playbook ("myInstance1_from_playbook" and "myInstance2_from_playbook")

```
PLAY [servers] ****************************************************************

TASK [test : debug] ***********************************************************
ok: [localhost] => {
    "msg": "echo inst1"
}

TASK [test : command] *********************************************************
changed: [localhost]

TASK [test : debug] ***********************************************************
ok: [localhost] => {
    "msg": "echo inst2"
}

TASK [test : command] *********************************************************
changed: [localhost]

RUNNING HANDLER [test : handle myInstance1_from_playbook] *********************
    "msg": "inst1"
}

RUNNING HANDLER [test : handle myInstance2_from_playbook] *********************
ok: [localhost] => {
    "msg": "inst2"
}

PLAY RECAP ********************************************************************
localhost                  : ok=6    changed=2    unreachable=0    failed=0
```
