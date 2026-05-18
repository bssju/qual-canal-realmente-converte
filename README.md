# Análise de Mídia Paga — Attribution Modeling & Budget Optimization

Análise completa de campanhas de mídia paga para responder à pergunta: **qual canal realmente gera conversões — e como alocar o budget para maximizar ROAS?**

Dataset sintético calibrado com benchmarks reais de Google Ads / Meta Ads (WordStream, Meta Business, 2023): 2.000 campanhas em 5 canais ao longo de 2023.

---

## Resultados

### EDA — Métricas de campanha

| Canal | Investimento total | ROAS mediano | CPC mediano | CTR mediano | Conversões |
|---|---|---|---|---|---|
| Google Search | R$ 1.783.890 | 2,27x | R$ 3,12 | 4,4% | 23.479 |
| Meta Feed | R$ 262.571 | 2,28x | R$ 1,29 | 1,8% | 4.090 |
| Instagram Feed | R$ 199.064 | 1,66x | R$ 1,54 | 2,2% | 2.048 |
| Meta Stories | R$ 68.628 | 1,60x | R$ 0,93 | 1,2% | 945 |
| Google Display | R$ 19.752 | 0,00x* | R$ 0,73 | 0,4% | 78 |
| **Total** | **R$ 2.333.905** | **2,32x** | — | — | **30.640** |

*ROAS direto do Display é 0 — o canal serve para awareness e seu valor real aparece na atribuição multi-touch.

Correlações relevantes: spend → revenue = **0,86** (forte), CTR → ROAS = **0,37** (moderada), CPC → ROAS = **0,11** (fraca). A correlação baixa entre CPC e ROAS justifica o attribution modeling: pagar mais por clique não se traduz diretamente em mais receita — o canal e a posição na jornada importam.

### Jornadas multi-touch

30.640 conversões simuladas com distribuição de comprimento realista. A distribuição do **último toque** — que determina quem ganha crédito no last-click — já revela o viés:

| Canal | Frequência como último toque |
|---|---|
| Google Search | 45,7% |
| Meta Feed | 27,0% |
| Instagram Feed | 11,8% |
| Meta Stories | 9,5% |
| Google Display | 5,9% |

O Display aparece como último toque em apenas 5,9% das jornadas — mas está presente no topo do funil em muito mais. Esse é o viés que o Shapley vai corrigir.

### Attribution Modeling — distribuição de crédito por canal (% da receita total)

| Canal | Last-Click | Linear | Time-Decay | **Shapley Values** | Δ Shapley − Last-Click |
|---|---|---|---|---|---|
| Google Search | 45,8% | 34,1% | 34,8% | **34,1%** | **−11,8 p.p.** |
| Meta Feed | 27,0% | 23,6% | 23,8% | **23,7%** | −3,3 p.p. |
| Instagram Feed | 11,7% | 13,1% | 13,0% | **13,2%** | +1,5 p.p. |
| Meta Stories | 9,6% | 13,4% | 13,2% | **13,6%** | +4,0 p.p. |
| Google Display | 5,9% | 15,8% | 15,2% | **15,4%** | **+9,5 p.p.** |

**Valores absolutos atribuídos:**

| Canal | Last-Click (R$) | Shapley Values (R$) | Diferença (R$) |
|---|---|---|---|
| Google Search | 2.436.719 | 1.810.163 | −626.556 |
| Meta Feed | 1.433.491 | 1.261.314 | −172.177 |
| Instagram Feed | 621.367 | 703.115 | +81.748 |
| Meta Stories | 510.329 | 720.442 | +210.113 |
| Google Display | 313.566 | 820.439 | **+506.873** |

O Last-Click distorce a atribuição em dois sentidos opostos:

- **Supervaloriza Google Search em −11,8 p.p.:** o canal captura o clique final, mas frequentemente o usuário chegou via Display ou Stories antes. R$ 626.556 a mais de crédito do que merece.
- **Subvaloriza Google Display em +9,5 p.p.:** o canal raramente é o último clique, mas aparece no início de boa parte das jornadas que convertem. R$ 506.873 a menos de crédito do que merece.

