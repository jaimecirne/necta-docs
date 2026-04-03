# CHAPTER 3: MATERIALS AND METHODS

## 3.1 Animal Preparation and Data Acquisition
The empirical validation of the framework utilized the `c1608a01` dataset (Wunderle et al., 2013). Multichannel electrophysiological recordings (Local Field Potentials and multi-unit spikes) were obtained from the primary visual cortex of an adult cat. To isolate intrinsic cortical network dynamics from confounding behavioral variables (e.g., saccadic eye movements or fluctuations in conscious attention), the animal was maintained under general anesthesia and paralysis. High-density matrix electrodes (32–64 channels) were implanted spanning Area 17 (A17), the A17/18 Transition Zone (TZ), and Area 18 (A18). 

## 3.2 Data Ingestion Architecture (The Isaac Pipeline)
To process the high-density recordings, the *VisionICeIO* (Isaac) pipeline was developed to act as a bridge between legacy proprietary formats (e.g., SPASS binaries) and modern data structures. 
*   **Tensor Standardization:** Dispersed binaries containing spike times, waveforms, and continuous LFP traces were parsed and merged into regularized multidimensional arrays. Ragged arrays (e.g., varying spike counts per trial) were standardized using `NaN` padding.
*   **Zarr and Xarray Encapsulation:** The output is structured as a cloud-native `Zarr` store. By encapsulating the data within an `xarray.Dataset`, the framework leverages *lazy loading*, allowing the frontend to read metadata trees without exhausting system RAM.

## 3.3 Distributed Pre-Computation Engine (HPC)
Analyzing dynamic connectivity across continuous recordings poses a severe combinatorial challenge. NECTA addresses this through a distributed pre-computation engine orchestrated by **Dask**.
*   **Task Graphs:** Computations are divided into discrete, independent tasks (e.g., computing coherence for a specific frequency band and time window) and mapped across a cluster of worker nodes.
*   **Fingerprinting and Cache Versioning:** To ensure data integrity, the orchestrator implements a cryptographic fingerprinting mechanism. Any modification to analytical parameters (e.g., altering variance thresholds or VAR criteria) alters the global hash, safely invalidating obsolete `Zarr` caches.

## 3.4 Spectral Connectivity and Phase Analysis
To mitigate the artifacts of volume conduction inherent to LFP recordings, NECTA expands upon classical Magnitude Squared Coherence by employing phase-based estimators.
*   **Analytical Signal Extraction:** Raw LFP signals are processed using the Hilbert Transform to yield the analytical signal, granting instantaneous access to both phase $\phi(t)$ and amplitude $A(t)$.
*   **Imaginary Coherence (iCoh) & Weighted Phase Lag Index (wPLI):** The framework computes `iCoh` to strictly isolate interactions with non-zero phase delays. Furthermore, `wPLI` is calculated by weighting the phase differences by the magnitude of the imaginary component of the cross-spectrum, providing a highly robust metric against uncorrelated noise and instantaneous spatial spread.

## 3.5 Directed Functional Connectivity (PDC)
To infer causal interactions and hierarchical information flow, NECTA implements an advanced Partial Directed Coherence (PDC) pipeline.
1.  **Stationarity Filtering:** LFP windows exceeding a predefined variance threshold are identified as artifacts and discarded, preserving the assumptions of autoregressive modeling.
2.  **Automated Order Selection:** Instead of imposing a static model order, the Dask workers dynamically fit Multivariate Autoregressive (MVAR) models using Ordinary Least Squares (OLS). The algorithm iterates through orders (up to `max_order=20`) and selects the optimal $p$ that minimizes the Bayesian Information Criterion (BIC), balancing model fit against parametric complexity.
3.  **Frequency Transformation:** Autoregressive coefficients $A_r$ are transformed into the frequency domain $A(f)$ using Fast Fourier Transforms accelerated via `Numba` (FastMath), yielding the normalized PDC matrices.

## 3.6 Graph Theory and Statistical Validation
A crucial aspect of NECTA is its rigorous statistical and topological validation.
*   **Surrogate Data & FDR Correction:** For connectivity edges (e.g., PDC), significance is established by generating 100 phase-randomized surrogate datasets. Edge-wise p-values are computed and subjected to False Discovery Rate (FDR) correction (Benjamini-Hochberg procedure, $\alpha = 0.05$). Non-significant connections are mathematically zeroed.
*   **Topological Metrics:** The framework extracts core graph properties including Modularity (Louvain heuristic), Efficiency, and Assortativity (the tendency of nodes to connect with similar-degree nodes).
*   **Small-Worldness ($\sigma$) and Null Models:** NECTA calculates the small-world coefficient by comparing the empirical graph's clustering and path length against theoretical Erdös-Rényi models. Furthermore, macro-topology is validated against Configuration Models (via double-edge swapping) to ensure that the detected community structures preserve empirical degree distributions.

## 3.7 Interactive Visualization Dashboard
The final tier of NECTA is a Single Page Application (SPA) built with `Dash` and `Cytoscape`. The interface does not process raw signals; it performs on-the-fly slicing of the pre-computed `Zarr` cache using `xarray.sel()`. Changes in the global state (e.g., sliding the temporal window) trigger instantaneous callbacks that serialize exact data slices into JSON, seamlessly re-rendering complex Connectome Matrices, Temporal Integration scatter plots, and anatomical Information Flow graphs.