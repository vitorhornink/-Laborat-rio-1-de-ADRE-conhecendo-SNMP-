# Configuração de Rede com Servidor de Log

A empresa fictícia **RedesBR** deseja centralizar os arquivos de log de 3 computadores da rede para facilitar o monitoramento. 
Para isso, será configurado um servidor `rsyslog` para receber logs de computadores clientes na rede.

---

## Tarefas

- Designar um dos PCs como servidor de log.
- Configurar o `rsyslog` no servidor e nos clientes.
- Verificar se os logs estão sendo enviados e recebidos corretamente.
- Demonstrar a leitura dos logs no servidor.

---

## Passo a Passo

### 1. Designar o Servidor de Log

Escolher um dos computadores da rede como **servidor central de logs**. 

---

### 2. Configurar o rsyslog no Servidor

#### 2.1 Editar a configuração do rsyslog
Editar o arquivo `/etc/rsyslog.conf` ou criar arquivos adicionais em `/etc/rsyslog.d/`:

sudo nano /etc/rsyslog.conf

conf
module(load="imudp") # para habilitar UDP
input(type="imudp" port="514")

module(load="imtcp") # para habilitar TCP
input(type="imtcp" port="514")

#### 2.2 Criar diretório para armazenar logs remotos

sudo mkdir -p /var/log/remotelogs

#### 2.3 Adicionar regra para salvar logs dos clientes
Crie um novo arquivo de configuração:

sudo nano /etc/rsyslog.d/remote.conf

Adicione a seguinte linha para registrar os logs de todos os clientes:
conf
$template RemoteLogs,"/var/log/remotelogs/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs

#### 2.4 Reiniciar o serviço

sudo systemctl restart rsyslog

### 3. Configurar os Clientes
#### 3.1 Editar a configuração do rsyslog em cada cliente
sudo nano /etc/rsyslog.conf
Adicione ao final do arquivo (ajustando para o IP real do servidor):

*.* @IPCLIENTE01:514    # para UDP
#*.* @@IPCLIENTE01:514  # para TCP (opcional)
#### 3.2 Reiniciar o serviço nos clientes

sudo systemctl restart rsyslog

### 4. Verificar Funcionamento
#### 4.1 Testar envio de log a partir do cliente
No cliente:
logger "Teste de envio de log para o servidor central"

#### 4.2 Verificar se o log chegou no servidor
No servidor:
sudo cat /var/log/remotelogs/[NOME_DO_CLIENTE]/logger.log
Você deve ver a mensagem "Teste de envio de log para o servidor central".
