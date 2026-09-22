# Servidor VPN pessoal com WireGuard

> **Nível:** Avançado

## Objetivo

Acessar a rede de casa remotamente (arquivos, câmeras, Home Assistant, Jellyfin, etc.) de forma segura e criptografada de ponta a ponta, sem precisar expor cada serviço individualmente na internet. Esse é o projeto que mais mexe com conceitos de rede "de verdade" — NAT, roteamento, firewall e criptografia de chave pública — então vale fazer depois de já ter pelo menos um outro serviço rodando em casa para "proteger".

## Hardware sugerido

- Pode rodar no mesmo Raspberry Pi/mini PC que já hospeda os outros serviços, já que o WireGuard é extremamente leve.
- Requisito principal não é hardware, e sim ter acesso ao roteador de casa para configurar redirecionamento de porta (port forwarding).

## Software

- **WireGuard**: VPN moderna, com implementação enxuta (poucas milhares de linhas de código, o que facilita auditoria) e desempenho melhor que soluções mais antigas como OpenVPN.

## Passo a passo (visão geral)

1. Instalar o WireGuard no servidor:
   ```bash
   sudo apt install wireguard
   ```
2. Gerar o par de chaves (privada/pública) do servidor:
   ```bash
   wg genkey | tee privatekey | wg pubkey > publickey
   ```
3. Criar o arquivo de configuração do servidor (`/etc/wireguard/wg0.conf`) definindo o IP virtual da VPN (ex: `10.10.0.1/24`), a porta de escuta (ex: `51820/UDP`) e a chave privada.
4. Gerar um par de chaves para cada dispositivo cliente (celular, notebook) e adicionar cada um como um bloco `[Peer]` na configuração do servidor, com seu IP virtual correspondente.
5. Subir a interface e habilitar no boot:
   ```bash
   sudo wg-quick up wg0
   sudo systemctl enable wg-quick@wg0
   ```
6. No roteador de casa, configurar o redirecionamento (port forward) da porta UDP escolhida para o IP interno do servidor.
7. Configurar o app do WireGuard no celular/notebook com a configuração de peer correspondente (via QR code ou arquivo `.conf`) e testar a conexão fora da rede de casa (ex: usando dados móveis).
8. Validar que, conectado à VPN, é possível acessar os outros serviços da casa (servidor de arquivos, Jellyfin, Home Assistant) pelo IP interno da rede.

## Próximos passos / evolução

- Restringir no firewall (`ufw`/`iptables`) quais portas/serviços cada peer pode acessar, em vez de dar acesso total à rede interna.
- Configurar `AllowedIPs` mais granular por peer, se quiser isolar o acesso de alguns dispositivos.
- Trocar a porta padrão e monitorar tentativas de conexão para reduzir superfície de ataque.
- Combinar com o Pi-hole para que os dispositivos conectados à VPN também tenham bloqueio de anúncios/rastreadores fora de casa.
