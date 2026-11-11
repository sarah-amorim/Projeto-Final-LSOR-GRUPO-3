# Projeto Final LSOR 

**INSTITUTO FEDERAL DE EDUCAÇÃO, CIÊNCIA E TECNOLOGIA DE ALAGOAS (IFAL) — Campus Maceió**
**Curso:** Bacharelado em Sistemas de Informação (BSI)
**Disciplina:** Laboratório de Sistemas Operacionais e Redes (LSOR) — 2026.2
**Professor:** Alaelson de Castro Jatobá Neto
**Turma:** bsi-26-2 · **Grupo:** 3
**Rede do grupo:** `192.168.26.48/28` · **Domínio:** `grupo3.bsi-26-2.maceio.lab`
**Apresentação:** 02/12/2026 · **Data deste relatório:** 11/11/2026

| Integrante | Matrícula | Usuário (LDAP) | uidNumber |
|:--|:--|:--|:--:|
| Sarah Amorim dos Santos | 2024014201 | sarah.amorim | 10001 |
| Pedro Henrique Cavalcante Rocha | 2023007730 | pedro.rocha | 10002 |
| Nome Completo 3 | 0000000000 | nome.sobrenome | 10003 |
| Nome Completo 4 | 0000000000 | nome.sobrenome | 10004 |

> Nenhuma senha, chave privada ou *hash* `{SSHA}` é publicada neste repositório.

---

## 1. Apresentação

Na Etapa 1, o grupo montou uma rede privada com IPs fixos, nomes no domínio do grupo e um gateway NAT. No Projeto Final, essa rede vira um ambiente de serviços completo, como o de uma pequena empresa, com **login unificado**: cada integrante tem uma única conta e senha, guardadas no LDAP, usadas em todas as VMs e serviços.

Nesta fase, o projeto roda no **VirtualBox do laboratório**, reaproveitando as 9 VMs da Etapa 1 (placas em Bridge).

## 2. Objetivo

Transformar a rede da Etapa 1 em um ambiente de serviços com login unificado, de modo que:

- o `gw` faça roteamento, NAT e **firewall** (`FORWARD` com política `DROP`);
- o DNS responda pelo domínio do grupo, com servidor **primário e secundário** e zona direta e reversa;
- o **OpenLDAP** guarde usuários e grupos POSIX, e as 8 VMs autentiquem nele via **SSSD**;
- o servidor de arquivos publique as mesmas pastas por **Samba e SFTP**, com permissões por grupo;
- o servidor web (Apache + PHP) e o banco (MariaDB) formem uma aplicação em duas camadas, com área `/admin` autenticada no LDAP;
- cada serviço seja **testado e documentado com evidências**.

**Escopo de 2026.2**

| Obrigatório | Bônus |
|:--|:--|
| Gateway (NAT + firewall), DNS primário e secundário, LDAP + login unificado (SSSD), arquivos (Samba e SFTP), web e banco | TLS no LDAP (StartTLS) e e-mail (Postfix + Dovecot) |

## 3. Topologia

![Topologia lógica com os serviços](img/topologia-servicos.png)
![Login unificado com LDAP](img/login-unificado.png)

Diagrama editável: <link do Lucid ou arquivo>

## 4. Endereçamento, nomes e serviços

- Rede do grupo: `192.168.26.48/28` (255.255.255.240) · Broadcast: `192.168.26.63`
- Rota padrão das VMs: LAN do `gw` (`192.168.26.49`)
- Rede do laboratório (WAN do `gw`): `172.20.20.0/22`, gateway/DNS `172.20.20.1`

