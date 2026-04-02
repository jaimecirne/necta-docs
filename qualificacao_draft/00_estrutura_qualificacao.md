# ESBOÇO DA TESE DE QUALIFICAÇÃO (PRE-TESE)
**Projeto:** NECTA: Neural Connectivity Time-resolved Analysis
**Autor:** Jaime Bruno Cirne de Oliveira
**Orientadora:** Prof. Dr. Kerstin E. Schmidt
**Programa:** Pós-Graduação em Bioinformática (e interface em Neurociências) / Instituto do Cérebro (ICe) - UFRN

---

## ESTRUTURA DO DOCUMENTO E ÍNDICE

O formato abaixo espelha o padrão institucional observado em trabalhos egressos do laboratório (e.g., Ferreiro 2018, Conde 2014, Moura 2021). As seções preenchidas integram a pesquisa teórica, as bases computacionais da plataforma (NECTA/Isaac) e as descobertas biológicas do pôster "BNF 2025" (Dynome no modelo felino).

---

### CAPA E PRÉ-TEXTUAIS
- **Capa Oficial:** Título, Instituição, Autor, Orientadora, Ano.
- **Folha de Rosto & Catalogação SISBI**
- **Banca Examinadora (Comitê de Acompanhamento)**
- **Agradecimentos (Opcional para qualificação, obrigatório na tese)**
- **Abstract / Resumo:** Resumo abrangendo o problema (estudo estático vs. Dynome), a ferramenta computacional (HPC/Dask/Zarr), as métricas (wPLI, iCoh, PDC) e os achados biológicos em Low Gamma (Hierarquia A17 -> TZ -> A18).

---

### 1. INTRODUCTION (INTRODUÇÃO)
**Objetivo da seção:** Apresentar a evolução dos conectomas e o problema biológico no córtex visual.
- **1.1 The Primary Visual Cortex and Mesoscopic Networks:** Histórico dos estudos em V1, layout espacial, colunas e áreas de transição (Referenciando Schmidt et al., 2023; Wunderle et al., 2013).
- **1.2 From Static Topologies to the "Dynome":** A transição de conectomas anatômicos estáticos para análises da neurodinâmica com resolução de milissegundos (Kopell et al., 2014; Allen et al., 2014).
- **1.3 Rhythmic Oscillations and Gamma Synchronization:** O papel cardinal das oscilações de alta frequência (Low/High Gamma) na sincronização das redes retinogeniculadas e integração cortical (Neuenschwander et al., 2023; Moura, 2021).
- **1.4 The Methodological Bottleneck in Electrophysiology:** A barreira de *Big Data* em eletrofisiologia de múltiplos canais (LFP/Spikes), os riscos de Falsos Positivos na matriz devido a *Volume Conduction* e as limitações dos scripts analíticos lentos e mononucleares.

---

### 2. WORKING HYPOTHESIS & OBJECTIVES
**Objetivo da seção:** Declarar formalmente o escopo do desenvolvimento.
- **2.1 Working Hypothesis:** A hipótese de que o uso de paralelização em larga escala combinada a estimadores fase-dependentes imunes a condução de volume (wPLI, iCoh) e algoritmos preditivos causais (PDC) revelarão propriedades dinâmicas inerentes à resposta visual, expondo uma topologia de mundo-pequeno reativa.
- **2.2 General Objective:** Desenvolver, validar e aplicar o framework NECTA.
- **2.3 Specific Objectives:**
  - Integrar e serializar formatos binários legados via pipeline `visioniceio` (Zarr).
  - Orquestrar cálculos estatísticos em Grafos de Tarefas distribuídos com Dask.
  - Implementar seleção auto-otimizada de modelos MVAR para conectividade causal (PDC).
  - Aplicar o framework no *dataset* empírico `c1608a01` para dissecar a resposta funcional de A17, A18 e TZ.

---

### 3. MATERIALS AND METHODS
**Objetivo da seção:** Todo o arcabouço técnico e matemático gerado (Documentos NECTA Docs).
- **3.1 Data Acquisition and Animal Model:** Detalhes experimentais empíricos. Registro multieletrodo (32-64 canais), gato adulto anestesiado e paralisado, suprimindo sacadas para purificar a dinâmica de rede intrínseca.
- **3.2 Data Ingestion & Storage Architecture (Isaac Pipeline):** Extração de SPASS, *padding* matemático, uso de tensores multidimensionais, conversão e *lazy loading* com Xarray e formato nativo de nuvem Zarr.
- **3.3 The NECTA Pre-computation Engine (HPC):**
  - Orquestração distribuída com Dask.
  - *Fingerprinting* para versionamento seguro do cache (Evitando corrupção de sessão).
