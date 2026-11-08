# Javalarm — apresentação do projeto

## O problema

Javalis e outros invasores destroem lavouras e ferem animais. Muitos pequenos produtores usam cerca elétrica para se proteger, mas a cerca pode perder tensão (por um fio rompido, mato encostando ou queda de energia) e o produtor só descobre depois do prejuízo. Percorrer o perímetro o tempo todo não é viável.

## A solução

O Javalarm é um sistema de **baixo custo e baixo consumo** que avisa o produtor assim que a cerca elétrica para de funcionar.

- Um **transmissor**, ligado à cerca por um circuito de medição seguro, percebe quedas abruptas de tensão.
- Ele envia um alerta por **LoRa**, uma tecnologia de rádio de longo alcance que funciona sem internet e com pouca energia, própria para áreas rurais.
- Um **receptor** perto do produtor mostra o aviso por uma luz (LED). Som e display fazem parte dos próximos passos.
- O receptor responde com uma **confirmação de recebimento**. A repetição automática do alerta quando a confirmação não chega ainda será implementada.

## Onde estamos

- Protótipo funcional demonstrado em bancada, com um eletrificador doméstico: circuito de medição, firmware e comunicação LoRa entre transmissor e receptor.
- Projeto desenvolvido na disciplina *Projetos em Engenharia de Computação* e agora em fase de reestruturação: organizando documentação, plano de trabalho e tarefas.
- **Ainda não há resultados de campo.** A meta atual é realizar um teste em campo, e o plano de trabalho está organizado para deixar o sistema pronto para ele.

## Próximos passos

1. Garantir que o sistema detecta falhas sem falsos alarmes.
2. Tornar o alerta confiável: repetição automática, identificação do aparelho e aviso quando o transmissor para de responder.
3. Medir o alcance do rádio e reduzir o consumo de energia para que a bateria dure o período do teste.
4. Instalar o sistema em uma propriedade rural, registrar o que acontece e aprender com o produtor.
5. Documentar tudo para que outros possam reproduzir e melhorar.

## Impacto

Aumenta a segurança rural, reduz prejuízos e fortalece a autonomia de pequenos produtores. O projeto se alinha aos ODS 7, 10, 11 e 12, conforme o README.

## Tecnologias

Placa Waveshare RP2040-LoRa (microcontrolador RP2040 com rádio LoRa SX1262), firmware em C/C++ (Pico SDK), circuito analógico de sensoriamento.
