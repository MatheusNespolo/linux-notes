# linux-notes

Repositório pessoal de conhecimento sobre **Linux em geral**, nascido a partir de experiências com **Beckhoff RT Linux** (automação industrial) e organizado para servir como referência tanto para o dia a dia com controladores industriais quanto para quem quer aprender/praticar Linux em casa.

## Estrutura

```
linux-notes/
├── docs/
│   ├── fundamentos/          # Conceitos gerais de Linux (qualquer distro Debian/Ubuntu-like)
│   └── beckhoff-rt-linux/    # Guias específicos do Beckhoff RT Linux
├── tools/                    # Ferramentas úteis (Rufus, Setup Helper, Setup Console)
└── projects/                 # Propostas de projetos hands-on para praticar em casa
```

## Fundamentos de Linux

Conceitos gerais, aplicáveis a qualquer distro baseada em Debian/Ubuntu, extraídos e generalizados da experiência com Beckhoff RT Linux:

- [Redes](docs/fundamentos/redes.md) — diagnóstico de rede, descoberta de host via IPv6 link-local, SSH, IP estático com `systemd-networkd`.
- [Gerenciamento de pacotes](docs/fundamentos/gerenciamento-pacotes.md) — `apt`, repositórios, autenticação em repositórios privados.
- [systemd e serviços](docs/fundamentos/systemd-e-servicos.md) — `systemctl`, `journalctl`, unidades de serviço.

## Beckhoff RT Linux

Guias específicos para quem trabalha com controladores industriais Beckhoff:

- [Instalação](docs/beckhoff-rt-linux/instalacao.md) — USB bootável com Rufus, BIOS, instalação do sistema.
- [Primeiros passos](docs/beckhoff-rt-linux/primeiros-passos.md) — descoberta de IP, IP estático, Package Manager, TwinCAT, EtherCAT.
- [Publisher/Subscriber](docs/beckhoff-rt-linux/publisher-subscriber.md) — contexto sobre comunicação via EtherCAT Automation Protocol (referência ao guia completo de TwinCAT no Obsidian Vault pessoal).

## Tools

- [tools/README.md](tools/README.md) — manual de bolso do **Rufus** (criação de imagem bootável), do **Beckhoff Linux Setup Helper** e do **Setup Console**, com pré-requisitos, disclaimers e quando usar cada um.

## Projetos hands-on

Propostas de projetos para praticar Linux em casa, do iniciante ao avançado:

- [projects/README.md](projects/README.md) — índice completo por nível de dificuldade.
- [Servidor de arquivos (NAS caseiro)](projects/servidor-arquivos.md) — Iniciante
- [Bloqueador de anúncios com Pi-hole](projects/pi-hole-dns.md) — Iniciante
- [Automação residencial](projects/automacao-residencial.md) — Intermediário
- [Servidor de mídia (Jellyfin)](projects/servidor-midia.md) — Intermediário
- [Backup automatizado](projects/backup-automatizado.md) — Intermediário
- [Servidor VPN (WireGuard)](projects/servidor-vpn.md) — Avançado

## Origem

Este repositório reorganiza e generaliza conteúdo originalmente escrito como notas soltas no meu [Obsidian Vault](../ObsidianVault), consolidando em um único lugar o que é conhecimento geral de Linux e o que é específico de Beckhoff RT Linux/TwinCAT.