- **3.4 Spectral and Directed Connectivity Methods:** (As equações exatas).
  - Filtro de Estacionaridade e Sinal Analítico (Transformada de Hilbert).
  - *Magnitude Coherence*, *Imaginary Coherence* e *wPLI*.
  - Partial Directed Coherence (PDC) com Busca e Otimização via Critério de Bayes (BIC) limitando em *max_order=20*.
- **3.5 Graph Theory, Topology, and Null Models:** 
  - Cálculo instantâneo de *Small-worldness* ($\sigma$).
  - Validação estatística rigorosa das arestas via FDR (Benjamini-Hochberg) e topologia com Modelos Nulos (Erdös-Rényi e Configuration Model).
- **3.6 Visualization Engine (Dash/Cytoscape):** A interface *Single Page Application*, o fatiamento *On-the-fly*, Matrizes de Calor e Redes interativas.

---

### 4. RESULTS
**Objetivo da seção:** Provar a validade do NECTA usando o Estudo de Caso (O pôster de 2025).
- **4.1 Baseline Characterization and the Static Backbone:** Mapeamento anatômico perfeito do algoritmo de agrupamento (Louvain) refletindo a biologia dos eletrodos em A17, TZ e A18 (Densidade a 15% em Low Gamma).
- **4.2 The Hierarchical Flow (Directed Connectivity):** 
  - O perfil hierárquico isolado pelo PDC otimizado. 
  - A17 liderando (Driver/Source) vs. A18 como Sumidouro. O papel intrigante do TZ como Modulador Dinâmico.
- **4.3 Temporal Integration and Dynamic Topology:** 
  - A variação de *Small-Worldness* ao longo da cronologia do estímulo.
  - Aumento da clusterização local que guia picos de eficiência global perfeitamente temporizados com o estímulo em movimento (Janelas de 500ms).

---

### 5. DISCUSSION
**Objetivo da seção:** Inserir a tese na literatura. [Nota: Este texto incorpora o rascunho de discussão focado na literatura do lab e bases clássicas].

A transição dos estudos puramente estáticos da percepção cortical (Wunderle et al., 2013; Schmidt et al., 2023) para a modelagem dinâmica do *Dynome* (Kopell et al., 2014) exige plataformas HPC robustas. O NECTA demonstrou, no conjunto `c1608a01`, que a estrutura topológica (*Small-Worldness*) sob estado de anestesia reage organicamente ao estímulo.

Superar a interferência metodológica, notadamente a Condução de Volume (*zero-phase lag*, Bastos & Schoffelen, 2015), foi possível devido à implementação arquitetural rigorosa do *wPLI* e Coerência Imaginária (*iCoh*). Ao aliar isso à Busca Auto-Otimizada do PDC pelo Critério de Informação Bayesiana (BIC), a ferramenta expurgou viés e reconstruiu as vias ascendentes visuais (Vezoli et al., 2021). A consolidação da sincronia Gamma como roteadora funcional da informação entre A17 e A18 expande o escopo delineado pelo Laboratório nas vias retinogeniculadas (Neuenschwander et al., 2023). Sob o viés técnico, fatiar matrizes Dask/Zarr de forma síncrona aos painéis do Cytoscape representa a quebra do atual gargalo biológico do "Big Data Eletrofisiológico".

---

### 6. CONCLUSION & FUTURE DIRECTIONS
- **Conclusões principais:** NECTA funciona não só como processador de sinais, mas como uma lente microscópica cronológica ("Chronological Connectome").
- **Próximos passos:** Expandir o framework cruzando dados com outras camadas (inter-hemisféricas) ou diferentes estados cerebrais (e.g., sono, vigília).

---

### 7. REFERENCES
*(Listar as publicações chave, com foco nos PDFs que extraímos via Europe PMC, como Bastos & Schoffelen, Kopell, Vezoli, Wunderle, Conde, Ferreiro, Moura, etc).*