
# Registro de uso de IA

## Ferramenta

Microsoft Copilot

## Contexto

A ferramenta foi utilizada como apoio durante a investigação dos testes falhos e na interpretação dos requisitos descritos no desafio.

A implementação das correções foi realizada manualmente após análise do código-fonte e execução local dos testes.

## Interações relevantes

### Investigação inicial

**Pergunta:**

Analisar a saída do `pytest` e identificar possíveis causas para os testes falhos.

**Resultado:**

Foram levantadas hipóteses relacionadas a:

- conversão cambial utilizando divisão em vez de multiplicação;
- cálculo incorreto de room nights em reservas que atravessam meses;
- aplicação incorreta das regras de status;
- ordenação incorreta do ranking de hotéis.

---

### Conversão cambial

**Pergunta:**

Como localizar a origem do erro do teste de conversão cambial?

**Resultado:**

Foi sugerida a inspeção da função responsável pela conversão de moedas.

Durante a análise foi identificado que o código utilizava:

```python
amount / rate
```

em vez de:

```python
amount * rate
```

---

### Room nights

**Pergunta:**

Como investigar o cálculo de noites de hospedagem?

**Resultado:**

Foi sugerida a busca por uso de `.day` nas datas.
A investigação encontrou:

```python
(checkout.day - checkin.day) * rooms
```

que foi substituído por:

```python
(checkout - checkin).days * rooms
```

---

### Regras de status

**Pergunta:**

Como localizar a causa do teste relacionado a `CANCELLED` e `NO_SHOW`?

**Resultado:**

Foi sugerida a busca por filtros envolvendo `CANCELLED`.
A investigação identificou uma política equivalente a:

```python
status != "CANCELLED"
```

que permitia a cobrança de reservas `NO_SHOW`.
A regra foi corrigida para considerar exclusivamente:

```python
status == "CONFIRMED"
```

---

### Ranking de hotéis

**Pergunta:**

Como investigar a ordenação dos hotéis no ranking?

**Resultado:**

Foi identificada uma ordenação crescente por valor:

```python
key=lambda item: (item[1]["amount"], item[0])
```

A implementação foi ajustada para respeitar a especificação:

```python
key=lambda item: (-item[1]["amount"], item[0])
```

---

## Validação

Após as correções:

```bash
pytest -q
```

**Resultado:**

```text
5 passed in 0.34s
```

Também foi executada a amostra oficial de validação descrita no desafio, produzindo os valores esperados.

## Observação

A IA foi utilizada como ferramenta de apoio para investigação e revisão.
Toda a implementação das correções, execução dos testes, validação dos resultados e preparação da entrega foram realizadas manualmente pelo candidato.
