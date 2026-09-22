# Servidor de automação residencial

> **Nível:** Intermediário

## Objetivo

Controlar luzes, tomadas inteligentes, sensores e outros dispositivos da casa a partir de um servidor Linux local, sem depender de nuvens de terceiros para automações básicas funcionarem. É um projeto interessante para quem já trabalha com automação industrial, porque os conceitos se repõem em outra escala: dispositivos de campo (sensores/atuadores), um "controlador" central que roda lógica, e um protocolo de comunicação padronizado entre eles.

## Hardware sugerido

- Raspberry Pi 4 (2 GB ou mais) dedicado só para essa função, rodando o tempo todo.
- Opcional: um dongle Zigbee (ex: Sonoff Zigbee 3.0 USB Dongle) ou Z-Wave, caso queira usar sensores/lâmpadas desses padrões em vez de só Wi-Fi.

## Software

- **Home Assistant**: plataforma open-source de automação residencial, com interface web, painel (dashboard) customizável e suporte a milhares de integrações (lâmpadas, tomadas, sensores, previsão do tempo, etc.). Pode ser instalado como Home Assistant OS (imagem dedicada) ou via Docker em cima de um Linux já existente.
- **Node-RED**: ferramenta de automação baseada em fluxos visuais (blocos conectados por fios), ótima para criar lógicas mais elaboradas ("se o sensor de presença disparar à noite, acender a luz do corredor"). Pode rodar sozinho ou integrado ao Home Assistant.
- **Mosquitto**: broker MQTT, o "barramento de mensagens" que permite que dispositivos e o Home Assistant troquem informações de forma leve e desacoplada — parecido em espírito com um barramento de campo, mas via rede IP.

## Passo a passo (visão geral)

1. Instalar o Home Assistant. As duas rotas mais comuns:
   - **Home Assistant OS**: gravar a imagem oficial no cartão SD do Raspberry Pi e acessar via `http://homeassistant.local:8123` depois do boot.
   - **Via Docker** em cima de um Raspberry Pi OS já instalado, se preferir manter controle total do SO:
     ```bash
     docker run -d --name homeassistant --restart=unless-stopped \
       -v /home/pi/homeassistant:/config --network=host \
       ghcr.io/home-assistant/home-assistant:stable
     ```
2. Instalar o Mosquitto (broker MQTT) como add-on do Home Assistant ou via `apt`/Docker separadamente.
3. Escolher um dispositivo de exemplo para integrar primeiro. Opções comuns para começar:
   - Uma tomada inteligente com firmware **Tasmota** ou **ESPHome** (baseada em ESP8266/ESP32), que publica seu estado via MQTT.
   - Um sensor Zigbee (ex: sensor de porta/presença) pareado através do dongle Zigbee, usando a integração Zigbee2MQTT ou ZHA do Home Assistant.
4. No Home Assistant, criar uma automação simples usando a interface (ex: "às 18h, ligar a tomada da luminária") e validar que o dispositivo responde.
5. Opcionalmente, instalar o Node-RED (como add-on do Home Assistant) e recriar essa mesma automação como um fluxo visual, comparando as duas abordagens.

## Próximos passos / evolução

- Adicionar mais dispositivos e organizar o dashboard por cômodo.
- Criar automações mais elaboradas com condições (presença + horário + luminosidade) no Node-RED.
- Rodar tudo isolado em uma VLAN de IoT separada da rede principal, por segurança.
- Fazer o paralelo consciente com o mundo industrial: o Home Assistant + MQTT aqui cumpre um papel parecido com um CLP (PLC) rodando lógica de automação e trocando dados com dispositivos de campo via um protocolo como EtherCAT/Modbus — a diferença é a escala, o determinismo (aqui não há tempo real) e o nível de criticidade.
