# 🛡️ Guia de Segurança e Gerenciamento de Servidor Linux

Este guia cobre as ferramentas essenciais para proteger e acessar seu servidor via terminal.

---

## 📑 Sumário
1. [🧱 UFW - Uncomplicated Firewall](#-ufw---uncomplicated-firewall)
2. [🔄 Atualização e Manutenção do Sistema](#-atualização-e-manutenção-do-sistema)
3. [👤 Gerenciamento de Usuários e Permissões](#-gerenciamento-de-usuários-e-permissões)
4. [🚫 Fail2Ban - Prevenção de Intrusão](#-fail2ban---prevenção-de-intrusão)
5. [🔑 SSH - Acesso Remoto](#-ssh---acesso-remoto)
6. [🔄 AutoSSH e Túnel Reverso](#-autossh-e-túnel-reverso)
7. [⚙️ SSHD - Configuração do Servidor SSH](#-sshd---configuração-do-servidor-ssh)
8. [🖧 Rede e Diagnóstico de Rede](#-rede-e-diagnóstico-de-rede)
9. [📅 Crontab - Agendamento de Tarefas](#-crontab---agendamento-de-tarefas)
10. [⚙️ systemd e serviços](#-systemd-e-serviços)
11. [💾 Backup e restauração](#-backup-e-restauração)
12. [✅ Checklist de Servidor](#-checklist-de-servidor)

---

## 🐧 Terminais: Bash e Linux

### Manipulação de Arquivos
- `ls -la`: Lista todos os arquivos com detalhes e permissões (incluindo ocultos).
- `cd <caminho>`: Muda para o diretório especificado.
- `mkdir -p <caminho>`: Cria diretórios e subdiretórios se não existirem.
- `touch <arquivo>`: Cria um arquivo vazio ou atualiza a data de modificação.
- `rm -rf <caminho>`: Remove arquivos ou pastas recursivamente e sem pedir confirmação.
- `cp -r <origem> <destino>`: Copia pastas e seus conteúdos de forma recursiva.
- `mv <origem> <destino>`: Move ou renomeia arquivos/diretórios.
- `cat <arquivo>`: Exibe todo o conteúdo do arquivo de uma vez.
- `tail -f <arquivo>`: Exibe as últimas linhas e continua monitorando novas linhas (logs).
- `grep "termo" <arquivo>`: Filtra linhas que contenham o termo buscado.

### Gestão do Sistema (Comandos úteis)
- `free -h`: Mostra o uso de memória RAM em formato legível (GB, MB).
- `df -h`: Mostra o espaço livre em todas as partições do disco.
- `du -sh <pasta>`: Calcula o tamanho total ocupado por uma pasta no disco.
- `htop`: Interface interativa para monitorar CPU, RAM e processos.
- `systemctl start <serviço>`: Inicia um serviço do sistema.
- `systemctl status <servico>`: Verifica se um serviço está rodando ou se falhou.
- `journalctl -u <servico> -f`: Acompanha os logs específicos de um serviço do sistema.
- `chmod +x <arquivo>`: Torna um arquivo executável pelo sistema.
- `chown -R <user>:<group> <pasta>`: Muda o dono e o grupo de uma pasta recursivamente.


## 🧱 UFW - Uncomplicated Firewall
O UFW é uma interface simplificada para o iptables.

### Comandos Básicos
- `sudo ufw status`: Verifica se o firewall está ativo e lista as regras.
- `sudo ufw enable`: Ativa o firewall.
- `sudo ufw disable`: Desativa o firewall.
- `sudo ufw default deny incoming`: Bloqueia todas as conexões de entrada por padrão (recomendado).
- `sudo ufw default allow outgoing`: Permite todas as conexões de saída por padrão.

### Liberando e Bloqueando Portas
- `sudo ufw allow 22`: Libera a porta 22 (SSH).
- `sudo ufw allow 80/tcp`: Libera apenas o protocolo TCP na porta 80 (HTTP).
- `sudo ufw allow 443`: Libera a porta 443 (HTTPS).
- `sudo ufw deny 23`: Bloqueia a porta 23.
- `sudo ufw allow from 192.168.1.50`: Libera todo o acesso para um IP específico.

### Gerenciando Regras
- `sudo ufw status numbered`: Lista as regras com números de índice.
- `sudo ufw delete 2`: Deleta a regra de número 2.

---

## 🔄 Atualização e Manutenção do Sistema
Manter o servidor atualizado é a base para segurança e estabilidade.

### Comandos Essenciais
- `sudo apt update`: Atualiza a lista de pacotes disponíveis.
- `sudo apt upgrade`: Atualiza os pacotes instalados.
- `sudo apt full-upgrade`: Atualiza e resolve dependências mais complexas.
- `sudo apt autoremove`: Remove pacotes obsoletos e dependências não usadas.
- `sudo apt install unattended-upgrades`: Instala atualizações automáticas de segurança.

### Boas Práticas
- Rode atualizações em horários de baixa utilização.
- Verifique se há backups antes de aplicar grandes upgrades.
- Use `unattended-upgrades` apenas para atualizações de segurança em produção.

---

## 👤 Gerenciamento de Usuários e Permissões
Controle de contas e permissões é crítico para evitar acessos indevidos.

### Comandos de Usuário
- `sudo adduser usuario`: Cria um novo usuário de forma interativa.
- `sudo userdel -r usuario`: Remove usuário e seu diretório home.
- `sudo usermod -aG sudo usuario`: Adiciona um usuário ao grupo `sudo`.
- `sudo passwd usuario`: Define ou altera a senha do usuário.

### Permissões e Propriedade
- `sudo chown usuario:grupo arquivo`: Muda o proprietário e grupo.
- `sudo chmod 750 arquivo`: Define permissões de leitura/escrita/execução.
- `sudo chmod 600 ~/.ssh/authorized_keys`: Permissão segura para chaves SSH.

### Dicas de Segurança
- Use contas de usuário sem privilégios para operação normal.
- Não permita login direto de `root` via SSH.
- Mantenha o `sudo` restrito a contas confiáveis.

---

## 🚫 Fail2Ban - Prevenção de Intrusão
O Fail2Ban monitora logs do sistema e bane temporariamente IPs que mostram sinais maliciosos (como muitas falhas de login).

### Comandos Principais
- `sudo systemctl status fail2ban`: Verifica se o serviço está rodando.
- `sudo fail2ban-client status`: Lista as "jails" (cadeias de monitoramento) ativas.
- `sudo fail2ban-client status sshd`: Mostra estatísticas de banimento específicas do SSH.
- `sudo fail2ban-client set sshd unbanip 1.2.3.4`: Desbane um IP específico manualmente.
- `sudo systemctl restart fail2ban`: Reinicia o serviço após mudanças na configuração.

---

## 🔑 SSH - Acesso Remoto (Cliente)
Comandos para conectar-se a outros servidores.

- `ssh usuario@ip-do-servidor`: Conexão padrão.
- `ssh -p 2222 usuario@ip`: Conexão especificando uma porta diferente.
- `ssh -i ~/.ssh/minha_chave usuario@ip`: Conexão usando uma chave privada específica.

### Copiando Chave para o Servidor
- `ssh-copy-id -i ~/.ssh/id_rsa.pub usuario@ip-do-servidor`: Envia sua chave pública para o servidor (permite login sem senha).

---

## 🔄 AutoSSH e Túnel Reverso
O `autossh` é uma ferramenta que monitora e reinicia conexões SSH automaticamente caso elas caiam. É ideal para manter túneis persistentes.

### Instalação
- `sudo apt install autossh` (Debian/Ubuntu)
- `sudo yum install autossh` (CentOS/RHEL)

### Túnel Reverso (Acesso Local de Fora)
Útil para acessar uma máquina que está atrás de um firewall ou NAT (como sua máquina de casa) através de um servidor público (VPS).

**Comando:**
```bash
autossh -M 0 -f -N -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -R 8080:localhost:80 usuario@vps-ip
```

**Explicação dos parâmetros:**
- `-M 0`: Desativa o monitoramento de porta do autossh (usa o `ServerAlive` do SSH).
- `-f`: Roda em segundo plano (background).
- `-N`: Não executa comandos remotos (apenas cria o túnel).
- `-R 8080:localhost:80`: Mapeia a porta **8080 do servidor remoto** para a porta **80 da sua máquina local**.
- `-o "..."`: Garante que o SSH perceba rápido se a conexão caiu para o `autossh` reiniciar.

### Acessando o Túnel
No servidor remoto (VPS), você agora pode acessar o serviço da sua máquina local via:
- `curl http://localhost:8080`

---

## ⚙️ SSHD - Configuração do Servidor SSH

O arquivo de configuração fica em `/etc/ssh/sshd_config`.

### Boas Práticas de Segurança
Ao editar o arquivo (`sudo nano /etc/ssh/sshd_config`), considere estas mudanças:

1. **Mudar a Porta Padrão:**
   `Port 2222` (Mudar de 22 para algo aleatório diminui ataques automatizados).
2. **Desativar Login do Root:**
   `PermitRootLogin no` (Força o uso de um usuário comum com `sudo`).
3. **Desativar Login por Senha (Usar apenas Chaves):**
   `PasswordAuthentication no` (Extremamente seguro).
4. **Limitar Tentativas de Login:**
   `MaxAuthTries 3`.

### Aplicando Mudanças
Sempre teste a configuração antes de reiniciar para evitar ficar trancado fora:
- `sudo sshd -t`: Testa a sintaxe do arquivo de configuração.
- `sudo systemctl restart sshd`: Reinicia o serviço SSH na maioria das distribuições.
- `sudo systemctl restart ssh`: Pode ser usado em distros que mapeiam o serviço para `ssh`.

---

## 🖧 Rede e Diagnóstico de Rede
Uma boa visibilidade de rede ajuda a identificar problemas de conectividade e segurança.

### Informações de Endereço e Roteamento
- `ip addr show`: Mostra os endereços IP e interfaces de rede.
- `ip addr show eth0`: Mostra apenas a interface `eth0`.
- `ip route show`: Exibe a tabela de roteamento.
- `ip neigh`: Lista a tabela ARP local.
- `hostname -I`: Exibe os endereços IP locais.
- `curl ifconfig.me` ou `curl icanhazip.com`: Ver IP público do servidor.

### Diagnóstico de Conectividade
- `ping 8.8.8.8`: Testa conectividade com um host remoto pelo IP.
- `ping google.com`: Testa resolução DNS e conectividade.
- `traceroute 8.8.8.8`: Exibe o caminho da rede até o destino.
- `tracepath google.com`: Alternativa sem privilégios para `traceroute`.
- `mtr google.com`: Combina ping e traceroute em tempo real.

### Diagnóstico DNS
- `dig google.com`: Consulta DNS usando o `dig`.
- `dig @8.8.8.8 google.com`: Consulta DNS em um servidor específico.
- `nslookup google.com`: Consulta DNS com saída simples.
- `host google.com`: Consulta DNS com nome do host.
- `systemd-resolve --status`: Exibe as configurações de resolução DNS em sistemas `systemd`.

### Conexões e Servidores de Rede
- `ss -tuln`: Lista sockets TCP/UDP em escuta.
- `ss -tnp`: Mostra conexões TCP ativas com PID e programa.
- `netstat -tulpen`: Exibe portas e processos (se disponível).
- `sudo ss -s`: Estatísticas resumidas de sockets.
- `sudo lsof -i -P -n`: Lista arquivos abertos em uso pela rede.

### Testes de Rota e Latência
- `iperf3 -c servidor`: Testa throughput de rede com `iperf3`.
- `curl -I https://example.com`: Verifica status HTTP e cabeçalhos.
- `wget --spider https://example.com`: Verifica acesso HTTP/HTTPS sem baixar o conteúdo.

---

## 📊 Monitoramento do Servidor
Monitorar recursos e logs ajuda a detectar problemas antes que causem falhas.

### Uso de CPU e Memória
- `top`: Visão em tempo real de processos e uso de CPU/memória.
- `htop`: Versão mais interativa de `top` (recomendada, instale com `sudo apt install htop`).
- `vmstat 1`: Mostra estatísticas de CPU, memória e E/S a cada segundo.
- `free -h`: Exibe memória total, usada e livre em formato legível.

### Uso de Disco
- `df -h`: Mostra uso de disco por sistema de arquivos.
- `du -sh /var/log`: Exibe o tamanho total de um diretório.
- `sudo du -h --max-depth=1 /var`: Mostra uso por subdiretório.
- `ncdu /`: Interface interativa para analisar uso de disco (`sudo apt install ncdu`).

### Logs do Sistema
- `journalctl -xe`: Exibe logs recentes e eventos críticos.
- `journalctl -u sshd`: Logs do serviço SSH.
- `journalctl --since "1 hour ago"`: Logs das últimas horas.
- `sudo tail -f /var/log/syslog`: Segue atualizações em tempo real no syslog.
- `sudo tail -f /var/log/auth.log`: Segue logs de autenticação e SSH.

### Rotina de Limpeza e Rotação de Logs
- `sudo systemctl status rsyslog`: Verifica o serviço de logs.
- `sudo logrotate -f /etc/logrotate.conf`: Força rotação de logs.
- Verifique `/etc/logrotate.d/` para configurações específicas de serviços.

---

## 📅 Crontab - Agendamento de Tarefas
O `cron` é um utilitário de sistema que permite agendar tarefas para serem executadas periodicamente.

### Comandos Básicos
- `crontab -e`: Abre o editor para criar ou editar suas tarefas agendadas.
- `crontab -l`: Lista todas as tarefas agendadas para o usuário atual.
- `crontab -r`: Remove todas as tarefas agendadas do usuário atual.

### Sintaxe do Cron
Uma linha no crontab tem 5 campos de tempo seguidos pelo comando:
```text
* * * * * comando_a_ser_executado
- - - - -
| | | | |
| | | | +----- Dia da semana (0 - 6) (Domingo=0)
| | | +------- Mês (1 - 12)
| | +--------- Dia do mês (1 - 31)
| +----------- Hora (0 - 23)
+------------- Minuto (0 - 59)
```

### Exemplos Comuns
- `0 5 * * * /backup.sh`: Roda o script de backup todos os dias às 05:00 da manhã.
- `*/15 * * * * comando`: Roda o comando a cada 15 minutos.
- `0 0 1 * * comando`: Roda o comando à meia-noite do primeiro dia de cada mês.
- `0 22 * * 1-5 comando`: Roda às 22:00, de segunda a sexta-feira.

### Atalhos Úteis
- `@reboot`: Executa uma vez, na inicialização do sistema.
- `@daily`: Executa uma vez por dia (mesmo que `0 0 * * *`).
- `@hourly`: Executa uma vez por hora (mesmo que `0 * * * *`).

## ⚙️ systemd e serviços

### Unit file básico
```ini
[Unit]
Description=Meu Serviço Python
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/srv/projetos/meu_app
ExecStart=/srv/projetos/meu_app/.venv/bin/gunicorn -k uvicorn.workers.UvicornWorker app:app -b 0.0.0.0:8000
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Comandos úteis
- `sudo systemctl daemon-reload`
- `sudo systemctl enable meu-servico`
- `sudo systemctl start meu-servico`
- `sudo systemctl restart meu-servico`
- `sudo systemctl status meu-servico`
- `sudo journalctl -u meu-servico -f`

### Boas práticas com services
- Separe logs de aplicação e logs do sistema.
- Use `Restart=on-failure` ou `Restart=unless-stopped` para serviços críticos.
- Mantenha `WorkingDirectory` e `ExecStart` consistentes entre dev e produção.
- Teste a unidade localmente antes de habilitá-la.

## 💾 Backup e restauração

- Faça backup regular de dados críticos, incluindo bancos de dados e configurações (`/etc`, `/var/lib`, `/home` quando necessário).
- Use ferramentas como `rsync`, `borg`, `restic` ou snapshots LVM/ZFS.
- Armazene backups em locais separados do servidor principal.
- Teste o processo de restauração periodicamente.
- Mantenha políticas de retenção claras: diário, semanal, mensal.
- Inclua no backup arquivos de configuração do sistema e do serviço.

## ✅ Checklist de Servidor

- [ ] Firewall ativo com regras mínimas.
- [ ] SSH configurado com chaves, porta não padrão e root desabilitado.
- [ ] Fail2Ban ou proteção equivalente habilitada.
- [ ] Serviço systemd configurado e monitorado.
- [ ] Logs rotacionados e monitorados.
- [ ] Backups automatizados e testados.
- [ ] Atualizações de segurança planejadas e aplicadas.
- [ ] Monitoramento de CPU, memória, disco e rede em produção.
- [ ] Acesso remoto seguro e reversível em caso de problema.

---

## 💡 Dica de Ouro
...Ao configurar o firewall ou mudar a porta do SSH, **nunca feche sua sessão atual** antes de abrir uma nova em outro terminal e confirmar que o acesso ainda funciona. Se algo der errado, você ainda terá a sessão aberta para corrigir.
