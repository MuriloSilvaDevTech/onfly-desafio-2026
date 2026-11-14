# Declaração de uso de IA

## Ferramenta e modelo

- **Ferramenta:** Claude (Anthropic), pela interface de chat (claude.ai / app).
- **Modelo:** Claude Sonnet 5.5.

## Uso declarado

Utilizei IA como apoio na conclusão deste projeto, com dois focos:

1. **Solucionar o erro do ranking de hotéis** (`top_hotels`).
2. **Construir os READMEs solicitados** para a entrega.

## O que a IA fez

- **Ranking de hotéis:** a partir de capturas de tela do teste `test_top_hotels_are_ranked_descending`, do trecho de código de ordenação e do `AssertionError` (`['H04', 'H03', 'H02']` no lugar de `['H01', 'H02', 'H03']`), explicou a causa: `sorted()` ordena de forma crescente e o fatiamento `[: top_n]` pegava os hotéis de menor valor. Sugeriu a correção com `-amount` e desempate por `hotel_id` crescente, com alertas sobre o tipo de `amount` (`Decimal` x string).
- **READMEs:** redigiu o `../README-ENTREGA.md` (defeitos corrigidos, causa raiz em uma frase por defeito, como o problema foi investigado, como validar e resultado do `pytest`) e esta declaração.
- **Apoio na validação:** na sessão, a IA também aplicou no código as correções dos demais defeitos descritos no `../README-ENTREGA.md` e conferiu o resultado contra os testes fornecidos e a amostra de validação do enunciado.

## O que eu fiz

- Identifiquei os defeitos por **análise do código e busca de palavras-chave** que indicavam a causa raiz de cada problema, conforme descrito no `../README-ENTREGA.md`.
- Encontrei o erro do ranking nos testes, levei o caso à IA e analisei a explicação recebida.
- Revisei as correções e os textos gerados pela IA, e executo o `pytest -q` localmente para confirmar o resultado.
- Mantive os testes fornecidos sem alterações e preservei o contrato do CLI e os arquivos de dados.
- Sou responsável pela versão final entregue.

## Sessão

A exportação da sessão com a IA acompanha esta declaração em `sessao.md` (ou formato equivalente) na mesma pasta.
