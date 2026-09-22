# S2-CP02 - Projeto Motiva | Atualização Remota de Firmware (OTA)

Implementação de um sistema de Atualização Remota de Firmware (OTA) para microcontrolador ESP32. O projeto simula um nó IoT para monitoramento de vegetação que realiza medições periódicas, consulta um repositório remoto via HTTP em busca de novas versões de software e aplica atualizações autônomas na memória flash.

## Integrantes

* Bento Donato Garcia - 561621

* Enzo Ribeiro Domingues Piazentin - 564216

* Guilherme Domingues Califoni - 565157

* Antonio Lucas Santana Tavares - 565516

* Gustavo Schimith - 564800

## 1. Arquitetura do Sistema

O firmware opera em um ESP32 (simulado via Wokwi) conectado à rede `Wokwi-GUEST`.

* **Protocolo de Atualização:** HTTP GET.

* **Endpoint de Verificação:** Requisição periódica (a cada 3 ciclos) ao manifesto `version.json`.

* **Processamento JSON:** Utilização da biblioteca `ArduinoJson` para parsing dos atributos de versão e URL do binário.

* **Gravação na Flash:** Gerenciada pela biblioteca nativa `Update.h`.

### 1.1. Particionamento de Memória (Dual Bank)

O sistema utiliza um esquema customizado no arquivo `partitions.csv` para garantir tolerância a falhas durante o processo OTA:

* `app0` (Offset 0x10000): Partição ativa primária.

* `app1` (Offset 0x150000): Partição de destino reservada para o download da nova versão.

* `otadata` (Offset 0xe000): Ponteiro atualizado dinamicamente apenas após a verificação de integridade da escrita na flash, instruindo o bootloader a alternar a inicialização para a partição `app1`.

## 2. Especificações de Firmware

### Firmware 1.0 (Base)

* Coleta de 5 amostras pseudoaleatórias por ciclo (intervalo de 2s por amostra).

* Processamento: Cálculo de média aritmética.

* Sinalização de hardware: LED Azul estático.

* Temporização: Rotina assíncrona baseada em `millis()` para controle de ciclos (48s).

### Firmware 2.0 (Payload OTA)

* Coleta de 5 amostras mantida.

* Tratamento de dados: Implementação de algoritmo de ordenação de vetores em memória.

* Processamento: Cálculo da mediana (3º elemento do vetor ordenado) para supressão de ruídos de leitura.

* Lógica de Controle: Histerese baseada na mediana.

  * Mediana >= 16.0 cm: Estado `ALERTA` (Acionamento: LED Vermelho).

  * Mediana <= 14.0 cm: Estado `NORMAL` (Acionamento: LED Verde).

  * 14.0 cm < Mediana < 16.0 cm: Retenção do estado anterior.

## 3. Procedimento de Teste

1. Inicializar a simulação Wokwi através do link do Firmware 1.0.

2. Monitorar o terminal serial (baud rate 115200) acompanhando as rotinas de leitura e temporização.

3. Aguardar a finalização de 3 sessões (ciclos de 48s).

4. O microcontrolador efetuará o request do `version.json`, fará o matching da versão e iniciará a stream de download do `firmware_v2.bin`.

5. Após o log de sucesso na gravação, o ESP32 executará um soft reset (`ESP.restart()`).

6. Validar a inicialização do Firmware 2.0 no terminal serial e testar as transições de estado (Histerese/LEDs).

## 4. Referências e Links

* Simulação FW 1.0: https://wokwi.com/projects/475553625279093761

* Simulação FW 2.0: https://wokwi.com/projects/475717766334377985

* Manifesto JSON: `version.json`

* Binário FW 2.0: `firmware_v2.bin`
