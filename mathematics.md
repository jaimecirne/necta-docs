# Fundamentação Matemática do NECTA

O NECTA (*Neural Electrophysiological Connectivity Time-resolved Analysis*) incorpora diferentes estimadores de conectividade, sendo a **Coerência Direcionada Parcial (PDC - *Partial Directed Coherence*)** o núcleo de conectividade causal/efetiva da plataforma. 

A implementação do PDC no NECTA (`directed_metrics.py`) não utiliza pacotes simplificados, optando por um fluxo rigoroso de modelagem, processamento rápido e controle estatístico de Falsas Descobertas.

## 1. Controle de Estacionaridade (Filtro de Qualidade)

Antes da modelagem, cada janela temporal de LFP é submetida a um filtro de estacionaridade. A plataforma computa a variância média da janela; caso o sinal ultrapasse um limite paramétrico de ruído, a janela é classificada como ruidosa/não-estacionária e sumariamente rejeitada. Essa triagem é imperativa, pois janelas com artefatos de alta variância quebram as premissas dos modelos autoregressivos, corrompendo a matriz de resultados.

## 2. Modelagem Autoregressiva Vetorial (MVAR)

O LFP pré-processado é ajustado em um modelo multivariado no domínio do tempo, onde o estado presente de todos os canais é explicado pelo seu histórico:

$$Y(t) = \sum_{r=1}^{p} A_r Y(t-r) + \epsilon(t)$$

Onde $A_r$ são as matrizes de coeficientes de lag e $\epsilon(t)$ é o ruído não correlacionado.

- **Otimização Mínimos Quadrados:** Para evitar a sobrecarga (overhead) da biblioteca estática (`statsmodels`), os coeficientes são inferidos empregando uma abordagem puramente matricial de Mínimos Quadrados Ordinários (OLS) via `np.linalg.lstsq`.
- **Seleção Dinâmica de Ordem ($p$):** A ordem do modelo não é cravada arbitrariamente. Ela é escolhida otimizando os resíduos (matriz de covariância do erro) frente à complexidade, avaliando Critérios de Informação clássicos (BIC, AIC ou HQIC).

## 3. Cálculo da PDC no Domínio da Frequência

Uma vez estabelecidos os coeficientes $A_r$, o cálculo migra para o domínio da frequência aplicando a Transformada Discreta de Fourier.

$$A(f) = I - \sum_{r=1}^{p} A_r e^{-j 2 \pi f \frac{r}{f_s}}$$

Para lidar com a exigência computacional dessa etapa em matrizes densas de alta taxa de amostragem, essa transformada é processada por funções compiladas e acelaradas em C (FastMath via **Numba**).

O valor escalar do fluxo de informação da fonte $j$ para o alvo $i$ na frequência $f$ é dado pela intensidade dessa via sobre toda a energia que deixa o canal de origem:

$$PDC_{ij}(f) = \frac{|A_{ij}(f)|}{\sqrt{\sum_{k} |A_{kj}(f)|^2}}$$

*(Esta restrição assegura que as somas das magnitudes quadradas por coluna sejam iguais a 1)*

## 4. Validação Estatística Robusta (Surrogates & FDR)

O mero valor escalar da PDC pode incorrer em achados casuais. A plataforma utiliza controle estatístico estrito para separar conexões reais de correlações espúrias:

1. **Dados Substitutos (Surrogate Data):** O NECTA recria o universo nulo do sinal. Os dados reais têm sua fase randomizada via Transformada de Fourier, destruindo interações causais enquanto preservam o espectro de energia exato.
2. **P-Values Baseados em Contagem:** O fluxo MVAR/PDC é rodado para essas réplicas nulas ($N=100$). A métrica de significância local (o valor-p empírico da aresta) surge comparando-se a taxa com que os valores da rede falsa igualam ou superam a rede verdadeira.
3. **FDR - Correção de Benjamini-Hochberg:** Finalmente, para domar o risco de Falsos Positivos inerente às comparações múltiplas das arestas, não basta aceitar um $p < 0.05$. O algoritmo controla a *False Discovery Rate* (FDR). Conexões reprovadas pelo rigor do FDR são zeradas matematicamente na matriz persistida no cache.