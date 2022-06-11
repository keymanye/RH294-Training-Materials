#### ansible playbook编写
- ansible检查环境
```bash
[training@controlnode ~]$ ansible  --version
ansible [core 2.13.0]
  config file = /home/training/.ansible.cfg
  configured module search path = ['/home/training/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /home/training/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.7 (default, Sep 13 2021, 08:18:39) [GCC 8.5.0 20210514 (Red Hat 8.5.0-3)]
  jinja version = 3.0.3
  libyaml = True
```
- 使用ansible-doc命令列出所有的模块
  安装ansible rpm包，不要使用系统自带ansible-core这个包
```bash
[training@controlnode ~]$  sudo rpm -e ansible-core
[training@controlnode ~]$  sudo dnf install -y ansible.noarch

[training@controlnode ~]$ ansible-doc -l
add_host               Add a host (and alternatively a group) to the ansible-playbook in-memory inventory
apt                    Manages apt-packages
apt_key                Add or remove an apt key
apt_repository         Add and remove APT repositories
assemble               Assemble configuration files from fragments
assert                 Asserts given expressions are true
async_status           Obtain status of asynchronous task
blockinfile            Insert/update/remove a text block surrounded by marker lines
... ...
```
```bash
[training@controlnode ansibledemo]$ ansible-doc  -l  |grep yum
yum                                                           Manages packages with the `yum' package manager
yum_repository                                                Add or remove YUM repositories
[training@controlnode ansibledemo]$ ansible-doc  -l  |grep dnf
dnf                                                           Manages packages with the `dnf' package manager
[training@controlnode ansibledemo]$ ansible-doc  -l  |grep http
apache2_mod_proxy                                             Set and/or get members' attributes of an Apache httpd 2.4 m...
avi_httppolicyset                                             Module for setup of HTTPPolicySet Avi RESTful Object
bigip_device_httpd                                            Manage HTTPD related settings on BIG-IP
bigip_gtm_monitor_http                                        Manages F5 BIG-IP GTM http monitors
bigip_gtm_monitor_https                                       Manages F5 BIG-IP GTM https monitors
bigip_monitor_http                                            Manages F5 BIG-IP LTM http monitors
bigip_monitor_https                                           Manages F5 BIG-IP LTM https monitors
bigip_profile_http                                            Manage HTTP profiles on a BIG-IP
bigip_profile_http2                                           Manage HTTP2 profiles on a BIG-IP
```

- 使用ansible-doc命令查询copy模块的语法及参数
```bash
[training@controlnode ~]$ ansible-doc  copy
#模块的位置
> ANSIBLE.BUILTIN.COPY    (/usr/lib/python3.9/site-packages/ansible/modules/copy.py)

        The `copy' module copies a file from the local or remote machine to a location on the remote
        machine. Use the [ansible.builtin.fetch] module to copy files from remote locations to the
        local box. If you need variable interpolation in copied files, use the
        [ansible.builtin.template] module. Using a variable in the `content' field will result in
        unpredictable output. For Windows targets, use the [ansible.windows.win_copy] module
        instead.

ADDED IN: historical

  * note: This module has a corresponding action plugin.

OPTIONS (= is mandatory):

- attributes
        The attributes the resulting filesystem object should have.
        To get supported flags look at the man page for `chattr' on the target system.
        This string should contain the attributes in the same order as the one displayed by
        `lsattr'.
        The `=' operator is assumed as default, otherwise `+' or `-' operators need to be included
        in the string.
        (Aliases: attr)[Default: (null)]
        type: str
        added in: version 2.3 of ansible-core


- backup
        Create a backup file including the timestamp information so you can get the original file
        back if you somehow clobbered it incorrectly.
        [Default: False]
        type: bool
        added in: version 0.7 of ansible-core


- checksum
        SHA1 checksum of the file being transferred.

  - decrypt
          This option controls the autodecryption of source files using vault.
          [Default: True]
          type: bool
          added in: version 2.4 of ansible-core


  = dest
          Remote absolute path where the file should be copied to.
          If `src' is a directory, this must be a directory too.
          If `dest' is a non-existent path and if either `dest' ends with "/" or `src' is a directory,
          `dest' is created.
          If `dest' is a relative path, the starting directory is determined by the remote host.
          If `src' and `dest' are files, the parent directory of `dest' is not created and the task
          fails if it does not already exist.
  EXAMPLES:

  - name: Copy file with owner and permissions
    ansible.builtin.copy:
      src: /srv/myfiles/foo.conf
      dest: /etc/foo.conf
      owner: foo
      group: foo
      mode: '0644'

  - name: Copy file with owner and permission, using symbolic representation
    ansible.builtin.copy:
      src: /srv/myfiles/foo.conf
      dest: /etc/foo.conf
      owner: foo
      group: foo
      mode: u=rw,g=r,o=r

  - name: Another symbolic mode example, adding some permissions and removing others
    ansible.builtin.copy:
      src: /srv/myfiles/foo.conf
      dest: /etc/foo.conf
      owner: foo
      group: foo
      mode: u+rw,g-wx,o-rwx
