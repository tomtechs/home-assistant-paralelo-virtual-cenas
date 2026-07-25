# Paralelo Virtual por Cenas no Home Assistant

Automação para utilizar as teclas livres de um interruptor como seletores de cenas no Home Assistant.

Neste exemplo, foi utilizado um interruptor de quatro teclas:

| Tecla | Função                       |
| ----- | ---------------------------- |
| L1    | Retorno físico de iluminação |
| L2    | Cena Estudar                 |
| L3    | Cena Dormir                  |
| L4    | Cena Desligar Tudo           |

A tecla **L1 continua funcionando normalmente**, ligada diretamente a uma iluminação física.

As teclas **L2, L3 e L4 não possuem cargas conectadas**. Elas são utilizadas exclusivamente como comandos virtuais para o Home Assistant.

---

## Funcionamento

As teclas funcionam como um seletor de cenas com indicação visual pelo próprio LED do interruptor.

### L2 — Estudar

Ao ligar a tecla L2:

* Desliga L3 e L4;
* Apaga a iluminação indireta;
* Acende a iluminação principal;
* Mantém o LED da tecla L2 ligado.

Ao desligar a tecla L2:

* Apaga somente a iluminação principal;
* Não altera as outras cenas.

### L3 — Dormir

Ao ligar a tecla L3:

* Desliga L2 e L4;
* Apaga a iluminação principal;
* Acende o cortineiro com brilho baixo;
* Mantém o LED da tecla L3 ligado.

Ao desligar a tecla L3:

* Apaga somente o cortineiro;
* Não altera as outras cenas.

### L4 — Desligar Tudo

Ao ligar a tecla L4:

* Desliga L2 e L3;
* Apaga todas as iluminações;
* Mantém o LED da tecla L4 ligado.

Na prática, o LED da tecla indica qual cena está selecionada.

---

## Requisitos

Para utilizar esta automação, você precisa ter:

* Home Assistant instalado e funcionando;
* Interruptor integrado ao Home Assistant;
* Teclas virtuais disponíveis como entidades do tipo `switch`;
* Iluminações disponíveis como entidades do tipo `light`;
* Teclas utilizadas na automação sem cargas físicas conectadas.

> [!IMPORTANT]
> Antes de utilizar uma saída do interruptor como paralelo virtual, confirme que ela não está alimentando nenhuma carga elétrica.

No meu caso:

* L1 possui retorno físico;
* L2, L3 e L4 são utilizadas exclusivamente pelo Home Assistant.

---

## Entidades utilizadas

### Teclas do interruptor

```yaml
switch.interruptor_quarto_julia_l2
switch.interruptor_quarto_julia_l3
switch.interruptor_quarto_julia_l4
```

### Iluminações

```yaml
light.led_cortineiro_julia
light.led_teto_quarto_yasmin
```

Você deve substituir essas entidades pelos nomes existentes na sua instalação do Home Assistant.

---

## Instalação

No Home Assistant, acesse:

```text
Configurações
→ Automações e cenas
→ Criar automação
→ Criar nova automação
→ Menu de três pontos
→ Editar em YAML
```

Apague o conteúdo existente e cole o YAML abaixo.

---

## Automação completa

```yaml
alias: "Quarto Júlia - Paralelo virtual por cenas"
description: >
  Usa L2, L3 e L4 como seletores de cenas, mantendo o LED das
  teclas sincronizado com a cena selecionada.

triggers:

  # ============================================================
  # L2 - ESTUDAR
  # ============================================================

  - trigger: state
    entity_id: switch.interruptor_quarto_julia_l2
    from: "off"
    to: "on"
    id: estudar_ligar

  - trigger: state
    entity_id: switch.interruptor_quarto_julia_l2
    from: "on"
    to: "off"
    id: estudar_desligar

  # ============================================================
  # L3 - DORMIR
  # ============================================================

  - trigger: state
    entity_id: switch.interruptor_quarto_julia_l3
    from: "off"
    to: "on"
    id: dormir_ligar

  - trigger: state
    entity_id: switch.interruptor_quarto_julia_l3
    from: "on"
    to: "off"
    id: dormir_desligar

  # ============================================================
  # L4 - DESLIGAR TUDO
  # ============================================================

  - trigger: state
    entity_id: switch.interruptor_quarto_julia_l4
    from: "off"
    to: "on"
    id: tudo_desligado

conditions: []

actions:

  - choose:

      # ========================================================
      # L2 LIGADA - ATIVAR ESTUDAR
      # ========================================================

      - conditions:
          - condition: trigger
            id: estudar_ligar

        sequence:

          - action: switch.turn_off
            target:
              entity_id:
                - switch.interruptor_quarto_julia_l3
                - switch.interruptor_quarto_julia_l4

          - action: light.turn_off
            target:
              entity_id: light.led_cortineiro_julia

          - action: light.turn_on
            target:
              entity_id: light.led_teto_quarto_yasmin
            data:
              brightness_pct: 90
              color_temp_kelvin: 4500

      # ========================================================
      # L2 DESLIGADA - DESATIVAR ESTUDAR
      # ========================================================

      - conditions:
          - condition: trigger
            id: estudar_desligar

        sequence:

          - action: light.turn_off
            target:
              entity_id: light.led_teto_quarto_yasmin

      # ========================================================
      # L3 LIGADA - ATIVAR DORMIR
      # ========================================================

      - conditions:
          - condition: trigger
            id: dormir_ligar

        sequence:

          - action: switch.turn_off
            target:
              entity_id:
                - switch.interruptor_quarto_julia_l2
                - switch.interruptor_quarto_julia_l4

          - action: light.turn_off
            target:
              entity_id: light.led_teto_quarto_yasmin

          - delay:
              milliseconds: 300

          - action: light.turn_on
            target:
              entity_id: light.led_cortineiro_julia
            data:
              brightness_pct: 5

      # ========================================================
      # L3 DESLIGADA - DESATIVAR DORMIR
      # ========================================================

      - conditions:
          - condition: trigger
            id: dormir_desligar

        sequence:

          - action: light.turn_off
            target:
              entity_id: light.led_cortineiro_julia

      # ========================================================
      # L4 LIGADA - DESLIGAR TUDO
      # ========================================================

      - conditions:
          - condition: trigger
            id: tudo_desligado

        sequence:

          - action: switch.turn_off
            target:
              entity_id:
                - switch.interruptor_quarto_julia_l2
                - switch.interruptor_quarto_julia_l3

          - action: light.turn_off
            target:
              entity_id:
                - light.led_cortineiro_julia
                - light.led_teto_quarto_yasmin

mode: queued
max: 10
max_exceeded: silent
```

