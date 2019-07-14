
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

```
]
---
.left-column[
### O mnie
### Agenda
]
.right-column[
- ### Czym jest Ansible?
- ### jak go używać //TODO
- ### jak nie używąć //TODO
]
---

class: center, middle

# Czym jest Ansible?


---
.left-column[
### Playbook
### Inwentarz
### Inwentarz w YAML
]

.right-column[
Najprostszy przykład playbooka:
`./task1.yml`
``` yaml
---
- host:       demo
  tasks:
  - name:     Change hostname
    hostname:
      name:   demo.hyski.pl

```
wymaga on jeszcze pliku `./hosts.ini`
``` ini
demo.hyski.pl
```

ale można lepiej `./hosts.yaml`
``` yaml
all:
  hosts:
    demo.hyski.pl
```
]
---