```


- 配置vim编辑器，设置缩进等参数
```bash
[training@controlnode ~]$ vim /etc/vimrc
#放在文件最后
autocmd  FileType  yaml  setlocal  ai  ts=2  sw=2  et
```
- 编写playbook
```YAML
---
name: RH294 training first playbook
hosts: web
tasks:
  - name: describle task 1  info
      copy:
        src: rh294-copy-test.txt
        dest: /tmp
        mode: "0600"

  - name: my trianing task 2
      command: touch /tmp/test1.txt
...
```
- 检查playbook的语法
```bash
[training@controlnode RH294-Training-Materials]$ ansible-playbook --syntax-check unit4/first-playbook.yml
ERROR! We were unable to read either as JSON nor YAML, these are the errors we got from each:
JSON: Expecting value: line 1 column 1 (char 0)

Syntax Error while loading YAML.
  mapping values are not allowed in this context

The error appears to be in '/home/training/RH294-Training-Materials/unit4/first-playbook.yml': line 6, column 11, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  - name: describle task 1  info
      copy:
          ^ here
[training@controlnode RH294-Training-Materials]$ cat unit4/first-playbook.yml

[training@controlnode RH294-Training-Materials]$ vim unit4/first-playbook.yml
[training@controlnode RH294-Training-Materials]$ cat unit4/first-playbook.yml
---
name: RH294 training first playbook
hosts: web
tasks:
  - name: describle task 1  info
    copy:
      src: rh294-copy-test.txt
      dest: /tmp
      mode: "0600"

  - name: my trianing task 2
    command: touch /tmp/test1.txt


...
[training@controlnode RH294-Training-Materials]$ ansible-playbook --syntax-check unit4/first-playbook.yml
ERROR! A playbook must be a list of plays, got a <class 'ansible.parsing.yaml.objects.AnsibleMapping'> instead

The error appears to be in '/home/training/RH294-Training-Materials/unit4/first-playbook.yml': line 2, column 1, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

---
name: RH294 training first playbook
^ here
[training@controlnode RH294-Training-Materials]$ vim unit4/first-playbook.yml
[training@controlnode RH294-Training-Materials]$ cat unit4/first-playbook.yml
---
- name: RH294 training first playbook
  hosts: web
  tasks:
    - name: describle task 1  info
      copy:
        src: rh294-copy-test.txt
        dest: /tmp
        mode: "0600"

    - name: my trianing task 2
      command: touch /tmp/test1.txt

[training@controlnode RH294-Training-Materials]$ ansible-playbook --syntax-check unit4/first-playbook.yml

playbook: unit4/first-playbook.yml
...
```

- 测试运行playbook
```bash

#在playbook里面写的文件的相对路径都是以playbook yaml文件作为参照物
[training@controlnode RH294-Training-Materials]$ ansible-playbook  -C  unit4/first-playbook.yml

PLAY [RH294 training first playbook] *****************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [server-A]

