# Gerenciamento de pacotes com `apt`

Fundamentos de gerenciamento de pacotes em distros baseadas em Debian (Debian, Ubuntu, Raspberry Pi OS, e qualquer derivada) usando a ferramenta `apt` (Advanced Package Tool).

## O que é o `apt` e o conceito de repositórios

`apt` é a ferramenta de linha de comando usada para instalar, atualizar e remover pacotes de software em distros Debian-like. Ele não baixa pacotes de qualquer lugar: consulta uma lista de **repositórios** configurados na máquina — servidores HTTP/HTTPS que hospedam pacotes `.deb` já compilados, junto com metadados (versões, dependências, assinaturas de segurança).

Cada repositório é identificado por uma URL, uma distribuição (ex.: `bookworm`, `trixie`, `jammy`) e uma ou mais "áreas" (ex.: `main`, `contrib`, `non-free`, ou nomes específicos do fornecedor). Repositórios podem ser:

- **Oficiais**, mantidos pela própria distro (Debian, Ubuntu).
- **De terceiros**, mantidos por fabricantes de software ou hardware que distribuem pacotes próprios (ex.: repositórios de fornecedores de drivers, ferramentas de automação industrial, ou softwares proprietários).

## Onde os repositórios são configurados

### `/etc/apt/sources.list`

Arquivo principal que lista os repositórios "de fábrica" da distro. Formato de cada linha (formato clássico "one-line"):

```
deb https://deb.debian.org/debian bookworm main contrib non-free
```

Em distros mais recentes, esse arquivo pode estar no formato `.sources` (DEB822), mais legível:

```
Types: deb
URIs: https://deb.debian.org/debian
Suites: bookworm
Components: main contrib non-free
```

Em geral, **não é necessário editar esse arquivo manualmente** — ele é gerenciado pela distro.

### `/etc/apt/sources.list.d/`

Diretório onde ficam repositórios adicionais, um arquivo por fornecedor/repositório. É o lugar correto para adicionar um repositório de terceiros sem mexer no arquivo principal do sistema.

Exemplo: para adicionar um repositório de um fornecedor `exemplo.com`, crie um arquivo dedicado:

```bash
sudo nano /etc/apt/sources.list.d/exemplo.list
```

Com conteúdo como:

```
deb https://deb.exemplo.com/debian trixie main
```

Ou, apontando para uma área de testes/desenvolvimento do mesmo fornecedor (comum quando se quer acesso a versões mais recentes/instáveis de um pacote antes do lançamento oficial):

```
deb https://deb.exemplo.com/debian trixie-testing main
```

Cada arquivo `.list` (ou `.sources`) dentro desse diretório é lido automaticamente pelo `apt` — não é preciso registrá-lo em nenhum outro lugar.

## Autenticação em repositórios privados

Alguns repositórios (normalmente de fornecedores comerciais) exigem login antes de permitir o download dos pacotes. As credenciais não vão dentro do arquivo `.list` — ficam separadas, em arquivos de autenticação.

### `/etc/apt/auth.conf.d/`

Diretório destinado a arquivos de credenciais no formato usado pelo `netrc`. Todo arquivo `.conf` dentro dessa pasta é lido automaticamente pelo `apt` ao tentar acessar um repositório.

Crie um arquivo dedicado para o fornecedor:

```bash
sudo nano /etc/apt/auth.conf.d/exemplo.conf
```

Com o formato:

```
machine deb.exemplo.com
login usuario@exemplo.com
password minha-senha-ou-token

machine deb-mirror.exemplo.com
login usuario@exemplo.com
password minha-senha-ou-token
```

Onde:

- **`machine`**: o hostname do repositório (deve bater exatamente com o host usado na URL do arquivo `.list`/`.sources`). Pode haver múltiplos blocos `machine` no mesmo arquivo, um para cada host (ex.: repositório principal e espelho/mirror).
- **`login`**: usuário da conta cadastrada junto ao fornecedor do repositório.
- **`password`**: senha ou token de acesso.

Quando você roda `apt update` ou `apt install`, o `apt` identifica automaticamente que aquele repositório exige autenticação, procura um bloco `machine` correspondente nos arquivos de `/etc/apt/auth.conf.d/` e usa as credenciais encontradas — sem precisar de nenhuma flag extra na linha de comando.

## Comandos essenciais

```bash
sudo apt update
```
Atualiza a lista local de pacotes disponíveis, consultando todos os repositórios configurados (não instala nada, só sincroniza metadados). Deve ser rodado sempre depois de adicionar/alterar um repositório.

```bash
apt list --upgradable
```
Lista os pacotes já instalados que têm uma versão mais nova disponível nos repositórios configurados.

```bash
sudo apt install <pacote>
```
Instala um pacote (e suas dependências) a partir dos repositórios configurados.

```bash
sudo apt upgrade
```
Atualiza todos os pacotes instalados para suas versões mais recentes disponíveis, sem remover ou instalar pacotes adicionais (para isso existe `apt full-upgrade`/`apt dist-upgrade`).

## Boas práticas de segurança

- **Permissões do arquivo de autenticação**: arquivos em `/etc/apt/auth.conf.d/` contêm senhas em texto plano. Restrinja a leitura só ao root:

  ```bash
  sudo chmod 600 /etc/apt/auth.conf.d/exemplo.conf
  sudo chown root:root /etc/apt/auth.conf.d/exemplo.conf
  ```

- **Prefira tokens de acesso a senhas de conta**, quando o fornecedor oferecer essa opção — um token pode ser revogado individualmente sem trocar a senha da conta principal.
- **Evite apontar para áreas de teste/instáveis** (`testing`, `unstable`, `nightly`) em ambientes de produção — use-as apenas durante desenvolvimento, e volte para uma área estável antes de ir para produção.
- **Confie apenas em repositórios de fontes conhecidas.** Adicionar um repositório de terceiros dá a esse fornecedor a capacidade de instalar software com privilégios de root na sua máquina.
- **Não versione** arquivos de `/etc/apt/auth.conf.d/` em repositórios Git nem os inclua em backups não criptografados.

## Ver também

- [Redes em Linux](./redes.md)
- [systemd e serviços](./systemd-e-servicos.md)
