# CHAPTER 2: WORKING HYPOTHESIS AND OBJECTIVES

## 2.1 Working Hypothesis
Traditional analyses of multichannel electrophysiological data often collapse the temporal dimension, yielding static graphs that obscure the transient coordination of neural ensembles. Furthermore, attempts to extract time-resolved networks are frequently confounded by volume conduction and crippled by the computational intractability of massive stochastic testing. 

We hypothesize that integrating highly scalable parallel computing with phase-based, zero-lag immune connectivity estimators (e.g., Imaginary Coherence and Weighted Phase Lag Index) and dynamically optimized causal models (Partial Directed Coherence) will overcome these traditional bottlenecks. By deploying such a framework, it will be possible to decode the "Dynome" of the primary visual cortex—revealing that the anatomical structural backbone (Areas 17, 18, and the Transition Zone) supports highly flexible, time-varying topological reorganizations (such as transient peaks in Small-Worldness) that align with active phases of sensory processing.

## 2.2 General Objective
The primary objective of this thesis is to develop, validate, and empirically apply **NECTA (Neural Electrophysiological Connectivity Time-resolved Analysis)**, an open-source, high-performance computational framework designed for the interactive exploration and statistical validation of dynamic mesoconnectomes from raw high-density electrophysiological recordings.

## 2.3 Specific Objectives
To achieve the general goal, the project is subdivided into the following specific aims:

1. **Data Ingestion and Standardization:** To develop a robust pipeline (the *Isaac* ecosystem) capable of converting dispersed legacy electrophysiological binaries (e.g., SPASS format) into standardized, cloud-native, n-dimensional tensors (Zarr/Xarray) using lazy-loading mechanisms.
2. **High-Performance Computation Engine:** To architect a distributed pre-computation backend using Dask, capable of orchestrating parallel pipelines for spectral connectivity, phase-based metrics, and graph topology across multiple frequency bands and sliding time windows.
3. **Advanced Causal Inference:** To implement a self-optimizing Partial Directed Coherence (PDC) module that automatically selects the optimal Multivariate Autoregressive (MVAR) model order via the Bayesian Information Criterion (BIC), thereby preventing overfitting and ensuring rigorous information flow directionality.
4. **Statistical Rigor and Null Modeling:** To embed strict statistical validation protocols, including False Discovery Rate (FDR) corrections applied to phase-randomized surrogate data, and topological null models (Erdös-Rényi and Configuration Models) to validate graph communities.
5. **Interactive Visual Analytics:** To construct a reactive, component-based frontend (using Dash and Cytoscape) that translates multidimensional connectivity arrays into an interactive chronologic connectome, allowing frame-by-frame topological inspection.
6. **Empirical Validation *In Vivo*:** To apply the NECTA framework to an empirical dataset (`c1608a01`) obtained from the feline visual cortex (A17/TZ/A18) to characterize the hierarchical directionality of gamma-band synchronization and the temporal evolution of network efficiency during visual stimulation.