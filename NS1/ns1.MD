Defina o hostname de cada VM. Exemplo no ns1:
```
sudo hostnamectl set-hostname ns1.grupo3.bsi-26-2.maceio.lab
sudo nano /etc/hosts
```
No /etc/hosts deixe só a própria VM:
```
127.0.0.1   localhost
127.0.1.1   ns1.grupo3.bsi-26-2.maceio.lab ns1
```
Pacotes
```
sudo apt-get update
sudo apt-get install -y bind9 bind9-utils dnsutils
```
Opções: /etc/bind/named.conf.options
```

```
Zonas: /etc/bind/named.conf.local
```

```
Zona direta: /etc/bind/db.grupo3
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
Zona reversa: /etc/bind/db.10.26.2
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
Validar e subir
```
sudo named-checkconf
sudo named-checkzone grupo3.bsi-26-2.maceio.lab /etc/bind/db.grupo3
sudo named-checkzone 2.26.10.in-addr.arpa /etc/bind/db.10.26.2
sudo systemctl restart named
sudo systemctl status named --no-pager
dig @127.0.0.1 www.grupo3.bsi-26-2.maceio.lab +short     # 10.26.2.117
```