Ambiente De desenvolvimento
Instale Python, ansible, Vagrant e VirtualBox

Estrutura de Loop:

    - name: 'Instala pacotes do sistema operacional'
      apt:
        name: '{{ item }}'
        state: latest
      become: yes
      with_items:
        - php5
        - apache2
        - libapache2-mod-php5

No playbook , pode-se referenciar all ou o grupo de maquinas:

- hosts: wordpress
  tasks:
    - name: 'Instala pacotes de dependencia do sistema operacional'
      apt: 
        name: ['php5', 'apache2', 'libapache2-mod-php5', 'php5-gd', 'libssh2-php', 'php5-mcrypt', 'mysql-server-5.6', 'python-mysqldb', 'php5-mysql']
        state: latest
      become: yes


*) O nome do arquivo em group_vars pode ser all ou o nome de um grupo do arquivo hosts
*) Uma variável não deve ter um ponto, espaço ou hífen no nome.