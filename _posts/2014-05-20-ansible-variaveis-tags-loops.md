---
layout: post
title: "Ansible - variáveis, tags e loops"
description: "Utilize os recursos de variáveis, tags e loops no Ansible"
category: devops
keywords: server, linux, unix, services, servidor, ansible, gerenciamento, ssh, provisionamento, devops
---

Este é um complemento do post [anterior](http://infoslack.com/server/automatize-o-gerenciamento-de-servidores-com-ansible/) sobre Ansible, veremos um pouco
sobre o uso de variáveis, tags e implementação de loops.

### Utilizando variáveis

Com o uso de [variáveis](http://docs.ansible.com/playbooks_variables.html), suas receitas de ansible podem ser configuradas em um só
lugar. No exemplo a seguir temos um playbook para instalar o ruby, vamos implementar o
uso de variáveis:

```yaml
---
- name: Add the key used to Ruby pkg
  apt_key: url=http://apt.hellobits.com/hellobits.key state=present

- name: Add repos for Ruby install
  copy: src=hellobits.list dest=/etc/apt/sources.list.d/

- name: Install Ruby
  apt: name=ruby-2.1 update_cache=yes

- name: Install bundler
  command: gem install bundler --no-rdoc --no-ri
```

Na instalação do ruby queremos escolher o nosso repositório e fornecer a  url
para o `apt_key`, além disso queremos escolher a versão que será instalada.

Seguindo a organização da nossa configuração do ansible, criaremos um
diretório chamado `group_vars` e dentro dele um arquivo chamado `all`:

```bash
$ mkdir group_vars
$ touch group_vars/all
```

A estrutura deverá ficar parecida com a seguinte:

```bash
├── hosts
├── group_vars
│   └── all
├── roles
│   ├── nginx
│   │   └── tasks
│   │       └── main.yml
│   └── ruby
│       └── tasks
│           └── main.yml
└── server.yml
```

Agora é possível atribuir valores a variáveis no arquivo `all`:

```yaml
---
# Ruby variables
ruby_url: http://apt.hellobits.com/hellobits.key
ruby_version: ruby-2.1
```

Agora que temos as duas variáveis com os valores atribuídos, podemos chamá-las
no playbook de instalação do ruby:

{% gist infoslack/552c811e71acae4c48cd %}

Dessa forma agora é possível alterar os valores das variáveis de vários
playbooks em um só lugar: `all`.

### Utilizando tags

Outro recurso interessante do ansible são as [tags](http://docs.ansible.com/playbooks_tags.html), com elas é possível
executar tarefas de forma isolada ou ignorá-las. Ainda no playbook de
instalação do ruby podemos adicionar uma tag e ver como funciona:

{% gist infoslack/2a82f4ccb78489cb4e7b %}

Agora podemos executar somente a tarefa de instalação do ruby:

```bash
$ ansible-playbook -i hosts server.yml --tags "ruby"
```

Ou podemos executar todas as tarefas menos a de instalação do ruby:

```bash
$ ansible-playbook -i hosts server.yml --skip-tags "ruby"
```

### Implementando loops

Imagine que em um dos playbooks você precisa fazer a instalação de vários
pacotes, o recurso de [loops](http://docs.ansible.com/playbooks_loops.html) fornece a opção `with_items` onde podemos
listar o nome dos pacotes que queremos instalar:

{% gist infoslack/04f17edbf777fd07a940 %}

A variável `item` recebe um valor de `with_items` a cada iteração.

### Finalizando

Ansible é uma ferramenta incrível e vale investir um pouco de tempo para
conhecê-la melhor.

O [@fnando](https://twitter.com/fnando) vai falar um pouco sobre [Ansible no Howto](http://howtocode.com.br/cursos/ansible) e breve
creio que lance um curso sobre isso, fique ligado!

Happy Hacking ;)

### Referências

- [http://docs.ansible.com/playbooks_variables.html](http://docs.ansible.com/playbooks_variables.html)

- [http://docs.ansible.com/playbooks_tags.html](http://docs.ansible.com/playbooks_tags.html)

- [http://docs.ansible.com/playbooks_loops.html](http://docs.ansible.com/playbooks_loops.html)

- [https://github.com/infoslack/simple-ansible](https://github.com/infoslack/simple-ansible)
