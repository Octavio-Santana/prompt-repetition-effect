# Repetir o Prompt Melhora a Acurácia?

## Um Estudo Experimental com SmolLM3-3B

Recentemente, li um artigo que investigava um fenômeno curioso: **a simples repetição da instrução no prompt pode melhorar o desempenho de modelos de linguagem** ([ref](https://arxiv.org/abs/2512.14982)).

A hipótese é contraintuitiva. Em tese, repetir a mesma instrução não adiciona nova informação semântica. Ainda assim, evidências sugerem que essa redundância pode reforçar o foco atencional do modelo e melhorar a qualidade das respostas.

Motivado por essa ideia, resolvi conduzir um experimento controlado para verificar se o efeito se mantém em um modelo open-weight relativamente pequeno: o SmolLM3-3B.

Este texto apresenta os resultados dessa investigação.

---

## Contexto do Experimento

O objetivo foi simples: testar se repetir a instrução de classificação melhora a performance em uma tarefa real.

A tarefa escolhida foi **classificação binária de sentimento** (positivo ou negativo).
Utilizei um conjunto de 500 amostras reais.

Foram comparadas duas configurações:

### Prompt Normal

Uma única instrução seguida do texto:

```python
"""Classifique o sentimento como positivo ou negativo.
Texto: ...
Sentimento:"""
```

### Prompt Repetido

A mesma instrução e o mesmo texto duplicados no contexto:

```python
"""Classifique o sentimento como positivo ou negativo.
Texto: ...

Classifique o sentimento como positivo ou negativo.
Texto: ...
Sentimento:"""
```

Importante:
Não houve fine-tuning, ajuste de temperatura ou qualquer alteração nos pesos do modelo. Apenas modificamos a estrutura do prompt.

---

## Resultados de Acurácia

Os resultados foram os seguintes:

* **Prompt Normal:** 80% de acurácia
* **Prompt Repetido:** 88,2% de acurácia

Isso representa um ganho absoluto de **8,2 pontos percentuais**.

Em termos práticos:

* 400 classificações corretas com prompt normal
* 441 classificações corretas com prompt repetido
* 41 previsões adicionais corretas

Para um modelo de 3 bilhões de parâmetros, esse é um ganho expressivo obtido apenas via engenharia de prompt.

---

## O Efeito é Estatisticamente Significativo?

Como os dois métodos foram avaliados exatamente nas mesmas 500 amostras, aplicamos o **teste de McNemar**, apropriado para comparar classificadores pareados.

Os resultados foram:

* 47 casos onde o prompt normal acertou e o repetido errou
* 88 casos onde o normal errou e o repetido acertou

Estatística χ² = 11,85
p-value = 0,000576

Isso significa que a probabilidade de esse ganho ter ocorrido por acaso é inferior a 0,1%.

Ou seja:

**O efeito é estatisticamente altamente significativo.**

Não se trata de ruído amostral.

---

## Custo Computacional

Naturalmente, há um trade-off.

### Tokens médios

* Normal: 366 tokens
* Repetido: 730 tokens

Praticamente o dobro.

### Latência média

* Normal: 0,264 s
* Repetido: 0,409 s

A latência aumentou cerca de 55%.

Portanto, o ganho de 8,2% em acurácia vem acompanhado de:

* ~2x uso de tokens
* +0,145 segundos por requisição

Dependendo do cenário (API paga, sistema em tempo real, edge deployment), isso pode ou não ser aceitável.

---

## Um Detalhe Interessante

A repetição não melhora todos os casos.

Embora 88 exemplos tenham sido corrigidos pela repetição, 47 pioraram.

Isso sugere que o efeito não é monotônico nem universal.
Há algum padrão estrutural determinando quando a repetição ajuda e quando atrapalha.

Essa é, possivelmente, a parte mais interessante da investigação.

---

## Por Que Isso Acontece?

Algumas hipóteses plausíveis:

1. **Reforço Atencional**
   A duplicação pode aumentar o peso da instrução nos mecanismos de atenção.

2. **Viés do Treinamento**
   O modelo pode ter sido exposto a dados com redundância instrucional.

3. **Ancoragem de Contexto**
   A repetição reduz ambiguidade e estabiliza o enquadramento da tarefa.

É importante destacar: nada no prompt adiciona nova informação.
A melhoria surge exclusivamente da redundância estrutural.

