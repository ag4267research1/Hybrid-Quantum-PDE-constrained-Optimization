# Hybrid-Quantum-PDE-constrained-Optimization

Classical and Quantum Linear Solver (QLSA) Experiments

This repository contains reproducible experiments demonstrating how to solve the state and adjoint equations arising in PDE-constrained optimization, both classically and using a simplified Quantum Linear Solver Algorithm (QLSA) implemented in Qiskit.

The purpose is to illustrate, on a small example, how adjoint-based optimization workflows can be integrated with quantum linear-algebra routines.

---

## Problem Setting

We consider the PDE-constrained optimization problem  
$-u''(x) = z(x)$ on $(0,1)$  
with boundary conditions  
$u(0) = u(1) = 0$.

The cost functional is  
$J(u,z) = \frac12 \int_0^1 (u(x) - u_d(x))^2\,dx + \frac{\alpha}{2}\int_0^1 z(x)^2\,dx$.

From the optimality conditions, we obtain:

1. State equation: $-u'' = z$
2. Adjoint equation: $-p'' = u - u_d$
3. Gradient with respect to control: $g = \alpha z - p$

Both PDEs use homogeneous Dirichlet boundary conditions.

---

## Discretization (Finite Differences)

We discretize the interval using $N = 3$ subintervals, giving two interior points.  
Let $h = 1/3$.

The second-order finite-difference discretization of $-u''$ yields the matrix

$A = \begin{pmatrix} 2 & -1 \\ -1 & 2 \end{pmatrix}$.

Thus the discrete equations are:

- State equation: $A u = h^2 z$
- Adjoint equation: $A p = -h^2 (u - u_d)$
- Gradient: $g = \alpha h^2 z - p$

The same matrix $A$ appears in both state and adjoint problems.

---

## Classical Workflow

The classical algorithm does the following:

1. Construct the matrix $A$.
2. Choose a control vector $z$.
3. Solve the state system  
   $u = A^{-1}(h^2 z)$.
4. Form the adjoint right-hand side  
   $b_{\text{adj}} = -h^2 (u - u_d)$.
5. Solve the adjoint system  
   $p = A^{-1} b_{\text{adj}}$.
6. Compute the gradient  
   $g = \alpha h^2 z - p$.

All solves are performed using `numpy.linalg.solve`.

---

## Quantum Workflow (QLSA / HHL-Style Simulation)

To solve the adjoint equation in a quantum-inspired manner, we implement a minimal QLSA/HHL-like circuit whose purpose is to approximate the solution of the linear system

$A p = b_{\text{adj}}$.

The steps are:

1. **State preparation**  
   Normalize the adjoint right-hand side and encode it as a single-qubit state  
   $|b\rangle = \frac{b_0}{\|b\|}|0\rangle + \frac{b_1}{\|b\|}|1\rangle$.

2. **Hamiltonian simulation**  
   Construct  
   $U = e^{i A_{\text{scaled}} t}$  
   using the matrix exponential and wrap it in a `UnitaryGate`.

3. **Quantum Phase Estimation (QPE)**  
   Use a phase ancilla to encode approximate eigenvalue information of $A$.

4. **Controlled rotation**  
   Perform a controlled $R_y$ rotation to approximate the action of $1/\lambda$ on the eigenvalues.

5. **Inverse QPE (uncomputation)**  
   Remove correlations between phase ancilla and the system qubit.

6. **Extraction of the solution**  
   Run the circuit in the statevector simulator, compute the reduced density matrix of the system qubit, and compare it to the classical adjoint vector.

This implementation is a pedagogical demonstration of the QLSA structure.

---

## Hybrid Comparison

We compute and compare:

1. Classical solution time for solving $A p = b_{\text{adj}}$.
2. Quantum simulation time for the QLSA circuit.
3. Accuracy of the quantum solution by normalizing the classical and quantum adjoint vectors and computing their overlap.

This comparison demonstrates:

- How adjoint systems naturally appear in PDE-constrained optimization.
- How quantum linear solvers could be integrated into adjoint-based optimization workflows.
- The expected difference in runtime between classical solves and simulated quantum circuits.

---


