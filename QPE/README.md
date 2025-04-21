# Quantum Phase Estimation
Let's define the problem more carefully. We are given a *unitary operator* $U$ and one of its *eigenstates* $|\psi\rangle$. The operator is *unitary*, so we can write:

$$U|\psi\rangle=e^{i\phi}|\psi\rangle,$$

where $\phi$ is the *phase* of the *eigenvalue* (remember, unitaries have eigenvalues with an absolute value of 1).

The goal is to estimate $\phi$, hence the name *phase estimation*.

## Part 1: Representing the Phase

A first step is to find a quantum circuit that performs the transformation $$|\psi\rangle|0\rangle \to |\psi\rangle|\phi\rangle.$$
We could then obtain $\phi$ directly by measuring the second register. We call this the **estimation register**.

However, because the complex exponential has period $2\pi$, technically the phase is not unique. Instead, we define $\phi=2\pi\theta$ so that $\theta$ is a number between $0$ and $1$.

How can we represent $\theta$ on a quantum computer? The answer is the first part of the algorithm: **we present $\theta$ in binary.**

When we write the number 0.15625, it is being expressed as a sum of multiples of powers of 10:

$$0.15625 = 1 \times 10^{-1} + 5 \times 10^{-2} + 6 \times 10^{-3} + 2 \times 10^{-4} + 5 \times 10^{-5}$$

But nothing is stopping us from using 2 instead of 10. In binary, the same number is $0.00101_2$:

$$0.00101_2 = 0 \times 2^{-1} + 0 \times 2^{-2} + 1 \times 2^{-3} + 0 \times 2^{-4} + 1 \times 2^{-5}$$

(You can confirm this by computing $1/8 + 1/32$ on a calculator). Similarly, $0.5_{10}$ is $0.1_2$ in binary, and $0.3125_{10}$ is $0.0101_2$.

Ok, now back to quantum. A binary fraction is useful because we can encode it using qubits, e.g., $|110010\rangle$ for $\theta=0.110010$. The phase is retrieved by measuring the qubits. The **precision** of the estimate isdetermined by **the number of qubits**. For example, the binary expansion of $0.8$ is $0.11001100...$ which does not terminate. From now on, we’ll use n for the number of estimation qubits.

## Part 2: Quantum Fourier Transform

The second part of the algorithm is to follow advice given to many physicists: "When in doubt, take the Fourier transform."; or in ourcase, "Whenin doubt, take the quantum Fourier transform (QFT)".

$$\text{QFT}|k\rangle = \frac{1}{\sqrt{2^n}} \sum_{k=0} e^{2\pi i \theta k} |k\rangle.$$

Note that this results in a uniform superposition, where each basis state has an additional phase. If we can prepare that state, then applying the *inverse QFT* would give $|\theta\rangle$ in the estimation register. This looks more promising, especially if we notice the appearance of the eigenvalue $e^{2\pi i \theta}$, although with an extra factor of $k$. We can obtian this factor by applying the unitary $k$ times to the state $|\psi\rangle$:

$$U^k|\psi\rangle = e^{2\pi i \theta k}|\psi\rangle.$$

Therefore, we will use $\psi$ and $U$ to generate factos that are of interest to us in each of the basic states. It would then be enough to create an operator such that:

$$|\psi\rangle|k\rangle \rightarrow U^k|\psi\rangle|k\rangle.$$

In this way, if we apply this operator to the uniform superposition we obtain:

$$\frac{1}{\sqrt{2^n}} \sum_{k=0} |\psi\rangle|k\rangle \rightarrow \frac{1}{\sqrt{2^n}} \sum_{k=0} U^k|\psi\rangle|k\rangle = |\psi\rangle \left( \frac{1}{\sqrt{2^n}} \sum_{k=0} e^{2\pi i \theta k} |k\rangle \right)$$

This exactly what we want! We refer to this as **ControlledSequence** operation.

## Part 3: Controlled sequence

We follow another timeless piece of physics advice: "If stuck, start with the simplest case". Let's see what happens with two qubits. After applying the *Hadamards*, the operator we need is

$$|\psi\rangle|00\rangle + |\psi\rangle|01\rangle + |\psi\rangle|10\rangle + |\psi\rangle|11\rangle \rightarrow U^0|\psi\rangle|00\rangle + U^1|\psi\rangle|01\rangle + U^2|\psi\rangle|10\rangle + U^3|\psi\rangle|11\rangle$$

We can extend this idea to any number of qubits. The following animation illustrates this effect.

![](https://pennylane.ai/_images/controlledSequence.gif)

With six qubits, an example would be

$$|\psi\rangle|010111\rangle \rightarrow U^{16}U^4U^2U^1|\psi\rangle|010111\rangle = U^{23}|\psi\rangle|010111\rangle$$

Note that $010111_2$ is $23_{10}$.


So we have the answer: **apply $U^{2^m}$ controlled on the $m$-th estimation qubit**.


Bringing it all together, here is the quantum phase estimation algorithm in all its glory:

## The QPE algorithm

1. Start with the state $|\psi\rangle|0\rangle$. Apply Hadamard gate to all estimation qubits to implement the transformation

$$|\psi\rangle|0\rangle \rightarrow |\psi\rangle \left( \frac{1}{\sqrt{2^n}} \sum_{k=0} |k\rangle \right).$$

2. Apply **ControlledSequence** operation, i.e., $U^{2^m}$ controlled on the $m$-th estimation qubit. This gives

$$|\psi\rangle \left( \frac{1}{\sqrt{2^n}} \sum_{k=0} |k\rangle \right) \rightarrow |\psi\rangle \left( \frac{1}{\sqrt{2^n}} \sum_{k=0} e^{2\pi i \theta k} |k\rangle \right).$$

3. Apply the *inverse QFT* to theestimation qubits

$$|\psi\rangle \left( \frac{1}{\sqrt{2^n}} \sum_{k=0} e^{2\pi i \theta k} |k\rangle \right) \to |\psi\rangle|0\rangle.$$

4. Measure the estimation qubits to recover $\theta$.

![](https://pennylane.ai/_images/qpe.png)


QPE is doing something incredible: it can calculate eigenvalues **without ever diagonalizing a matrix**. This is true even if we relax the assumption that the input is an eigenstate. By linearity, for an arbitrary state expanded in the eigenbasis of $U$ as

$$|\Psi\rangle = \sum_i c_i |\psi_i\rangle ,$$

QPE outputs the eigenphase $\theta_i$ with probability $|c_i|^2$.
