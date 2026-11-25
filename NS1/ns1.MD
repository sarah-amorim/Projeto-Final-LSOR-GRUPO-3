# Configurações da máquina virtual ```ns1```
No projeto, o ns1 (abreviação de Name Server 1) é a máquina virtual configurada com o software BIND9 para atuar como o servidor DNS primário da rede. Ele é o servidor central responsável por traduzir os nomes das máquinas em endereços IP e vice-versa para todas as outras VMs do grupo, permitindo que os serviços se comuniquem pelo nome em vez de dependerem de IPs decorados ou de arquivos ```/etc/hosts``` locais.

### Definição do Hostname

O hostname é o nome único que identifica uma máquina na rede. No projeto, ele é configurado como o FQDN ( Domínio Totalmente Qualificado), que une o nome do serviço ao domínio do grupo, por exemplo, ```ns1.grupo0.bsi-26-2.maceio.lab```. No Projeto Final, cada máquina assume o nome do serviço que hospeda (gw, ns1, ns2, www, bd, arquivos, ldap e cliente). Essa padronização é fundamental porque facilita a administração e o diagnóstico de problemas durante o trabalho em grupo.

```
sudo hostnamectl set-hostname ns1.grupo3.bsi-26-2.maceio.lab
sudo nano /etc/hosts
```

No ```/etc/hosts``` deixe só a própria VM:

![Arquivo /etc/hosts](../imagens/etc-hosts.png)

### Instalação e configuração do BIND
O BIND (sigla para Berkeley Internet Name Domain) é o software de servidor DNS mais utilizado na internet para traduzir nomes de domínio em endereços IP e vice-versa, originalmente criado na Universidade da Califórnia em Berkeley e atualmente mantido pelo ISC (Internet Systems Consortium).

```
sudo apt-get update
sudo apt-get install -y bind9 bind9-utils dnsutils
```

![Confirmação de instalação do BIND](../imagens/bind-instalado.png)

O arquivo ```/etc/bind/named.conf.options``` define o comportamento global e as regras de segurança do servidor DNS BIND9, estabelecendo quem pode fazer consultas, como resolver nomes externos e onde armazenar dados temporários.

![Arquivo /etc/bind/named.conf.options](../imagens/named_conf_options.png)

O arquivo ```/etc/bind/named.conf.local``` é utilizado no BIND9 para declarar as zonas locais pelas quais o servidor DNS é responsável de forma autoritativa, informando onde estão armazenados os registros e como ocorre a sincronização com o servidor secundário (ns2).

![Arquivo /etc/bind/named.conf.local](../imagens/named_conf_local.jpeg)

O arquivo ```/etc/bind/db.grupo3``` é o arquivo de zona direta do Grupo 3, responsável por mapear cada nome de máquina do domínio para o seu respectivo endereço IPv4.

![Arquivo /etc/bind/db.grupo3](../imagens/db-grupo3.jpeg)

O arquivo ```/etc/bind/db.192.168.26``` é o arquivo de zona reversa do Grupo 3, responsável por realizar o caminho oposto ao da zona direta: traduzir um endereço IPv4 de volta para o seu nome de domínio completo (FQDN), algo essencial para auditoria de logs, verificação de segurança de serviços como SSH e e-mail, e testes de diagnóstico com comandos como dig -x.

![Arquivo /etc/bind/db.10.26.2](../imagens/db_192_168_26.jpeg)

### Validação
Essa sequência de comandos serve para testar a sintaxe das configurações, aplicar as alterações e confirmar se o servidor DNS primário (ns1) está respondendo corretamente antes de avançar para a configuração do servidor secundário (ns2).
```
sudo named-checkconf
sudo named-checkzone grupo3.bsi-26-2.maceio.lab /etc/bind/db.grupo3
sudo named-checkzone 26.168.192.in-addr.arpa /etc/bind/db.192.168.26
sudo systemctl restart named
sudo systemctl enable named
sudo systemctl status named --no-pager
sudo ufw status
```
![Saída da validação](../imagens/validacao_saida.jpeg)


