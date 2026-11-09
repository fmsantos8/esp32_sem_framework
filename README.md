# ESP32 sem o framework do TCC

Aplicação convencional para ESP-WROOM-32 DevKit V1, com **ESP-IDF 6.1.x**. Usa diretamente GPIO, I2C master, UART e GPTimer do ESP-IDF; não utiliza HAL portável, arena, BSP nem componentes do framework do TCC.

## Estrutura

`main/main.c` contém `app_main`, configuração dos periféricos e callback do GPTimer. `main/application.c` implementa o fluxo sequencial; `sht31d.c` e `ssd1306.c` acessam os handles nativos de dispositivos I2C. `telemetry.c` utiliza a UART do SDK, com aceitação parcial e timeout finito. `graphics.c`, `fonts.c`, `bitmaps.c` e `render.c` preservam a apresentação visual.

## Compilar e gravar

Em um terminal com o ESP-IDF 6.1 ativado:

```sh
idf.py set-target esp32
idf.py build
idf.py -p /dev/ttyUSB0 flash
idf.py -p /dev/ttyUSB0 monitor
```

O alvo padrão já é ESP32; `set-target` é necessário apenas ao configurar outro diretório de build. A compilação não depende dos projetos com framework. As saídas ficam em `build/`: ELF, BIN, MAP, bootloader, tabela de partições e argumentos de gravação. O console da aplicação está desabilitado para reservar UART0 à telemetria; o monitor exibe os registros a 115200 8N1.

`sdkconfig.defaults` seleciona CPU de 160 MHz e fixa a tarefa principal no núcleo 0, onde o GPTimer é construído. Power management permanece desabilitado, mantendo estável o clock APB da UART. O laço principal segue o processamento contínuo da referência; as esperas são controladas pelo contador de alarmes atendidos do GPTimer. O timer só inicia após inicializar os dispositivos e a aplicação.

| Função | Recurso |
| --- | --- |
| Botão externo | GPIO18, ativo baixo, pull-up externo de 10 kΩ |
| LED externo | GPIO19, ativo alto, resistor série de 1 kΩ |
| I2C0 SCL / SDA | GPIO22 / GPIO21, 100 kHz, pull-ups externos |
| Sensor / display | SHT31-D `0x44` / SSD1306 `0x3C` |
| UART0 TX / RX | GPIO1 / GPIO3, 115200 8N1 |
| Base da aplicação | GPTimer a 1 MHz, alarme periódico a cada 1000 contagens |

I2C tem timeout de 100 ms; a UART tem orçamento de 1000 ms. As operações I2C de escrita e leitura são separadas e terminam em STOP. Falha de inicialização interrompe a execução antes de acender o LED, como na referência.

Consulte [SEM_FRAMEWORK.md](https://github.com/fmsantos8/testes_framework/blob/main/docs/SEM_FRAMEWORK.md) para o comportamento detalhado e a validação pareada.


## Evidências da avaliação

Os fontes e as configurações desta publicação correspondem aos arquivos identificados nos builds de 02/11/2026 e na campanha de 07–08/11/2026. A configuração comparada no artigo é default (-Og). Os binários avaliados, métricas, registros e o manifesto que relaciona seus hashes aos commits publicados estão em [testes_framework](https://github.com/fmsantos8/testes_framework). Este repositório contém a implementação convencional; sua compilação não depende do framework proposto.
