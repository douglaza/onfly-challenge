# Entrega - Desafio Onfly

## Dados do candidato

- Nome: Douglas A C Silva
- Data: 11/11/2026
- Repositório: https://github.com/douglaza/onfly-challenge

---

## Investigação e abordagem

Iniciei a investigação executando a suíte de testes fornecida pelo desafio:

```bash
pytest -q
```

As falhas reportadas indicavam divergências entre a implementação do CLI e as regras de negócio descritas no README. Para cada teste com falha, segui o mesmo processo:

- analisar a mensagem de erro e o valor esperado;
- identificar qual regra de negócio estava sendo violada;
- localizar o código responsável pela implementação da regra;
- corrigir a causa raiz do problema;
- executar novamente o teste específico relacionado à falha;
- reexecutar toda a suíte de testes ao final das correções.

Durante a investigação, comparei constantemente:

- os requisitos descritos no README;
- os resultados esperados pelos testes;
- a implementação presente no código-fonte.

Após a correção dos defeitos identificados, também executei a amostra oficial de validação disponibilizada no desafio para confirmar o comportamento esperado do relatório.

## Defeitos corrigidos

### 1) Conversão cambial incorreta

**Causa raiz:** a conversão para BRL utilizava divisão pela taxa de câmbio em vez de multiplicação.

**Impacto:** os valores convertidos para BRL ficavam significativamente menores do que o esperado, afetando totais financeiros, médias e rankings.

**Correção:** a função de conversão passou a utilizar a regra especificada pelo desafio.

Antes:

```python
amount / rate
```

Depois:

```python
amount * rate
```

Também foi removido o uso de `float`, mantendo todos os cálculos monetários utilizando `Decimal`.

### 2) Cálculo incorreto de room nights

**Causa raiz:** o cálculo utilizava apenas o dia do mês (`checkout.day - checkin.day`), desconsiderando mudanças de mês e ano.

**Impacto:** reservas que atravessavam a virada de mês podiam gerar quantidade negativa ou incorreta de noites.

**Correção:** o cálculo passou a utilizar a diferença entre objetos `date`, conforme definido na especificação.

Antes:

```python
(checkout.day - checkin.day) * rooms
```

Depois:

```python
(checkout - checkin).days * rooms
```

### 3) Tratamento incorreto dos status faturáveis

**Causa raiz:** a política de faturamento excluía apenas reservas com status `CANCELLED`, permitindo que reservas `NO_SHOW` fossem consideradas nos totais financeiros.

**Impacto:** valores faturados, médias e rankings eram calculados com reservas que não deveriam participar das agregações financeiras.

**Correção:** a regra passou a considerar como faturáveis apenas reservas com status `CONFIRMED`.

Antes:

```python
status != "CANCELLED"
```

Depois:

```python
status == "CONFIRMED"
```

### 4) Ordenação incorreta do ranking de hotéis

**Causa raiz:** o ranking era ordenado de forma crescente pelo valor faturado, enquanto a especificação exigia ordenação decrescente.

**Impacto:** hotéis com menor faturamento apareciam nas primeiras posições do ranking.

**Correção:** a ordenação foi ajustada para respeitar a regra definida pelo desafio:

- total faturado em ordem decrescente;
- `hotel_id` em ordem ascendente para desempate.

Antes:

```python
key=lambda item: (item[1]["amount"], item[0])
```

Depois:

```python
key=lambda item: (-item[1]["amount"], item[0])
```

## Decisões técnicas

### Valores monetários

Todas as operações financeiras utilizam `Decimal`, conforme exigido pelo desafio. O arredondamento ocorre apenas durante a serialização para JSON utilizando arredondamento `ROUND_HALF_UP`.

### Regras de negócio validadas

Foram validadas as seguintes regras:

- conversão cambial para BRL;
- contabilização de status;
- exclusão de `CANCELLED` e `NO_SHOW` dos totais financeiros;
- cálculo de room nights;
- agrupamento por mês utilizando o mês do check-in;
- ranking de hotéis por faturamento;
- critério de desempate por `hotel_id`;
- formatação monetária com duas casas decimais;
- cálculo da média por diária;
- ordenação dos resultados conforme especificação.

## Como validar

### Executar os testes

```bash
pytest -q
```

**Resultado esperado:**

```text
5 passed
```

### Executar o CLI com o dataset principal

```bash
PYTHONPATH=src python -m onfly_report \
  --input data/reservas_hoteis.csv \
  --rates data/taxas_cambio.csv
```

**Saída obtida durante a validação:**

```text
Reservas: 200 | Confirmadas: 153 | Noites: 909 | Total BRL: 249543.79
```

O comando deve gerar o arquivo:

```text
output/report.json
```

### Validar utilizando a amostra oficial do desafio

```bash
PYTHONPATH=src python -m onfly_report \
  --input data/exemplo-validacao.csv \
  --rates data/taxas_cambio.csv \
  --output output/exemplo.json
```

**Saída obtida:**

```text
Reservas: 12 | Confirmadas: 9 | Noites: 28 | Total BRL: 7566.00
```

Os valores gerados conferem com os resultados esperados descritos no README do desafio.

## Resultado do pytest

**Comando executado:**

```bash
pytest -q
```

**Saída:**

```text
..... [100%]
5 passed in 0.34s
```

## Considerações finais

Durante as correções foram preservados:

- o comando do CLI;
- os argumentos da aplicação;
- o schema do relatório;
- a semântica dos campos;
- os arquivos de entrada fornecidos pelo desafio.

Nenhum teste fornecido foi alterado.

Todos os testes da suíte disponibilizada passaram com sucesso após as correções.
