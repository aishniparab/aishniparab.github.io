---
layout: post
title: 'Mathematical discoveries from program search with large language models: A Summary'
date: 2025-02-11
description: 
tags: evolutionary-algorithms search article-notes
categories: 
thumbnail: 
bibliography: /assets/bibliography/2025-02-11-funsearch.bib
---
A recent paper, *Mathematical Discoveries from Program Search with Large Language Models*<d-cite key="romera2024mathematical"></d-cite>, introduces **FunSearch**, a novel approach that redefines how we solve complex mathematical and computational problems. Instead of searching for solutions directly, **FunSearch searches for functions that describe how to solve a problem**. These functions are essentially programs that encode the structure and logic of the underlying problem. Rather than merely presenting a result, these programs articulate the logical process behind the solution. By evaluating a program across a range of inputs, its effectiveness in solving the problem can be assessed.

Large language models (LLMs) have demonstrated remarkable proficiency in generating code, making them well-suited for producing such programs. However, they suffer from **hallucinations**—producing incorrect or unverifiable outputs. LLMs also struggle to move beyond known examples, often generating solutions that resemble patterns from their training data rather than discovering genuinely new insights.The FunSearch paper demonstrated that **combining LLMs with evolutionary algorithms** can overcome these limitations, systematically explore vast function spaces, and achieve groundbreaking discoveries in two NP-hard problems: the Cap Set problem and Online Bin Packing.

FunSearch depends on a few key aspects:
1. **Pre-trained (frozen) LLM**: Codey, an LLM built on top of the PaLM2 model family, fine-tuned on a large corpous of code is used to sample a program.
2. **Evaluation**: The program is evaluated symbolically on a range of inputs and the score of the program is computed for various inputs, and then aggregated using the mean function (see appendix for score function). The scored programs are then stored in the program database. Incorrect programs or programs that timed-out within imposed time and memory limits are discarded. 
3. **Program database**: A database is used to maintain a diverse set of correct programs. To maintain diversity and prevent convergence to local optima, an island model is used, where multiple subpopulations evolve independently with occasional migration. The lowest-performing half of the islands are periodically reset by replacing their populations with clones of top individuals from the surviving islands.
4. **Best-shot prompting:** *k* programs are sampled from the database by first selecting an island, then choosing a program within it, favoring higher-scoring and shorter ones. The sampled programs, sorted by score, are assigned version labels (e.g.,  v_0  for the lowest-scoring,  v_1  for the second lowest, etc.). These programs are then combined into a single prompt, with the version appended as a suffix to the function name (e.g., function_v0). In practice,  k = 2  is used, as higher values yield diminishing returns. This structured prompt is then fed to the LLM to generate a new program.
5. **Skeleton-based initialization:** The initial program contains boilerplate code and predefined problem structure while leaving the core logic open for evolution. 

The figure below illustrates these key components.
{% include figure.liquid loading="eager" path="assets/img/funsearch_pipeline.png" class="img-fluid rounded z-depth-1" zoomable=true style="width: 500px; height: 300px;" %}

FunSearch operates in a **distributed manner**, with three key components communicating asynchronously:
* **Programs Database:** Stores and serves candidate programs.
* **Samplers:** Generate new functions using the pretrained LLM.
* **Evaluators:** Assess the effectiveness of programs.

Since LLM sampling and evaluation occur concurrently, FunSearch can scale by adding more samplers or evaluators. The experiments conducted in this paper used **15 samplers and 150 CPU evaluators** (5 CPU servers each running 32 evaluators), all running in parallel. Since there is inherent randomness in the system architecture, the experiments were repeated multiple times. 

## Applying FunSearch to NP-Hard Problems
The authors of this paper demonstrate the power of FunSearch by applying it to two NP-hard problems, the Cap Set problem and the Online Bin Packing problem. 

The Cap Set Problem is a question in extremal combinatorics that asks how large a set of numbers can be while avoiding any three that form a simple pattern like an arithmetic progression (a sequence of numbers where the difference between consecutive terms is the same, such as 2, 4, 6 or 3, 6, 9). A simple example is picking numbers from \{1, 2, 3, 4, 5, 6, 7\} while avoiding any three that are evenly spaced, such as \{1, 3, 5\} or \{2, 4, 6\}. While this example is in one dimension, the Cap Set Problem generalizes to higher-dimensional spaces, where numbers are replaced by vectors in an n-dimensional space over a finite field, making the problem significantly more complex. 

The Online Bin Packing Problem is a classic optimization problem in theoretical computer science and operations research, where items of varying sizes arrive sequentially and must be packed into bins of fixed capacity without knowledge of future items. The goal is to minimize the number of bins used while making irrevocable placement decisions as each item arrives. A simple example is packing numbers representing item sizes, such as \{4, 7, 2, 5, 6\}, into bins of size 10. If the items arrive one by one, an online strategy might place 4 and 7 in separate bins before realizing later that 4 and 6 could have fit together, demonstrating the challenge of not knowing the full sequence in advance. 

