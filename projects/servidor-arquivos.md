# Servidor caseiro de armazenamento de arquivos (NAS caseiro)

> **Nível:** Iniciante

## Objetivo

Ter um ponto único de armazenamento na rede local para guardar backups da família, fotos, documentos e arquivos de trabalho, acessível de qualquer computador ou celular em casa sem depender de nuvem paga. É também o projeto mais natural para começar a mexer com Linux "de verdade", porque toca em conceitos básicos de sistema de arquivos, permissões, rede e serviços — sem exigir nada muito além disso.

## Hardware sugerido

- Raspberry Pi 4 ou 5 (2 GB de RAM já é suficiente para uso leve) + HD/SSD externo via USB 3.0.
- Alternativa: um mini PC antigo ou uma VM em uma máquina que já esteja ligada em casa.
- Se possível, use um disco dedicado só para os dados (não o cartão SD/boot), para evitar desgaste e facilitar upgrades de disco no futuro.

## Software

- **Samba**: implementa o protocolo SMB, usado pelo Windows para compartilhar pastas. É a opção mais simples para quem tem PCs Windows em casa.
- **NFS**: protocolo nativo do mundo Unix/Linux, mais leve que o Samba quando todos os clientes são Linux.
- **Nextcloud** ou **OpenMediaVault**: soluções mais completas, com interface web, gerenciamento de usuários, apps de sincronização, e no caso do Nextcloud, funcionalidades parecidas com Google Drive/Dropbox.

Para começar, a recomendação é ir de Samba puro — é o caminho mais curto para ter algo funcional e entender o que está acontecendo por baixo dos panos.

## Passo a passo (visão geral)

1. Instalar o Linux no dispositivo (Raspberry Pi OS Lite é suficiente, não precisa de interface gráfica).
2. Conectar e montar o disco externo em um ponto fixo, por exemplo `/mnt/dados`, e garantir que ele monte automaticamente no boot (via `/etc/fstab`).
3. Instalar o Samba:
   ```bash
   sudo apt update
   sudo apt install samba
   ```
4. Criar uma pasta compartilhada e configurar o `/etc/samba/smb.conf` com um bloco apontando para ela, por exemplo:
   ```ini
   [dados]
   path = /mnt/dados
   browseable = yes
   read only = no
   valid users = seu_usuario
   ```
5. Criar um usuário Samba (separado da senha do sistema):
   ```bash
   sudo smbpasswd -a seu_usuario
   ```
6. Reiniciar o serviço (`sudo systemctl restart smbd`) e testar o acesso a partir do Windows digitando `\\ip-do-servidor\dados` no Explorador de Arquivos.
7. Repetir o processo pensando em pastas separadas por finalidade (backups, fotos, documentos) e permissões diferentes para cada uma, se fizer sentido.

## Próximos passos / evolução

- Configurar **RAID** com `mdadm` (RAID 1 espelhado, por exemplo) para ter redundância caso um disco falhe.
- Automatizar backups de outros computadores da casa para o servidor usando `rsync` agendado via `cron` ou `systemd timers`.
- Adicionar acesso remoto seguro (ver o projeto de VPN com WireGuard) para acessar os arquivos de fora de casa sem expor o Samba diretamente à internet.
- Migrar para o Nextcloud quando quiser sincronização automática, versionamento de arquivos e apps mobile.
