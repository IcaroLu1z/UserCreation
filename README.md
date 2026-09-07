# UserCreation
Scripts Ansible para automação de criação de usuários no Laboratório de Computação de Sistemas Inteligentes (CSILab, UFOP).


O Ansible é um mecanismo de automação. Ele cuida do gerenciamento de configuração, implantação de aplicações, provisionamento em nuvem, execução de tarefas ad hoc, automação de rede e orquestração de múltiplos nós. Para o caso do CSILab, ele é usado para automatizar o gerenciamento de usuários.


O Ansible utiliza **Inventory** e  **Playbooks**. 
- **Inventory:** O Ansible automatiza tarefas em nós gerenciados ou "hosts" da infraestrutura usando uma lista ou grupo de listas conhecido como Inventory. 
- **Playbooks:** São blueprints de automação, no formato YAML, que o Ansible usa para implantar e configurar nós em um inventário.

Para executar os **playbooks**, utilize o seguinte comando na `bridge`:
```
ansible-playbook -i hosts.ini <playbook>.yml --ask-become-pass
```
A opção `--ask-become-pass` é necessária para passar a senha de administrador.

Também é possível limitar a execução do playbook para servidores específicos usando a opção `--limit`. Exemplo:
```
ansible-playbook -i hosts.ini create_user.yml --limit bridge,maeve --ask-become-pass
```

Também é possível executar o playbook em modo `--check`, em que o Ansible não faz nenhuma mudança, em vez disso, ele tenta prever as mudanças que podem ocorrer. Combinado com a opção `--diff`, o Ansible mostrará o que pode mudar e como. Ótimo para debug.
```
ansible-playbook -i hosts.ini create_user.yml --check --diff --ask-become-pass
```

Documentação do Ansible: [https://docs.ansible.com/projects/ansible/latest/index.html]

## - hosts.ini
Esse é o arquivo *Inventory* que define os nós presentes no laboratório. Nele temos definidos três grupos: `all_servers`, `pub_servers`, `ppg_servers`.
- O grupo `all_servers` define todos os servidores do laboratório (incluindo a `bridge` como localhost). 
- O grupo `pub_servers` define os servidores disponíveis para os usuários gerais do laboratório. 
- O grupo `ppg_servers` define os servidores disponíveis para os alunos do Programa de Pós-Graduação.
## - create_user.yml
Este é o playbook principal que implementa a criação de um novo usuário em ***todo*** cluster do laboratório. 
- **Uso:** Esse **playbook** deve ser usado quando um novo aluno ingressar no laboratório e ainda não existir um usuário referente a esse aluno.
- Este **playbook** implementa:
    - A criação do usuário em cada servidor com o próximo UID disponível na `bridge`.
    - A criação do grupo do usuário (GID) com o mesmo valor do UID.
    - A criação dos diretórios `/home/<user>` em cada servidor e `/media/work/<user>` na NAS.
    - A criação de uma senha alfanumérica aleatória de 12 caracteres para o usuário.
    - Configuração do `~/.nanorc` e `~/.bashrc`.
    - Criação de uma chave SSH para o usuário e sua propagação para os servidores.
    - A inclusão do usuário no grupo `docker` para permitir a execução de containers.

## - copy_user.yml
Este playbook implementa o mecanismo de cópia de um usuário existente na `bridge` para um servidor em que ele ainda não exista.
- **Uso:** Esse **playbook** deve ser usado quando o usuário, por algum motivo, existir na bridge, mas não em um servidor especifico. Exemplo: Na época que o usuário foi criado o servidor `maeve` estava desconectado da rede.
- Este **playbook** implementa:
    - A leitura do UID e GID do usuário na `bridge`, a criação do usuário no servidor alvo usando esses valores
    - A leitura do hash da senha do usuário presente na `bridge` e a cópia do hash para o servidor alvo.
    - Configuração de permissões para o diretório `/media/work/<user>` na NAS.
    - Configuração do `~/.nanorc` e `~/.bashrc`.
    - Propagação da chave SSH para o servidor alvo.
    - A inclusão do usuário no grupo `docker`.

## - sync_users.yml + create_missing_user.yml
Estes playbook implementam o mecanismo de sincronização (cópia) de múltiplos usuários que existem na bridge, mas não nos servidores alvos. O playbook está dividido em dois arquivos: **create_missing_user.yml** implementa a cópia do usuário e o **sync_users.yml** chama **create_missing_user.yml** para cada usuário em cada servidor alvo.
- **Uso:** Esse **playbook** deve ser usado quando múltiplos usuários, por algum motivo, existem na bridge, mas não em servidores específicos. Exemplo: O laboratório adquiriu um novo servidor.
- Este **playbook** implementa:
    - A leitura do UID e GID dos usuários na `bridge`, e a criação do usuários nos servidores alvos.
    - A leitura dos hashes da senha dos usuários presentes na `bridge` e a cópia dos hashes para os servidores alvos.
    - Configuração de permissões para o diretório `/media/work/<user>` na NAS.
    - Configuração dos `~/.nanorc` e `~/.bashrc`.
    - Propagação das chaves SSH para os servidores alvos.
    - A inclusão dos usuários no grupo `docker`.

## - remove_user.yml
Este playbook implementa a remoção de usuários do cluster do laboratório.
- **Uso:** Esse **playbook** deve ser usado quando o aluno, por algum motivo, não pertencer mais ao laboratório.
- Este **playbook** implementa:
    - A remoção do usuário de cada servidor alvo.
    - A remoção dos diretórios `/home/<user>` em cada servidor e `/media/work/<user>` na NAS, se desejado.