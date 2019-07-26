
class: center, middle

# Ansible metodą prób i błędów
## Czyli wprowadzenie do Ansible na przykładzie wdorożenia w Fast White Cat

---
.left-column[

### O mnie
]
.right-column[

``` yaml
---
- Speaker:
    Name:     Adam Hyski
    Twitter:  @AdamHyski
    Page:     https://hyski.pl
    Mail:     adam@hyski.pl
    GitHub:   github.com/AdamHyski

```
Gdzie będzie prezentacja?

]
---
name: Agenda
.left-column[
### O mnie
### Nomenklatura
]
.right-column[
- ### Czym jest Ansible?
- ### jak go używać [//]: TODO
- ### jak nie używąć [//]:  TODO
]
---

class: center, middle

# Czym jest Ansible?


---
### Playbook

Najprostszy przykład playbooka:
``` yml
---
# playbooks/task1.yml
- hosts:       demo
  tasks:
  - name:     Change hostname
    hostname:
      name:   demo.hyski.pl

```
???
czym są taski a czy playbooki
--

wymaga on jeszcze  `./hosts.yaml`
``` yaml
# file: ./hosts.yaml
all:
  hosts:
    demo.hyski.pl
```
--

> uruchamiamy:

``` shell
$ ansible-playbook playbooks/task1.yml
```
---
class: center, middle
![computer says no](./img/qwertee_computer-says-no.png)
---
# Komputer powiedział „nie”
## Dlaczego?
--

.center[Nie ma pythona 🐍]

---
# Jak sobie z tym poradziłem?
wyciąg z jednego z pierwszych playbooków:

``` yaml
---
- hosts:           demo
  gather_facts:   False

  pre_tasks:
  - name:         Install python for Ansible
    raw:          test -e /usr/bin/python || (apt -y update && apt install -y python-minimal)
    register:     output
    changed_when: output.stdout != ""
    tags:         always
  - setup:        # aka gather_facts

  tasks:
  - name:         Change hostname
    hostname:
      name:       demo.hyski.pl

```
.red[🛑proszę nie używać ]
???
opowiedzieć co to robi  
---
## Z czego to wynika?
za
[Python Clock:](https://pythonclock.org/)
```
Python 2.7 will not be maintained past 2020.
Originally, there was no official date.
Recently, that date has been updated to January 1, 2020
```
???
- jak działa Ansible po stronie servera  
--
#### Mamy też ładny komunikat jak bymśmy dalej próbowali tego używać
 [//]: TODO
![deprecation](./img/deprecation.png)
---
## To jak należy to zrobić?
``` yml
---
# file: playbooks/task1.yml
- hosts:       demo
  tasks:
  - name:     Change hostname
    hostname:
      name:   demo.hyski.pl

```

``` yaml
---
# file: hosts.yaml
all:
  hosts:
    demo:
      ansible_host:                demo.hyski.pl
      ansible_python_interpreter:  /usr/bin/python3

```

--
#### Można tu też dodać użytkownika
`ansible_user: ubuntu`
???
Jak poradzić sobie w firmie gdzie mamy loginy imienne?
Elastyczna konfiguracja.
--


`export ANSIBLE_REMOTE_USER=adam.hyski`
---

class: center, middle

# Playbooki
---

class: center, middle

![deprecation](./img/The_playbook.png)
---
## Jak wyglądają
``` yml
---
# file: playbooks/task2.yml
- hosts:       demo
  tasks:
  - name:     Change hostname
    hostname:
      name:   demo.hyski.pl
  - name:     Install htop
    package:
      name:   htop
      state:  present

```
--
### Alternetywnie
``` yml
- name:                Innstall htop
  apt:
    update_cache:      yes
    cache_valid_time:  3600
    name:              htop
    state:             latest
```
???
- czemu nie polecam package
- jak to będzie wykonywane
- co jeśli któryś się wysypie? a się wysypie
---
## Struktura plików
### Za Ansible Docs Best Practices
``` ls
production        # inventory file for production servers
staging           # inventory file for staging environment

group_vars/
   group1.yml     # here we assign variables to particular groups
   group2.yml
host_vars/
   hostname1.yml  # here we assign variables to particular systems
   hostname2.yml

library/          # if any custom modules, put them here (optional)
module_utils/     # if any custom module_utils to support modules, put them here (optional)
filter_plugins/   # if any custom filter plugins, put them here (optional)

site.yml          # master playbook
webservers.yml    # playbook for webserver tier
dbservers.yml     # playbook for dbserver tier
roles             # (...)

```
---
## Struktura plików
### Wypracowany kompromis
``` ls
.
├── ansible.cfg                // konfiguracja
├── hosts.yml                  // Host inventories
├── group_vars                 // Group Vars (dir)
│   ├── vault                  // Secret group_vars (dir)
├── host_vars                  // Host host_vars (dir)
│   ├── vault                  // Secret host_vars (dir)
├── playbooks                  // Playbook (dir)
│   ├── task1.yml   
│   └── task2.yml
└──  roles                      // Rules (dir)


```
---
## Pętle
``` yaml
- name: remove the nginx and php-fpm package
  package:
    name: "{{ item }}"
    state: absent
  with_items:
    - nginx
    - php7.0-fpm
    - percona-server-server-5.6
```
--
## można bez pętli
``` yaml
- name: remove the nginx and php-fpm package
  apt:
    name:
      - nginx
      - php7.0-fpm
      - percona-server-server-5.6
    state: absent
```
???
Jak już mówiłem apt jest bardziej elastyczne
---

## Pliki: file
``` yaml
- name:      Change file ownership, group and permissions
  file:
    path:    /etc/foo.conf
    owner:   foo
    group:   foo
    mode:    '0644'
```
--
``` yaml
- name:      Workaround h.264 Opera problem
  file:
     path:   "/usr/lib/x86_64-linux-gnu/opera/libffmpeg.so"
     src:    "/usr/lib/chromium-browser/libffmpeg.so"
     state:  link
     force:  true
  tags:
     - h264
```
--
```yaml
- name:      Recursively change ownership of a directory
  file:
    path:    /etc/foo
    state:   directory
    recurse: yes
    owner:   foo
    group:   foo
```
??? tworzymy pliki zmieniamy uprawnienia
---
## Pliki: copy
``` yaml
- name:         Copy a new "ntp.conf and backing up the original
  copy:
    src:        /mine/ntp.conf
    dest:       /etc/ntp.conf
    owner:      root
    group:      root
    mode:       '0644'
    backup:     yes
```
--
``` yaml
- name:         Copy a new "sudoers" file, after passing validation
  copy:
    src:        /mine/sudoers
    dest:       /etc/sudoers             # (*) Przypis na następnym slajdzie
    validate:   /usr/sbin/visudo -cf %s
```
--
``` yaml
- name:         Copy a "sudoers" file on the remote machine for editing
  copy:
    src:        /etc/sudoers
    dest:       /etc/sudoers.edit        # (*) Przypis na następnym slajdzie
    remote_src: yes
    validate:   /usr/sbin/visudo -cf %s
```
---
## ( * ) Visudo i /etc/sudoers

``` shell
# file: /etc/sudoers
(...)
# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
```
``` bash
# This will cause sudo to read and parse any files in the /etc/sudoers.d
# directory that do not end in '~' or contain a '.' character.
```
--
``` yaml
- name:           make sudo
  template:       
    src:          root.j2
    dest:         "/etc/sudoers.d/{{ item | regex_replace('\.', '_') }}"
    mode:         0440
    owner:        root
    group:        root
  with_items:     "{{ users.sudoers }}"
```
``` j2
# {{ ansible_managed }}
# file: root.j2
{{item}} ALL=(ALL) NOPASSWD: ALL

```
---
## Pliki: szablony
### czyli parę słów o Jinja2

``` yaml
vars:
  firewall:
    other_in:
      - { name: 'MySQL for x', port: 3306, addr: 100.200.300.400 }
      - { name: 'SSH for y',   port: 22,   addr: 100.200.300.400 }
```
``` j2
# {{ ansible_managed }}

{% for other in firewall.other_in %}
# {{ other.name }}
proto tcp dport ( {{ other.port }} ) saddr ({{ other.addr }}) ACCEPT;
{% endfor %}

```
--

``` j2
# {{ ansible_managed }}
# file: /etc/proftpd/ftpd.passwd
{% for user in ftp.users %}
{{user.name}}:{{ user.pass |password_hash('sha512') }}:{{user.uid}}:{{user.gid}}::{{user.home}}:/bin/false
{% endfor %}
```

---
## Zmienne w playbookach
`./task3.yml`
``` yml
---
- hosts:       demo
  vars:       demo.hyski.pl
    hostname:
  tasks:
  - name:     Change hostname
    hostname:
      name:   "{{hostname}}"
```
???
- czemu tego nie używać ?
---
### host_vars
``` yaml
# file: host_vars/demo.yaml
users:
  sudoers:
    - adam.hyski
    - anna.nowak
    - jan.kowalski
```
--
``` yaml
- name:           make sudo
  template:       
    src:          root.j2
    dest:         "/etc/sudoers.d/{{ item | regex_replace('\.', '_') }}"
    mode:         0440
    owner:        root
    group:        root
  with_items:     "{{ users.sudoers }}"
```
---
### group_vars
``` yaml
---
# file: hosts.yaml
all:
  hosts:
    demo:
  children:
    webserwers:
    docker:
      hosts:
        demo:
```
``` yaml
---
# file: group_vars/docke.yaml
is_docker: "true"

```
``` yaml
---
- hosts:         all
  tasks:
  - name:       Install pip packages
    pip:
      name:     docker-compose
      state:    present
    when:       is_docker     # proszę tego przykładu tak nie używać
```
---
## vault
``` shell
$ openssl rand -base64 18 | sed 's/\//_/' >  ~/.ansible_valt
```
``` shell
$ ansible-vault create --vault-id ~/.ansible_valt ./vault/demo.yml
```
```  shell
$ ansible-vault edit --vault-id ~/.ansible_valt ./vault/demo.yml
```
--
#### aliasy
``` bash
export ANSIBLE_VAULT_PASSWORD_FILE= ~/.ansible_valt
alias ave='ansible-vault edit --vault-id ~/.ansible_valt'
alias avc='ansible-vault create --vault-id ~/.ansible_valt'
alias av='ansible-vault --vault-id ~/.ansible_valt'
```
#### gui?
``` bash
$ EDITOR='kate' ansible-vault create --vault-id ~/.ansible_valt ./vault/demo.yml
$ EDITOR='geany' ave ./vault/demo.yml     
```
---
## Vault używanie

``` yaml
# file: playbooks/task4.yml
# Add a user to a password file and ensure permissions are set
- htpasswd:
    path:     /etc/passwdfile
    name:     "{{auth.user}}"
    password: "{{auth.user}}"
    owner:    root
    group:    www-data
    mode:     0640
```
``` yaml
---
# file: host_vars/demo.yml
auth:
  user:   janedoe

```
``` shell
# file: host_vars/vault/demo.yml
$ANSIBLE_VAULT;1.1;AES256
36653934383363313435333666313664303436313762353963653535343163383963666131373436
3134303466633833643465316334373937663734396632620a666137316363633332383330316537
36343564366362316665373962323966346330333338373935353665306635323561666530663333
3235663039323065640a396262313362643162656533666134626635353361316431663763326137
33333530373035636264653139326264376262613933623930346465636530316532623539303334
3661636536306538613738636663306266613731623331373630

```

---
## Tagi
``` yaml
- name:       Install pip packages
  pip:
    name:     docker-compose
    state:    present
  tags:
    - docker
    - install
- apt:
    name:     htop
    state:    present
  tags:
    - tools
    - install
```
Użycie:

``` shell
$ ansible-playbook install.yml --tag tools
$ ansible-playbook install.yml --tag install
$ ansible-playbook install.yml --tag docker
```
---

class: center, middle

# Role
---
# Struktura plików
Gwoli przypomnienia
``` shell
.
├── ansible.cfg                // konfiguracja
├── hosts.yml                  // Host inventories
├── group_vars                 // Group Vars (dir)
│   ├── vault                  // Secret group_vars (dir)
├── host_vars                  // Host host_vars (dir)
│   ├── vault                  // Secret host_vars (dir)
├── playbooks                  // Playbook (dir)
│   ├── task1.yml   
│   └── task2.yml
└──  roles                      // Rules (dir)

```
---
# Struktura plików ról
``` shell
common/             # this hierarchy represents a "role"
  tasks/            #
      main.yml      #  <-- tasks file can include smaller files if warranted
  handlers/         #
      main.yml      #  <-- handlers file
  templates/        #  <-- files for use with the template resource
      ntp.conf.j2   #  <------- templates end in .j2
  files/            #
      bar.txt       #  <-- files for use with the copy resource
      foo.sh        #  <-- script files for use with the script resource
  vars/             #
      main.yml      #  <-- variables associated with this role
  defaults/         #
      main.yml      #  <-- default lower priority variables for this role
  meta/             #
      main.yml      #  <-- role dependencies
  library/          # roles can also include custom modules
  module_utils/     # roles can also include custom module_utils
  lookup_plugins/   # or other types of plugins, like lookup in this case

```
---
## Handlers
przykładowa rola:
``` ansible
---
# file: roles/common/tasks/main.yml

- name: be sure ntp is installed
  yum:
    name: ntp
    state: present
  tags: ntp

- name: be sure ntp is configured
  template:
    src: ntp.conf.j2
    dest: /etc/ntp.conf
  notify:                     # <-- pojawia się coś nowego
    - restart ntpd
  tags: ntp

- name: be sure ntpd is running and enabled
  service:
    name: ntpd
    state: started
    enabled: yes
  tags: ntp
```
???
Proszę zwrócić uwagę na
notify:
    - restart ntpd
---
## Handlers cd.
plik z handlerem
``` ansible
---
# file: roles/common/handlers/main.yml
- name: restart ntpd
  service:
    name: ntpd
    state: restarted
```
???
- kolejność działania
- kolejność działania handlerów
---

class: center, middle

# Pytania?
---

class: center, middle

# Gdzie prezentacja
---

class: center, middle

# Dziękuję
