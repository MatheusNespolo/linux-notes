# systemd e serviços

Fundamentos de `systemd` para quem vem de outras distros (ex.: SysV init, Alpine/OpenRC) ou de outros sistemas operacionais, e precisa entender como serviços, rede e logs são gerenciados em distros Linux modernas.

## O que é o systemd

`systemd` é o sistema de inicialização (init system) usado pela maioria das distros Linux modernas, incluindo Debian, Ubuntu e suas derivadas. Ele é o primeiro processo que roda no boot (PID 1) e é responsável por:

- Iniciar, parar e monitorar **serviços** (daemons) — gerenciado via `systemctl`.
- Gerenciar **interfaces de rede**, através do componente `systemd-networkd` (ver [Redes em Linux](./redes.md)).
- Centralizar **logs** de todo o sistema e dos serviços, através do `journald`, consultado via `journalctl`.
- Gerenciar montagem de discos, timers (substituindo o `cron` em muitos casos), sockets, e outras unidades do sistema.

Cada coisa que o systemd gerencia é chamada de **unidade** (unit): um serviço é uma unidade `.service`, uma configuração de rede é uma unidade `.network`, um timer é uma unidade `.timer`, e assim por diante.

## Comandos essenciais do `systemctl`

`systemctl` é a ferramenta principal para controlar serviços (unidades `.service`).

```bash
systemctl status <servico>
```
Mostra o estado atual do serviço: se está rodando (`active (running)`), parado, com falha, e as últimas linhas de log relacionadas.

```bash
sudo systemctl start <servico>
```
Inicia o serviço imediatamente (não afeta se ele inicia automaticamente no boot).

```bash
sudo systemctl stop <servico>
```
Para o serviço imediatamente.

```bash
sudo systemctl restart <servico>
```
Para e inicia o serviço novamente — usado tipicamente depois de alterar um arquivo de configuração que o serviço não recarrega sozinho.

```bash
sudo systemctl enable <servico>
```
Configura o serviço para iniciar automaticamente no próximo boot (não inicia agora). Combine com `--now` para fazer as duas coisas de uma vez:

```bash
sudo systemctl enable --now <servico>
```

```bash
sudo systemctl disable <servico>
```
Remove o serviço da inicialização automática no boot.

```bash
sudo systemctl daemon-reload
```
Recarrega as definições de unidades a partir do disco. Necessário sempre que você **cria ou edita manualmente** um arquivo de unidade `.service` (não é necessário para arquivos `.network`, que são recarregados com `networkctl reload`, nem para a maioria das mudanças feitas por pacotes instalados via `apt`).

## Consultando logs com `journalctl`

O `journald` centraliza logs de todos os serviços gerenciados pelo systemd, além de mensagens do kernel.

```bash
journalctl -u <servico>
```
Mostra o histórico de logs de um serviço específico.

```bash
journalctl -u <servico> -f
```
Segue o log em tempo real (equivalente a `tail -f`), útil para acompanhar um serviço enquanto ele é reiniciado ou enquanto se reproduz um problema.

```bash
journalctl -u <servico> --since "10 min ago"
```
Filtra por janela de tempo.

```bash
journalctl -p err
```
Mostra apenas mensagens de nível "erro" ou mais grave, de qualquer serviço.

## Relação com a configuração de rede

O componente `systemd-networkd` é o serviço (`systemd-networkd.service`) responsável por aplicar as configurações de interfaces de rede definidas em arquivos `.network` dentro de `/etc/systemd/network/` — o formato e os passos de configuração de IP estático estão detalhados em [Redes em Linux](./redes.md).

Comandos relacionados a esse serviço específico:

```bash
systemctl status systemd-networkd    # verifica se o serviço de rede está ativo
sudo systemctl restart systemd-networkd   # reinicia o serviço de rede (mais "pesado" que reload)
```

Na prática, para simplesmente aplicar uma mudança em um arquivo `.network`, o comando dedicado `networkctl reload` (em vez de reiniciar o serviço inteiro via `systemctl`) é a forma recomendada — ele recarrega só a configuração de rede, sem afetar outras unidades. Veja o passo a passo completo em [Redes em Linux](./redes.md#configuração-de-ip-estático-com-systemd-networkd).

## Ver também

- [Redes em Linux](./redes.md)
- [Gerenciamento de pacotes (`apt`)](./gerenciamento-pacotes.md)
