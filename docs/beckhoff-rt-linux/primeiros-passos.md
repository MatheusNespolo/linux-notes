# Primeiros passos com o Beckhoff RT Linux

Depois de instalado (veja [`instalacao.md`](instalacao.md)), estes são os
primeiros passos práticos para deixar o controlador utilizável: descobrir o
IP, fixar um endereço estático, configurar o gerenciador de pacotes, instalar
o TwinCAT e preparar a interface de rede para EtherCAT.

## 1. Determinar o endereço IP

### Com monitor

Se o equipamento permite conectar um monitor local (ex.: CX9240), faça login
no terminal com as credenciais padrão:

- **Usuário:** `Administrator`
- **Senha:** `1`

> Troque essa senha padrão assim que possível por questões de segurança.

No terminal, liste os adaptadores de rede:

```bash
ip addr show
```

Por padrão, o adaptador Ethernet recebe IP via DHCP (ex.: `169.254.199.63`).
Com esse IP em mãos, já é possível conectar via SSH.

### Sem monitor (via SSH a partir do Windows)

Quando não há monitor local, é possível localizar o dispositivo na rede
usando IPv6 link-local a partir de um PC Windows.

1. Identifique a interface de rede do Windows com `ipconfig` e anote o número
   da interface (o final `%xx` do endereço link-local). Exemplo de saída:

   ```
   Ethernet adapter Ethernet 5:
      Connection-specific DNS Suffix . : example.com
      Link-local IPv6 Address . . . . : fe80::5197:ef72:a352:b7f7%17
      IPv4 Address. . . . . . . . . . : 172.17.42.17
      Subnet Mask . . . . . . . . . . : 255.255.252.0
      Default Gateway . . . . . . . . : 172.17.40.1
   ```

   Aqui, o número da interface é `17`.

2. Faça ping no endereço multicast link-local para descobrir dispositivos
   IPv6 na rede local (substitua `17` pelo número da sua interface):

   ```powershell
   ping ff02::1%17
   ```

   Uma resposta bem-sucedida traz várias respostas, uma por dispositivo:

   ```
   Pinging ff02::1%10 with 32 bytes of data:
   Reply from fe80::201:5ff:fe50:5911: icmp_seq=1 ttl=64 time<1ms
   Reply from fe80::201:5ff:fe3d:6913: icmp_seq=1 ttl=64 time<1ms
   ```

   Timeouts nessa etapa não são necessariamente um problema — pode seguir
   para o próximo comando.

3. Encontre o endereço MAC do dispositivo alvo com o PowerShell:

   ```powershell
   Get-NetNeighbor -LinkLayerAddress 00-01-05* -AddressFamily IPv6
   ```

   Isso lista os endereços MAC/IPv6 de dispositivos na rede cujo MAC comece
   com `00-01-05`. Identifique o Beckhoff RT Linux pelo MAC impresso na
   etiqueta ("name plate") do PC industrial e anote o IPv6 correspondente.

4. Confirme que o dispositivo está acessível (troque `%??` pelo número da sua
   interface):

   ```powershell
   ping fe80::201:5ff:fe3d:6913%??
   ```

5. Conecte via SSH:

   ```powershell
   ssh Administrator@fe80::201:5ff:fe3d:6913%??
   ```

   A conexão pode usar o algoritmo MAC (Message Authentication Code)
   `hmac-sha2-512-etm@openssh.com`, que garante integridade e autenticidade
   dos dados transmitidos via SHA-512.

## 2. Fixar um endereço IP estático

O IP estático é definido por um arquivo de configuração em
`/etc/systemd/network/`. Os parâmetros da interface (endereço, gateway, DNS)
ficam nesse arquivo, e o serviço `systemd-networkd` os aplica
automaticamente.

> **Importante:** não edite `/usr/lib/systemd/network/20-wired.network` — é a
> configuração padrão (DHCP) pré-instalada. Arquivos em
> `/etc/systemd/network/` têm precedência e sobrescrevem essa configuração,
> então crie um arquivo próprio em vez de alterar o padrão.

Procedimento:

1. Liste as interfaces Ethernet disponíveis:

   ```bash
   ip addr show
   ```

   Exemplos de nomes: `lo`, `end0`, `end1`.

2. Crie um arquivo de configuração para a interface desejada:

   ```bash
   sudo nano /etc/systemd/network/10-end0-static.network
   ```

3. Preencha com os parâmetros da rede, ajustando conforme necessário:

   ```ini
   [Match]
   Name=end0

   [Network]
   Address=192.168.10.20/24
   ```

4. Salve e feche o arquivo.

5. Recarregue a configuração de rede:

   ```bash
   sudo networkctl reload
   ```

6. Verifique se as mudanças foram aplicadas:

   ```bash
   networkctl status     # status do serviço de rede
   ip addr show          # parâmetros das interfaces
   ip route show         # rotas
   ```

## 3. Configurar o Package Manager (apt)

O Beckhoff RT Linux usa `apt` para instalar pacotes, hospedados nos
"Package Servers" da Beckhoff, que exigem autenticação com uma conta
myBeckhoff (crie uma em myBeckhoff caso ainda não tenha).

### Arquivo de autenticação para o apt

O diretório `/etc/apt/auth.conf.d/` guarda arquivos de autenticação lidos
automaticamente pelo `apt`. Crie o arquivo `bhf.conf`:

```bash
sudo nano /etc/apt/auth.conf.d/bhf.conf
```

Conteúdo (substitua pelas suas credenciais myBeckhoff):

```
machine deb.beckhoff.com
login example@mail.com
password xyz123

machine deb-mirror.beckhoff.com
login example@mail.com
password xyz123
```

- **machine**: nome do Package Server (`deb.beckhoff.com`,
  `deb-mirror.beckhoff.com`).
- **login / password**: credenciais da conta myBeckhoff.

Salve com `Ctrl+O` e saia do editor com `Ctrl+X`.

### Apontar para a área instável (opcional, uso em desenvolvimento)

Para obter pacotes de uma área instável/testing do repositório Beckhoff
durante desenvolvimento, edite:

```bash
sudo nano /etc/apt/sources.list.d/bhf.list
```

Adicione a linha:

```
https://deb.beckhoff.com/debian trixie-testing main
```

Ao rodar `apt update` ou `apt install`, o `apt` valida as credenciais no
arquivo de autenticação e baixa os pacotes do Package Server correspondente.

## 4. Instalar o TwinCAT

Com a autenticação e as fontes configuradas, atualize o sistema:

```bash
sudo apt update
```

Veja quais pacotes podem ser atualizados:

```bash
apt list --upgradable
```

Instale o TwinCAT:

```bash
sudo apt install tc31-xar-um
```

## 5. Configurar a interface de rede para EtherCAT

Liste os dispositivos Ethernet real-time disponíveis para uso com EtherCAT:

```bash
sudo TcRteInstall -h    # ajuda e exemplos
sudo TcRteInstall -l    # lista interfaces e vínculo com o RT Ethernet driver
sudo TcRteInstall --bind []
```

Somente dispositivos não usados para Ethernet "normal" aparecem na lista.
Selecione a interface real-time correspondente e vincule.

Reinicie para aplicar as mudanças:

```bash
sudo shutdown -r now
```

Depois do reboot, o Beckhoff RT Linux pode ser localizado e definido como
"target system" no TwinCAT.

> **Nota:** por padrão, o Beckhoff RT Linux só permite **rotas ADS seguras**.
> Crie uma rota ADS segura para o PC de engenharia antes de tentar conectar
> como target system.