| VM da Etapa 1 | Serviço (hostname) | Interface | IP/máscara | FQDN |
|:--|:--|:--|:--|:--|
| Gateway (WAN) | `gw` | enp0s3 | 172.20.23.X/22 | gw.grupo3.bsi-26-2.maceio.lab |
| Gateway (LAN) | `gw` | enp0s8 | 192.168.26.49/28 | — |
| VM1 (g3-pc1-vm1) | `ns1` (DNS primário) | enp0s3 | 192.168.26.50/28 | ns1.grupo3.bsi-26-2.maceio.lab |
| VM2 (g3-pc1-vm2) | `ns2` (DNS secundário) | enp0s3 | 192.168.26.51/28 | ns2.grupo3.bsi-26-2.maceio.lab |
| VM3 (g3-pc2-vm1) | `www` (Apache + PHP + SFTP) | enp0s3 | 192.168.26.52/28 | www.grupo3.bsi-26-2.maceio.lab |
| VM4 (g3-pc2-vm2) | `bd` (MariaDB) | enp0s3 | 192.168.26.53/28 | bd.grupo3.bsi-26-2.maceio.lab |
| VM5 (g3-pc3-vm1) | `arquivos` (Samba + SFTP) | enp0s3 | 192.168.26.54/28 | arquivos.grupo3.bsi-26-2.maceio.lab |
| VM6 (g3-pc3-vm2) | `ldap` (OpenLDAP) | enp0s3 | 192.168.26.55/28 | ldap.grupo3.bsi-26-2.maceio.lab |
| VM7 (g3-pc4-vm1) | `mail` (bônus) | enp0s3 | 192.168.26.56/28 | mail.grupo3.bsi-26-2.maceio.lab |
| VM8 (g3-pc4-vm2) | `cliente` (testes na rede do grupo) | enp0s3 | 192.168.26.57/28 | cliente.grupo3.bsi-26-2.maceio.lab |

> Na migração para o servidor do laboratório, muda só o prefixo (`192.168.26.x` → `10.10.101.x`) e o nome das placas (`enp0s3`/`enp0s8` → `ens18`/`ens19`).

## 5. Hardware das VMs

| VM | Memória | vCPUs | Disco |
|:--|:--|:--|:--|
| gw | | | |
| ns1 | | | |
| ns2 | | | |
| www | | | |
| bd | | | |
| arquivos | | | |
| ldap | | | |
| mail | | | |
| cliente | | | |

Todas as placas em **Placa em Ponte (Bridge)**:

![Adaptadores de rede](img/virtualbox-adaptadores.png)

## 6. Preparação do ambiente

### 6.1 Renomear as VMs e revisar o Netplan

```bash
sudo hostnamectl set-hostname ns1.grupo3.bsi-26-2.maceio.lab
ls /etc/netplan/          # deve haver um único arquivo, com dhcp4: false
ip -br addr
ip route                  # deve haver uma única rota padrão
```

```text
<colar as saídas e comentar: quantos IPs e quantas rotas padrão aparecem?>
```

### 6.2 Remover as contas locais da Etapa 1

As contas dos integrantes criadas na Etapa 1 são removidas (mantendo o `administrador`), senão o Linux usa a conta local e ignora a do LDAP.

```bash
<comandos usados>
getent passwd | grep -c <usuário>
```

```text
<colar a saída>
```

### 6.3 Snapshots

Snapshot de cada VM antes de configurar cada serviço: <registrar print ou lista>.

---

## 7. Gateway — `gw`

**Requisitos:** `ip_forward` persistente, NAT só para a rede do grupo e firewall em `FORWARD` com política `DROP`.

```bash
sudo sysctl net.ipv4.ip_forward
sudo iptables -t nat -A POSTROUTING -s 192.168.26.48/28 -o enp0s3 -j MASQUERADE
sudo iptables -P FORWARD DROP
sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.26.48/28 -j ACCEPT
# <regras de acesso a partir do PC do laboratório (DNAT) — ver Aula 10>
sudo netfilter-persistent save
```

| Regra | Origem | Destino | Portas |
|:--|:--|:--|:--|
| Respostas | qualquer | qualquer | `ESTABLISHED,RELATED` |
| Saída do grupo | 192.168.26.48/28 | qualquer | todas |
| <acesso do PC do lab> | | | |

Verificação:

