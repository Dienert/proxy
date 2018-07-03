# Digestaps

O Digestaps é um servidor de proxy local utilizado para automatizar a autenticação em um proxy Squid.

## Conteúdo

- [Instruções de Uso - Linux](#instru%C3%A7%C3%B5es-de-uso-linux)
- [Instruções de Uso - Windows](#instru%C3%A7%C3%B5es-de-uso-windows)
- [Instruções de Uso - Mac](#instru%C3%A7%C3%B5es-de-uso-mac)

## Motivações

Esta solução foi construída como um paliativo para os problemas que passaram a ocorrer após a mudança de um proxy Squid para o método de autenticação Digest.

## Instruções de Uso - Linux

### Download

Baixe o arquivo zip do Digestaps e salve no diretório de sua preferência. E.g.: `~/Downloads` ou `~/apps`

```bash
mkdir ~/apps
wget --no-check-certificate -O ~/apps/ntlmaps-Linux.tar.gz -c https://github.com/Dienert/proxy/blob/master/ntlmaps-Linux.tar.gz
```

### Instalação - Linux

Descompacte o arquivo digestaps.zip no diretório desejado. E.g.: `~/apps/`

```bash
cd ~/apps && tar -zxfv ntlmaps-Linux.tar.gz && rm -rf ntlmaps-Linux.tar.gz
```

### Configuração

#### Criptografia da Senha de Rede

Instale o pacote python-crypto para fornecer suporte a criptografia no python:

- [Pacote para Linux em plataforma amd64](python-crypto_2.6.1-5+b2_amd64.deb)
- [Pacote para Linux em plataforma i386](python-crypto_2.6.1-5+b2_i386.deb)
- [Pacote para Linux Red Hat em plataforma x86_64](python-crypto-2.6.1-1.el6.rfx.x86_64.rpm)


Acesse o diretório do ntlmaps e execute o utilitário `encrypt` para criptograr a senha de acesso ao proxy:

```bash
# Execute o utilitário e informe sua senha de rede
cd ~/apps/ntlmaps-Linux && python scripts/encrypt
```

__Saída do Comando__

```bash
Encrypted password: 5iYAck4FlylnWWJT8Eg/hhUBjFqW7qsXH9VkfEfnE9w=
```

Anote esse texto e cole no arquivo server.cfg no próximo passo.

#### Server.cfg

Abra o arquivo `~/apps/ntlmaps-Linux/server.cfg` com o editor de sua preferência:

```bash
# Editor para Interface gráfica
gedit ~/apps/ntlmaps-Linux/server.cfg

# Editor para Linha de Comando - Avançado
vim ~/apps/ntlmaps-Linux/server.cfg

# Editor para Linha de Comando - Iniciante
nano ~/apps/ntlmaps-Linux/server.cfg
```

Edite o arquivo `server.cfg` com as seguintes informações:

```
LISTEN_PORT:3128
PARENT_PROXY:IP_OU_NOME_DO_PROXY
PARENT_PROXY_PORT:8080
PARENT_PROXY_TIMEOUT:5
HOSTS_TO_BYPASS_PARENT_PROXY:localhost 127.0.0.1 localaddress *.localdomain.com 10.* 
URL_LOG:1
MAX_CONNECTION_BACKLOG:1024

Accept-Encoding: gzip, deflate

USER:<nome.sobrenome>
PASSWORD:<senha_criptografada>
```

### Inicialização

Crie o script de inicialização com o comando abaixo:

```bash
echo -e '#!/bin/bash\n\ncd ~/apps/ntlmaps-Linux && ~/apps/ntlmaps-Linux/scripts/ntlmaps' >> ~/start-proxy.sh && chmod +x ~/start-proxy.sh
```

Em seguinda, utilize o comando abaixo para adicionar o proxy ao boot do sistema:

```bash
(crontab -l 2>/dev/null; echo "@reboot setsid $HOME/start-proxy.sh>/dev/null 2>&1 < /dev/null &") | crontab -
```

### Configuração do Arquivo PAC

Os arquivos de configuração PAC são utilizados pelo sistema para determinar quais endereços devem ser processados pelo proxy.

O procedimento abaixo configura um servidor web estático para disponibilização do arquivo PAC:

Baixe o arquivo PAC no diretório do Digestaps e inicie um servidor web estático:

```bash
mkdir ~/apps/ntlmaps-Linux/pac && wget -c --no-check-certificate -P ~/apps/ntlmaps-Linux/pac/ digestaps.pac
```

Crie o arquivo de inicialização do proxy

```bash
echo -e '#!/bin/bash\n\ncd ~/apps/ntlmaps-Linux/pac && python -m SimpleHTTPServer' >> ~/start-pac-server.sh && chmod +x ~/start-pac-server.sh
```

Por último, execute o comando abaixo para iniciar o proxy junto ao sistema:

```bash
(crontab -l 2>/dev/null; echo "@reboot setsid $HOME/start-pac-server.sh>/dev/null 2>&1 < /dev/null &") | crontab -
```

### Configuração do Sistema

Com o serviço de proxy local no ar, o próximo passo é configurar o sistema operacional para uso nas ferramentas internas.

Abra o arquivo `.bashrc` para editar as variáveis de ambiente do sistema:

```bash
# Editor para Interface gráfica
gedit ~/.bashrc

# Editor para Linha de Comando - Avançado
vim ~/.bashrc

# Editor para Linha de Comando - Iniciante
nano ~/.bashrc
```

Adicione as linhas abaixo no arquivo:

```bash
# Terminal Settings (WGET, Apt e etc.)
export http_proxy=http://localhost:3128
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export rsync_proxy=$http_proxy
export no_proxy=localhost,127.0.0.1,localaddress,*.localdomain.com,10.*

# Configurações para proxy automático em ambientes Gnome
gsettings set org.gnome.system.proxy autoconfig-url 'http://localhost:8000/digestaps.pac'
gsettings set org.gnome.system.proxy mode 'auto'
```

__Opcional:__ Os desenvolvedores podem adicionar as configurações abaixo para configurar o proxy das ferramentas Git e NPM:

```bash
# NodeJS
npm config set proxy $http_proxy
npm config set https-proxy $http_proxy

# Git
git config --global http.proxy $http_proxy
git config --global https.proxy $http_proxy

# Configurações do GIT
git config --global url."https://".insteadOf git://
```

### Aplicando as configurações

Ao finalizar a edição do arquivo .bashrc, abra um terminal e execute o comando abaixo para aplicar as configurações no sistema operacional:

```bash
source ~/.bashrc
```

### Configuração - APT-GET

Crie ou abra o arquivo `/etc/apt/apt.conf`:

```bash
# Editor para Interface gráfica
gedit /etc/apt/apt.conf

# Editor para Linha de Comando - Avançado
vim /etc/apt/apt.conf

# Editor para Linha de Comando - Iniciante
nano /etc/apt/apt.conf
```

Adicione as linhas abaixo no corpo do arquivo:

```bash
Acquire::http::proxy "http://localhost:3128/";
Acquire::https::proxy "https://localhost:3128/";
Acquire::ftp::proxy "ftp://localhost:3128/";
```

### Informações Adicionais

Este proxy funciona com ferramentas que suportem proxies do tipo Basic Authentication, como o Dropbox. Nestes casos, basta informar o endereço do proxy local nas configurações da ferramenta.

P.S.: Na IDE Eclipse basta acrescentar as informações de proxy para os tipos HTTP, HTTPS e FTP, não adicione as informações de proxy na configuração SOCKS.

### Validando o Proxy

O navegador Google Chrome oferece uma ferramenta para validar o proxy vigente, abra a URL:

- [chrome://net-internals/#proxy](chrome://net-internals/#proxy)

## Instruções de Uso - Windows

Até a presente data não está disponível um pacote binário para o Digestaps para o sistema operacional Windows, portanto, os procedimentos abaixo dependem do uso do Runtime Python e são baseados no uso do código fonte do digestaps.

### Pré-Requisitos

Baixe e instale o runtime Python em seu sistema:

- 32 bits: <https://www.python.org/ftp/python/2.7.10/python-2.7.10.msi>
- 64 bits: <https://www.python.org/ftp/python/2.7.10/python-2.7.10.amd64.msi>

Baixe e instale a biblioteca Python Crypto:
- 32 bits: <http://www.voidspace.org.uk/downloads/pycrypto26/pycrypto-2.6.win32-py2.7.exe>
- 64 bits: <http://www.voidspace.org.uk/downloads/pycrypto26/pycrypto-2.6.win-amd64-py2.7.exe>

### Download

Baixe o pacote com o código fonte do Digestaps e salve no diretório de sua preferência. E.g.: `C:\\`

__Versão Simples__

```
https://github.com/Dienert/proxy/blob/master/ntlmaps-Windows.rar
```

__Versão com Criptografia__

```
https://github.com/jctosta/digestaps/archive/master.zip
```

### Instalação

Descompacte o arquivo `ntlmaps-Windows.rar`.

### Configuração

#### Criptografia da Senha de Rede (Opcional)

> Obs: Esse procedimento somente deve ser executado se a versão com suporte a criptografia for escolhida.

Acesse o diretório do ntlmaps e execute o utilitário `encrypt` para criptograr a senha de acesso ao proxy:

- Acesse o diretório C:\\ntlmaps-Windows em um prompt de comando e execute o comando abaixo:

```bash
python scripts\\encrypt
```

__Saída do Comando__

```bash
Encrypted password: 5iYAck4FlylnWWJT8Eg/hhUBjFqW7qsXH9VkfEfnE9w=
```

Anote esse texto e cole no arquivo server.cfg no próximo passo.

Abra o arquivo `C:\\ntlmaps-Windows\\server.cfg` no editor de sua preferência e altere as seguintes informações:

```
LISTEN_PORT:3128
PARENT_PROXY:IP_OU_NOME_DO_PROXY
PARENT_PROXY_PORT:8080
PARENT_PROXY_TIMEOUT:5
HOSTS_TO_BYPASS_PARENT_PROXY:localhost 127.0.0.1 localaddress *.localdomain.com 10.* 
URL_LOG:1
MAX_CONNECTION_BACKLOG:1024

Accept-Encoding: gzip, deflate

USER:<nome.sobrenome>
PASSWORD:<senha_criptografada>
```

Salve o arquivo.

### Inicialização

Abra o arquivo `C:\\ntlmaps-Windows\\runserver.bat` com um editor de texto e altere a localização do executável do Python:

__Valor atual__

```bat
@echo off
"c:\program files\python\python.exe" scripts/ntlmaps
```

__Após alteração__

```bat
@echo off
"c:\python27\python.exe" scripts/ntlmaps
```

Ao finalizar, inicie o Digestaps utilizando o arquivo bat.

### Configuração do Sistema

Acesse as configurações de proxy do sistema e adicione a localização do arquivo PAC:

- `digestaps.pac`

Para contornar os problemas de autenticação no GitLab, crie um servidor web local:
 - Coloque o arquivo digestaps.pac numa pasta
 - Execute as seguintes linhas de comando (que poderá ser colocada num arquivo ".bat" para execução automática a partir do menu "Iniciar"):

 ```
 cd <nome_da_pasta_com_o_arquivo_pac>
 python -m SimpleHTTPServer
 ```

Nas ferramentas que possuam configurações de proxy próprias, utilize o endereço `http://localhost:3128`.

Configurar as variáveis de ambiente http_proxy e https_proxy com o valor `http://localhost:3128`

### Inicialização Automática

//@TODO: Documentar o processo para incialização automática no Windows

## Instruções de Uso - Mac

### Download

Baixe o arquivo zip do Digestaps e salve no diretório de sua preferência. E.g.: `~/Downloads` ou `~/apps`

```bash
mkdir ~/apps
wget -O ~/apps/digestaps.zip -c https://github.com/jctosta/digestaps/archive/master.zip
```

### Instalação - Mac

Descompacte o arquivo digestaps.zip no diretório desejado. E.g.: `~/apps/`

```bash
cd ~/apps && unzip digestaps.zip
mv digestaps-master nltmaps-Mac
```

### Configuração

#### Criptografia da Senha de Rede

Acesse o diretório do ntlmaps e execute o utilitário `encrypt` para criptograr a senha de acesso ao proxy:

```bash
# Execute o utilitário e informe sua senha de rede
python ~/apps/ntlmaps-Mac/scripts/encrypt
```

__Saída do Comando__

```bash
Encrypted password: 5iYAck4FlylnWWJT8Eg/hhUBjFqW7qsXH9VkfEfnE9w=
```

Anote esse texto e cole no arquivo server.cfg no próximo passo.

#### Server.cfg

Abra o arquivo `~/apps/ntlmaps-Mac/server.cfg` com o editor de sua preferência:

```bash
# Editor para Interface gráfica
gedit ~/apps/ntlmaps-Mac/server.cfg

# Editor para Linha de Comando - Avançado
vim ~/apps/ntlmaps-Mac/server.cfg

# Editor para Linha de Comando - Iniciante
nano ~/apps/ntlmaps-Mac/server.cfg
```

Edite o arquivo `server.cfg` com as seguintes informações:

```
LISTEN_PORT:3128
PARENT_PROXY:IP_OU_NOME_DO_PROXY
PARENT_PROXY_PORT:8080
PARENT_PROXY_TIMEOUT:5
URL_LOG:1
MAX_CONNECTION_BACKLOG:1024

Accept-Encoding: gzip, deflate

USER:<nome.sobrenome>
PASSWORD:<senha_criptografada>
```

### Inicialização

Crie o script de inicialização com o comando abaixo:

```bash
echo -e '#!/bin/bash\n\ncd ~/apps/ntlmaps-Mac && ~/apps/ntlmaps-Mac/scripts/ntlmaps' >> ~/start-proxy.sh && chmod +x ~/start-proxy.sh
```

Inicie o proxy:

```bash
cd ~ && ./start-proxy.sh
```

Para adicionar ao boot do sistema, utilize o comando `crontab -e` e adicione a linha abaixo:

```bash
@reboot $HOME/start-proxy.sh>/dev/null 2>&1 < /dev/null & disown
```

### Configuração do Sistema

Baixe o arquivo PAC no diretório do Digestaps e inicie um servidor web estático:

```bash
# Crie o diretório local para o arquivo .PAC
mkdir ~/apps/ntlmaps-Mac/pac
# Baixe o arquivo no diretório criado no passo anterior
wget -c --no-check-certificate -P ~/apps/ntlmaps-Mac/pac/ digestaps.pac
# Inicie o servidor web no diretório
echo -e '#!/bin/bash\n\ncd ~/apps/ntlmaps-Mac/pac && python -m SimpleHTTPServer' >> ~/start-pac-server.sh && chmod +x ~/start-pac-server.sh
cd ~/apps/ntlmaps-Mac/pac && python -m SimpleHTTPServer
```

Adicione o script ao crontab (`crontab -e`) para inicializar o servidor no boot da máquina:

```bash
@reboot $HOME/start-pac-server.sh>/dev/null 2>&1 < /dev/null & disown
```

Para o uso em ferramentas de linha de comando basta definir as variáveis de ambiente adequadas:

```bash
# Terminal Settings (WGET, Apt e etc.)
export http_proxy=http://localhost:3128
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export rsync_proxy=$http_proxy
export no_proxy=localhost,127.0.0.1,localaddress,*.localdomain.com,

# NodeJS
npm config set proxy $http_proxy
npm config set https-proxy $http_proxy

# Git
git config --global http.proxy $http_proxy
git config --global https.proxy $http_proxy

# Configurações do GIT
git config --global url."https://".insteadOf git://
```

### Configuração do Sistema Operacional

É possível configurar o proxy do sistema operacional de duas maneiras, através da interface gráfica de usuário ou por do comando `networksetup`, neste tutorial será exibido apenas o procedimento para configuração através do command line.

Utilize o comando abaixo para ativar o proxy:

```bash
networksetup -setautoproxyurl "Wi-Fi" "http://localhost:8000/digestaps.pac"
```

Para desabilitar e retornar a configuração padrão utilize:

```bash
networksetup -setautoproxystate "Wi-Fi" off
```

### Informações Adicionais

Este proxy funciona com ferramentas que suportem proxies do tipo Basic Authentication, como o Dropbox. Nestes casos, basta informar o endereço do proxy local nas configurações da ferramenta.

P.S.: Na IDE Eclipse basta acrescentar as informações de proxy para os tipos HTTP, HTTPS e FTP, não adicione as informações de proxy na configuração SOCKS.

### Validando o Proxy

O navegador Google Chrome oferece uma ferramenta para validar o proxy vigente, abra a URL:

- <chrome://net-internals/#proxy>
