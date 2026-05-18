===============================================================================
                       PROJETO 01 - VAGRANT E ANSIBLE
                    DISCIPLINA: ADMINISTRAÇÃO DE SISTEMAS ABERTOS
                    PROFESSOR: LEONIDAS LIMA | PERÍODO: 2026.1
         INSTITUTO FEDERAL DA PARAÍBA (IFPB) - CAMPUS JOÃO PESSOA
===============================================================================

-------------------------------------------------------------------------------
1. INTEGRANTES DA EQUIPE
-------------------------------------------------------------------------------
* Augusto Emanuel Silva Pereira   - Matricula: 20242380021
* Sarah Ferreira Fernandes        - Matricula: 20242380031

-------------------------------------------------------------------------------
2. DESCRIÇÃO DO PROJETO
-------------------------------------------------------------------------------
Este projeto consiste na implementação e orquestração automatizada de uma 
infraestrutura de rede local completa utilizando Infraestrutura como Código (IaC). 
Através do Vagrant (provedor VirtualBox), são provisionadas 4 máquinas virtuais 
rodando Debian 12 (Bookworm). Toda a configuração do sistema operacional, 
segurança, rede e serviços essenciais é realizada de forma 100% automatizada 
utilizando Ansible Playbooks.

-------------------------------------------------------------------------------
3. ARQUITETURA DA INFRAESTRUTURA
-------------------------------------------------------------------------------
A rede é composta por quatro nós interconectados em uma rede privada isolada
(192.168.56.0/24), com o servidor DHCP interno do VirtualBox desativado para 
permitir o controle total da atribuição de endereçamento:

1. Servidor de Arquivos (arq.emanuel.sarah.devops)
   - IP Fixo: 192.168.56.121
   - Papel: Servidor central de armazenamento (LVM), Servidor NFS, Servidor 
     DHCP da rede e Servidor DNS Autoritativo (Bind9).
   - Hardware Adicional: 3 discos rígidos virtuais extras de 10 GB cada.

2. Servidor de Banco de Dados (db.emanuel.sarah.devops)
   - IP DHCP Fixo (Reserva por MAC): 192.168.56.131
   - Papel: Hospedar o SGDB MariaDB Server e realizar montagem dinâmica de 
     volume via Autofs/NFS.

3. Servidor de Aplicação Web (app.emanuel.sarah.devops)
   - IP DHCP Fixo (Reserva por MAC): 192.168.56.111
   - Papel: Hospedar o Servidor HTTP Apache com uma página institucional 
     dinâmica e montagem Autofs/NFS.

4. Host Cliente (cli.emanuel.sarah.devops)
   - IP Dinâmico (Escopo DHCP): Atribuído dinamicamente na faixa 192.168.56.50 
     a 192.168.56.100.
   - Papel: Estação de trabalho do usuário configurada com suporte a interface 
     gráfica remota via tunelamento SSH (X11 Forwarding) e navegador 
     Firefox ESR instalado.

-------------------------------------------------------------------------------
4. TECNOLOGIAS E SERVIÇOS IMPLEMENTADOS
-------------------------------------------------------------------------------
* Vagrant & VirtualBox: Provisionamento de máquinas eficientes usando clones 
  conectados (linked_clones) e desativação de DHCP padrão via gatilhos (triggers).
* LVM (Logical Volume Manager): Agregação dos 3 discos de 10 GB no Volume Group 
  "dados" e criação de um Logical Volume "ifpb" de 15 GB formatado em ext4 e 
  montado em /dados.
* NFS & Autofs: Compartilhamento do diretório /dados/nfs do servidor arq para 
  as demais VMs da rede, com montagem automática sob demanda em /var/nfs via autofs.
* ISC-DHCP-Server: Serviço ativo no arq configurado com escopo dinâmico e 
  mapeamento estático de endereços IP baseado no endereço MAC das interfaces.
* Bind9 (DNS Autoritativo): Resolução de nomes para o domínio customizado 
  emanuel.sarah.devops com zonas diretas e reversas configuradas.
* Segurança SSH & Sistema:
  - Sincronização de horário através do chrony apontando para o pool.ntp.br.
  - Fuso horário definido para America/Recife.
  - Criação do grupo local "ifpb" e contas de usuários customizadas.
  - Restrição estrita de acesso SSH via chaves criptográficas públicas.
  - Banner de segurança obrigatório exibido antes de qualquer autenticação SSH.
  - Configuração de privilégios administrativos via /etc/sudoers sem senha.

-------------------------------------------------------------------------------
5. COMO EXECUTAR O PROJETO
-------------------------------------------------------------------------------
Passo 1: Certifique-se de possuir o VirtualBox, Vagrant e Ansible instalados.
Passo 2: No diretório do projeto, suba a infraestrutura executando:
   $ vagrant up

Passo 3: Caso precise reconfigurar ou auditar os ambientes:
   $ ansible-playbook playbooks/config.yml
