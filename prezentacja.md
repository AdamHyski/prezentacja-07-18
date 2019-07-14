
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

# Przykład kodu
Najprosszy przykład taska:
`task1.yml`
``` yaml
---
- host:       Demo
  tasks:
  - name:     Change hostname
    hostname:
      name:   demo.hyski.pl

```

---
