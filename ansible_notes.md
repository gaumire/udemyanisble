# Ansible Notes


## Roles
* Organized and consolidated version of playbooks that can be reused.
* A folder where we put all our task, files, variables, handlers that does a specific task.
* Provides an encapsulation layer and enables efficient management.
* Reduces duplicate work. You'd know exactly where the change for task or handlers needs to be done.
* To create new roles, copy tasks and handlers from respective files to `main.yml` in respective folders.
* Location of src templates in tasks needs to remove `templates/` prefix as with roles ansible assumes templates are in the templates directory.
* Also with roles, ansible takes all the source files that are to be copied to remote servers form the `files/` directory inside roles.
* Commands:

    ```
    # mkdir -p app/roles
    # ansible-galaxy init role1
    # ansible-galaxy init role2

    ## Role Directory

    role1
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    └── vars
        └── main.yml
    ```

## Variables
* Variable precedence varies in 1.x and 2.x and 2.x has a lot more levels. But injecting variabels using '-e' always wins.
* Variables can be injected from tasks with something like

    ```
    - name: blabla
      module: attributes
      vars:
        key: value
    ```
* For convenience pick one plae where you put your vars and always use that, that way it gets a lot easier on you. A `group_vars` file is recommended.
* To gather all ansible-facts for a host, do :

    ```
    # ansible -m setup $host
    ```
* The `defaults/main.yml` file in roles provides with default variables to be used in roles. They can be overridden.

## `with_dict` vs  `with_items`
* It actually depends on the type variable that's being iterated. For e.g, for `with_items` generally a list of items is used and for cases where the variable is a kvp/hash key map type of data, `with_dict` is preferred.


## Removal of one or more item/file/directory using ansible.
* First, create a task that lists all the items required and register it.
* In another task:
    * Configure the removal command.
    * Run it for each item that doesn't meet a pre_defined criteria:
    * For e.g,
        ```
        vars:
            myfiles:
                - file1
                - file2

        - name: get existing files
          shell: ls -1 /etc/filedir/
          register: existing_files

        - name: remove nx files
          file: /etc/filedir/{{item}} state=absent
          with_items: existing_files.stdout_lines
          when: item not in myfiles
        ```

## `group_vars` and `host_vars` structure
* If you put it in the right place, it automatically gets picked up and we can segment it my group/hosts.
* Create a director called `group_vars` in the project root and create further yml files for each group or hosts.

    ```
    ├── group_vars
        └── all.yml
        └── webserver.yml

    File: all.yml
    ---
    var1 : data1
    var2 : data2
    var3 : data3
    ```

## Vault
* Encrypts a var file with a passphrase. Entire file will be encrypted, rendered unreadable normally.
* Supply passphrase while using ansible-playbook.
* Vault encrypts the entire content of a file, so you would loose visibility as in what keys are encrypted. [ encryption is generally for values I guess ]
* You can pair a vault file with an unencrypted vars file. And refer between them for encrypted and unencrypted values.
* Commands :

    ```
    # ansible-vault create vault
    # ansible-vault edit vault
    # ansible-vault --ask-vault-pass
    ```
* You can also configure default password file location in ansible configuration.
* only supports one passphrase per execution, so all vault files should have same password :(

## Galaxy
* online repo for roles that other peoples can contribute to.
* Commands :

    ```
    ansible-galaxy install username.rolename
    ```

## General Optimization
* Look for places where you don't need to gather facts and disable them.
* When using apt module, set a cache timeout for apt-cache.
* Limit your playbook runs to hosts that actually need the changes applied. Use either hosts or tags.
* For tasks which result doesn't matter when it's changed because there may not be any physical changes on the server e.g, `service status`. Use `changed_when`.
* You can also use `failed_when` to add some additional params to check if the task has actually failed rather than using default module logic.

## Debug
* use `debug` to print debug messages and variables

    ```
    - name: print list of files
      shell: ls -1 /etc/myfolder/
      register: active_files

    - debug: myvar=active_files.stdout_lines
    ```
* run playbook with `--step` parameter to enable step by step task runs.