```bash
sudo iptables -t nat -L -n -v
sudo iptables -L -n -v
```

```text
<colar as saídas>
```

**O que a saída comprova:** <1 ou 2 frases>

## 8. DNS — `ns1` (primário) e `ns2` (secundário)

### 8.1 `ns1`: opções e zonas

`/etc/bind/named.conf.options`:

```text
acl grupo { 192.168.26.48/28; localhost; };
acl turma { 192.168.26.0/24; };
options {
    directory "/var/cache/bind";
    allow-query     { grupo; turma; };
    allow-recursion { grupo; };
    allow-transfer  { 192.168.26.51; };
    also-notify     { 192.168.26.51; };
    forwarders      { 172.20.20.1; };
    dnssec-validation no;
};
```

`/etc/bind/named.conf.local`:

```text
zone "grupo3.bsi-26-2.maceio.lab" {
    type primary;
    file "/etc/bind/db.grupo3";
    allow-transfer { 192.168.26.51; };
    also-notify    { 192.168.26.51; };
};
zone "26.168.192.in-addr.arpa" {      // confirmar o nome da zona reversa
    type primary;
    file "/etc/bind/db.192.168.26";
};
```

### 8.2 Arquivos de zona

```text
<colar db.grupo3: SOA, NS ns1/ns2, MX mail, um A por VM>
<colar db.192.168.26: um PTR por VM>
```

### 8.3 `ns2`: zonas secundárias

```text
<colar named.conf.local do ns2 com type secondary e masters { 192.168.26.50; }>
```

### 8.4 Validação

```bash
sudo named-checkconf
sudo named-checkzone grupo3.bsi-26-2.maceio.lab /etc/bind/db.grupo3
sudo named-checkzone 26.168.192.in-addr.arpa /etc/bind/db.192.168.26
sudo rndc reload
```

```text
<colar as saídas>
```

### 8.5 DNS de todas as VMs apontando para ns1 e ns2

```yaml
      nameservers:
        search: [grupo3.bsi-26-2.maceio.lab]
        addresses: [192.168.26.50, 192.168.26.51]
```

```bash
resolvectl status
```

```text
<colar a saída>
```

## 9. LDAP — `ldap`

Base: `dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab`

```bash
sudo apt-get install -y slapd ldap-utils
sudo dpkg-reconfigure slapd      # domínio: grupo3.bsi-26-2.maceio.lab
sudo slapcat | grep ^dn
```

```text
<colar a saída>
```

### 9.1 Estrutura, grupos e usuários

| Grupo | gidNumber | Membros | Uso |
|:--|:--:|:--|:--|
| `lsor-g3` | 10000 | todos os integrantes | `sudo`, pasta do grupo, site e `/admin` |
| `sftpusers` | 10001 | `novato` | só SFTP no `arquivos` |

LDIF usado (**sem** o campo `userPassword`):

```ldif
dn: ou=usuarios,dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab
objectClass: organizationalUnit
ou: usuarios

dn: ou=grupos,dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab
objectClass: organizationalUnit
ou: grupos

dn: cn=lsor-g3,ou=grupos,dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab
objectClass: posixGroup
cn: lsor-g3
gidNumber: 10000
memberUid: nome.sobrenome
# ... um memberUid por integrante

dn: uid=nome.sobrenome,ou=usuarios,dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: nome.sobrenome
cn: Nome Completo
sn: Sobrenome
mail: nome.sobrenome@grupo3.bsi-26-2.maceio.lab
uidNumber: 10001
gidNumber: 10000
homeDirectory: /home/nome.sobrenome
loginShell: /bin/bash
# userPassword omitido (gerado com slappasswd)
```

```bash
ldapadd -x -D cn=admin,dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab -W -f base.ldif
ldapsearch -x -LLL -H ldap://ldap.grupo3.bsi-26-2.maceio.lab \
  -b dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab "(objectClass=posixAccount)" uid uidNumber gidNumber
```

```text
<colar as saídas>
```

