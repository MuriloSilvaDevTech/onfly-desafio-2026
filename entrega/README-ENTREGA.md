# Entrega — Consolidador de reservas de hotel (Onfly)

Todos os defeitos identificados no CLI foram **corrigidos**. Os testes fornecidos não foram alterados, e o relatório gerado sobre `data/exemplo-validacao.csv` bate com a amostra de validação do enunciado.

## 1. Defeitos corrigidos e causa raiz

| # | Defeito | Arquivo | Causa raiz (uma frase) |
|---|---------|---------|------------------------|
| 1 | Reservas `NO_SHOW` entravam nos totais financeiros e de noites | `services/status_policy.py` | `is_billable` usava `status != "CANCELLED"`, deixando `NO_SHOW` passar; o correto é `status == "CONFIRMED"`. |
| 2 | Noites de quarto erradas na virada de mês/ano | `services/stay_calculator.py` | O cálculo subtraía `checkout.day - checkin.day` (dia do mês) em vez de usar a diferença real entre datas. |
| 3 | Câmbio para BRL com valor invertido e impreciso | `services/currency_converter.py` | O código dividia o valor pela taxa (`amount / rate`) e passava por `float`, em vez de `Decimal` com `amount * rate_to_brl`. |
| 4 | `top_hotels` ordenado do menor para o maior valor | `services/report_builder.py` | `sorted` usava `(amount, hotel_id)` em ordem crescente, então o fatiamento `[:top_n]` pegava os hotéis de menor valor. |
| 5 | Relatório ignorava parte das reservas do arquivo | `repositories/csv_reservation_repository.py` | A leitura interrompia o loop com `break` ao atingir 200 linhas, mas o CSV tem 300 reservas. |

### Correções aplicadas

```python
# status_policy.py
return status == "CONFIRMED"

# stay_calculator.py
return (checkout - checkin).days * rooms

# currency_converter.py
return amount * rate

# report_builder.py
ranked_hotels = sorted(hotels.items(), key=lambda item: (-item[1]["amount"], item[0]))

# csv_reservation_repository.py
# removido o limite `if len(rows) == 200: break`
```

## 2. Como investiguei e abordei o problema

A investigação foi feita por **análise do código** e por **busca de palavras-chave** que indicassem a causa raiz de cada sintoma, e não por tentativa e erro:

1. **Leitura do fluxo ponta a ponta** seguindo a ordem do enunciado (`ler → converter tipos → contar status → filtrar CONFIRMED → agregar`), do `cli.py` até o `report_builder.py` e os serviços auxiliares.
2. **Busca por palavras-chave** ligadas às regras de negócio:
   - `CANCELLED`, `CONFIRMED`, `NO_SHOW` → localizou a política de status (defeito 1).
   - `.day`, `checkin`, `checkout` → localizou o cálculo de noites (defeito 2).
   - `float`, `/`, `rate` → localizou a conversão de moeda (defeito 3).
   - `sorted`, `reverse`, `top` → localizou a ordenação do ranking (defeito 4).
   - `break`, `200`, `len(rows)` → localizou o corte de linhas na leitura do CSV (defeito 5).
3. **Leitura das mensagens dos testes**: no teste do ranking, o `AssertionError` mostrou `['H04', 'H03', 'H02']` no lugar de `['H01', 'H02', 'H03']`, o que apontou ordem crescente na ordenação.
4. **Conferência com os dados reais**: `data/reservas_hoteis.csv` tem 300 reservas; o resumo do CLI passou a mostrar `Reservas: 300` depois da correção do defeito 5.
5. **Comparação com a amostra de validação** do enunciado (`data/exemplo-validacao.csv`) após cada correção.

## 3. Como validar

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

# 1) Testes fornecidos (sem alterações)
pytest -q

# 2) Amostra de validação
PYTHONPATH=src python -m onfly_report \
  --input data/exemplo-validacao.csv \
  --rates data/taxas_cambio.csv \
  --output output/exemplo.json

# 3) Dataset completo
PYTHONPATH=src python -m onfly_report \
  --input data/reservas_hoteis.csv \
  --rates data/taxas_cambio.csv \
  --output output/report.json
```

Valores esperados da amostra (`output/exemplo.json`):

| Campo | Esperado | Obtido |
|-------|----------|--------|
| `total_reservations` | 12 | 12 |
| `confirmed_reservations` | 9 | 9 |
| `cancelled_reservations` | 2 | 2 |
| `no_show_reservations` | 1 | 1 |
| `total_room_nights` | 28 | 28 |
| `total_amount_brl` | 7566.00 | 7566.00 |
| `average_nightly_rate_brl` | 270.21 | 270.21 |

`by_month` (2025-01: 8 / 25 / 6684.00 e 2025-02: 1 / 3 / 882.00) e `top_hotels` (H01, H02, H12, H10, H09) também coincidem com o enunciado.

No dataset completo, o CLI lê as 300 reservas: `Reservas: 300 | Confirmadas: 235 | Noites: 1347 | Total BRL: 396170.00`.

## 4. Resultado do pytest

```text
$ pytest -q
.....                                                                    [100%]
5 passed
```

| Teste | Resultado |
|-------|-----------|
| `test_currency.py::test_currency_conversion_to_brl` | passou |
| `test_nights.py::test_nights_across_month_boundary` | passou |
| `test_regression_report.py::test_regression_report_on_sample` | passou |
| `test_status_rules.py::test_status_rules_exclude_cancelled_and_no_show` | passou |
| `test_top_hotels.py::test_top_hotels_are_ranked_descending` | passou |

Antes das correções, os testes de câmbio, noites, regressão, status e ranking falhavam, como previsto no enunciado.
