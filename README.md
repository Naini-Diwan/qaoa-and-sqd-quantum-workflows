# qaoa-and-sqd-quantum-workflows

Two Qiskit-based workflows that each map a classically hard problem onto a quantum circuit, execute it on IBM hardware), and recover the answer through classical post-processing. One targets a combinatorial optimization problem, the other a quantum chemistry ground-state problem. Both end in a diagonalization step over a space too large to search exactly.

## Contents

| Notebook | Problem | Quantum step | Classical recovery step |
|---|---|---|---|
| `notebook-1` | Max-Cut on a graph | QAOA (Quantum Approximate Optimization Algorithm) | Exact diagonalization of the cost Hamiltonian (problem small enough to solve directly) + noise/error-mitigation study across backends |
| `notebook-2` | Ground-state energy of a molecule (N₂) | LUCJ ansatz circuit sampled on IBM hardware | Sample-based Quantum Diagonalization (SQD): diagonalize the Hamiltonian in a subspace spanned by sampled bitstrings, refined through an iterative Monte Carlo-style configuration recovery loop |

## What's inside

### Notebook 1: Combinatorial optimization with QAOA

- Formulates Max-Cut as an Ising Hamiltonian (Pauli Z and identity terms).
- Builds and transpiles a QAOA circuit, optimizes variational parameters with `scipy`.
- Cross-checks the QAOA result against the exact ground state, obtained by direct diagonalization of the cost Hamiltonian (feasible because the problem instance is small).
- Benchmarks three different IBM backends (e.g. `ibm_brisbane`, `ibm_sherbrooke`, `ibm_torino`) under realistic noise, comparing accumulated gate error, transpiler layout choices, and solution-probability outcomes.
- Applies and compares error-mitigation strategies: zero-noise extrapolation (linear, polynomial, exponential), dynamical decoupling, Pauli twirling, and the NEAT (Noisy Estimator Analyzer Tool) estimator.
- Runs a larger bonus instance end-to-end on real quantum hardware with full mitigation.

### Notebook 2: Quantum chemistry ground-state estimation with SQD

- Introduces the scaling problem directly: the many-electron Hamiltonian grows combinatorially with basis set size (spin orbitals, active electrons), quickly exceeding what exact diagonalization can handle.
- Builds a hardware-efficient ansatz (Local Unitary Cluster Jastrow, LUCJ) tailored to the target device's qubit connectivity.
- Samples bitstrings from the ansatz circuit (shots on real hardware / pre-collected samples).
- Runs the SQD post-processing loop:
  1. **Configuration recovery** — corrects noisy bitstrings using known particle-number symmetries and current orbital-occupancy estimates.
  2. **Subsampling** — draws batches of corrected bitstrings (e.g. 50 per batch, multiple batches, parallelizable).
  3. **Subspace diagonalization** — projects the full Hamiltonian onto the subspace spanned by each batch's bitstrings and diagonalizes there, rather than over the full exponential Hilbert space.
  4. **Aggregation** — combines batch results into an updated ground-state energy and occupancy estimate, iterating until convergence.
- Studies how ansatz and layout choices affect accuracy: basis set size, qubit layout selection by two-qubit error, and added long-range interaction terms in the ansatz.
- Closes with a real-hardware run combining the full configuration-recovery loop with error mitigation.

## Why these two are grouped together

Despite different target problems (graph optimization vs. molecular energy), both notebooks follow the same shape:

1. Map a classical problem to a Hamiltonian.
2. Encode a trial state in a parameterized quantum circuit.
3. Execute on simulator and real QPU hardware, subject to transpilation and noise.
4. Recover the solution classically by diagonalizing the problem in a space small enough to handle, exactly for the optimization case, in a sampled subspace for the chemistry case, with the second case additionally relying on stochastic (Monte Carlo-style) resampling to refine that subspace.

## Requirements

```
qiskit>=2.0.0
qiskit-ibm-runtime>=0.40
qiskit-addon-sqd>=0.11.0
ffsim
rustworkx
pylatexenc
scipy
matplotlib
```

An IBM Quantum Platform account (open plan) is required for real-hardware execution cells; all notebooks run end-to-end on simulators without one.

## Notes

- Real-hardware cells consume QPU time against the open plan's monthly allotment; simulator-only runs are unaffected.
- Some cells are long-running (several minutes).
- Pre-collected hardware samples are included for the chemistry notebook so the post-processing pipeline can be run without consuming QPU time.

## Author's Note
If you have any questions/suggestions regarding this work, feel free to contact me.

Regards,
Naini Diwan
