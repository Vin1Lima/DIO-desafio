# Segundo Cérebro: Firmware ESP32 com Boas Práticas

Notebook montado no **Gemini Notebook** (antigo NotebookLM) para servir de consultor no projeto de firmware para ESP32, respondendo apenas com base em fontes que eu selecionei e conferi.

## 🎯 Objetivo

> "Quero um segundo cérebro que me ajude a projetar firmware para ESP32 com ESP-IDF e FreeRTOS seguindo boas práticas de arquitetura, uso de memória, tarefas e consumo de energia."

## 📚 Fontes e por que confio nelas

**Critérios para uma fonte entrar:** ser oficial (Espressif/FreeRTOS) ou de referência reconhecida em embarcados, estar atualizada para o ESP-IDF 5.x ou superior e explicar o *porquê* das práticas, não só o *como*.

| # | Fonte | Formato | Por que entrou |
|---|---|---|---|
| 1 | [ESP-IDF – FreeRTOS Overview](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/freertos.html) | Texto | Documentação oficial da Espressif; explica como o FreeRTOS é integrado ao ESP-IDF. |
| 2 | [ESP-IDF – FreeRTOS (IDF / SMP)](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/freertos-smp.html) | Texto | Oficial; cobre as diferenças do FreeRTOS no ESP32 dual-core (afinidade de núcleo, pilha em bytes). |
| 3 | [ESP-IDF – Error Handling](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/error-handling.html) | Texto | Oficial; define `esp_err_t`, `ESP_ERROR_CHECK` e erros recuperáveis vs. fatais. |
| 4 | [ESP-IDF – Sleep Modes](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/sleep_modes.html) | Texto | Oficial; base para economia de energia (light/deep sleep, fontes de wakeup). |
| 5 | [ESP32 Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf) | PDF | Manual oficial do chip, com memória e periféricos em detalhe. |
| 6 | [Interrupt (Memfault) – A Guide to Watchdog Timers](https://interrupt.memfault.com/blog/firmware-watchdog-best-practices) | Texto | Blog técnico editado pela Memfault, escrito por engenheiros de firmware; cobre práticas de confiabilidade que a doc oficial não cobre. |
| 7 | [Digi-Key – Introduction to RTOS Part 2](https://www.youtube.com/watch?v=JIr7Xm_riRs) | Vídeo | Série didática de um grande distribuidor de componentes. Os exemplos usam Arduino IDE, então foi usado só pelos conceitos de RTOS. |

## 🧭 Diretriz do notebook

```
Comporte-se como um engenheiro sênior de firmware embarcado especialista em ESP32,
ESP-IDF e FreeRTOS. Responda apenas com base nas fontes, cite de onde veio cada
informação e avise quando as fontes não cobrirem o assunto. Explique de forma prática,
com exemplos de código curtos e o motivo de cada boa prática. Responda em português do Brasil.
```

## 💬 Perguntas e respostas

### 1. Quando devo usar uma tarefa, uma fila ou um semáforo no FreeRTOS do ESP-IDF?
- **Tarefa:** para cada fluxo de execução concorrente (ex.: leitura de sensor). Roda em loop infinito, pode ser fixada em um núcleo com `xTaskCreatePinnedToCore()` e, se terminar, deve chamar `vTaskDelete(NULL)`.
- **Fila:** para passar dados entre tarefas ou de uma ISR para uma tarefa. Os dados são copiados; para estruturas grandes, passe ponteiros. Em ISR, use as funções `...FromISR`.
- **Mutex:** para proteger recurso compartilhado (I2C, SPI). Tem herança de prioridade, quem pega deve devolver e não pode ser usado em ISR.
- **Semáforo / Task Notification:** para sinalizar eventos. O notebook destacou que Task Notifications são mais rápidas e gastam menos RAM.

**Fontes citadas:** FreeRTOS (IDF/SMP), FreeRTOS Overview, vídeo Digi-Key.
![Resposta 1](prints/pergunta1.png)

### 2. Como dimensionar a pilha de uma tarefa e detectar estouro?
- No ESP-IDF a pilha é informada **em bytes** (no FreeRTOS original é em *words*).
- O mínimo aceito é 768 bytes; o notebook sugeriu começar entre 2048 e 4096 bytes e ajustar depois de medir.
- Evitar buffers grandes como variáveis locais; usar heap ou alocação estática.
- Estouro de pilha é tratado como erro fatal, não recuperável.

**Fontes citadas:** FreeRTOS (IDF/SMP), FreeRTOS Overview, Error Handling.
![Resposta 2](prints/pergunta2.png)

### 3. Boas práticas para economizar energia com light sleep e deep sleep
- **Light sleep:** preserva RAM e estado; o notebook citou consumo na faixa de ~800 µA e despertar em menos de 1 ms.
- **Deep sleep:** desliga CPU e quase toda a RAM, ficando só o domínio RTC; consumo na faixa de µA, mas perde o contexto.
- Desligar Wi-Fi/Bluetooth (`esp_wifi_stop()`) antes de dormir.
- Para manter a conexão, usar Modem-sleep com light sleep automático (Power Management).
- No light sleep, preferir o modo Deep Power-Down da flash a cortar a alimentação dela.

**Fontes citadas:** Sleep Modes, ESP32 Technical Reference Manual.
![Resposta 3](prints/pergunta3.png)

### 4. Como tratar erros com `esp_err_t`/`ESP_ERROR_CHECK` e usar watchdog corretamente?
- APIs retornam `esp_err_t`; `ESP_OK` (0) é sucesso; `esp_err_to_name()` converte o código em texto para log.
- `ESP_ERROR_CHECK` aborta o programa: usar só na inicialização, para falhas críticas.
- No fluxo normal, usar `ESP_RETURN_ON_ERROR` / `ESP_GOTO_ON_ERROR` para tratar a falha sem reiniciar o chip.
- Watchdog: não alimentar de forma incondicional numa única tarefa; monitorar o progresso de cada tarefa (Task Watchdog).
- O notebook avisou que as fontes **não detalham** toda a configuração do Task WDT no menuconfig, mostrando que ele respeita o limite das fontes.

**Fontes citadas:** Error Handling, Interrupt (Watchdog), ESP32 Technical Reference Manual.
![Resposta 4](prints/pergunta4.png)

## 🧠 Materiais gerados no Estúdio

- [Mapa mental](materiais/mapa-mental.png)
- [Apresentação de slides](materiais/slides.pdf)

## 🔗 Link do notebook

[Abrir o notebook no Gemini Notebook](COLE_AQUI_O_LINK_COMPARTILHADO)
