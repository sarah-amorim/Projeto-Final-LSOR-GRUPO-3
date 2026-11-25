# Configurações da máquina virtual ```www```

No projeto, o **`www`** é a máquina virtual responsável por atuar como o **servidor web** da rede, hospedando a aplicação do grupo por meio do **Apache com PHP** (junto ao driver `php-mysql`).

Ela publica o site em uma arquitetura de duas camadas, conectando-se pela rede interna ao servidor **`bd`** (onde fica o banco de dados MariaDB) para ler e gravar informações. Além disso, a VM `www` está integrada ao **LDAP** em duas frentes: ela restringe o acesso à área administrativa (`/admin`) do site apenas aos usuários do grupo do projeto e permite que os integrantes enviem ou atualizem os arquivos do site via **SFTP**, contando também com um ambiente SFTP restrito e isolado (*chroot*) exclusivo para o usuário `novato`.