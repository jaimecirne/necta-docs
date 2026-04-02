# Estrutura de Dados Eletrofisiológicos (Formato Zarr)

O motor analítico do **NECTA** depende da ingestão de um formato de dados padronizado, denso e de alto desempenho. O fluxo de captação laboratorial gera arquivos legados esparsos e de formatos proprietários (ex: formato SPASS, com binários múltiplos para formas de onda, estímulos e potenciais locais).

A plataforma utiliza o ecossistema e pipeline **Isaac (VisionICeIO)** para realizar a conversão massiva, unificação e padronização desses registros num formato otimizado nativo de nuvem: o **Zarr**. O formato é acessado internamente no Python pelo pacote `xarray`.

Este documento especifica a estrutura do arquivo `.zarr` gerado pelo pipeline Isaac e que atua como **dado de entrada direto** para o ecossistema NECTA.

## 1. Topologia do Armazenamento

A estrutura segue uma árvore relacional para acomodar metadados e grandes matrizes de dados. Todo o experimento fica encapsulado num único arquivo de contêiner de armazenamento (`NomeExperimento.zarr`).

Ao ser lido pelo `xarray`, o diretório Zarr expõe um objeto iterável da classe `xarray.Dataset`, mapeado pelas seguintes matrizes e seus eixos dimensionais físicos:

## 2. Matrizes de Dados (Data Variables)

O dado original é restruturado em tensores multidimensionais. O pipeline preenche espaços vazios com `NaN` (Padding) para assegurar o formato retangular de matrizes densas.

### 2.1. Potencial de Campo Local (LFP - *Local Field Potential*)
*   **Nome da Variável:** `lfp`
*   **Formato de Dados:** `int16` ou `float32`
*   **Dimensões Físicas:** `(electrodes, trials, lfp_time)`
*   **Descrição:** O traçado contínuo macroscópico da atividade extracelular, fundamental para análises espectrais e modelos como Coerência e PDC.

### 2.2. Ocorrências Discretas (Spike Times)
*   **Nome da Variável:** `spike_times`
*   **Formato de Dados:** `float32`
*   **Dimensões Físicas:** `(electrodes, trials, spikes_idx)`
*   **Descrição:** Os *timestamps* exatos dos disparos neuronais (potenciais de ação unitários) identificados. Como os *spikes* variam por trial, há preenchimento de `NaN` nos índices remanescentes de cada vetor até o limite do trial mais denso.

### 2.3. Formas de Onda (Waveforms)
*   **Nome da Variável:** `waveforms`
*   **Formato de Dados:** `float16`
*   **Dimensões Físicas:** `(electrodes, trials, spikes_idx, snippet_time)`
*   **Descrição:** A morfologia isolada de cada *spike* captado, contendo a captura sub-milisegundo do potencial de ação correspondente. 

### 2.4. Matriz Auxiliar (Spike Count)
*   **Nome da Variável:** `n_spikes`
*   **Formato de Dados:** `int32`
*   **Dimensões Físicas:** `(trials, electrodes)`
*   **Descrição:** Uma matriz de contagem escalar para validação. Indica a quantidade exata real de *spikes* válidos para um par canal/trial (ignorando as máscaras `NaN`).

## 3. Metadados do Experimento (Global Attributes)

Atributos globais originados de dicionários/arquivos de configuração de hardware legado (ex: arquivos `_LPF.txt` ou `-ifo.txt`) são integrados nativamente nas propriedades (`.attrs`) do `xarray.Dataset`. Eles garantem a interoperabilidade dos cálculos do NECTA, provendo chaves universais como:

*   **Frequência de Amostragem (Sampling Rate):** Indispensável para reconstrução dos espectros do domínio da frequência.
*   **Quantificação de Canais / Eletrodos.**
*   **Identificadores Clínicos / Sujeito-Animal.**

---

*Esta abstração converte a granularidade biológica analógica captada pelos amplificadores em uma matriz algorítmica matemática regular, permitindo ao NECTA fatiar os arrays otimizados num ambiente de processamento paralelo e interativo.*