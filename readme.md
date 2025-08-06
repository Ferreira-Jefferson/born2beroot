# Born2beRoot

Este README foi criado para ajudar no desenvolvimento do projeto Born2beRoot da 42, com dicas práticas, comandos explicados e uma lista completa dos comandos usados no projeto. A ordem abaixo segue a linha natural de configuração, ele não te dará todos os comandos na mão, mas é um bom guia para quem não quer só fazer sem nem parar para pensar sobre o que está fazendo.

---

## 1. Configuração da Máquina Virtual

- **VirtualBox**: Crie uma VM com 20GB+ de disco, 1 CPU, 2GB RAM.
- **Sistema**: Instale o Debian sem interface gráfica.

## 2. Hostname

```bash
sudo hostnamectl set-hostname seuLogin42
```

Atualize o `/etc/hosts`:

```
127.0.1.1    seuLogin42
```

## 3. Particionamento com LVM e Criptografia

Durante a instalação:

- Crie `/boot` fora do LVM
- Crie partição criptografada com LUKS para o restante
- Dentro dela, crie LVM com os volumes:
  - `/` (root)
  - `/home`
  - `/var`
  - `/srv`
  - `swap`

## 4. Criação de usuário

```bash
sudo useradd -m -G sudo,user42 seuLogin
sudo passwd seuLogin
```

## 5. Configuração do Sudo

Edite com `visudo`:

```bash
sudo visudo
```

Adicione:

```
Defaults    passwd_tries=3
Defaults    badpass_message="Senha incorreta, tente novamente."
Defaults    logfile="/var/log/sudo/log"
Defaults    log_input
Defaults    log_output
Defaults    requiretty
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

## 6. Política de Senhas

Configure via `pam_pwquality.so`, `login.defs`, `chage` e `passwd`:

```bash
sudo chage -M 30 -W 7 seuLogin
```

Modifique `/etc/login.defs` e `/etc/security/pwquality.conf` conforme regras.

## 7. SSH

```bash
sudo apt install openssh-server
sudo nano /etc/ssh/sshd_config
```

Altere:

```
Port 4242
PermitRootLogin no
```

Reinicie:

```bash
sudo systemctl restart ssh
```

## 8. Firewall (UFW)

```bash
sudo apt install ufw
sudo ufw allow 4242
sudo ufw enable
sudo ufw status verbose
```

## 9. AppArmor

Verifique:

```bash
sudo aa-status
```

Se estiver desativado, ative:

```bash
sudo systemctl enable apparmor
sudo systemctl start apparmor
```

## 10. Script `monitoring.sh`

Crie em `/usr/local/bin/monitoring.sh`:

```bash
chmod +x /usr/local/bin/monitoring.sh
```

### Agendamento com cron:

```bash
echo "*/10 * * * * root /usr/local/bin/monitoring.sh | wall" | sudo tee /etc/cron.d/monitoring
```

### Execução no boot (alternativa):

Adicione no fim de `/etc/profile`:

```bash
/usr/local/bin/monitoring.sh | wall
```

---

## 📋 Comandos usados e explicações

### uname -a

- `uname`: mostra informações do kernel e sistema
- `-a`: exibe todos os detalhes (nome do kernel, hostname, versão, arquitetura, etc.)

### lscpu | grep Socket | awk '{print \$2}'

- `lscpu`: mostra detalhes da CPU
- `grep Socket`: filtra a linha que mostra o número de soquetes físicos
- `awk '{print $2}'`: imprime a segunda coluna, que contém o número

### nproc

- Mostra o número de CPUs lógicas (núcleos com hyperthreading)

### free -m

- Mostra uso da memória em MB
- `awk` usado para extrair memória usada, total e calcular percentual

### df -h --block-size=G --total

- `df`: mostra uso de disco
- `-h`: formato legível
- `--block-size=G`: exibe em gigabytes
- `--total`: soma todos os sistemas de arquivos
- `awk` + `cut`: extrai valores usados, totais e porcentagem

### ps -eo %cpu --sort=-%cpu

- Lista uso de CPU por processo
- `awk` soma todos os valores para estimar o uso total da CPU

### who -b

- Exibe data e hora do último boot do sistema

### cat /etc/fstab | grep /dev/mapper | wc -l

- Verifica se há partições com LVM em uso
- `wc -l`: conta quantas linhas correspondem, se > 0, LVM está ativo

### ss -t state established | wc -l

- `ss`: mostra conexões de rede
- `-t`: apenas TCP
- `state established`: apenas conexões ativas
- `wc -l`: conta número de conexões

### w | wc -l

- `w`: lista usuários logados
- `wc -l`: conta quantas linhas existem, subtrai 2 (cabeçalhos)

### ip address | grep enp | ...

- Pega o IP (`inet`) e MAC address (`ether`) da interface de rede ativa (ex: `enp0s3`)

### journalctl \_COMM=sudo | grep COMMAND | wc -l

- Conta quantos comandos `sudo` foram executados no sistema
- Usa o log do `journalctl` filtrado por `COMMAND`

---
---
Se seguir este guia e praticar cada etapa, você estará bem preparado para a avaliação do Born2beRoot e se precisar de uma referência que segure em suas mãos, dá uma olhada no [Born2BeRoot Guide](https://noreply.gitbook.io/born2beroot).

## License

This project is licensed under the GNU General Public License v3.0 (GPLv3). See the [LICENSE](LICENSE) file for details.

---

*Project developed as part of the 42 School curriculum.*

