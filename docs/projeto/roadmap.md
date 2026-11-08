# Roadmap de desenvolvimento

**Versão:** 3. **Última revisão:** 2026-11-08, conferida contra o código da branch `main` (commit `f181fb7`) e contra o relatório final da disciplina em `docs/`. **Responsável pelo projeto:** a definir (D1). **Base:** código, README, relatório final e as atas de 27/10 e 03/11/2026. Estado verificado do código em [status-e-roadmap.md](status-e-roadmap.md).

> **Meta atual: realizar um teste em campo.** Tudo o que está antes dele neste roadmap existe para deixar a plataforma pronta para esse teste. O que não é pré-requisito do teste está na seção 10, "Depois do teste em campo".

## 1. Problema

Um produtor rural com cerca elétrica para proteger a lavoura e os animais (contra javalis, por exemplo) precisa saber **na hora** que a cerca perdeu tensão, sem percorrer o perímetro. A propriedade fica em área isolada, sem internet confiável e sem energia estável em todos os pontos.

Três necessidades decorrem disso: **detectar a falha com confiança**, **ser avisado a tempo** e **funcionar sozinho no campo**.

## 2. Meta: teste em campo

### 2.1 O que queremos provar

Que um par transmissor/receptor, instalado numa cerca elétrica real e sem intervenção da equipe durante o teste, **detecta as falhas, avisa com confirmação e registra o que aconteceu**, de modo que a equipe possa medir o desempenho e decidir os próximos passos.

### 2.2 Critérios de sucesso do teste

Duração planejada do teste: 14 dias (confirmada em D10).

| Critério                                          | Meta                                                                         |
|---------------------------------------------------|------------------------------------------------------------------------------|
| Falhas provocadas detectadas e avisadas           | 100 %, em pelo menos 20 provocações                                          |
| Tempo entre a falha e o alerta no receptor        | Até 10 s                                                                     |
| Falsos alarmes                                    | No máximo 1 por semana                                                       |
| Alertas com confirmação (ACK) recebida            | Pelo menos 95 %, em até 3 tentativas                                         |
| Operação contínua sem intervenção                 | Os 14 dias, sem troca de bateria                                             |
| Eventos registrados para análise                  | 100 %, com data/hora, tipo, intensidade do sinal (RSSI) e tentativas de ACK  |

### 2.3 Prontidão para o teste (gate)

O teste só começa quando **todos** os itens abaixo estiverem cumpridos. Esse é o critério de saída da Fase 3.

| #  | Item de prontidão                                                                              | Tarefa     |
|----|------------------------------------------------------------------------------------------------|------------|
| G1 | Detecção validada em bancada, com falsos alarmes e falhas não detectadas medidos               | T1.4       |
| G2 | Segurança elétrica do circuito sensor revisada                                                 | T1.6       |
| G3 | ACK com retransmissão validado, frequência definida e alcance medido na distância do teste     | T2.2, T2.3 |
| G4 | Autonomia calculada e medida maior que a duração do teste, com margem                          | T3.3       |
| G5 | Caixa vedada e fixação prontas                                                                 | T3.4       |
| G6 | *Heartbeat*, *watchdog* e aviso de bateria fraca funcionando                                   | T2.4, T3.5 |
| G7 | Registro de eventos funcionando no transmissor e no receptor                                   | T3.7       |
| G8 | Ensaio geral concluído (operação contínua em bancada com a versão final)                       | T3.8       |
| G9 | Local, produtor, duração e plano do teste aprovados                                            | T4.1, T4.2 |

## 3. Ponto de partida

Resumo do que a conferência do código mostrou. O detalhe, com arquivos e linhas, está em [status-e-roadmap.md](status-e-roadmap.md).