Linear e Time-Decay chegam a conclusões semelhantes ao Shapley, mas sem o embasamento axiomático. O Shapley é o único modelo que satisfaz eficiência, simetria, nulidade e aditividade simultaneamente.

### Budget Optimization

**Curvas de resposta estimadas (ROAS = a · spend^b, b < 1 confirma retornos decrescentes):**

| Canal | a | b |
|---|---|---|
| Google Search | 2,401 | 0,010 |
| Google Display | 1,599 | 0,010 |
| Meta Feed | 2,428 | 0,010 |
| Meta Stories | 1,964 | 0,010 |
| Instagram Feed | 1,745 | 0,010 |

| Cenário | Receita esperada | ROAS | Ganho vs. atual |
|---|---|---|---|
| Alocação atual | R$ 5.422.499 | 2,32x | — |
| **SLSQP** (contínuo) | **R$ 6.195.557** | **2,65x** | **+14,3% (+R$ 773.058)** |
| **ILP** (discreto, blocos de R$ 1k) | **R$ 6.200.405** | **2,66x** | **+14,3% (+R$ 777.906)** |

A diferença entre SLSQP e ILP é de apenas **+R$ 4.848 (−0,08%)** — o custo da granularidade discreta é negligenciável nesta escala. O ILP é preferível operacionalmente porque gera alocações em múltiplos de R$ 1.000, compatíveis com os mínimos de budget das plataformas e com o fluxo de trabalho de equipes de mídia.

**Realocação recomendada:**

| Canal | Atual | SLSQP | ILP | Δ SLSQP vs atual |
|---|---|---|---|---|
| Google Search | R$ 1.783.890 | R$ 971.115 | R$ 819.000 | −45,6% |
| Google Display | R$ 19.752 | R$ 116.695 | R$ 116.000 | **+490,8%** |
| Meta Feed | R$ 262.571 | R$ 1.012.705 | R$ 1.166.000 | **+285,7%** |
| Meta Stories | R$ 68.628 | R$ 116.695 | R$ 116.000 | +70,0% |
| Instagram Feed | R$ 199.064 | R$ 116.695 | R$ 116.000 | −41,4% |
| **Total** | **R$ 2.333.905** | **R$ 2.333.905** | **R$ 2.333.000** | — |

A otimização redistribui budget de Google Search (saturado, ROAS marginal decrescente) para Meta Feed e Google Display (subinvestidos, maior ROAS marginal disponível).

---

## Estrutura

```
media-paga-attribution/
├── media-paga-attribution.ipynb
└── README.md
```

---

## Etapas do Projeto

### 1. EDA de métricas de campanha

Análise exploratória cobrindo CPC, CTR, ROAS e CPA por canal e tipo de campanha, com verificação de sazonalidade mensal (Q4 com multiplicador de 1,35× pelo período Black Friday/Natal) e correlações entre métricas.

A EDA motiva o attribution modeling: o ROAS direto do Google Display (0,57x agregado) parece ruim, mas a correlação spend → revenue de 0,86 no dataset geral indica que o problema não é o canal em si — é a escala ínfima de investimento (R$ 19.752 de um total de R$ 2,3M) que impede avaliar seu potencial real.

---

### 2. Simulação de jornadas multi-touch

Datasets reais de campanhas raramente expõem sequências de touchpoints individuais (privacidade das plataformas). O padrão em pesquisa de attribution modeling é simular jornadas com distribuição calibrada por canal e posição:

- **Primeiro toque:** Display (30%) e Stories (20%) predominam — awareness
- **Último toque:** Search (45%) e Meta Feed (28%) predominam — conversão
- **Distribuição de comprimento:** 40% com 2 toques, 25% com 3, 25% com 1, 10% com 4
- **Total gerado:** 30.640 conversões, valor médio R$ 173,48

