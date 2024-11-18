# born2beroot
Configuration of a Debian virtual machine in VirtualBox with a focus on security, system administration, and automated monitoring.

# Tutorial
### SUDO
- **Entrar como root**: `su`
- **Instalar sudo**: `apt install sudo`
- **Reiniciar a máquina**: `sudo reboot`
- **Mostrar versão e configurações do sudo**: `sudo -V`

### GRUPOS E USUÁRIOS
- **Criar usuário**: `sudo adduser rcurty-g`
- **Criar grupo**: `sudo addgroup user42`
- **Verificar se o grupo foi criado**: `getent group nombre_grupo`
- **Listar todos os grupos existentes**: `cat /etc/group`
- **Adicionar usuário ao grupo**: `sudo adduser rcurty-g user42`
- **Verificar se o usuário está no grupo**: `getent group group_name` (ex.: `getent group user42`)
- **Verificar se o usuário está nos grupos sudo e user42**: `nano /etc/group`

### SSH
- **Atualizar repositórios**: `sudo apt update`
- **Instalar OpenSSH para login remoto**: `sudo apt install openssh-server`
- **Verificar se o SSH está ativo**: `sudo service ssh status`
- **Editar configuração SSH**:
  - `nano /etc/ssh/sshd_config` (Altere `#Port 22` para `Port 4242`, e `PermitRootLogin prohibit-password` para `PermitRootLogin no`)
  - `nano /etc/ssh/ssh_config` (Altere `#Port 22` para `Port 4242`)
- **Reiniciar o serviço SSH**: `sudo service ssh restart`
- **Verificar status e confirmar porta 4242**: `sudo service ssh status`

### UFW (Firewall)
- **Instalar o UFW**: `sudo apt install ufw`
- **Ativar o firewall**: `sudo ufw enable`
- **Permitir conexões na porta 4242**: `sudo ufw allow 4242`
- **Verificar configuração do firewall**: `sudo ufw status`

### SENHA FORTE - CONFIGURAÇÕES DE SUDO
- **Criar arquivo de configuração do sudo**: `touch /etc/sudoers.d/sudo_config`
- **Criar diretório para logs do sudo**: `mkdir /var/log/sudo`
- **Editar o arquivo de configuração do sudo**: `nano /etc/sudoers.d/sudo_config`

#### Configurações a serem inseridas no arquivo:
Defaults passwd_tries=3
Defaults badpass_message="Error"
Defaults logfile="/var/log/sudo/sudo_config"
Defaults log_input, log_output
Defaults iolog_dir="/var/log/sudo"
Defaults requiretty
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

- **Explicação**:
  - `passwd_tries=3`: Limite de tentativas de senha.
  - `badpass_message`: Mensagem personalizada de erro.
  - `logfile`: Arquivo para logs do sudo.
  - `log_input, log_output`: Ativar registro de comandos.
  - `iolog_dir`: Diretório para logs de entrada e saída.
  - `requiretty`: Exigir TTY para segurança.
  - `secure_path`: Restringir diretórios utilizados pelo sudo.

### SENHA FORTE - CONFIGURAÇÕES DE LOGIN
- **Editar parâmetros de senha**: `nano /etc/login.defs` (Altere `PASS_MAX_DAYS` para 30 e `PASS_MIN_DAYS` para 2)
- **Instalar pacote de qualidade de senha**: `sudo apt install libpam-pwquality`
- **Editar arquivo de configurações de senha**: `nano /etc/pam.d/common-password`

#### Configurações para inserir:
minlen=10 ucredit=-1 dcredit=-1 lcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root

- **Explicação**:
  - `minlen=10`: Mínimo de 10 caracteres.
  - `ucredit=-1`: Pelo menos uma letra maiúscula.
  - `dcredit=-1`: Pelo menos um dígito.
  - `lcredit=-1`: Pelo menos uma letra minúscula.
  - `maxrepeat=3`: Não repetir o mesmo caractere mais de 3 vezes.
  - `reject_username`: Não pode conter o nome do usuário.
  - `difok=7`: Pelo menos 7 caracteres diferentes da senha antiga.
  - `enforce_for_root`: Aplica a política para o usuário root.

- **Verificar regras aplicadas ao usuário**: `sudo chage -l username` (ex.: `sudo chage -l root`)
- **Definir mínimo de 2 dias para alteração de senha**: `sudo chage -m 2 root`
- **Definir máximo de 30 dias para expiração de senha**: `sudo chage -M 30 root`

### CONECTAR VIA SSH
- **Configurar redirecionamento de porta**:
  - Em "Settings" > "Network" > "Bridged Adapter" para acesso à rede.
- **Obter IP da máquina virtual**: `hostname -I`
- **Conectar via SSH**:
  - Na máquina real: `ssh rcurty-g@ip -p 4242`
- **Verificar status do UFW**: `sudo ufw status numbered` para conferir se a porta 4242 está ativa

### SCRIPT DE MONITORAMENTO
inserir os comandos no arquivo monitoring.sh

## Comandos do Script
- **`uname -a`**  
  Mostra informações completas sobre o sistema (kernel, arquitetura, versão do SO, etc.).

- **`grep "physical id" /proc/cpuinfo | wc -l`**  
  Conta o número de CPUs físicas.

- **`grep processor /proc/cpuinfo | wc -l`**  
  Conta o número de processadores lógicos (núcleos virtuais).

- **`free --mega | awk '$1 == "Mem:" {print $2}'`**  
  Exibe a quantidade total de memória RAM em MB.

- **`free --mega | awk '$1 == "Mem:" {print $3}'`**  
  Exibe a quantidade de memória RAM em uso em MB.

- **`free --mega | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}'`**  
  Calcula o percentual de uso da RAM.

- **`df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_t += $2} END {printf ("%.1fGb\n"), disk_t/1024}'`**  
  Calcula o espaço total do disco (excluindo a partição `/boot`).

- **`df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}'`**  
  Calcula o espaço usado no disco.

- **`df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} {disk_t+= $2} END {printf("%d"), disk_u/disk_t*100}'`**  
  Calcula o percentual de espaço usado no disco.

- **`vmstat 1 2 | tail -1 | awk '{printf $15}'`**  
  Exibe o percentual de tempo ocioso da CPU.

- **`expr 100 - $cpul`**  
  Calcula a carga da CPU, subtraindo o tempo ocioso de 100%.

- **`who -b | awk '$1 == "system" {print $3 " " $4}'`**  
  Mostra a data e hora do último boot do sistema.

- **`lsblk | grep "lvm" | wc -l`**  
  Verifica se o LVM está ativo (retorna "yes" ou "no").

- **`ss -ta | grep ESTAB | wc -l`**  
  Conta o número de conexões TCP estabelecidas.

- **`users | wc -w`**  
  Conta quantos usuários estão logados no sistema.

- **`hostname -I`**  
  Exibe o endereço IP da máquina.

- **`ip link | grep "link/ether" | awk '{print $2}'`**  
  Exibe o endereço MAC da interface de rede.

- **`journalctl _COMM=sudo | grep COMMAND | wc -l`**  
  Conta o número de comandos executados com `sudo`.

- **`wall`**  
  Envia mensagens para todos os terminais logados, exibindo as informações coletadas pelo script.


### CRONTAB
- **Configurar para rodar a cada 10 min**:
  - `sudo crontab -u root -e` e adicione `*/10 * * * * sh /caminho/monitoring.sh`

### SIGNATURE
- **Obter assinatura da VM**:
  - Vá para a pasta da VM e rode `shasum born2beroot.vdi`