| Já existe                                                                                                                          | Ainda não existe                                                                                  |
|------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| Leitura analógica da cerca (ADC, média de 1 s, alerta abaixo de 0,5 V) e envio de alerta por LoRa                                  | Retransmissão quando o ACK não chega; validação do conteúdo do ACK                                |
| Receptor que sinaliza por LED e responde com ACK                                                                                   | Identificação do nó, número de sequência e tipo de mensagem no pacote                             |
| Circuito sensor montado (resistores, capacitor 47 µF e retificador de onda completa, valores no relatório)                         | *Heartbeat* (aviso de que o transmissor continua vivo), registro de eventos, *watchdog*           |
| Adaptação da biblioteca LoRaMac-node para a placa RP2040-LoRa (`src/`)                                                             | Modo de baixo consumo: o firmware mede o ADC em laço contínuo, sem *sleep*                        |
| Medidas de corrente (114 mA transmitindo, 32 mA em repouso) feitas com resistor de 5 Ω                                             | Frequência coerente com o módulo: o código fixa 865 MHz e a placa é a versão de 410–510 MHz       |
| Testes em bancada com eletrificador doméstico                                                                                      | Caixa, fixação, alimentação autônoma, testes em campo, CI, licença, testes automatizados          |

## 4. Objetivos

| #      | Objetivo                                                                     | Resultado esperado                                                                                          |
|--------|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| **O1** | O sistema **detecta a falha** da cerca com confiança suficiente para o campo | Queda de tensão detectada em bancada; falsos alarmes medidos e dentro da meta da seção 2.2                  |
| **O2** | O produtor **é avisado** a tempo, com confirmação                            | Alerta no receptor em até 10 s, com ACK e retransmissão, e alcance validado para o local do teste           |
| **O3** | O sistema **opera sozinho durante o teste**                                  | Autonomia maior que 14 dias, com margem; caixa vedada; recuperação automática após reinício                 |
| **O4** | O **teste em campo é executado** e gera resultados                           | Teste concluído, eventos registrados e relatório com os critérios da seção 2.2                              |

Adoção prolongada pelo produtor e notificação remota ficam para depois do teste (seção 10).

## 5. Decisões em aberto

Cada uma bloqueia ou muda parte do plano. A coluna "Antes de" diz até quando a decisão precisa estar tomada.

| ID  | Pergunta                                                                                                                                                  | Afeta               | Antes de | Dono                                   |
|-----|-----------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|----------|----------------------------------------|
| D1  | Quem é o responsável pelo projeto? Na disciplina o líder foi o Hiure, que também mantém contato com produtores rurais                                     | Todo o roadmap      | Fase 0   | Equipe                                 |
| D2  | Qual o nome oficial (Javalarm, `javalerta` ou Comm Protocol)? O repositório continua como fork de `joaopaulorocha052/comm_protocol`?                      | T0.4, T0.6          | Fase 0   | Equipe                                 |
| D3  | Onde e com quem será o teste? Qual o eletrificador e a cerca (modelo, tensão, extensão)?                                                                  | T1.2, T4.1          | Fase 1   | Equipe, a partir dos contatos do Hiure |
| D4  | A regra atual (média de 1 s abaixo de 0,5 V) basta? Qual o tempo mínimo fora da faixa e haverá aviso de "cerca restabelecida"?                            | T1.3                | Fase 1   | Equipe + produtor                      |
| D5  | Onde fica o receptor? Qual a distância do transmissor, há energia e abrigo no local?                                                                      | T2.3, T3.3          | Fase 2   | Equipe + produtor                      |
| D7  | Qual frequência usar? O código fixa 865 MHz, mas o módulo é a versão LF (410–510 MHz). Qual a faixa permitida pela regulamentação para este uso?          | T2.3                | Fase 2   | Equipe                                 |
| D8  | Como o transmissor é alimentado durante o teste: bateria, painel solar ou derivação da própria cerca?                                                     | T3.3                | Fase 3   | Equipe                                 |
| D9  | Onde fica a gestão das tarefas: manter Trello (usado na disciplina) ou migrar para GitHub Issues/Projects, como no Shitakiometer?                         | T0.3                | Fase 0   | Equipe                                 |
| D10 | Qual a duração do teste e como provocar falhas controladas sem risco (quem, quando, como)?                                                                | G4, T4.2            | Fase 3   | Equipe + produtor                      |

