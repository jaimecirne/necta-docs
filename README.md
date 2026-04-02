# NECTA - Documentação Técnica e Científica

Bem-vindo à documentação oficial do **NECTA** (*Neural Electrophysiological Connectivity Time-resolved Analysis*).

O NECTA é um framework computacional interativo construído para a visualização, análise e exploração de conectividade funcional e efetiva baseada em registros neurofisiológicos de alta densidade (LFP e *spikes*).

Esta documentação centraliza os aspectos metodológicos, arquiteturais e matemáticos da plataforma, servindo como base tanto para o desenvolvimento contínuo quanto para a redação acadêmica (teses, artigos e qualificações).

## Conteúdo da Documentação

1. [Arquitetura de Software](architecture.md): Detalha a estratégia de pré-cálculo massivamente paralelo (Dask/Zarr) e o front-end de baixa latência (Dash/Cytoscape).
2. [Fundamentação Matemática](mathematics.md): Descreve em profundidade a implementação dos cálculos de conectividade, com foco no modelo Autoregressivo Vetorial (VAR), Coerência Direcionada Parcial (PDC) e os rigorosos critérios de validação estatística (FDR).
3. [Estrutura de Dados (Zarr)](data_structure.md): Especifica o formato de entrada dos dados eletrofisiológicos (LFP, spikes e metadados) convertido de legados pelo pipeline Isaac.
4. [Ingestão de Dados e Gestão de Estado Global](data_ingestion_and_state.md): Detalha o carregamento assíncrono (*Lazy Loading*), a extração de fase e amplitude via Transformada de Hilbert (para *iCoh* e *wPLI*) e o gerenciamento do estado via `core.py`.
5. [Arquitetura do Motor de Pré-Cálculo e Orquestração (Dask)](precompute_orchestration.md): Explica o versionamento de cache via *fingerprinting*, os pipelines paralelos do LFP (Magnitude, iCoh, wPLI), e a Auto-Otimização dinâmica de ordem dos modelos VAR no cálculo de PDC.
6. [Arquitetura de Visualização e Dashboard Interativo](dashboard_architecture.md): Documenta o design em componentes da UI (Dash/Cytoscape), a gestão do Estado Global (`ids.py` e Callbacks rápidos via `xarray.sel()`), e os painéis especializados (Integração Temporal, Fluxo de Informação e Assortatividade).
7. [Aplicação Biológica e Estudo de Caso](biological_application.md): Documenta a validação científica da ferramenta no córtex visual (A17/A18) e a prova de conceito empírica do mesoconectoma dinâmico ("Dynome").

---
*Gerenciado como parte do projeto de doutorado em Bioinformática (UFRN - Instituto do Cérebro) de Jaime Bruno Cirne de Oliveira.*