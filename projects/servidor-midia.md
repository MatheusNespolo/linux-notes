# Servidor de mídia doméstico com Jellyfin

> **Nível:** Intermediário

## Objetivo

Centralizar filmes, séries, músicas e fotos em um servidor doméstico e assistir/ouvir a partir de qualquer TV, celular ou computador da casa (e, opcionalmente, de fora dela), sem depender de um serviço de streaming pago para o conteúdo que você já possui. O Jellyfin é a alternativa 100% open-source ao Plex/Emby, sem trava de recursos atrás de assinatura.

## Hardware sugerido

- Mini PC ou Raspberry Pi 4/5 com bom desempenho de CPU, especialmente se for necessário fazer transcodificação de vídeo em tempo real (converter o formato do vídeo na hora para o dispositivo que está assistindo). Se possível, algo com aceleração de vídeo por hardware.
- Armazenamento generoso (HD externo ou o mesmo disco do servidor de arquivos) para guardar a biblioteca de mídia.

## Software

- **Jellyfin**: servidor de mídia open-source, com apps para TV, celular, navegador e integração com clientes como Kodi.
- **Docker**: forma recomendada de instalar o Jellyfin, isolando a aplicação e facilitando atualizações.

## Passo a passo (visão geral)

1. Instalar o Docker no servidor Linux (se ainda não tiver, ver o projeto de servidor de arquivos para uma base comum de sistema).
2. Organizar a biblioteca de mídia em pastas com uma convenção de nomes reconhecida pelo Jellyfin (ex: `Filmes/Nome do Filme (Ano)/arquivo.mkv`, `Séries/Nome/Temporada 01/...`).
3. Subir o container do Jellyfin apontando para essas pastas como volumes:
   ```bash
   docker run -d --name jellyfin \
     -p 8096:8096 \
     -v /mnt/dados/config-jellyfin:/config \
     -v /mnt/dados/filmes:/media/filmes \
     -v /mnt/dados/series:/media/series \
     jellyfin/jellyfin
   ```
4. Acessar `http://ip-do-servidor:8096`, concluir o assistente inicial (criar usuário admin, apontar as bibliotecas de mídia) e deixar o Jellyfin escanear os arquivos.
5. Instalar o app do Jellyfin no celular/TV ou acessar via navegador e testar a reprodução de um vídeo, de preferência forçando um cenário de transcodificação (formato não suportado nativamente pelo dispositivo) para avaliar se o hardware aguenta.

## Próximos passos / evolução

- Configurar acesso externo seguro (via VPN WireGuard, de preferência, em vez de expor a porta diretamente na internet).
- Automatizar a organização de novos arquivos de mídia com scripts ou ferramentas como o *Sonarr/Radarr* (fora do escopo inicial, mas é o caminho natural de evolução).
- Adicionar múltiplos perfis de usuário e controle de conteúdo por perfil (ex: perfil infantil).
- Migrar a biblioteca para o mesmo array RAID do servidor de arquivos, se os dois projetos forem consolidados na mesma máquina.