## 6. Fases

Ordem pensada para chegar ao teste em campo com a menor quantidade de trabalho que o torne válido. As Fases 1, 2 e 3 podem se sobrepor; a Fase 4 começa cedo (T4.1 e T4.2) para que local e plano estejam prontos quando o gate fechar.

### Fase 0 — Base técnica e de processo (estimativa: 1–2 semanas)

Remove o que bloqueia as demais e atende às exigências das atas.

| ID   | Tarefa                                                                                                                                                                            | Objetivo  | Critério de pronto                                                                                  |
|------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------|-----------------------------------------------------------------------------------------------------|
| T0.1 | Definir o responsável pelo projeto (D1)                                                                                                                                           | Processo  | Nome registrado neste documento                                                                     |
| T0.2 | Aplicar a guideline de commits da equipe                                                                                                                                          | Processo  | Guideline referenciada em `docs/README.md`                                                          |
| T0.3 | Decidir a ferramenta (D9) e popular o quadro com as tarefas deste roadmap                                                                                                         | Processo  | Quadro com cards T0.x–T4.x, cada um citando ID e critério de pronto                                 |
| T0.4 | Padronizar a apresentação do repositório (D2): descrição e *topics* do "About" (hoje vazios), `LICENSE` próprio (só a biblioteca em `lib/` tem licença), `.gitignore` para `build/` | Qualidade | About preenchido; `LICENSE` e `.gitignore` na raiz; nome consistente em README, docs e atas         |
| T0.5 | Mapear o estado real do código e conferir o status contra a `main`                                                                                                                | Processo  | **Concluída em 2026-11-08** (commit `f181fb7`); resultado em [status-e-roadmap.md](status-e-roadmap.md) |
| T0.6 | Corrigir o README com o que o código e o relatório mostram: placa (Waveshare RP2040-LoRa, não Pico W), estrutura de pastas, notificação (só LED), frequência, funcionamento real do ACK e do sleep | Qualidade | README confere item a item com a seção 4 do status; cada pasta descrita (`comm_protocol/` = aplicações TX e RX, `src/` = adaptação da placa, `lib/` = LoRaMac-node, `docs/`) |
| T0.7 | Tornar o build reprodutível: o `CMakeLists.txt` já fixa Pico SDK 2.1.1, toolchain 14_2_Rel1 e picotool 2.1.1, mas o README manda clonar o `master` do SDK. Alinhar o README e documentar o passo a passo | Qualidade | Build limpo do transmissor e do receptor em máquina nova seguindo só `docs/firmware/compilacao.md` |
| T0.8 | Configurar CI de build (GitHub Actions) para compilar os dois firmwares a cada push/PR *(desejável; não bloqueia o gate)*                                                         | Qualidade | Badge de build no README; PR com build quebrado é bloqueado                                         |

### Fase 1 — Detectar com confiança (O1) (estimativa: 2–4 semanas)

