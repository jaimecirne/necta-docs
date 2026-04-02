# Arquitetura do Motor de Pré-Cálculo e Orquestração (Dask)

O coração numérico do NECTA reside no seu Motor de Pré-Cálculo (`precompute`). Desenvolvido com uma arquitetura distribuída (*Dask*), ele absorve a carga massiva das rotinas neurofisiológicas, processando ramificações algorítmicas de fase e estatística em paralelo.

## 1. O Orquestrador e o Cluster (`orchestrator.py`)

A submissão de tarefas não é monolítica; ela é administrada por um orquestrador central inteligente.

*   **Versionamento Seguro (Fingerprinting):** O NECTA garante a validade dos dados via `CACHE_VERSION`. Alterar limiares, ligar algoritmos secundários (*wPLI*, *iCoh*) ou reparametrizar a ordem estatística (ex: de AIC para BIC) altera instantaneamente o *hash* dos parâmetros. O orquestrador detecta a discrepância e invalida o cache Zarr correspondente. Esse mecanismo blinda o pesquisador contra dados obsoletos acoplados ao frontend acidentalmente.

## 2. O Grafo de Tarefas Expandido (`tasks.py`)

O roteador de rotinas (`tasks.py`) define os "nós" de computação do *Cluster Dask*, organizando as baterias analíticas por modalidade de rede:

### 2.1. Pipeline de Spikes (Conectividade Discreta)
Permanece atrelado ao cálculo robusto de correlação de Pearson processando histogramas temporais binarizados das frequências de disparo originais.

### 2.2. Pipeline Espectral Triplo (LFP)
A análise de coerência foi expandida em um tríplice arcabouço funcional. Para cada combinação de segmento de tempo e banda de frequência, o Dask instiga três ramificações matemáticas isoladas, com retorno associado a chaves dedicadas (`coh_mag_static`, `coh_wpli_static`, etc):
1.  **Magnitude Squared Coherence (`mag`):** A coerência analógica, fornecendo o panorama clássico de sincronização da potência.
2.  **Imaginary Coherence (`icoh`):** Um galho algorítmico computando unicamente a fração imaginária do espectro, contornando falsos positivos decorrentes da condução volumétrica instantânea.
3.  **Weighted Phase Lag Index (`wpli`):** Uma rotina avançada extraindo unicamente a pureza da sincronização de fase assíncrona, onde o peso da média direcional é escalonado pela densidade imaginária do espectro cruzado.

### 2.3. Pipeline Inteligente de PDC (`directed_metrics.py`)
A avaliação Causal de Granger (*Partial Directed Coherence*) abandonou configurações estáticas, portando-se como um módulo inteligente com busca automática de estado ótimo:
1.  **Otimização Dinâmica do VAR:** Antes de calcular a PDC, o *Worker Dask* avalia iterativamente a resiliência do modelo Autorregressivo Vetorial em ordens incrementais (indo até o limite superior *default* de `max_order=20`).
2.  **Critério de Decisão (BIC/AIC):** A ordem ótima ($p$) não é imposta, mas derivativa da minimização formal do *Bayesian Information Criterion* (ou AIC), contrabalançando a precisão de ajuste contra o *overfitting* de parâmetros.
3.  **Validação In-Loco:** As réplicas (*Surrogates* para controle do FDR) são giradas diretamente pelo mesmo *worker* após alcançar a ordem do modelo, enxugando custos de rede entre os nós do cluster.

## 3. Expansão de Grafos e Reestruturação de Dados (`dataset_builder.py`)

### 3.1. Assortatividade e Topologia
Junto ao núcleo pré-existente (Global Efficiency, Modularidade, Louvain, Small-worldness), as tarefas explícitas agora invocam a **Assortatividade** da rede (`assortativity_static/dynamic`), calculando a preferência natural de nós influentes do cérebro para se fixarem em outros polos neuronais de grau equivalente.

### 3.2. Acomodação Final do Zarr
O coletor universal (`dataset_builder.py`) capta a miríade de resultados assíncronos expandindo a topologia multivariada final do Xarray/Zarr, assegurando os atributos que pautam toda a transparência estatística (salvando meta-atributos diretos como `pdc_criterion`, `pdc_max_order` e os patamares de variância descartados).