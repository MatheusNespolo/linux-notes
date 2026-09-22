# Sistema de backup automatizado com Restic/Borg

> **Nível:** Intermediário

## Objetivo

Ter uma rotina de backup automática, incremental e criptografada dos dados importantes (do servidor de arquivos ou de qualquer outra máquina de casa), rodando sozinha em segundo plano e com restauração testada — porque um backup que nunca foi restaurado não é um backup confiável, é só uma cópia com sorte.

## Hardware sugerido

- O mesmo servidor de arquivos, fazendo backup para um disco separado localmente, e/ou
- Um segundo dispositivo (ex: outro Raspberry Pi na casa de um parente) para backup fora do local (offsite), e/ou
- Um repositório de backup em nuvem compatível (ex: Backblaze B2, S3) para redundância extra.

## Software

- **Restic**: ferramenta de backup moderna, com deduplicação, criptografia nativa e suporte a diversos backends (disco local, SFTP, S3, B2, etc.).
- **Borg Backup**: alternativa madura ao Restic, também com deduplicação e criptografia, muito usada com o front-end Vorta para agendamento gráfico.
- **systemd timers** (ou `cron`): para agendar as execuções automáticas do backup.

Este guia usa o Restic como exemplo, mas os passos com Borg seguem uma lógica equivalente.

## Passo a passo (visão geral)

1. Instalar o Restic:
   ```bash
   sudo apt install restic
   ```
2. Inicializar um repositório de backup (local, para começar):
   ```bash
   restic init --repo /mnt/backup/restic-repo
   ```
   (o Restic vai pedir uma senha de criptografia — guarde-a em um gerenciador de senhas, sem ela os backups são irrecuperáveis.)
3. Rodar o primeiro backup manual para validar que funciona:
   ```bash
   restic -r /mnt/backup/restic-repo backup /mnt/dados
   ```
4. Criar um script simples que executa o backup e aplica uma política de retenção (ex: manter últimos 7 diários, 4 semanais, 6 mensais):
   ```bash
   restic -r /mnt/backup/restic-repo backup /mnt/dados
   restic -r /mnt/backup/restic-repo forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
   ```
5. Agendar esse script com um **systemd timer** (uma unidade `.service` + uma `.timer`) para rodar, por exemplo, todas as madrugadas — ou, mais simples, via `cron`.
6. Testar a restauração periodicamente (não só o backup):
   ```bash
   restic -r /mnt/backup/restic-repo restore latest --target /tmp/teste-restore
   ```
   e conferir que os arquivos restaurados batem com o esperado.

## Próximos passos / evolução

- Adicionar um segundo destino de backup fora de casa (outro Raspberry Pi remoto via SSH, ou um bucket S3/B2), seguindo a regra 3-2-1 (3 cópias, 2 mídias diferentes, 1 fora do local).
- Configurar notificações (e-mail, Telegram, etc.) quando o backup falhar, para não descobrir o problema só na hora de precisar restaurar.
- Monitorar o tamanho do repositório ao longo do tempo e ajustar a política de retenção conforme necessário.
- Testar restauração completa de um "desastre simulado" (apagar os dados originais e restaurar do zero) pelo menos uma vez, para validar o processo de ponta a ponta.