TASK [describle task 1  info] ************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: If you are using a module and expect the file to exist on the remote, see the remote_src option
fatal: [server-A]: FAILED! => {"changed": false, "msg": "Could not find or access 'rh294-copy-test.txt'\nSearched in:\n\t/home/training/RH294-Training-Materials/unit4/files/rh294-copy-test.txt\n\t/home/training/RH294-Training-Materials/unit4/rh294-copy-test.txt\n\t/home/training/RH294-Training-Materials/unit4/files/rh294-copy-test.txt\n\t/home/training/RH294-Training-Materials/unit4/rh294-copy-test.txt on the Ansible Controller.\nIf you are using a module and expect the file to exist on the remote, see the remote_src option"}

PLAY RECAP *******************************************************************************************************************
server-A                   : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0


[training@controlnode RH294-Training-Materials]$ echo "first playbook" >  rh294-copy-test.txt

[training@controlnode RH294-Training-Materials]$ ls
aa.txt  ansible.cfg  inventory  README.md  rh294-copy-test.txt  unit3  unit4

[training@controlnode RH294-Training-Materials]$ ansible-playbook  -C  unit4/first-playbook.yml

PLAY [RH294 training first playbook] *****************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [server-A]

TASK [describle task 1  info] ************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: If you are using a module and expect the file to exist on the remote, see the remote_src option
fatal: [server-A]: FAILED! => {"changed": false, "msg": "Could not find or access 'rh294-copy-test.txt'\nSearched in:\n\t/home/training/RH294-Training-Materials/unit4/files/rh294-copy-test.txt\n\t/home/training/RH294-Training-Materials/unit4/rh294-copy-test.txt\n\t/home/training/RH294-Training-Materials/unit4/files/rh294-copy-test.txt\n\t/home/training/RH294-Training-Materials/unit4/rh294-copy-test.txt on the Ansible Controller.\nIf you are using a module and expect the file to exist on the remote, see the remote_src option"}

PLAY RECAP *******************************************************************************************************************
server-A                   : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0


[training@controlnode RH294-Training-Materials]$ mv rh294-copy-test.txt   unit4/
[training@controlnode RH294-Training-Materials]$ ansible-playbook  -C  unit4/first-playbook.yml

PLAY [RH294 training first playbook] *****************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [server-A]

TASK [describle task 1  info] ************************************************************************************************
changed: [server-A]

TASK [my trianing task 2] ****************************************************************************************************
skipping: [server-A]

PLAY RECAP *******************************************************************************************************************
server-A                   : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

- 运行playbook
```bash
[training@controlnode RH294-Training-Materials]$ ansible-playbook  unit4/multi-play.yml

PLAY [rh294 multiplay play 1] ************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [server-A]
ok: [server-b]

TASK [play 1  task 1] ********************************************************************************************************
ok: [server-A]
ok: [server-b]

TASK [play1 task2] ***********************************************************************************************************
ok: [server-A]
ok: [server-b]

PLAY [rh294 multiplay play 2] ************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [server-A]

TASK [play 2  task 1 copy index.html to apache document root] ****************************************************************
changed: [server-A]

PLAY RECAP *******************************************************************************************************************
server-A                   : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
server-b                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[training@controlnode RH294-Training-Materials]$ curl  http://a
httpd service  on webhost

[training@controlnode RH294-Training-Materials]$ curl  http://b
selinux test write on /root  #在之前配置中给定的html
```bash

- 编写多play的playbook
```bash
[training@controlnode RH294-Training-Materials]$ cat unit4/multi-play.yml
---
- name: rh294 multiplay play 1
  hosts: all
  tasks:
    - name: play 1  task 1
      dnf:
        name: httpd
        state: latest

    - name: play1 task2
      service:
        name: httpd
        state: started

- name: rh294 multiplay play 2
  hosts: web
  tasks:
    - name: play 2  task 1 copy index.html to apache document root
      copy:
        src: index.html
        dest: /var/www/html/index.html
        mode: "0644"
        owner: apache
        group: apache
...
[training@controlnode RH294-Training-Materials]$ ansible-playbook --syntax-check  unit4/multi-play.yml
[training@controlnode RH294-Training-Materials]$ ansible-playbook  unit4/multi-play.yml 
```
