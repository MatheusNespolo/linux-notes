# Comunicação Publisher/Subscriber (EtherCAT Automation Protocol)

Controladores Beckhoff RT Linux suportam troca de dados com outros
dispositivos Beckhoff (incluindo plataformas TwinCAT/BSD) através do padrão
**Publisher/Subscriber**, implementado sobre o **EtherCAT Automation
Protocol (Network Variables)** dentro de um projeto TwinCAT. Nesse modelo, um
controlador publica variáveis de saída na rede (Broadcast, Multicast ou
Unicast) e outro controlador as assina ("subscribe"), recebendo os valores
como entradas em seu próprio programa de PLC — sem exigir um mestre/escravo
EtherCAT tradicional entre eles.

Esse mecanismo é configurado inteiramente dentro do ambiente de engenharia
TwinCAT (declaração de variáveis com sufixo `AT %Q*`/`AT %I*`, criação do
dispositivo "EtherCAT Automation Protocol" no Publisher e no Subscriber,
seleção da interface de rede correta e, no Subscriber, a busca pelo
dispositivo Publisher via rota ADS). Por ser um tópico de engenharia
TwinCAT/PLC — e não de administração do sistema operacional Linux em si —,
ele foge do escopo deste repositório, que trata do Linux embarcado da
Beckhoff (rede, apt, systemd, etc.).

Um ponto relevante mesmo do ponto de vista de infraestrutura: como o
Beckhoff RT Linux só aceita rotas ADS seguras por padrão, é preciso
estabelecer uma rota ADS segura e estável entre o PC de engenharia e os dois
controladores (Publisher e Subscriber) antes de a comunicação funcionar —
isso pode ser feito com `adstool`, por exemplo:

```bash
adstool {IP} addroute --addr={IP} --netid={AMS NetId} --password=*
```

Além disso, é preciso garantir disponibilidade de rede adequada (firewalls,
IPs, roteadores e gateways configurados para permitir tráfego TCP/UDP
livremente entre os dispositivos envolvidos).

O passo a passo completo de configuração no TwinCAT (usando como exemplo os
controladores CX8290 rodando Beckhoff RT Linux e CX5240 rodando TwinCAT/BSD)
está documentado separadamente no Obsidian Vault pessoal do autor:

> Ver nota completa no Obsidian Vault:
> [`Connectivity/Publisher-Subscriber (Beckhoff RT Linux - CX8290).md`](../../../ObsidianVault/Connectivity/Publisher-Subscriber%20(Beckhoff%20RT%20Linux%20-%20CX8290).md)