| ID   | Tarefa                                                                                                                                                                                                      | Critério de pronto                                                              |
|------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| T1.1 | Documentar o circuito sensor: o relatório lista os componentes (4 resistores de 9 kΩ, 1 de 100 kΩ, 1 de 40 kΩ, capacitor de 47 µF e módulo retificador de onda completa), mas não há esquemático nem faixa de operação | `docs/hardware/circuito-sensor.md` com esquemático desenhado a partir da placa montada, valores e faixa de tensão |
| T1.2 | Registrar o sinal na saída do circuito sensor (entrada do ADC) com a cerca energizada, em operação normal e com falhas (fio solto, contato com vegetação, eletrificador desligado). O relatório documenta o sinal da bobina e cita ruído da rede elétrica e do eletrificador, mas não a saída do circuito final | Medições com data, equipamento, imagens do osciloscópio e conclusão sobre o ruído |
| T1.3 | Revisar a regra de falha, hoje fixa no código: média de 10 amostras em 1 s, alerta abaixo de 0,5 V, sem histerese nem tempo mínimo. Definir tempo mínimo e histerese (D4), tornar os parâmetros configuráveis e enviar também o evento de restabelecimento | Regra documentada, implementada e com os parâmetros em constantes nomeadas             |
| T1.4 | Teste de bancada: simular rompimento e retorno da cerca e medir detecções, falhas não detectadas, falsos alarmes e tempo até o alerta                                                                         | Tabela de resultados em `docs/testes/` com data e versão do firmware (**G1**)   |
| T1.5 | Calibrar a leitura analógica (o código converte com referência fixa de 3,3 V) e registrar a incerteza, inclusive com a alimentação por bateria                                                                | Erro medido e documentado                                                       |
| T1.6 | Revisar a segurança elétrica do circuito sensor (isolação, proteção contra surtos e alta tensão; o relatório cita pulsos de cerca de 8000 V)                                                                  | Checklist de segurança em `docs/hardware/` (**G2**)                             |

### Fase 2 — Avisar (O2) (estimativa: 2–3 semanas)

| ID   | Tarefa                                                                                                                                                                                                                                                | Critério de pronto                                                                       |
|------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| T2.1 | Definir e documentar o protocolo v2. Hoje o pacote tem 4 bytes (o primeiro vale 5), o ACK é a string "ACK" e o receptor trata qualquer pacote como alerta. Incluir ID do nó, número de sequência, tipo (alerta, restabelecido, *heartbeat*), palavra de sincronismo e ACK com ID e sequência | `docs/firmware/protocolo.md` com formato do pacote, estados, retentativas e *timeouts*  |
| T2.2 | Implementar retransmissão e validação do ACK no transmissor. Hoje o transmissor não retransmite, não confere o conteúdo do ACK e, sem resposta, volta a escutar indefinidamente. Testar com perda de pacotes e com receptor desligado                  | Até 3 tentativas com intervalo definido; tabela de resultados registrada (**G3**)        |
| T2.3 | Definir a frequência (D7) e medir o alcance. O código fixa 865 MHz em transmissor e receptor, ignorando as macros de região (padrão `REGION_EU433`), e a placa é a versão de 410–510 MHz. Medir também com fator de espalhamento acima do SF7 configurado hoje | Frequência e parâmetros de rádio documentados; tabela distância × RSSI × perda de pacotes na distância prevista do teste (**G3**) |
| T2.4 | Implementar *heartbeat*: o transmissor envia um pacote periódico e o receptor alerta quando perde pacotes consecutivos. Hoje o transmissor só fala quando detecta falha                                                                              | Alerta de "transmissor mudo" disparado ao desligar o transmissor (**G6**)                |
| T2.5 | Refazer a notificação no receptor: o ACK sai só depois de 6 s de LED piscando dentro de `OnRxDone`, o timeout de recepção bloqueia o laço por 4 s, e o laço reinicia a escuta a cada 10 ms. Responder o ACK imediatamente, notificar sem bloquear, repetir o alerta só enquanto a falha persistir, permitir silenciar e acionar o display de 7 segmentos já montado ou um buzzer | ACK em menos de 1 s; receptor continua escutando durante a notificação; comportamento descrito e testado |

### Fase 3 — Pronto para o campo (O3) (estimativa: 2–4 semanas; depende de compras)

Fecha o gate da seção 2.3. Escopo limitado ao que o teste exige. Com o consumo em repouso medido (32 mA), 14 dias exigiriam cerca de 10,8 Ah só para manter o transmissor ligado (32 mA × 336 h), por isso o modo de baixo consumo é pré-requisito do teste.