### 9.2 TLS (StartTLS) — bônus

- [ ] CA do grupo e certificado para `ldap.grupo3.bsi-26-2.maceio.lab`
- [ ] `olcTLSCACertificateFile`, `olcTLSCertificateFile` e `olcTLSCertificateKeyFile` em `cn=config`
- [ ] `ca-grupo3.crt` nas 8 VMs + `update-ca-certificates`

```bash
ldapsearch -x -ZZ -H ldap://ldap.grupo3.bsi-26-2.maceio.lab \
  -b dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab "(objectClass=posixAccount)" uid
```

> A chave da CA não vai para o GitHub.

## 10. Login unificado nas 8 VMs (SSSD)

```bash
sudo apt-get install -y sssd-ldap libnss-sss libpam-sss ldap-utils
sudo nano /etc/sssd/sssd.conf
sudo chmod 600 /etc/sssd/sssd.conf
sudo systemctl restart sssd
sudo pam-auth-update --enable mkhomedir
```

`/etc/sssd/sssd.conf` (versão **sem TLS**; com o bônus, usar as linhas de StartTLS):

```ini
[sssd]
services = nss, pam
domains = grupo3

[domain/grupo3]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://ldap.grupo3.bsi-26-2.maceio.lab
ldap_search_base = dc=grupo3,dc=bsi-26-2,dc=maceio,dc=lab
cache_credentials = true
ldap_auth_disable_tls_never_use_in_production = true
```

`sudo` pelo grupo do LDAP (`/etc/sudoers.d/lsor-g3`):

```text
%lsor-g3 ALL=(ALL:ALL) ALL
```

Verificação (repetir em **cada** VM):

```bash
getent passwd nome.sobrenome
id nome.sobrenome
grep nome.sobrenome /etc/passwd
```

```text
<colar as saídas; o grep não deve retornar nada: a conta não é local>
```

| VM | getent | id (lsor-g3) | login SSH | sudo |
|:--|:--:|:--:|:--:|:--:|
| gw | | | | |
| ns1 | | | | |
| ns2 | | | | |
| www | | | | |
| bd | | | | |
| arquivos | | | | |
| ldap | | | | |
| mail | | | | |

O `administrador` local foi mantido em todas as VMs (acesso de emergência).

## 11. Servidor de arquivos — `arquivos` (Samba e SFTP)

| Compartilhamento | Caminho | Acesso |
|:--|:--|:--|
| `publico` | `/srv/arquivos/publico` | leitura para todos, inclusive convidado |
| `grupo3` | `/srv/arquivos/grupo3` | leitura e escrita só para `lsor-g3` |

```bash
sudo mkdir -p /srv/arquivos/publico /srv/arquivos/grupo3
sudo chown :lsor-g3 /srv/arquivos/grupo3 && sudo chmod 2770 /srv/arquivos/grupo3
sudo smbpasswd -a nome.sobrenome
testparm
```

Trecho do `smb.conf`:

```text
<colar os blocos [publico] e [grupo3] com valid users, force group, create mask 0660 e directory mask 2770>
```

SFTP restrito do `novato` (trecho final do `sshd_config`):

