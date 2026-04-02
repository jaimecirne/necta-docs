# Fundamentação Matemática do NECTA

O NECTA (*Neural Electrophysiological Connectivity Time-resolved Analysis*) incorpora estimadores robustos de conectividade para sinais eletrofisiológicos, abordando tanto as interferências espaciais (condução de volume) quanto as causalidades direcionais. 

## 1. Conectividade Funcional Espectral (LFP)

O sistema mitiga o problema clássico da *Volume Conduction* (um mesmo sinal elétrico sendo captado instantaneamente por múltiplos eletrodos) dividindo a análise de fase e magnitude. Sendo $S_{xy}(f)$ o espectro cruzado entre os sinais $x$ e $y$ na frequência $f$:

### 1.1. Coerência de Magnitude Quadrada (Padrão)
Mede a sincronia linear geral. É altamente sensível a influências de fase zero (condução de volume).
$$C_{xy}(f) = \frac{|S_{xy}(f)|^2}{S_{xx}(f) S_{yy}(f)}$$

### 1.2. Coerência Imaginária (iCoh)
Isola o atraso temporal verdadeiro, ignorando por completo as interações instantâneas de fase zero.
$$iCoh_{xy}(f) = \frac{Im(S_{xy}(f))}{\sqrt{S_{xx}(f) S_{yy}(f)}}$$

### 1.3. Weighted Phase Lag Index (wPLI)
Uma evolução das métricas de fase. É baseada unicamente na assimetria da diferença de fase entre os sinais, ponderada pela magnitude imaginária do espectro cruzado para reduzir a sensibilidade a pequenos ruídos aleatórios.
$$wPLI_{xy} = \frac{|E[Im(S_{xy})]|}{|E[|Im(S_{xy})|]|}$$
*(Onde $E[\cdot]$ é o valor esperado calculado sobre as janelas/trials. Considerada uma das métricas mais imunes à condução de volume do estado-da-arte).*

## 2. Conectividade Efetiva Direcional: Otimização do PDC

Para inferência causal, o NECTA utiliza a Coerência Direcionada Parcial (PDC). A principal inovação matemática da plataforma (`directed_metrics.py`) reside no abandono de ordens estáticas de modelagem em favor da otimização dinâmica.

### 2.1. Ajuste Autoregressivo Vetorial (VAR)
Os sinais LFP são submetidos a um filtro de estacionaridade temporal (rejeição por excesso de variância) e ajustados por um modelo de ordem $p$:
$$X(t) = \sum_{r=1}^{p} A_r X(t-r) + E(t)$$

### 2.2. Seleção de Ordem Automática (Critérios de Informação)
O Dask itera a ordem $p$ de 1 até `max_order=20`, solucionando o sistema via Mínimos Quadrados Ordinários (OLS) acelerados em NumPy. A escolha do melhor modelo minimiza a seguinte função de custo paramétrica:

**Bayesian Information Criterion (BIC):** O critério padrão da plataforma penaliza severamente modelos complexos ($k$ parâmetros), induzindo à parcimônia contra o *overfitting*:
$$BIC = k \ln(n) - 2 \ln(\hat{L})$$
*(O AIC - Akaike Information Criterion - também é suportado como alternativa).*

### 2.3. Transformada PDC
Uma vez estabelecidos os coeficientes $A_r$ da ordem ótima, o cálculo migra para o domínio da frequência aplicando a Transformada de Fourier (*FastMath* via Numba).
$$A(f) = I - \sum_{r=1}^{p} A_r e^{-j 2 \pi f \frac{r}{f_s}}$$

O valor da conectividade de $j$ para $i$ é a fração da energia que sai de $j$ canalizada na direção de $i$:
$$PDC_{ij}(f) = \frac{|A_{ij}(f)|}{\sqrt{\sum_{k} |A_{kj}(f)|^2}}$$

### 2.4. Validação Estatística (Surrogates & FDR)
1. **Surrogate Data:** O sistema gira a fase da Transformada de Fourier dos dados reais (randomização), recriando $100$ distribuições nulas que preservam o espectro de energia exato, mas destroem a causalidade temporal.
2. **P-Values:** O PDC otimizado é recalculado nos nulos. O valor-p surge da contagem do percentil ($p < 0.05$ global em $95\%$).
3. **FDR (Benjamini-Hochberg):** Aplica a restrição rigorosa de *False Discovery Rate* para mitigar os Falsos Positivos do grau massivo de testes por arestas, mascarando como zero qualquer aresta não significativa.

## 3. Teoria dos Grafos e Topologia de Rede

As matrizes de conectividade resultantes alimentam o módulo `metrics.py` e `statistical_validation.py`:

### 3.1. Assortatividade
Mede a correlação de Pearson ($r$) entre os graus de nós adjacentes.
*   **$r > 0$ (Assortativa):** Nós influentes (*Hubs*) conectam-se entre si. Indica um cérebro/rede robusto.
*   **$r < 0$ (Disassortativa):** Hubs conectam-se primariamente a nós menores. Uma rede vulnerável a falhas direcionadas aos *hubs*.

### 3.2. Small-Worldness ($\sigma$)
Utiliza a aproximação teórica rápida baseada no agrupamento e distância do componente gigante:
$$\sigma = \frac{C/C_{rand}}{L/L_{rand}}$$
*(Viabilizado via fórmula de Erdös-Rényi instantânea para escalar em centenas de janelas dinâmicas).*

### 3.3. Null Models Estruturais (Configuration Model)
Para validar métricas globais (como Modularidade via *Louvain*), o NECTA cruza os resultados reais contra ensaios nulos estritos gerados por permutações do tipo *Double Edge Swap*, preservando exatamente a densidade grau a grau da matriz neurofisiológica original.