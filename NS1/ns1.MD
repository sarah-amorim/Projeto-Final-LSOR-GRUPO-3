No projeto, o ns1 (abreviação de Name Server 1) é a máquina virtual (VM1, com IP .2 na sub-rede do grupo) configurada com o software BIND9 para atuar como o servidor DNS primário da rede. Ele é o servidor central responsável por traduzir os nomes das máquinas em endereços IP e vice-versa para todas as outras VMs do grupo, permitindo que os serviços se comuniquem pelo nome em vez de dependerem de IPs decorados ou de arquivos /etc/hosts locais.

## Configurações aplicadas na VM

### Definição do Hostname
O hostname é o nome único que identifica uma máquina na rede. No projeto, ele é configurado como o FQDN (Fully Qualified Domain Name ou Nome de Domínio Totalmente Qualificado), que une o nome do serviço ao domínio do grupo, por exemplo, ```ns1.grupo0.bsi-26-2.maceio.lab```. No Projeto Final, cada máquina assume o nome do serviço que hospeda (gw, ns1, ns2, www, bd, arquivos, ldap e cliente). Essa padronização é fundamental porque facilita a administração e o diagnóstico de problemas durante o trabalho em grupo, garante que os logs, terminais e serviços identifiquem corretamente a origem das requisições, e alinha a identidade própria de cada servidor com os registros configurados no DNS e com os serviços que dependem de nomes para funcionar (como o cliente SSSD buscando o servidor ldap, o Apache aplicando virtual hosts ou a aplicação web conectando-se ao bd).
```
sudo hostnamectl set-hostname ns1.grupo3.bsi-26-2.maceio.lab
sudo nano /etc/hosts
```
No /etc/hosts deixe só a própria VM:
```
127.0.0.1   localhost
127.0.1.1   ns1.grupo3.bsi-26-2.maceio.lab ns1
```

### Instalação e configuração do BIND
O BIND (sigla para Berkeley Internet Name Domain) é o software de servidor DNS mais utilizado na internet para traduzir nomes de domínio em endereços IP e vice-versa, originalmente criado na Universidade da Califórnia em Berkeley e atualmente mantido pelo ISC (Internet Systems Consortium).

```
sudo apt-get update
sudo apt-get install -y bind9 bind9-utils dnsutils
```
O arquivo /etc/bind/named.conf.options define o comportamento global e as regras de segurança do servidor DNS BIND9, estabelecendo quem pode fazer consultas, como resolver nomes externos e onde armazenar dados temporários.
```
acl grupo { 10.26.2.115; 10.26.2.116; 10.26.2.117; 10.26.2.118;
            10.26.2.119; 10.26.2.120; 10.26.2.121; localhost; };

options {
    directory "/var/cache/bind";
    listen-on-v6 { none; };

    allow-query     { grupo; };
    allow-recursion { grupo; };
    allow-transfer  { none; };

    forwarders { 172.20.0.10; 172.20.0.11; };
    dnssec-validation no;
};
```
O arquivo /etc/bind/named.conf.local é utilizado no BIND9 para declarar as zonas locais pelas quais o servidor DNS é responsável de forma autoritativa, informando onde estão armazenados os registros e como ocorre a sincronização com o servidor secundário (ns2).
```
zone "grupo3.bsi-26-2.maceio.lab" {
    type primary;
    file "/etc/bind/db.grupo3";
    allow-transfer { 10.26.2.116; };
    also-notify    { 10.26.2.116; };
};

zone "2.26.10.in-addr.arpa" {
    type primary;
    file "/etc/bind/db.10.26.2";
    allow-transfer { 10.26.2.116; };
    also-notify    { 10.26.2.116; };
};
```
O arquivo /etc/bind/db.grupo3 é o arquivo de zona direta do Grupo 3, responsável por mapear cada nome de máquina do domínio para o seu respectivo endereço IPv4.
```
$TTL 3600
@       IN  SOA ns1.grupo3.bsi-26-2.maceio.lab. administrador.grupo3.bsi-26-2.maceio.lab. (
                2026112201  ; serial (AAAAMMDDnn): aumente a cada alteração
                3600        ; refresh
                900         ; retry
                604800      ; expire
                300 )       ; TTL negativo

        IN  NS      ns1.grupo3.bsi-26-2.maceio.lab.
        IN  NS      ns2.grupo3.bsi-26-2.maceio.lab.

ns1      IN  A   10.26.2.115
ns2      IN  A   10.26.2.116
www      IN  A   10.26.2.117
bd       IN  A   10.26.2.118
arquivos IN  A   10.26.2.119
ldap     IN  A   10.26.2.120
cliente  IN  A   10.26.2.121
```
O arquivo /etc/bind/db.10.26.2 é o arquivo de zona reversa do Grupo 3, responsável por realizar o caminho oposto ao da zona direta: traduzir um endereço IPv4 de volta para o seu nome de domínio completo (FQDN), algo essencial para auditoria de logs, verificação de segurança de serviços como SSH e e-mail, e testes de diagnóstico com comandos como dig -x.
```
$TTL 3600
@       IN  SOA ns1.grupo3.bsi-26-2.maceio.lab. administrador.grupo3.bsi-26-2.maceio.lab. (
                2026112201 3600 900 604800 300 )

        IN  NS      ns1.grupo3.bsi-26-2.maceio.lab.
        IN  NS      ns2.grupo3.bsi-26-2.maceio.lab.

115     IN  PTR     ns1.grupo3.bsi-26-2.maceio.lab.
116     IN  PTR     ns2.grupo3.bsi-26-2.maceio.lab.
117     IN  PTR     www.grupo3.bsi-26-2.maceio.lab.
118     IN  PTR     bd.grupo3.bsi-26-2.maceio.lab.
119     IN  PTR     arquivos.grupo3.bsi-26-2.maceio.lab.
120     IN  PTR     ldap.grupo3.bsi-26-2.maceio.lab.
121     IN  PTR     cliente.grupo3.bsi-26-2.maceio.lab.
```
### Validação
Essa sequência de comandos serve para testar a sintaxe das configurações, aplicar as alterações e confirmar se o servidor DNS primário (ns1) está respondendo corretamente antes de avançar para a configuração do servidor secundário (ns2).
```
sudo named-checkconf
sudo named-checkzone grupo3.bsi-26-2.maceio.lab /etc/bind/db.grupo3
sudo named-checkzone 2.26.10.in-addr.arpa /etc/bind/db.10.26.2
sudo systemctl restart named
sudo systemctl status named --no-pager
dig @127.0.0.1 www.grupo3.bsi-26-2.maceio.lab +short     # 10.26.2.117
```