---

## O que precisa ser alterado

### Entidades das teclas

Substitua:

```yaml
switch.interruptor_quarto_julia_l2
switch.interruptor_quarto_julia_l3
switch.interruptor_quarto_julia_l4
```

Pelas entidades correspondentes às teclas do seu interruptor.

Exemplo:

```yaml
switch.interruptor_quarto_l2
switch.interruptor_quarto_l3
switch.interruptor_quarto_l4
```

### Entidades das luzes

Substitua:

```yaml
light.led_cortineiro_julia
light.led_teto_quarto_yasmin
```

Pelas entidades das suas iluminações.

---

## Ajustes da cena Estudar

A intensidade e a temperatura de cor da iluminação principal são configuradas neste trecho:

```yaml
brightness_pct: 90
color_temp_kelvin: 4500
```

### Intensidade

O valor de `brightness_pct` pode variar de `1` a `100`.

Exemplo com 80%:

```yaml
brightness_pct: 80
```

### Temperatura de cor

Alguns exemplos:

| Temperatura | Característica     |
| ----------- | ------------------ |
| 2700 K      | Branco quente      |
| 3000 K      | Branco quente      |
| 4000 K      | Branco neutro      |
| 4500 K      | Branco neutro/frio |
| 6000 K      | Branco frio        |

Exemplo:

```yaml
color_temp_kelvin: 4000
```

> [!NOTE]
> A entidade da luz precisa suportar ajuste de temperatura de cor para utilizar `color_temp_kelvin`.

---

## Ajuste da cena Dormir

O brilho do cortineiro é configurado neste trecho:

```yaml
brightness_pct: 5
```

Você pode aumentar ou diminuir conforme o comportamento do seu LED.

Exemplo com 3%:

```yaml
brightness_pct: 3
```

> [!NOTE]
> Alguns controladores não conseguem acender corretamente em níveis muito baixos. Caso a luz falhe ao ligar, aumente gradualmente para 5%, 7% ou 10%.

---

## Como o intertravamento funciona

Quando uma cena é ligada, a automação desliga as outras teclas.

Exemplo ao ativar L3:

```yaml
- action: switch.turn_off
  target:
    entity_id:
      - switch.interruptor_quarto_julia_l2
      - switch.interruptor_quarto_julia_l4
```

Dessa forma:

* Apenas uma cena permanece selecionada;
* O LED do interruptor representa a cena ativa;
* As teclas funcionam como seletores físicos de cenas;
* O estado visual do interruptor permanece sincronizado com o Home Assistant.

---

## Observações importantes

> [!WARNING]
> Não utilize esta automação em saídas que estejam alimentando cargas físicas.

As teclas L2, L3 e L4 devem estar configuradas apenas como comandos virtuais.

Também é recomendado confirmar que o interruptor permite:

* Controle independente de cada canal;
* Acionamento pelo Home Assistant;
* Atualização correta do estado `on` e `off`;
* Controle do LED da tecla por meio do próprio estado do canal.

---

## Resultado

Com essa configuração, um interruptor tradicional passa a funcionar como um painel de cenas:

```text
L1 → Iluminação física comum
L2 → Estudar
L3 → Dormir
L4 → Desligar tudo
```

O LED de cada tecla indica qual cena está ativa, criando um paralelo virtual baseado em cenas e totalmente integrado ao Home Assistant.
