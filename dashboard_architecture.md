# Arquitetura de Visualização e Dashboard Interativo

A última camada do NECTA é a interface do usuário. Trata-se de uma aplicação web reativa, desenvolvida em **Dash** e estruturada como uma *Single Page Application* (SPA).

A interface atua estritamente como uma camada de observação. Ela não reprocessa sinais eletrofisiológicos brutos, mas interage cirurgicamente sobre o `core.PROCESSED_DATASET` instanciado na memória. O design arquitetural é fortemente modularizado em componentes independentes.

## 1. Estrutura de Navegação e Estado Global

A estrutura principal (`app.py` e `make_tabs_app`) emprega o padrão arquitetural de Painel Lateral (*Sidebar*) acoplado ao Corpo de Conteúdo.

### 1.1. Sidebar de Controle Central (`sidebar.py`)
A barra lateral atua como a guardiã do **Estado de Filtragem Global**.
Quando o pesquisador interage com um controle (e.g., mudando a Densidade da Rede para o "Top 15%", alterando a Banda Espectral ou o Tipo de Conectividade), esse evento reverbera pelo aplicativo. *Callbacks* em múltiplas abas são acionados simultaneamente, forçando a re-renderização harmônica das matrizes e dos grafos sem necessidade de recarregamento da página.

### 1.2. Dicionário de IDs (`ids.py`)
Dado o volume de componentes interativos (centenas de botões, sliders e gráficos), a aplicação bane strings fixas (*hardcoded*). Todas as chaves do DOM HTML (ex: `GLOBAL_THRESHOLD_SLIDER`) são instanciadas como constantes em `ids.py`. Isso protege o código contra falhas de amarração nos *Callbacks* e viabiliza refatorações estruturais maciças.

## 2. Motores Especializados de Visualização

O projeto segmenta a renderização conforme a ontologia matemática do dado em análise:

### 2.1. Topologia de Rede e Grafos (`graphview.py`)
A renderização de conectomas espaciais é provida pelo módulo `dash-cytoscape`.
*   **Mapeamento:** O algoritmo mapeia métricas como o *Degree* do nó biológico diretamente para a propriedade de tamanho (`width`/`height`) do componente visual. A espessura da aresta espelha o peso absoluto da conectividade ($w_{ij}$).
*   **Comunidades:** A organização de comunidades detectada (algoritmo de Louvain) controla a paleta de cores. Nós no mesmo módulo recebem a mesma matiz.
*   **Layouts:** A biblioteca de layouts de Força (e.g., `cose`) ajuda no desemaranhamento das estruturas visuais densas.

### 2.2. Inspeção de Matrizes de Adjacência (`connectomes.py`)
Para o escrutínio paramétrico exato, o sistema utiliza *Heatmaps* de alta densidade (`plotly.graph_objects.Heatmap`). Ele entrega uma visão bidimensional canônica e limpa, respondendo fluidamente à linha do tempo via animações por *slider*.

### 2.3. Séries Temporais Analíticas (`statistical_summary.py`)
Voltado à visão longitudinal ($t_0 \to t_n$). Curvas de dispersão e evolução (Plotly Express) correlacionam métricas de Grafos (como Agrupamento vs Eficiência), evidenciando estados neurodinâmicos macroscópicos.

## 4. Painéis Científicos Avançados

O NECTA vai além das visões genéricas e oferta módulos especializados para responder perguntas neurofisiológicas diretas:

*   **Padrões de Mistura / Assortatividade (`mixing_analysis.py`):** Apresenta matrizes reduzidas focadas na hierarquia (Região vs Região em vez de Canal vs Canal), atestando como "hubs" biológicos comunicam-se entre si.
*   **Fluxo de Informação (`information_flow_summary.py`):** Desenhado para métricas de Causalidade como o PDC. Renderiza a assimetria direcional destacando os *Drivers* (regiões-fonte massivas no Eixo Y) contra os *Sinks* (regiões-alvo no Eixo X).
*   **Integração Temporal Multimodal (`temporal_integration.py`):** Um painel vital de curadoria. Ele plota sobrepostos: a) O Sinal Biológico (Taxa de Spikes); b) A Topologia (*Small-Worldness*); e c) Qualidade do Sinal. Isso instrumentaliza o pesquisador a atestar instantaneamente se uma flutuação nas conexões cerebrais reflete fisiologia real ou mero ruído experimental (artefatos/epilepsia).

## 5. Ciclo de Vida: *Callbacks* e Gestão do Motor

### 5.1. Fluxo Interno de uma Interação Visual
A agilidade do sistema advém da forma como o Dash interage com o `xarray`:
1.  O *Slider* temporal é movido pelo usuário na UI.
2.  O *Callback* atrelado captura o novo valor `t`.
3.  Acesso imediato à variável `core.PROCESSED_DATASET`.
4.  O sistema aplica um recorte (slicing) massivo utilizando indexação inteligente: `xarray.sel(time_window=t, method='nearest')`.
5.  Filtros de limiar são computados no vetor.
6.  O resultado fatiado é serializado em JSON e devolvido exclusivamente ao componente de grafo correspondente para re-renderização vetorial no navegador.

### 5.2. O *Wizard* de Inicialização (`analysis_manager.py`)
Módulo responsável por construir o *job* de processamento antes do *dashboard* existir. Permite a exploração do sistema de arquivos do servidor (`dirpicker.py`) e configuração paramétrica das frequências e tempo, invocando a arquitetura assíncrona do *Dask* e suspendendo a UI com barras de progresso até a finalização e ingestão do dataset.

### 5.3. Histórico e I/O (`analysis_history.py`)
O sistema gerencia um estado persistente, permitindo recarregar conjuntos de dados pré-computados (cache do disco) e disparar funções de empacotamento (`export_dataset_to_zip`) para baixar as planilhas sumarizadas.