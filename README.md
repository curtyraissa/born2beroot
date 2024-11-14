# born2beroot
Configuration of a Debian virtual machine in VirtualBox with a focus on security, system administration, and automated monitoring.


SUDO
su -> entrar na raiz
apt install sudo -> instalar sudo
sudo reboot -> reiniciar a maquina
sudo -V -> mostrar a versao e argumentos passados para configurar e plugins

GRUPOS E USUARIOS
sudo adduser rcurty-g -> criar utilizador
sudo addgroup user42 -> criar grupo
getent group nombre_grupo -> verificar se o grupo foi criado
cat /etc/group -> verificar quais grupos existem
sudo adduser group -> add usuario no grupo. ex.: sudo adduser rcurty-g user42
getent group group_name - > verificar se o usuario esta no grupo. ex.: getent group user42
nano /etc/group -> verificar se no grupo sudo e user42 esta o nosso login

SSH
sudo apt update -> actualizar os repositórios definidos no ficheiro /etc/apt/sources.list
sudo apt install openssh-server -> Y -> instalar ferramenta OpenSSH para login remoto com protocolo SSH
sudo service ssh status -> verificar se esta activo
nano /etc/ssh/sshd_config -> editar arquivo, #Port 22 = Port 4242, PermitRootLogin prohibit-password = PermitRootLogin no
nano /etc/ssh/ssh_config -> editar arquivo, #Port 22 = Port 4242 
sudo service ssh restart -> reiniciar o servico ssh para atualizar as modificacoes
sudo service ssh status -> verificar o status e confirmar a porta 4242

UFW
sudo apt install ufw -> Y -> instalar o ufw 
sudo ufw enable -> ativar o firewall
sudo ufw allow 4242 -> permitir que a firewall permita ligacoes a porta 4242
sudo ufw status -> verificar se esta configurado corretamente


SENHA FORTE
touch /etc/sudoers.d/sudo_config -> criar ficheiro para inserir a config de senha
mkdir /var/log/sudo -> criar o diretorio sudo para armazenar os comandos
nano /etc/sudoers.d/sudo_config -> editar o ficheiro criado e introduzir os comandos abaixo
Defaults  passwd_tries=3
Defaults  badpass_message="Error"
Defaults  logfile="/var/log/sudo/sudo_config"
Defaults  log_input, log_output
Defaults  iolog_dir="/var/log/sudo"
Defaults  requiretty
Defaults  secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
explicacao: numero de tentativas, mensagem de erro, arquivos do comando sudo, diretorio do arquivo sudo, ativar modo TTY, restringir os diretorios utilizados por sudo.

SENHA FORTE 2
nano /etc/login.defs -> modificar parametros: PASS_MAX_DAYS 99999 => PASS_MAX_DAYS 30, PASS_MIN_DAYS 0 => PASS_MIN_DAYS 2
sudo apt install libpam-pwquality -> Y -> instalar pacote para continuar a conf.
nano /etc/pam.d/common-password -> editar arquivo depois de retry=3: minlen=10 ucredit=-1 dcredit=-1 lcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
//
minlen=10 ➤ O número mínimo de caracteres.
ucredit=-1 ➤ Deve conter pelo menos uma letra maiúscula. Colocamos o - como deve conter pelo menos um caracter, se colocarmos + queremos dizer no máximo esses caracteres.
dcredit=-1 ➤ Deve conter pelo menos um dígito.
lcredit=-1 ➤ Deve conter pelo menos uma letra minúscula.
maxrepeat=3 ➤ Não se pode ter o mesmo carácter mais de 3 vezes seguidas.
reject_username ➤ Não pode conter o nome do utilizador.
difok=7 ➤ Deve ter pelo menos 7 caracteres que não façam parte da senha antiga.
enforce_for_root ➤ Iremos implementar esta política para o utilizador de raiz.
//
sudo chage -l username -> verificar se o utilizador esta na regra correta, ex: sudo chage -l root
sudo chage -m 2 root -> inserir o minimo 2 dias
sudo chage -M 30 root -> insrir maximo de 30 dias

CONECTAR VIA SSH
settings, network, advanced, port forwarding, inserir 2222 e 4242 nas portas.
hostname -i -> para saber o ip da maquina virtual
ssh rcurty-g@ip -p 4242 -> na maquina virtual
ssh rcurty-g@ip -p 4242 -> na maquina real
sudo ufw status numbered -> para verificar se ta na porta 4242