| ID   | Tarefa                                                                                                                                                                                                                   | Critério de pronto                                                                  |
|------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| T3.1 | Refazer a medição de consumo e corrigir o README e o relatório. As correntes (114 mA e 32 mA, calculadas como V/5 Ω) são plausíveis, mas as potências publicadas (65 mW e 5,1 mW) usam a tensão no resistor de 5 Ω (0,57 V e 0,16 V) e não a tensão de alimentação. Medir a tensão de alimentação e o consumo em cada estado, incluindo *sleep* | Medições com método descrito; potências recalculadas; README corrigido              |
| T3.2 | Implementar o modo de baixo consumo no transmissor. O firmware atual mede o ADC em laço contínuo e o estado `SLEEP` é só o estado de medição; o relatório descreve acordar por interrupção do circuito sensor, o que não está no código. Acordar por interrupção ou por temporizador (para o *heartbeat*) | Consumo médio medido abaixo do orçamento definido em T3.3                           |
| T3.3 | Dimensionar a alimentação (D8) para 14 dias mais margem, a partir das medições de T3.1 e T3.2; medir em bancada                                                                                                           | Cálculo documentado e autonomia medida maior que o teste (**G4**)                   |
| T3.4 | Projetar e testar caixa vedada e fixação junto à cerca (versão simples, suficiente para o período do teste; o relatório cita um modelo de case de Raspberry Pi Pico como ponto de partida, ainda não feito)              | Sem entrada de água nem condensação durante 14 dias de ensaio (**G5**)              |
| T3.5 | Resiliência: *watchdog*, reinício automático após queda de energia e aviso de bateria fraca. Nenhum dos três existe no código                                                                                            | Comportamento testado e documentado (**G6**)                                        |
| T3.7 | Instrumentação do teste: registrar cada evento com data/hora, tipo, RSSI e tentativas de ACK. Hoje só há `printf` por USB, sem carimbo de tempo, e o RSSI recebido no *callback* é ignorado. Registrar no receptor de forma persistente e enviar no pacote a sequência e as tentativas do transmissor | Eventos legíveis após o teste, sem depender de cabo ligado durante a operação (**G7**) |
| T3.8 | Ensaio geral: operar o conjunto completo, com a versão final do firmware marcada com *tag*, em bancada ou perto da equipe, por pelo menos 48 h                                                                            | Registro do ensaio; nenhuma intervenção manual necessária (**G8**)                  |

### Fase 4 — Teste em campo (O4) (T4.1 e T4.2 começam na Fase 1; execução de 14 dias)

| ID   | Tarefa                                                                                                                                              | Critério de pronto                                                          |
|------|-----------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| T4.1 | Visita e conversa com o produtor, a partir dos contatos do Hiure: escolher o local e levantar D3, D4, D5 e D8                                       | Respostas e local registrados (**G9**)                                      |
| T4.2 | Escrever o plano do teste: duração (D10), o que medir, como provocar falhas controladas com segurança, critérios de sucesso (seção 2.2), quem acompanha | `docs/testes/plano-teste-campo.md` aprovado pela equipe e pelo produtor (**G9**) |
| T4.3 | Instalar o par transmissor/receptor no local e conferir que está funcionando                                                                         | Primeiro evento de teste recebido no local                                  |
| T4.4 | Executar o teste: provocações controladas, acompanhamento periódico e registro de ocorrências (alertas verdadeiros, falsos alarmes, falhas não detectadas) | Registro completo em `docs/testes/`                                         |
| T4.5 | Recolher os dados e escrever o relatório do teste, comparando com os critérios da seção 2.2                                                          | Relatório publicado em `docs/`; decisão sobre os próximos passos registrada |

## 7. Dependências e caminho crítico

