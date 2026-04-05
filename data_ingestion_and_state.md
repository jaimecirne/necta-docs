# Ingestão de Dados e Gestão de Estado Global (NECTA)

O subsistema de dados do NECTA atua como um pipeline de transformação contínua, convertendo dados brutos em estruturas de análise gerenciadas centralmente. A arquitetura segue o padrão de *Single Source of Truth* (Fonte Única da Verdade).

## 1. Leitura Física e Preparação Avançada (`data_loader.py`)

A camada base opera via `data_loader.py`, responsável pela ingestão adaptada para lidar com métricas de conectividade avançadas (protegidas contra *volume conduction*).

### 1.1. Estrutura Zarr e Lazy Loading
O sistema interage primariamente com o formato Zarr otimizado pelo pipeline Isaac. O carregamento emprega **Lazy Loading**: ao invés de alocar gigabytes na memória RAM, o `xarray.open_zarr` apenas mapeia a árvore de metadados. As variáveis críticas consumidas sob demanda são `spike_times`, `lfp` e `stim_label`.

### 1.2. Transformações Espectrais e Sinal Analítico
Para suportar o rigor da **Coerência Imaginária (iCoh)** e do **Weighted Phase Lag Index (wPLI)**, o sistema processa análises de fase:
*   **Transformada de Hilbert:** Converte sinais LFP reais em sinais analíticos complexos. Isso permite a extração instantânea da fase $\phi(t)$ e da amplitude $A(t)$ de cada canal.
*   **Segmentação Vetorial:** Os arrays de sinais contínuos são remodelados em matrizes de janelas (`n_segments`, `samples_per_seg`) otimizadas para processamento vetorizado ultra-rápido no NumPy.

### 1.3. Harmonização de Dados
*   **Spikes:** Os tempos de disparo sofrem *binning* para formar histogramas temporais precisos, permitindo métricas de correlação.
*   **LFP:** Os canais sofrem alinhamento por *trial*, gerando tensores prontos para o cruzamento de espectros cruzados (*cross-spectra*).

## 2. Gestão de Estado Global (`core.py`)

O módulo `core.py` atua como a espinha dorsal do dashboard em tempo de execução.

### 2.1. A Variável `PROCESSED_DATASET`
A aplicação inteira consome uma única variável global que retém o contexto do experimento ativo: `PROCESSED_DATASET: xr.Dataset | None = None`. Este dataset estendido hospeda coordenadas exclusivas para diferenciar a visualização anatômica e funcional de redes de Magnitude, Fase (*iCoh*) ou atraso (*wPLI*).

### 2.2. Exportação e Compactação
Utilitários de I/O em memória (`io.BytesIO`) e empacotamento (`zipfile`) garantem que o usuário baixe todo o lote de matrizes de adjacência (pesos das arestas) e sumarizações em um formato estruturado.

## 3. Geração de Relatórios e Auditoria (`report_generator.py`)

Visando facilitar a interpretação dos achados estatísticos e topológicos gerados, o `report_generator.py` atua na tradução das dimensões.
*   **Flattening:** Ele executa o "achatamento" dos tensores hiperespaçais do `xarray` convertendo-os em matrizes bidimensionais do `pandas.DataFrame`.
*   **Auditoria de Fase:** Assegura que o cientista exporte os painéis CSV com os valores puros do wPLI, PDC ou Coerência Clássica agrupados por banda de frequência e instante temporal, preservando o valor primário gerado.