```text
Match Group sftpusers
    ChrootDirectory /srv/sftp/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

```bash
sudo mkdir -p /srv/sftp/novato/arquivos
sudo chown root:root /srv/sftp/novato && sudo chmod 755 /srv/sftp/novato
sudo chown novato:sftpusers /srv/sftp/novato/arquivos
sudo sshd -t && sudo systemctl restart ssh
```

```text
<colar as saídas>
```

## 12. Servidor web e banco de dados — `www` e `bd`

### 12.1 `bd` (MariaDB)

```bash
sudo apt-get install -y mariadb-server
# bind-address = 192.168.26.53
# <criar o banco "projeto" com criar-banco.sql>
# <usuário da aplicação restrito ao IP do www, com SELECT/INSERT/UPDATE/DELETE>
```

```text
<colar: SHOW DATABASES; SHOW GRANTS FOR '<usuário>'@'192.168.26.52';>
```

### 12.2 `www` (Apache + PHP)

```bash
sudo apt-get install -y apache2 php php-mysql libapache2-mod-php
sudo a2enmod authnz_ldap ldap
```

```text
<colar o virtual host www.grupo3.bsi-26-2.maceio.lab, sem senhas>
```

- Pasta do site: grupo `lsor-g3`, permissão `2775`.
- Área `/admin`: autenticação no LDAP, só para `lsor-g3`. Sem TLS: sem `STARTTLS` no `AuthLDAPURL` e sem `LDAPTrustedGlobalCert`.
- Aplicação: site demo com as adaptações do grupo (o `config.php` **não** vai para o GitHub).

## 13. E-mail — `mail` (bônus)

- [ ] Postfix (`myhostname`, `mydomain`, `mydestination`, `mynetworks`, `home_mailbox = Maildir/`)
- [ ] Submission 587 com STARTTLS e SASL pelo Dovecot
- [ ] Dovecot IMAP 993, Maildir, autenticação via PAM/SSSD
- [ ] Registro MX apontando para `mail`

```text
<colar main.cf, trecho do master.cf e arquivos do Dovecot, se feito>
```

---

## 14. Testes

Registre, para cada teste, **o comando, a saída (ou print) e uma frase dizendo o que ela comprova**. Na fase do VirtualBox, os testes por nome são feitos da VM `cliente`, e os de navegador, Explorer e SFTP, do PC do laboratório pela WAN do `gw`.

| # | Serviço | Teste | Resultado esperado | Status |
|:--:|:--|:--|:--|:--:|
| 1 | Gateway | `ping` e `traceroute` até a VM `www` | Passa pelo `gw` do grupo | ☐ |
| 2 | Gateway | De uma VM: `ping 1.1.1.1` e `curl -I https://ubuntu.com` | Saída pelo NAT do `gw` | ☐ |
| 3 | DNS | `dig www.grupo3.bsi-26-2.maceio.lab` | IP `192.168.26.52` | ☐ |
| 4 | DNS | `dig @192.168.26.50 grupo3.bsi-26-2.maceio.lab SOA` e o mesmo com `@192.168.26.51` | Mesmo serial, flag `aa` | ☐ |
| 5 | DNS | `dig -x 192.168.26.56` | `mail.grupo3.bsi-26-2.maceio.lab.` | ☐ |
| 6 | DNS | `dig MX grupo3.bsi-26-2.maceio.lab` | `10 mail.grupo3...` | ☐ |
| 7 | DNS | Desligar o `ns1` e repetir o teste 3 | `ns2` responde sozinho | ☐ |
| 8 | Login | `ssh` em `www`, `bd` e `mail` + `sudo -l` | Mesma senha do LDAP; `sudo` liberado | ☐ |
| 9 | Login | Trocar a senha no LDAP (`ldappasswd`) e repetir o teste 8 | Senha nova vale em todas as VMs | ☐ |
| 10 | LDAP | `ldapsearch -x -ZZ ...` (sem `-ZZ` se não houver TLS) | Usuários e grupos listados | ☐ |
| 11 | Arquivos | Abrir `\\arquivos...\grupo3` e gravar um arquivo | Integrantes gravam; `novato` negado | ☐ |
| 12 | Arquivos | SFTP ao `arquivos` e baixar o arquivo do teste 11 | Mesmo arquivo nos dois protocolos | ☐ |
| 13 | Arquivos | `sftp novato@arquivos...` e `ssh novato@arquivos...` | SFTP funciona na pasta; SSH recusado | ☐ |
| 14 | Web | Enviar o site por SFTP e abrir `http://www.grupo3.bsi-26-2.maceio.lab/` | "Conectado ao MariaDB … em bd…" | ☐ |
| 15 | Web | `/admin`: cadastrar, editar e remover registro de cada tabela | Alterações refletidas no `bd`; `novato` não entra | ☐ |
| 16 | E-mail (bônus) | Thunderbird com IMAP 993 e SMTP 587 entre dois integrantes | `status=sent` no `mail.log` | ☐ |
| 17 | Firewall | `nc -vz 192.168.26.53 22` e `nc -vz 192.168.26.53 3306` | 22 aberta; 3306 bloqueada pelo `gw` | ☐ |

