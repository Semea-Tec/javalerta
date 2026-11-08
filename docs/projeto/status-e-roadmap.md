# Status e roadmap

**Projeto:** detecção de falhas em cercas elétricas com alerta via LoRa, para proteger lavouras e animais (Javalarm / `javalerta` / Comm Protocol; ver D2 no roadmap). **Fase:** reestruturação, consolidando documentação e planejamento. **Meta atual:** realizar um teste em campo; as pendências abaixo são as necessárias para chegar a ele, e o que fica para depois está na seção 10 do roadmap. **Última revisão desta página:** 2026-11-08, conferida contra o código da branch `main` (commit `f181fb7`, de 2025-08-21, único ramo do repositório) e contra o relatório final da disciplina em `docs/`.

Plano com objetivos, fases, prazos e critérios de pronto: [roadmap.md](roadmap.md). Contexto e motivação: [apresentacao.md](apresentacao.md).

## 1. Objetivos

- **Detecção:** identificar quedas abruptas de tensão na cerca elétrica.
- **Alerta confiável:** enviar o aviso por LoRa, com protocolo de confirmação (ACK).
- **Baixo consumo:** operar por longos períodos em área rural isolada.

## 2. Materiais

| Item                                                                                                  | Situação                                                                                              |
|-------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| 2 placas Waveshare RP2040-LoRa, versão LF (410–510 MHz), com SX1262                                   | Em uso (transmissor e receptor); o README cita Raspberry Pi Pico W, mas a pinagem em `src/board-config.h` e o relatório indicam a RP2040-LoRa |
| Circuito sensor: 4 resistores de 9 kΩ, 1 de 100 kΩ, 1 de 40 kΩ, capacitor de 47 µF e módulo retificador de onda completa | Montado em placa perfurada; valores só no relatório, sem esquemático                                  |
| Display de 7 segmentos no receptor                                                                    | Montado, mas sem código no firmware                                                                   |
| Eletrificador doméstico de cerca                                                                      | Usado nos testes de bancada (pulsos de cerca de 8000 V segundo o relatório)                           |
| Osciloscópio, multímetro, resistor de 5 Ω (2 de 10 Ω em paralelo)                                     | Usados nas medições de sinal e de corrente                                                            |
| Alimentação (pilhas ou bateria)                                                                       | Autonomia não medida; sem *sleep* o consumo medido em repouso é de 32 mA                              |
| Caixa/case e fixação na cerca                                                                         | Não existem; o relatório cita um modelo de case de Raspberry Pi Pico como ponto de partida           |
| Cartilha de uso (`docs/images/cartilha_uso.png`)                                                      | Disponível; não validada com o produtor                                                               |

## 3. Concluído

### 3.1 Sensoriamento e firmware

- Leitura analógica da cerca no transmissor: ADC no GPIO 26, média de 10 amostras a cada 100 ms (1 s por leitura), alerta quando a média fica abaixo de 0,5 V.
- Rádio SX1262 configurado em LoRa (largura de banda de 125 kHz, fator de espalhamento 7, taxa de codificação 4/5, 22 dBm, CRC ligado), pacote de 4 bytes.
- Adaptação da biblioteca LoRaMac-node (incluída em `lib/`) para a placa RP2040-LoRa, em `src/` (SPI, GPIO, rádio SX1262, temporização).
- CMake configurado (Pico SDK 2.1.1) para gerar os executáveis `transmitter` e `receiver`.

### 3.2 Comunicação

- O transmissor envia o alerta e passa a escutar; o receptor, ao receber qualquer pacote, sinaliza com o LED e responde com uma mensagem "ACK".

### 3.3 Validação

- Circuito sensor por detecção direta (resistores, capacitor e retificador) escolhido depois de a abordagem por bobina de indução se mostrar instável por ruído de terra.
- Sistema demonstrado em bancada, com eletrificador doméstico, e medições de corrente do transmissor.
- Não há testes em campo.

### 3.4 Documentação

- README com resumo, objetivos, arquitetura, protocolo, consumo, instruções de build e gravação.
- Diagrama de arquitetura, fluxogramas do protocolo, cartilha de uso e relatório final em PDF (`docs/`).

## 4. Pendências e próximos passos

> Plano com objetivos, fases, prazos e critérios de pronto: [roadmap.md](roadmap.md). Esta seção é só o retrato das pendências.

### 4.1 Processo e organização

- Definir o responsável pelo projeto.
- Aplicar a guideline de commits da equipe.

### 4.2 Detecção

- Documentar o circuito sensor (esquemático, faixa de tensão, segurança elétrica).
- Registrar o sinal na saída do circuito final, com e sem falha, e tratar o ruído.
- Revisar a regra de falha (média de 1 s abaixo de 0,5 V, sem histerese nem tempo mínimo) e medir falsos alarmes e falhas não detectadas em bancada.

### 4.3 Comunicação

- Definir o protocolo v2 (ID do nó, sequência, tipo de mensagem, sincronismo) e implementar retransmissão e validação do ACK.
- Definir a frequência (hoje 865 MHz fixa, com placa de 410–510 MHz), conferir a regulamentação e medir alcance e perda de pacotes.
- Implementar *heartbeat*.
- Refazer a notificação do receptor: ACK imediato, sem bloqueio, silenciar, display ou buzzer.
- Notificação remota (possivelmente reaproveitando o gateway do Shitakiometer) fica para depois do teste em campo.

### 4.4 Energia e robustez

- Refazer a medição de consumo e recalcular as potências com a tensão de alimentação.
- Implementar modo de baixo consumo (pré-requisito do teste: 14 dias exigem cerca de 10,8 Ah no consumo atual em repouso) e dimensionar a alimentação.
- Projetar caixa vedada simples, *watchdog* e aviso de bateria fraca.
- Registrar cada evento (data/hora, tipo, RSSI, tentativas de ACK) de forma persistente.
- Fazer um ensaio geral de pelo menos 48 h em bancada com a versão final do firmware.

### 4.5 Qualidade

- Build reprodutível: alinhar o README à versão 2.1.1 do Pico SDK já fixada no CMake e documentar o passo a passo.
- CI de build no GitHub Actions; sem testes automatizados.

### 4.6 Teste em campo

- Escolher o produtor e o local do teste, e levantar tipo de eletrificador e cerca, distância e energia disponível.
- Definir a duração do teste e escrever o plano (como provocar falhas controladas com segurança, o que medir, critérios de sucesso).
- Instalar um par transmissor/receptor, executar o teste, registrar eventos e produzir o relatório.
- Revisão da cartilha de uso com o produtor fica para depois do teste.
