# Redes em Linux

Fundamentos de diagnóstico e configuração de rede em qualquer distro Linux moderna baseada em `systemd` (Debian, Ubuntu e derivados). Útil tanto para servidores quanto para dispositivos embarcados/industriais sem monitor conectado.

## Diagnóstico básico

Os três comandos abaixo cobrem a maior parte das dúvidas do dia a dia sobre rede.

### `ip addr show`

Mostra todas as interfaces de rede da máquina e os endereços IP (IPv4 e IPv6) atribuídos a cada uma.

```bash
ip addr show
```

Saída típica (resumida):

```
1: lo: <LOOPBACK,UP,LOWER_UP> ...
    inet 127.0.0.1/8 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    inet 192.168.1.50/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::1234:5678:9abc:def0/64 scope link
```

Repare que cada interface pode ter mais de um endereço: um IPv4 "global" (roteável, ex.: `192.168.1.50`) e um IPv6 "link" (válido só na rede local, começa com `fe80::`). O comando `ip a` é um atalho equivalente.

### `ip route show`

Mostra a tabela de rotas: para onde o tráfego é enviado dependendo do destino, incluindo a rota padrão (gateway).

```bash
ip route show
```

```
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.50
```

Use para confirmar se a máquina tem um gateway configurado e se está na sub-rede esperada.

### `ping`

Testa conectividade básica com outro host, por nome ou IP.

```bash
ping 192.168.1.1
ping google.com
```

Em Linux, `ping` roda indefinidamente até ser interrompido com `Ctrl+C` (diferente do Windows, que por padrão envia só 4 pacotes).

## Descobrindo um host Linux na rede sem monitor

Um cenário comum: você tem uma máquina Linux (um servidor headless, uma placa embarcada, um controlador industrial) ligada na rede, mas sem monitor/teclado conectados, e precisa descobrir o IP para acessar via SSH.

Se o DHCP e o DNS estiverem configurados corretamente, muitas vezes basta `ping <hostname>` ou consultar a tabela de leases do roteador. Mas quando isso não é possível (rede sem DHCP, hostname desconhecido, ou o host ainda não tem IPv4 configurado), a técnica abaixo usando **IPv6 link-local** costuma funcionar mesmo em redes "cruas", porque todo host Linux moderno gera automaticamente um endereço IPv6 link-local ao subir a interface de rede — sem precisar de DHCP ou de nenhuma configuração prévia.

### O que é um endereço link-local

Endereços na faixa `fe80::/10` são **link-local**: válidos apenas dentro do mesmo segmento físico de rede (mesmo cabo/switch/VLAN), nunca roteados. Todo host com IPv6 habilitado atribui automaticamente um endereço desses a cada interface, com base no MAC address ou em um valor aleatório.

Como o mesmo endereço `fe80::...` pode existir simultaneamente em interfaces diferentes de uma mesma máquina Windows (Wi-Fi, Ethernet, VPN, etc.), o sistema operacional exige que você especifique **em qual interface** local usar esse endereço. É para isso que serve o sufixo `%<interface>` (chamado de "zone ID" ou "scope ID").

### Passo a passo a partir de um PC Windows

1. **Descubra o índice da interface Windows** que está fisicamente conectada à mesma rede do host Linux, usando `ipconfig`:

   ```
   ipconfig
   ```

   Procure a interface correta na saída e anote o número depois do `%` no endereço link-local:

   ```
   Ethernet adapter Ethernet 5:
      Connection-specific DNS Suffix . : example.com
      Link-local IPv6 Address . . . . . : fe80::5197:ef72:a352:b7f7%17
      IPv4 Address. . . . . . . . . . . : 172.17.42.17
      Subnet Mask . . . . . . . . . . . : 255.255.252.0
      Default Gateway . . . . . . . . . : 172.17.40.1
   ```

   Nesse exemplo, o índice da interface é `17`.

2. **Envie um ping multicast "all-nodes"** para descobrir quais hosts IPv6 respondem naquele segmento de rede. O endereço `ff02::1` é um multicast especial que todo host IPv6 escuta:

   ```
   ping ff02::1%17
   ```

   Se houver resposta, você verá algo como:

   ```
   Pinging ff02::1%17 with 32 bytes of data:
   Reply from fe80::201:5ff:fe50:5911: icmp_seq=1 ttl=64 time<1ms
   Reply from fe80::201:5ff:fe3d:6913: icmp_seq=1 ttl=64 time<1ms
   ```

   Dependendo do firewall da rede, o `ping` pode não retornar nada além do primeiro pacote enviado — isso não impede o próximo passo.

3. **Descubra o MAC address correspondente** usando o PowerShell, filtrando pela tabela de vizinhança (ARP/NDP) que o Windows já populou ao fazer o ping anterior:

   ```powershell
   Get-NetNeighbor -AddressFamily IPv6
   ```

   Ou filtrando por um prefixo de MAC conhecido (por exemplo, o fabricante do equipamento):

   ```powershell
   Get-NetNeighbor -LinkLayerAddress 00-01-05* -AddressFamily IPv6
   ```

   A saída lista pares de endereço MAC e IPv6 de todos os dispositivos vistos na rede. Identifique o host desejado (por exemplo, comparando o MAC com a etiqueta física do equipamento) e anote o endereço IPv6.

