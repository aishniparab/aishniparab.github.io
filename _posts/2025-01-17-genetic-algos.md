---
layout: post
title: 'Genetic Programming: A Conceptual Crash Course'
date: 2025-02-11
description: 
tags: evolutionary-algorithms search
categories: 
thumbnail: 
bibliography: 2025-01-17-genetic-algos.bib
---

Genetic programming (GP) is a computational technique that originated in the 1960s <d-cite key="forrest1996genetic"></d-cite>. It evolves programs through a process similar to natural selection, optimizing for a predefined fitness function. The goal of GP is to create programs that approximate a target function. In the example shown, the target function to approximate is \( y = x^2 \).

## Key Steps in Genetic Programming:
1. **Generate Random Initial Programs**: The process begins with randomly generating programs (solutions).
2. **Evaluate Fitness**: The fitness of each program is evaluated by comparing its output to the expected values of the target function.
3. **Apply Mutation and Crossover**: New programs are created by mutating existing ones or combining parts of two programs through crossover.
4. **Repeat the Process**: This cycle is repeated iteratively until a satisfactory program emerges that approximates the target function with minimal error.

## Example:
The specific task is to evolve a function to approximate \( y = x^2 \) where \( x \) takes values from the set \{ -2, -1, 0, 1, 2 \}, and the corresponding expected outputs \( y \) are \{ 4, 1, 0, 1, 4 \}.

- **Terminals**: y, x, 1, 2, ...
- **Operators**: \( +, -, \times, /, \% \)
- **Fitness Function**: The fitness of a program is calculated using the formula:

  \[
  \text{Fitness} = \sum_{i=1}^{n} (y_{\text{output}} - y_{\text{expected}})^2
  \]

Where \( y_{\text{output}} \) is the program's output, and \( y_{\text{expected}} \) is the target output.

## Process Example:
**Initial Programs**:
1. \( P1 \) generates: \( y = x + 1 \) with fitness = 20
2. \( P2 \) generates: \( y = x^2 \) with fitness = 16
3. \( P3 \) generates: \( y = x + x \) with fitness = 16

**Selection**:
Select \( P2 \) and \( P3 \) for crossover as they are the most fit.

**Crossover**:
Combining parts of \( P2 \) and \( P3 \), we produce the new program \( P4 \), which is \( y = x \times x \) (fitness = 0).

**Mutation**:
Further mutation of the program can refine it. Here, replacing part of \( P3 \) produces program \( P5 \), which also converges to the desired result.

**Final Convergence**:
Both \( P4 \) and \( P5 \) converge to the target output with a fitness of 0, meaning they perfectly approximate the function \( y = x^2 \).

This process highlights the power of genetic programming in evolving increasingly efficient solutions through iterative selection, mutation, and crossover processes.