```
T1.1 ──► T1.2 ──► T1.3 ──► T1.4 ──┐
T1.1 ──► T1.6 ────────────────────┤
T2.1 ──► T2.2 ──┐                 │
T2.3 ───────────┼─────────────────┼──► GATE ──► T4.3 ──► T4.4 ──► T4.5
T2.4, T2.5 ─────┘                 │      ▲
T3.1 ─► T3.2 ─► T3.3 ─► T3.4 ─────┤      │
T3.5, T3.7 ──► T3.8 ──────────────┘      │
T4.1 ──► T4.2 ───────────────────────────┘
```

- **T1.2, T1.3 e T1.4** são o caminho crítico técnico: sem detecção confiável o teste não prova nada.
- **T2.1 e T2.2** (protocolo v2 e retransmissão) são o segundo caminho crítico: hoje a confirmação não é validada nem repetida.
- **T3.2** (baixo consumo) é o item de maior risco de prazo; depende de T3.1 e, se pedir compras, de hardware.
- **T4.1 e T4.2** (local e plano) não dependem de código; começar cedo evita que o gate feche e o teste fique esperando.
- **T3.8** (ensaio geral) só faz sentido depois de T2.4, T3.5 e T3.7.

## 8. Marcos

| Marco | Entrega                                                                          | Fase |
|-------|----------------------------------------------------------------------------------|------|
| M0    | Responsável definido; README corrigido; quadro populado                          | 0    |
| M1    | Detecção validada em bancada, com falsos alarmes medidos (G1, G2)                | 1    |
| M2    | Alerta com ACK e retransmissão funcionando, com frequência definida e alcance medido (G3) | 2 |
| M3    | **Pronto para o campo:** gate G1–G9 cumprido e ensaio geral concluído            | 3    |
| M4    | Par instalado no local e primeiro evento de teste recebido                       | 4    |
| M5    | Teste concluído e relatório publicado                                            | 4    |

A ata de 27/10 registra que é preciso ter mais resultados e testes em campo antes de avançar nas conversas com a Embrapa e o Globo Rural. O marco **M5** é o que gera esse resultado, e **M3** é pré-requisito dele.

## 9. Riscos

| Risco                                                                                                        | Efeito                                                       | Mitigação                  |
|--------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|----------------------------|
| Ruído da rede elétrica e do eletrificador na leitura (citado no relatório); regra de falha sem histerese     | Falsos alarmes; resultado do teste inconclusivo              | T1.2, T1.3, T1.4           |
| ACK sem validação nem retransmissão; ACK só é recebido se cair na janela de escuta do transmissor            | Alerta perdido sem ninguém saber                             | T2.2, T2.5                 |
| Receptor aceita qualquer pacote do mesmo rádio, sem ID nem sincronismo                                       | Alarmes falsos por outros dispositivos LoRa                  | T2.1                       |
| Frequência de 865 MHz fora da faixa do módulo LF (410–510 MHz) e possivelmente fora do permitido             | Alcance reduzido; risco regulatório                          | T2.3, D7                   |
| Transmissor para sem avisar (bateria, travamento)                                                            | Cerca sem monitoramento sem ninguém saber; perda de dados    | T2.4, T3.5, T3.8           |
| Consumo em repouso de 32 mA (sem *sleep*)                                                                    | Bateria acaba em dias, antes do fim do teste                 | T3.1, T3.2, T3.3           |
| Alta tensão no circuito sensor, inclusive durante as falhas provocadas                                       | Risco para pessoas e equipamento                             | T1.6, plano de teste (T4.2) |
| Eventos não registrados                                                                                      | Teste sem dados para analisar                                | T3.7, T3.8                 |
| Local ou produtor indefinido                                                                                 | Gate fecha e não há onde testar                              | T4.1 começa na Fase 1      |
| README e relatório divergem do código (placa, interrupção e *sleep*, display, som, retransmissão)            | Retrabalho; apresentação com informação errada               | T0.6                       |
| Hardware demora a chegar                                                                                     | Fase 3 atrasa                                                | Pedir cedo; T3.1 em paralelo |
| Responsável não definido                                                                                     | Roadmap sem dono                                             | D1                         |

