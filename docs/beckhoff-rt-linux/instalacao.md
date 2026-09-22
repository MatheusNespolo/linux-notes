# Instalação do Beckhoff RT Linux

Guia de instalação do Beckhoff RT Linux em um PC industrial, cobrindo a
criação da mídia de boot, o ajuste de BIOS e o procedimento de instalação em
si.

## 1. Criar um USB bootável

Antes de instalar o Beckhoff RT Linux em um PC industrial, é preciso criar um
USB "bootável" com a imagem do sistema operacional.

### Pré-requisitos

- Ferramenta **Rufus** (recomendada a versão **3.13** — versões mais novas
  podem não ser compatíveis com o upload/download dessa imagem). Veja
  detalhes de download e uso em [`../../tools/README.md`](../../tools/README.md#rufus).
- Arquivo de imagem do Beckhoff RT Linux.
- Pendrive USB com pelo menos 2 GB de armazenamento.

### Procedimento

1. Inicie o Rufus em um PC com Windows.
2. Em **"Select"**, selecione o arquivo de imagem a ser gravado.
3. Em **"Device"**, selecione o pendrive USB de destino.
4. Clique em **"Start"** para iniciar a gravação.

> **PCs com processador Arm:** nesses casos não há boot via USB — a imagem é
> copiada diretamente no cartão microSD do dispositivo, também usando o
> Rufus (mesmo fluxo Select → Device → Start, apontando para o leitor de
> microSD). Mais detalhes em [`../../tools/README.md`](../../tools/README.md#nota-sobre-pcs-com-processador-arm).

## 2. Checar as configurações de BIOS

Antes de dar boot pelo USB, confirme que a BIOS do PC industrial está
configurada para permitir isso.

Para o Beckhoff RT Linux, o modo de boot precisa ser **UEFI** ou **Dual
Boot** (use Dual Boot se quiser alternar entre sistemas operacionais
diferentes instalados em mídias distintas).

Procedimento:

1. Reinicie o PC industrial e pressione **Del** para entrar na configuração
   da BIOS.
2. Vá em **Boot > "Boot mode select"** e escolha **UEFI** ou **DUAL**.
3. Pressione **F4** para salvar e sair da BIOS.

## 3. Instalar o Beckhoff RT Linux

Com o USB bootável em mãos e a BIOS configurada, conecte o USB ao PC
industrial e inicie a instalação.

### Pré-requisitos

- USB bootável com a imagem do Beckhoff RT Linux (passo 1).
- Cartão/unidade de memória interna com pelo menos 4 GB de espaço livre.

### Procedimento

1. Conecte o USB ao PC industrial.
2. Ligue o PC e pressione **F7** para abrir o menu de boot.
3. Selecione a entrada UEFI referente ao USB e confirme com **Enter**.
4. Selecione a opção **"TC/LUR Install"** para instalar a imagem.
5. Crie a senha do usuário administrador (`Administrator`) quando solicitado
   e siga as instruções na tela.
6. Reinicie o PC — ele deve subir com a imagem recém-instalada.

Com o sistema instalado, siga para
[`primeiros-passos.md`](primeiros-passos.md) para configurar rede, apt e o
TwinCAT.
