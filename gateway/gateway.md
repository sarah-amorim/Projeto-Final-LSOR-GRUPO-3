# Gateway

O **gateway (`gw`)** é a máquina que conecta a rede interna isolada do Grupo 3 (`192.168.26.48/28`) à rede do laboratório e à internet, atuando como a rota padrão (`192.168.26.49`) para todas as demais VMs.

Com uma placa voltada para fora (WAN) e outra para a rede interna (LAN), ele realiza duas tarefas persistentes: o **roteamento** (`ip_forward = 1`), que repassa os pacotes entre as duas redes, e o **NAT** (`MASQUERADE` no `iptables`), que traduz os IPs privados do grupo para o IP externo do `gw` na saída. Sem ele, as VMs não conseguem baixar pacotes nem os servidores DNS alcançam os encaminhadores externos para resolver nomes da internet.
