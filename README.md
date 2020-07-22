---
esphome :
  nome : campainha
  plataforma : ESP8266
  placa : esp01_1m

# Conexão WiFi, corrija estes
# com valores para o seu WiFi.
wifi :
  SSID : ! wifi_ssid segredo
  password : ! wifi_password segredo

# Ative o log.
logger :

# Ative a API do Home Assistant.
api :

# Ative atualizações sem fio.
ota :

# Ativar servidor da Web.
web_server :
  porta : 80

# Sincronize a hora com o Assistente de casa.
tempo :
  - plataforma : homeassistant
    id : homeassistant_time

# Sensores de texto com informações gerais.
text_sensor :
  # Exponha a versão ESPHome como sensor.
  - plataforma : versão
    nome : Campainha ESPHome Version
  # Exponha informações de WiFi como sensores.
  - plataforma : wifi_info   endereço_ip :
      nome : IP da campainha
    ssid :
      nome : Campainha SSID
    bssid :
      nome : Campainha BSSID

# Sensores com informações gerais.
sensor :
  # Sensor de Uptime.
  - plataforma : tempo de atividade
    nome : Campainha Uptime

  # Sensor de WiFi Signal.
  - plataforma : wifi_signal
    nome : Sinal WiFi de campainha
    update_interval : 60s

# Global para armazenar o estado ligado / desligado do carrilhão
globais :
  - id : carrilhão
    tipo : bool
    restore_value : true
    valor_inicial : ' verdadeiro '
# Interruptores expostos.
interruptor :
  # Alterne para reiniciar a campainha.
  - plataforma : reiniciar
    nome : Reinício da campainha

  # Alterne para ligar / desligar o sinal sonoro.
  - plataforma : gpio
    id : retransmissão
    invertido : verdadeiro
    nome : Campainha Campainha
    pino : GPIO0

  # Alterne para ativar / desativar o sinal sonoro quando
  # o botão da campainha é pressionado.
  #
  # Ele cria um switch "virtual" baseado em
  # em uma variável global.
  - plataforma : modelo
    nome : Campainha Campainha Ativa
    id : chime_active
    restore_state : false
    turn_on_action :
      - globals.set :
          id : carrilhão
          valor : ' verdadeiro '
    turn_off_action :
      - globals.set :
          id : carrilhão
          valor : ' false '
    lambda : | -
      id de retorno (carrilhão);
# Sensor binário representando o
# Pressionar o botão da campainha.
binary_sensor :
  - plataforma : gpio
    id : button
    nome : Botão da campainha
    pino :
      # Conectado ao GPIO no ESP-01S.
      número : GPIO2
      modo : INPUT_PULLUP
      invertido : verdadeiro
    filtros :
      # Filtro pequeno, para rebater o botão, pressione.
      - delayed_on : 25ms
      - delayed_off : 25ms
    on_press :
      # Ligue apenas o sinal sonoro quando estiver ativo.
      então :
        se :
          condição :
            - switch.is_on : chime_active
          então :
            - switch.turn_on : relay
    on_release :
      # Ao soltar, toque a campainha.
      - switch.turn_off : relé
