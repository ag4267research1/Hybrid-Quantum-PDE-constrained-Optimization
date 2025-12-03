# Hybrid-Quantum-PDE-constrained-Optimization
 
Classical and Quantum Linear Solver (QLSA) Experiments

This repository contains a set of reproducible experiments demonstrating how to solve the **state** and **adjoint** equations arising in **PDE-constrained optimization**, both classically and using a simplified **Quantum Linear Solver Algorithm (QLSA)** implemented in Qiskit.

The goal is to illustrate, at a small computational scale, how adjoint-based optimization workflows can be coupled with quantum linear-algebra routines.

---

## Problem Setting

We consider the PDE-constrained optimization problem
$$
- u''(x) = z(x), \qquad x \in (0,1),
$$
with boundary conditions
$$
u(0) = u(1) = 0.
$$

The cost functional is
$$
J(u, z) =
\frac{1}{2} \int_0^1 (u(x) - u_d(x))^2 \, dx
+ \frac{\alpha}{2} \int_0^1 z(x)^2 \, dx.
$$

This leads to three equations:

1. **State equation**
   $$
   -u'' = z
   $$

2. **Adjoint equation**
   $$
   -p'' = u - u_d
   $$

3. **Gradient with respect to the control**
   $$
   g = \frac{\partial J}{\partial z} = \alpha z - p.
   $$

All equations use homogeneous Dirichlet boundary conditions.

---

## Discretization (Finite Differences)

We discretize the interval using \( N = 3 \) subintervals, producing two interior points.
Let \( h = 1/3 \).  
Using the standard second-order finite difference stencil, the discrete Laplacian with Dirichlet boundaries becomes the matrix

$$
A = \begin{pmatrix}
2 & -1 \\
-1 & 2
\end{pmatrix}.
$$

Thus the discrete equations are:

- State system:
  $$
  A u = h^2 z.
  $$

- Adjoint system:
  $$
  A p = -h^2 (u - u_d).
  $$

- Gradient:
  $$
  g = \alpha h^2 z - p.
  $$

---

## Classical Workflow

The classical computation proceeds as follows:

1. Build the matrix \( A \).
2. Select a control vector \( z \).
3. Compute the state:
   $$
   u = A^{-1}(h^2 z).
   $$
4. Build the adjoint right-hand side:
   $$
   b_{\text{adj}} = -h^2 (u - u_d).
   $$
5. Solve the adjoint system:
   $$
   p = A^{-1} b_{\text{adj}}.
   $$
6. Compute the gradient:
   $$
   g = \alpha h^2 z - p.
   $$

All classical solves use `numpy.linalg.solve`.  
This yields the exact state, adjoint, and gradient for the finite-difference problem.

---

## Quantum Workflow (QLSA/HHL-Style Simulation)

To solve the **adjoint equation** using quantum methods, we implement a minimal QLSA/HHL-style circuit.

The idea is to solve the linear system
$$
A p = b_{\text{adj}},
$$
using a quantum circuit that approximates
$$
|p\rangle \propto A^{-1} |b_{\text{adj}}\rangle.
$$

The quantum steps consist of:

1. **State preparation**  
   Normalize \( b_{\text{adj}} \) and encode it as a 1-qubit state
   $$
   |b\rangle = \frac{b_0}{\|b\|} |0\rangle +
               \frac{b_1}{\|b\|} |1\rangle .
   $$

2. **Hamiltonian simulation**  
   Build
   $$
   U = e^{i A_{\text{scaled}} t}
   $$
   using `scipy.linalg.expm` and wrap it as a `UnitaryGate`.

3. **Quantum Phase Estimation (QPE)**  
   Use one phase ancilla to extract approximate eigenvalue information of \(A\).

4. **Controlled rotation**  
   Apply a rotation conditioned on the estimated eigenvalue, which approximates the operation \(1/\lambda\).

5. **Inverse QPE (uncomputation)**  
   Restore ancilla qubits to their initial states.

6. **Measurement and extraction**  
   The resulting system qubit encodes a direction proportional to the adjoint solution \(p\).  
   We extract the statevector, compute the reduced density matrix, and compare it to the classical adjoint vector.

This quantum workflow is not a full HHL implementation but a structurally accurate pedagogical demonstration of how adjoint systems can be handled using quantum linear solver methods.

---

## Hybrid Comparison

We compare:

1. **Classical solution time** for  
   $$
   A p = b_{\text{adj}}
   $$
   using `numpy.linalg.solve`.

2. **Quantum simulation time** for the QLSA circuit on the `aer_simulator` statevector backend.

3. **Accuracy comparison**  
   We normalize both classical and quantum adjoint vectors and compute their overlap.

This demonstrates:

- How adjoint equations naturally fit into hybrid quantum-classical PDE optimization workflows.
- How quantum linear solvers may provide scalability advantages for larger systems.
- How Qiskit simulation time compares to classical solves in the small-scale regime.

---

## Requirements

