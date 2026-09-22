# Bloqueador de anúncios/DNS local com Pi-hole

> **Nível:** Iniciante

## Objetivo

Bloquear anúncios e rastreadores para todos os dispositivos da rede doméstica (não só um navegador) através de um servidor DNS local que filtra domínios indesejados antes que a requisição saia para a internet. É um projeto pequeno e rápido, mas que já ensina bastante sobre DNS e sobre como serviços de rede rodam em segundo plano em um Linux.

## Hardware sugerido

- Qualquer coisa serve: um Raspberry Pi Zero/Zero 2 W já é suficiente, já que o Pi-hole consome pouquíssimo recurso.
- Também funciona bem como um segundo serviço rodando no mesmo dispositivo do servidor de arquivos ou de automação residencial, se preferir consolidar.

## Software

- **Pi-hole**: aplicação que atua como servidor DNS local, filtrando domínios de anúncios/rastreamento com base em listas de bloqueio (blocklists) atualizáveis.

## Passo a passo (visão geral)

1. Instalar um Linux leve no dispositivo (Raspberry Pi OS Lite).
2. Instalar o Pi-hole com o script oficial de instalação:
   ```bash
   curl -sSL https://install.pi-hole.net | bash
   ```
   (vale a pena ler o script antes de rodar, já que ele pede privilégios de root — é um bom exercício de desconfiar de "curl | bash" por padrão.)
3. Durante a instalação, anotar o IP fixo do dispositivo na rede e a senha gerada para o painel web.
4. Acessar o painel web (`http://ip-do-pihole/admin`) e conferir que o serviço está rodando e coletando consultas DNS.
5. Configurar o roteador de casa para distribuir o IP do Pi-hole como servidor DNS primário via DHCP, de forma que todos os dispositivos da rede passem a usá-lo automaticamente.
6. Testar acessando algum site com bastante anúncios/rastreadores e observar no painel do Pi-hole quantas requisições foram bloqueadas.

## Próximos passos / evolução

- Adicionar listas de bloqueio extras (ex: listas anti-telemetria, anti-malware).
- Configurar um segundo Pi-hole (em outro dispositivo) como DNS secundário, para redundância caso o primeiro caia.
- Habilitar DNS sobre HTTPS/TLS entre o Pi-hole e os servidores DNS upstream, para mais privacidade.
- Combinar com o servidor VPN (WireGuard) para ter a proteção do Pi-hole também quando estiver fora de casa.