## 10. Depois do teste em campo (fora do escopo atual)

Itens que saíram do caminho até o teste. A decisão de retomá-los depende dos resultados do relatório (T4.5).

| ID | Item                                                                                                                              | Observação                                          |
|----|-----------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------|
| P1 | Notificação remota via gateway LoRa → Telegram/WhatsApp, possivelmente reaproveitando o gateway do Shitakiometer                  | Depende de haver internet no local                  |
| P2 | Placa definitiva (PCB) no lugar da placa perfurada                                                                                | Só se o resultado do teste justificar               |
| P3 | Revisar a cartilha de uso com o produtor, na linguagem dele                                                                       | Feedback do teste alimenta a revisão                |
| P4 | Operação prolongada (um ciclo produtivo completo) e adoção pelo produtor                                                          | Novo roadmap após M5                                |
| P5 | Autonomia de longo prazo (painel solar, semanas ou meses sem manutenção)                                                          | Depende dos dados de consumo de T3.1 e T3.2         |
| P6 | Suporte a vários sensores por receptor (o ID do nó em T2.1 já prepara o caminho) e interface móvel                                | Próximos passos já citados no relatório da disciplina |
| P7 | Retomar conversas com Embrapa e Globo Rural usando o relatório do teste                                                           | Conforme ata de 27/10                               |

## 11. Documentação a criar

Estrutura proposta, espelhando a do Shitakiometer (`Docs/projeto`, `Docs/arquitetura` etc.). Hoje `docs/` guarda as imagens e o relatório final em PDF; padronizar a estrutura faz parte de T0.6.

| Arquivo                                    | Conteúdo                                                                         | Tarefa relacionada |
|--------------------------------------------|----------------------------------------------------------------------------------|--------------------|
| `docs/README.md`                           | Índice da documentação e link para a guideline de commits                        | T0.2               |
| `docs/projeto/apresentacao.md`             | Contexto, problema e solução em linguagem acessível                              | Antes de 11/11     |
| `docs/projeto/status-e-roadmap.md`         | Retrato do estado atual, com a verificação no código                             | T0.5               |
| `docs/projeto/roadmap.md`                  | Este documento                                                                   | T0.1               |
| `docs/projeto/relatorio-final-disciplina.pdf` | Relatório final da disciplina (mover o PDF que já está em `docs/`)            | T0.6               |
| `docs/arquitetura/arquitetura.md`          | Como transmissor e receptor se encaixam; pontos de atenção                       | T0.6               |
| `docs/arquitetura/decisoes-de-hardware.md` | Por que RP2040-LoRa e SX1262, por que esse circuito sensor, alternativas descartadas (a abordagem por bobina e o motivo) | T1.1 |
| `docs/hardware/circuito-sensor.md`         | Esquemático, valores, segurança elétrica                                         | T1.1, T1.6         |
| `docs/firmware/protocolo.md`               | Pacote, ACK, retentativas, *timeouts*                                            | T2.1               |
| `docs/firmware/compilacao.md`              | Build, versões, gravação na placa                                                | T0.7               |
| `docs/testes/`                             | Registros de bancada, alcance e ensaio geral, com data e versão                  | T1.4, T2.3, T3.8   |
| `docs/testes/plano-teste-campo.md`         | Plano do teste em campo                                                          | T4.2               |
| `docs/testes/relatorio-teste-campo.md`     | Relatório do teste em campo                                                      | T4.5               |

## 12. Como manter este documento

- Revisar a cada reunião semanal; atualizar o status das tarefas e a data no topo.
- Ao concluir uma tarefa, atualizar o documento da área em `docs/` e o [status-e-roadmap.md](status-e-roadmap.md) no mesmo commit.
- Os IDs (`T0.3`, `D2`, `G4`…) servem para abrir cards no quadro; cada card deve citar o ID e o critério de pronto.
- Mudança de escopo (mover item entre as fases e a seção 10) deve ser registrada na reunião e na data de revisão.