Essa estrutura cria exatamente o viés que o last-click vai explorar — e que o Shapley vai corrigir.

---

### 3. Attribution Modeling — quatro modelos com premissas distintas

#### Por que quatro modelos?

| Modelo | Premissa | Limitação |
|---|---|---|
| Last-Click | Crédito total ao último toque | Ignora toda a jornada de awareness |
| Linear | Crédito igual entre todos os toques | Cego à posição no funil |
| Time-Decay | Mais crédito a toques recentes (meia-vida: 7 dias) | Enviesa contra canais de awareness em ciclos longos |
| **Shapley Values** | Contribuição marginal média em todos os subconjuntos possíveis | Computacionalmente custoso (O(2ⁿ·n)), mitigado por amostragem |

#### Shapley Values — por que é o modelo de referência

O Shapley value de canal $i$ é:

$$\phi_i = \sum_{S \subseteq N \setminus \{i\}} \frac{|S|!(n-|S|-1)!}{n!} \left[v(S \cup \{i\}) - v(S)\right]$$

onde $v(S)$ é a receita atribuível ao subconjunto $S$ de canais. Com 5 canais, são $2^5 = 32$ subconjuntos — calculados de forma exata sobre amostra de 2.000 jornadas (erro de estimação < 1%) e escalados para a base completa de 30.640 conversões.

Os quatro axiomas que o Shapley satisfaz e os outros modelos violam: **eficiência** (soma das atribuições = receita total), **simetria** (canais com contribuição igual recebem crédito igual), **nulidade** (canal sem contribuição recebe zero) e **aditividade** (atribuição de múltiplos períodos = soma das atribuições individuais).

---

### 4. Budget Optimization — duas abordagens complementares

#### Curvas de resposta — retornos decrescentes

A função de receita por canal segue $\text{Revenue}(x) = a \cdot x^{1+b}$, onde $b < 1$ garante retornos decrescentes. Os parâmetros são estimados por ajuste de curva (scipy.optimize.curve_fit) sobre dados observados agrupados por decil de investimento.

#### SLSQP — otimização contínua

Maximiza $\sum_i a_i \cdot x_i^{1+b_i}$ sujeito a $\sum_i x_i = B_{total}$ e $0{,}05B \leq x_i \leq 0{,}50B$. Converge para receita de **R$ 6.195.557 (+14,3%)**.

#### ILP com PuLP — alocação discreta

Lineariza a função côncava por aproximação piecewise linear em 20 segmentos por canal, formulada como programação linear inteira. Resolvido com CBC (solver open-source do PuLP). Status: **Optimal**. Receita: **R$ 6.200.405 (+14,3%)**.

A diferença de receita entre SLSQP e ILP é de apenas **R$ 4.848** — o ILP chega levemente acima do contínuo porque a discretização favorece Meta Feed (canal de maior ROAS marginal) na margem.

---

## Resposta à pergunta central

**O modelo Last-Click distorce sistematicamente a atribuição — e induz alocações de budget subótimas.** O Google Display recebe apenas 5,9% da atribuição no last-click, mas 15,4% no Shapley: o canal está financiado a R$ 19.752 quando deveria receber R$ 116.000+ para operar na faixa de ROAS marginal positivo.

A realocação baseada em Shapley + ILP eleva o ROAS esperado de 2,32× para 2,66× — **ganho de +14,3% de receita (R$ 777.906) sem aumento de budget**, redistribuindo investimento de Google Search (saturado) para Meta Feed e Google Display (subinvestidos).

---

## Tecnologias

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![scipy](https://img.shields.io/badge/scipy-8CAAE6?style=flat)
![PuLP](https://img.shields.io/badge/PuLP-FF6B35?style=flat)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-F7931E?style=flat&logo=scikit-learn&logoColor=white)

---

*Juliana Burato — 2026*