4. **Confirme que o host responde** ao ping direto, sempre incluindo o sufixo da interface local:

   ```
   ping fe80::201:5ff:fe3d:6913%17
   ```

5. **Conecte via SSH**, também com o sufixo `%<interface>`:

   ```
   ssh usuario@fe80::201:5ff:fe3d:6913%17
   ```

   Essa técnica funciona porque o SSH aceita endereços IPv6 com zone ID diretamente na linha de comando, tanto no Windows (OpenSSH nativo) quanto no PuTTY/Linux/macOS (com sintaxe de interface própria de cada um).

> Essa abordagem é útil sobretudo em bring-up de equipamentos novos (a máquina ainda não tem IPv4 fixo nem está cadastrada em nenhum DNS), mas também serve como ferramenta geral de diagnóstico de rede local.

## SSH básico

A sintaxe fundamental de conexão remota:

```bash
ssh usuario@host
```

Onde `host` pode ser um hostname, um IPv4 ou um IPv6 (com `%interface` quando for link-local, a partir do Windows).

Boas práticas ao configurar acesso remoto:

- **Troque a senha padrão** no primeiro acesso (`passwd`), principalmente em imagens de fábrica que vêm com usuário/senha conhecidos publicamente.
- **Prefira autenticação por chave SSH** em vez de senha, especialmente para acesso remoto e automatizado:

  ```bash
  ssh-keygen -t ed25519
  ssh-copy-id usuario@host
  ```

- Depois de confirmar que o login por chave funciona, desabilite login por senha em `/etc/ssh/sshd_config` (`PasswordAuthentication no`) e reinicie o serviço SSH.

## Configuração de IP estático com `systemd-networkd`

Distros Debian/Ubuntu modernas frequentemente usam `systemd-networkd` para gerenciar interfaces de rede (em vez de `/etc/network/interfaces` ou NetworkManager). A configuração é feita por arquivos `.network` colocados em `/etc/systemd/network/`.

### Por que criar o arquivo em `/etc/` e não editar o padrão de fábrica

Muitas distros já trazem uma configuração padrão de DHCP para interfaces Ethernet, normalmente em:

```
/usr/lib/systemd/network/20-wired.network
```

**Não edite esse arquivo diretamente.** A convenção do `systemd` é que arquivos em `/etc/systemd/network/` têm prioridade sobre os equivalentes em `/usr/lib/systemd/network/` — este último é gerenciado pelo pacote do sistema e pode ser sobrescrito em uma atualização. Qualquer personalização deve ir em `/etc/`, que nunca é tocado por atualizações de pacote.

### Passo a passo

1. Identifique o nome da interface que você quer configurar:

   ```bash
   ip addr show
   ```

   Nomes comuns em distros modernas seguem o padrão "predictable network interface names" (`enp3s0`, `eth0`, `end0`, etc.) — cada um se refere fisicamente a uma porta de rede diferente.

2. Crie um arquivo de configuração dedicado em `/etc/systemd/network/`, com um prefixo numérico que define a ordem de aplicação (menor número = maior prioridade):

   ```bash
   sudo nano /etc/systemd/network/10-eth0-static.network
   ```

3. Defina o conteúdo do arquivo com duas seções: `[Match]` (para selecionar a interface pelo nome) e `[Network]` (para os parâmetros de IP):

   ```ini
   [Match]
   Name=eth0

   [Network]
   Address=192.168.10.20/24
   Gateway=192.168.10.1
   DNS=8.8.8.8
   ```

   - `Address`: IP fixo com máscara em notação CIDR.
   - `Gateway`: rota padrão (opcional, se a interface não for a saída para a internet).
   - `DNS`: um ou mais servidores DNS (pode repetir a linha para adicionar mais de um).

4. Salve o arquivo e aplique a nova configuração sem precisar reiniciar a máquina:

   ```bash
   sudo networkctl reload
   ```

5. Verifique se a mudança foi aplicada corretamente:

   ```bash
   networkctl status          # status geral do serviço de rede e de cada interface
   networkctl status eth0     # status detalhado de uma interface específica
   ip addr show                # confirma o IP atribuído
   ip route show                # confirma a rota padrão
   ```

Se o `networkctl status` mostrar a interface como "unmanaged" ou o IP antigo continuar aparecendo, verifique se não existe outro serviço de rede (NetworkManager, `dhcpcd`) disputando o controle da mesma interface — normalmente só um gerenciador de rede deve estar ativo por vez.

## Ver também

- [Gerenciamento de pacotes (`apt`)](./gerenciamento-pacotes.md)
- [systemd e serviços](./systemd-e-servicos.md)
