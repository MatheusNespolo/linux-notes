# Tools — Utilitários para Beckhoff RT Linux

Manual de bolso das ferramentas usadas no dia a dia para instalar, configurar e
colocar em produção controladores industriais com Beckhoff RT Linux.

Os binários em si **não fazem parte deste repositório** (são executáveis
grandes, alguns proprietários, e não fazem sentido versionados em git). Este
arquivo documenta o que cada ferramenta faz, onde obtê-la e como usá-la. Na
máquina do autor, essas ferramentas vivem em:

```
C:\Users\matheusn\Documents\Imagens Linux\Tools\
├── BeckhoffLinuxSetupHelper\
│   ├── BeckhoffSetupHelper.exe
│   ├── README.md
│   └── THIRD-PARTY-NOTICES.md
└── Setup Console\
    ├── Linux Setup Console.lnk
    └── linux-ipc-setup-console-3.2.1-x64.exe
```

Trate esse caminho como referência local — se você está reproduzindo este
setup em outra máquina, baixe as ferramentas nas fontes indicadas abaixo.

---

## Rufus

**O que é:** ferramenta gratuita para criar pendrives USB "bootáveis" a partir
de um arquivo de imagem (`.img`/`.iso`). É o que se usa para gravar a imagem
do Beckhoff RT Linux em um USB antes de instalar o sistema em um PC
industrial.

**Download:** [rufus.ie](https://rufus.ie/)

> [!WARNING]
> Versões mais recentes do Rufus podem **não ser compatíveis** com o
> upload/download da imagem do Beckhoff RT Linux. A versão recomendada e
> testada é a **3.13**. Prefira baixar essa versão específica em vez da mais
> recente disponível no site.

### Passo a passo resumido

1. Abra o Rufus em um PC com Windows.
2. Em **"Select"**, escolha o arquivo de imagem do Beckhoff RT Linux a ser
   gravado.
3. Em **"Device"**, selecione o pendrive USB de destino (mínimo 2 GB).
4. Clique em **"Start"** e aguarde a gravação terminar.

### Nota sobre PCs com processador Arm

Em controladores com processador Arm, não existe boot via USB do mesmo jeito:
a imagem do sistema é copiada **diretamente no cartão microSD** do
dispositivo. O Rufus também é usado para isso — o procedimento é o mesmo
(Select → Device → Start), só que o "Device" de destino é o leitor de cartão
microSD em vez do pendrive.

---

## Beckhoff Linux Setup Helper

**O que é:** uma ferramenta gráfica (GUI), construída em .NET 8, para
orquestrar o deploy e a configuração de controladores Beckhoff RT Linux —
automatiza tarefas repetitivas de pós-instalação (rede, apt, TwinCAT, etc.)
que, de outra forma, seriam feitas manualmente via SSH.

### Pré-requisitos

- Windows 10/11 com **.NET 8 Runtime** instalado.
- Acesso de rede do host Windows ao controlador Beckhoff alvo.
- Sistema operacional alvo: Beckhoff RT Linux (baseado em Debian).

### Disclaimer

> [!CAUTION]
> Esta é uma ferramenta de terceiros, fornecida "como está" (AS-IS), sem
> garantias. O uso é de responsabilidade exclusiva de quem executa. Não é
> afiliada, endossada ou patrocinada pela Beckhoff Automation GmbH & Co. KG.
> "TwinCAT", "EtherCAT" e "Beckhoff" são marcas registradas da Beckhoff
> Automation GmbH & Co. KG.

### Como obter/rodar

- Executável: `BeckhoffSetupHelper.exe` (não versionado neste repositório).
- Basta rodar o `.exe` em uma máquina Windows com .NET 8 Runtime instalado —
  não requer instalação prévia.
- Consulte o `README.md` e o `THIRD-PARTY-NOTICES.md` que acompanham a
  ferramenta para detalhes de licenciamento de bibliotecas de terceiros
  usadas internamente.

---

## Setup Console (`linux-ipc-setup-console`)

**O que é:** console de configuração oficial para os IPCs (Industrial PCs) da
Beckhoff que rodam Linux. Usado para tarefas de setup inicial do dispositivo
diretamente a partir do Windows.

**Executável de referência:** `linux-ipc-setup-console-3.2.1-x64.exe`

### Como executar

1. Conecte o PC industrial à mesma rede do host Windows (ou via cabo direto,
   conforme a topologia usada).
2. Execute o `.exe` (ou o atalho `Linux Setup Console.lnk`, se disponível).
3. Siga o fluxo da ferramenta para localizar o dispositivo na rede e aplicar
   a configuração desejada.

---

## Resumo

| Ferramenta | Função | Quando usar |
|---|---|---|
| **Rufus** (v3.13) | Grava a imagem do Beckhoff RT Linux em USB (ou microSD, em PCs Arm) | Antes de qualquer instalação nova de Beckhoff RT Linux em um PC industrial |
| **Beckhoff Linux Setup Helper** | GUI que orquestra deploy/configuração pós-instalação (rede, apt, TwinCAT) | Para automatizar a configuração de um ou mais controladores já com o SO instalado |
| **Setup Console** | Console oficial de setup dos IPCs Beckhoff Linux | Para configuração inicial rápida de um IPC recém-instalado |

Para o passo a passo completo de instalação usando essas ferramentas, veja
[`../docs/beckhoff-rt-linux/instalacao.md`](../docs/beckhoff-rt-linux/instalacao.md).