### Evidências

#### Teste 1
```bash
<comando>
```
```text
<saída>
```
**O que comprova:** <frase>

*(repetir o bloco para os testes 2 a 17)*

---

## 15. Problemas encontrados

| Problema | Causa | Como foi resolvido |
|:--|:--|:--|
| | | |

## 16. Desafios opcionais realizados

- [ ] TLS no LDAP
- [ ] E-mail (Postfix + Dovecot)
- [ ] HTTPS no `www`
- [ ] Webmail (Roundcube)
- [ ] Senha única no Samba
- [ ] Zona reversa *classless* (RFC 2317)
- [ ] Replicação do LDAP (*syncrepl*)
- [ ] Backup automático (`mysqldump` e `slapcat` com `cron`)

## 17. Conclusão

<o que foi implantado, o que funcionou, o que ficou pendente e o que o grupo aprendeu>

## 18. Referências

[1] NETFILTER PROJECT. **iptables(8)** e **iptables-extensions(8)**. <https://ipset.netfilter.org/iptables-extensions.man.html>
[2] CANONICAL. **Configuring networks**. Ubuntu Server documentation. <https://ubuntu.com/server/docs/configuring-networks>
[3] INTERNET SYSTEMS CONSORTIUM. **BIND 9 Administrator Reference Manual**. <https://bind9.readthedocs.io/>
[4] CANONICAL. **Domain Name Service (DNS)**. <https://ubuntu.com/server/docs/how-to/networking/install-dns/>
[5] MOCKAPETRIS, P. **RFC 1034** e **RFC 1035**. IETF, 1987. <https://www.rfc-editor.org/rfc/rfc1034> · <https://www.rfc-editor.org/rfc/rfc1035>
[6] ELZ, R.; BUSH, R. **RFC 2181**. IETF, 1997. <https://www.rfc-editor.org/rfc/rfc2181>
[7] CANONICAL. **Install and configure LDAP**. <https://documentation.ubuntu.com/server/how-to/openldap/install-openldap>
[8] OPENLDAP FOUNDATION. **OpenLDAP Software Administrator's Guide**. <https://www.openldap.org/doc/admin26/>
[9] HOWARD, L. **RFC 2307**. IETF, 1998. <https://www.rfc-editor.org/rfc/rfc2307>
[10] CANONICAL. **How to set up SSSD with LDAP**. <https://documentation.ubuntu.com/server/how-to/sssd/with-ldap>
[11] SSSD PROJECT. **sssd-ldap(5)**. <https://manpages.ubuntu.com/manpages/noble/man5/sssd-ldap.5.html>
[12] CANONICAL. **Set up Samba as a file server**. <https://ubuntu.com/server/docs/how-to/samba/file-server/>
[13] OPENBSD PROJECT. **sshd_config(5)**. <https://man.openbsd.org/sshd_config>
[14] APACHE SOFTWARE FOUNDATION. **Apache HTTP Server 2.4 — Virtual Hosts** e **mod_authnz_ldap**. <https://httpd.apache.org/docs/2.4/>
[15] MARIADB FOUNDATION. **Configuring MariaDB for Remote Client Access**. <https://mariadb.com/kb/en/configuring-mariadb-for-remote-client-access/>
[16] CANONICAL. **Mail services** (Postfix e Dovecot). <https://documentation.ubuntu.com/server/how-to/mail-services>
[17] KUROSE, James F.; ROSS, Keith W. **Redes de Computadores e a Internet**. 8. ed. Pearson, 2022.
[18] LACROIX, Jay. **Mastering Ubuntu Server**. 4. ed. Packt Publishing, 2023.
