# Arquitetura do Sistema NECTA

O processamento de dados neurofisiológicos multicanal (ex: LFP) esbarra num gargalo computacional grave quando se busca análise em tempo real. Matrizes de conectividade multivariadas exigem alto custo computacional, inviabilizando a exploração interativa.

Para contornar esse problema, o NECTA foi arquitetado em um **modelo desacoplado de duas etapas (Two-Step Workflow)**.

## 1. Motor de Pré-Cálculo Paralelo (Back-end)

A primeira etapa foca inteiramente em força bruta matemática e é executada fora do ambiente de visualização (via Jupyter Notebook interativo `NECTA_Precomputation_and_Launch.ipynb`).

- **Paralelização Massiva:** Utiliza o framework **Dask** para distribuir o cálculo em múltiplos *workers*. Cada janela de tempo (ou epoch) é processada de maneira independente, paralelizando a modelagem MVAR, as métricas e as validações estatísticas.
- **Armazenamento Otimizado:** O resultado multidimensional (Tempo x Frequência x Canal Fonte x Canal Alvo) gerado é gravado em disco utilizando o formato **Zarr**. O acesso e a serialização dos arrays Zarr são gerenciados pelo `xarray`, que rotula as dimensões, permitindo fatiamento (slicing) eficiente em disco e compressão de alto desempenho.

## 2. Dashboard de Baixa Latência (Front-end)

A interface analítica é a segunda etapa. Foi construída usando **Dash (Plotly)** integrado ao **Cytoscape** (`dash-cytoscape`) para a renderização de redes complexas.

- **Ausência de Recomputação:** Quando o Dashboard é iniciado, ele apenas monta o cache `.zarr` já pré-processado na memória.
- **Interatividade Instantânea:** Interações do usuário — como deslizar a barra de tempo (Sliding Window), alterar limiares de conexão (absolutos ou por densidade) ou mudar de métrica — operam em tempo sub-segundo. A interface não recalcula conexões, apenas realiza fatiamento e mascaramento nas matrizes em memória.
- **Teoria dos Grafos Aplicada:** A topologia de rede é analisada on-the-fly usando a biblioteca `networkx` e `python-louvain` para determinar características dinâmicas:
  - Tamanho dos nós mapeado por métricas de centralidade (*Degree*, *Betweenness*, *Eigenvector*).
  - Detecção de comunidades (*clusters*), com garantia de persistência de coloração das comunidades ao longo das janelas temporais.