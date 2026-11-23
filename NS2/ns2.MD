# Configurações da máquina virtual ```ns2```

No projeto, o ns2 (abreviação de Name Server 2) é a segunda máquina virtual de DNS do grupo, que também roda o software BIND9, mas assume o papel de servidor DNS secundário.   A principal razão de existir do ns2 é eliminar um ponto único de falha na rede. Se o grupo dependesse apenas do ns1 e essa máquina travasse, reiniciasse ou tivesse o serviço named parado, nenhuma outra VM conseguiria resolver nomes, derrubando o login unificado via LDAP, o acesso ao banco de dados e os demais serviços. Com o ns2 configurado como segundo servidor DNS nas placas de rede de todas as máquinas, ele assume as respostas automaticamente caso o ns1 fique indisponível.  

### Definição do Hostname

O hostname é o nome único que identifica uma máquina na rede. No projeto, ele é configurado como o FQDN ( Domínio Totalmente Qualificado), que une o nome do serviço ao domínio do grupo, por exemplo, ```ns2.grupo0.bsi-26-2.maceio.lab```. 

```
sudo hostnamectl set-hostname ns2.grupo3.bsi-26-2.maceio.lab
sudo nano /etc/hosts
```

No ```/etc/hosts``` deixe só a própria VM:

![Arquivo /etc/hosts](../imagens/)