The figure below shows the specification for the Cap Set problem (a) and the Online Bin packing problem (b).
{% include figure.liquid loading="eager" path="assets/img/funsearch_spec.png" class="img-fluid rounded z-depth-1" zoomable=true style="width: 500px; height: 300px;"%}

The spec contains the evaluate function which takes in the solution and returns a score assessing it, the solve function contains the skeleton algorithm, which calls the function to evolve that contains the crutial logic. In Cap Set problem, priority function is the function to be evolved and in Online Bin Packing, the heuristic function is the fucntion to be evolved. The main function uses the solve function to solve the problem, then uses the evaluate function to score the solution. Sometimes, as shown in b, the main function can implement additional logic. This format allows FunSearch to focus on the most critical part of the program, avoiding failures due to mistakes in parts of the program that are already known. Additional information about the known parts of the problem can be provided in the form of docstrings, relevant primitive functions or import packages.

The specification consists of three key functions: **evaluate**, which scores a given solution; **solve**, which serves as the skeleton algorithm; and a function to be evolved, which encapsulates the critical decision-making logic. In the **Cap Set problem**, the **priority** function is evolved, while in **Online Bin Packing**, the **heuristic** function is evolved. The **main** function orchestrates the solution by calling **solve** and then using **evaluate** to assess its quality. In some cases, as seen in the Online Bin Packing example, additional logic is implemented within the main function. This modular structure allows **FunSearch** to focus on evolving the most crucial aspects of the program while leveraging known and well-defined components. Supporting information, such as docstrings, relevant utilities, or predefined imports, can al so be provided.

For the **cap set** problem, the experiment involved evolving a function that prioritizes which vectors to include in a set, ensuring no three sum to zero. Using this approach, the method discovered a larger cap set in dimension 8 than previously known and improved the lower bound on cap set capacity, demonstrating its ability to scale to higher dimensions and generate novel mathematical insights.

For the **bin packing** problem, the experiment evolved a heuristic that decides where to place incoming items in bins to minimize wasted space. The discovered heuristic outperformed traditional methods (first fit and best fit) across multiple datasets, generalizing well to larger problem instances and improving efficiency in practical scenarios.

The specification consists of three key functions: **evaluate**, which scores a given solution; **solve**, which serves as the skeleton algorithm; and a function to be evolved, which encapsulates the critical decision-making logic. In the **Cap Set problem**, the **priority** function is evolved, while in **Online Bin Packing**, the **heuristic** function is evolved. The **main** function orchestrates the solution by calling **solve** and then using **evaluate** to assess its quality. In some cases, as seen in the Online Bin Packing example, additional logic is implemented within the main function. This modular structure allows **FunSearch** to focus on evolving the most crucial aspects of the program while leveraging known and well-defined components. Supporting information, such as docstrings, relevant utilities, or predefined imports, helps ensure clarity and correctness.

## Key Takeaways
* **Searching in function space enables more efficient discovery**: Instead of directly searching for solutions—such as an enormous list of numbers in the Cap Set problem—FunSearch searches for functions (expressed as programs) that describe how to solve the problem. These functions provide a more structured and interpretable representation.
* **Programs offer an interpretable and concise representation**: Many problems of interest are highly structured rather than random. Solutions can often be described more concisely with a computer program than with other representations. For instance, the trivial representation of the admissible set A(24, 17) consists of over 200,000 vectors, whereas the program generating this set is only a few lines of code.
* **Kolmogorov-compressed inductive bias enables scalability**: FunSearch **favors concise programs**, allowing it to scale to much larger problem instances compared to traditional search methods. In a loose sense, FunSearch **optimizes for low Kolmogorov complexity**—the shortest program that produces a given output—whereas conventional search algorithms follow a different inductive bias. 
* **LLMs should be viewed as program generators, not problem solvers**: The pre-trained LLM does not rely heavily on problem-specific context. Instead, it acts as a generator of diverse, syntactically correct programs, occasionally producing novel and useful ideas.
* **LLMs have a natural bias for human-written code**: Unlike genetic algorithms, which evolve solutions without any preference for human readability, LLMs inherently favor structured, human-like code, making their outputs more interpretable and aligned with conventional programming practices.

## When is FunSearch Applicable?
FunSearch is most effective for problems that meet the following criteria:
* **Well-defined evaluator**: The problem must have a clear evaluation function that can assess the quality of a solution.
* **Rich scoring feedback**: The evaluation should provide **granular, informative feedback** to guide iterative improvements.
* **Structured skeleton with an isolated evolvable component**: The problem should allow for a predefined program structure (skeleton) while leaving a critical decision-making function open for evolution.

Problems like **MAX-SAT** are well-suited for FunSearch because the number of **satisfied clauses** naturally serves as a **scoring signal**. In contrast, theorem proving is more challenging to frame within this paradigm, as it lacks a straightforward scoring function that can provide meaningful intermediate feedback.