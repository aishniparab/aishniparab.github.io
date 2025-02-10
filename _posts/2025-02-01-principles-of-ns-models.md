---
layout: post
title: Core Design Principles of Neuro-Symbolic Models
date: 2025-02-01 
description: 
tags: neurosymbolic-models
categories: 
thumbnail: 
bibliography: 2025-02-01-principles-of-ns.bib

---

Neurosymbolic models operate within a design space centered on user intent, program generation, and task execution. User intent serves as the guiding principle, describing the task to be performed and specifying the formal language in which the output program should be expressed. These models include a mechanism to generate symbolic programs that align with this intent. Once generated, these programs are executed to produce data that fulfills the specified task. Importantly, the generated output may undergo further refinement or enhancement through a non-symbolic (neural) post-processing step. 

## Task Specification
### Inputs and Outputs
Typically, the input to a neuro-symbolic model can be a single visual entity, such as an image or 3D shape, to be reconstructed, or a collection of visual data with the goal of emulating its distribution. Additionally, inputs can encompass a black-box visual data generator whose behavior should be emulated, a natural language text prompt describing the desired type of visual data to generate, or objective functions that map a program’s visual output to a desirability score. Notably, these latter possibilities remain largely unexplored \cite{ritchie2023neurosymbolic}.

Consequently, a neuro-symbolic model can produce various types of outputs, including deterministic programs, probabilistic programs, or program distributions. A deterministic program generates a fixed visual output each time it is executed, while allowing its parameters to be adjusted to produce variations in the output (e.g., CAD programs). A probabilistic program, in contrast, encapsulates a generative model, enabling it to produce diverse visual outputs by sampling from its internal probabilistic structure during execution. A program distribution takes this a step further by generating a variety of programs themselves, each of which may independently produce outputs. This additional layer of abstraction allows the system to explore a broader space of possible solutions by sampling diverse programs, rather than being confined to variations within a single program.

## Program Specification
To enable a neurosymbolic model to generate programs, a specification language is essential for defining how these programs should be expressed. While general-purpose programming languages like Python or C++ are options, domain-specific languages (DSLs) are often preferred due to their ability to provide specialized operators and data types tailored to a particular domain. This specialization makes DSLs more concise, easier to modify, and reduces the search space of possible outputs, enhancing efficiency.
The design of a DSL involves key considerations, including the programming paradigm, primitive types, and mutability. The choice of programming paradigm shapes how the language operates: functional paradigms prioritize immutability and declarative expressions, imperative paradigms center on sequential instructions and state changes, and constraint paradigms define relationships resolved by solvers. Primitive elements in a DSL can either be manually crafted or augmented with machine learning through neural primitives, which are data types or functions represented by neural networks.
Mutability is another crucial aspect of DSL design. A language can remain fixed, undergo preprocessing modifications to identify and factor out common patterns into new procedures, or evolve as the model learns. In some cases, no predefined language is provided, allowing the neurosymbolic model to invent its own. In this case, the model must propose a set of primitive data types and functions capable of combining these types to effectively represent visual data. This approach, where the model invents its own language, remains an intriguing and largely unexplored area.
## Program Synthesizer
With a given task specification and a DSL, a neurosymbolic model must generate programs that fulfill the requirements of the specification within the constraints of the DSL. This process is essentially a **search problem**, involving the exploration of the program space defined by the DSL to identify those that meet the specification.

**Prior:** It is important to define *a priori* the preferred characteristics of programs, as any DSL will likely produce multiple programs that satisfy the input specification. Consistent with Occam's razor, existing research generally favors shorter, simpler programs.

Various **search algorithms** have been applied to program synthesis, each with distinct strategies and use cases. 

**Database Retrieval:** The simplest approach is database retrieval, where instead of generating a new program, the method retrieves an existing one from a database and adjusts its parameters to better meet user goals. 

**Enumerative Search:** Explicit enumeration, another approach, can be implemented either top-down or bottom-up, exploring the program space from the root or the leaves of an abstract syntax tree (AST). However, this method is practical only for identifying short programs in simple languages, as the complexity of larger spaces requires additional techniques to remain computationally feasible. In complex scenarios, additional techniques are necessary to efficiently navigate the expansive search space for programs. Heuristics are commonly used to reduce the search space, with greedy search serving as the most restrictive approach by prioritizing immediate results. Beam search and its variations are widely applied to achieve a balance between thorough exploration and computational efficiency. Top-down search methods, such as branch-and-bound, provide another effective strategy by systematically pruning less optimal paths. More recent innovations have focused on compressing the search space by compactly representing large sets of equivalent sub-programs, leveraging tools like version spaces [[^1]] and e-graphs [[^2].

**Stochastic Search**: When the program space defined by a language becomes too vast for enumeration to be practical, randomized search methods provide a viable alternative. These approaches typically begin with an initial set of programs, which may be generated randomly or guided by heuristics, and iteratively apply random modifications to improve the programs. Techniques like Markov Chain Monte Carlo (MCMC) and simulated annealing operate on a single candidate program at a time, refining it through stochastic updates. In contrast, algorithms such as Sequential Monte Carlo (SMC) and genetic programming maintain a population of programs, applying random changes across multiple candidates simultaneously.

**Constraint Satisfaction**: Some tasks can be addressed by converting the specification into a set of constraints, treating the problem as one of constraint satisfaction. A common approach involves translating the specification into a Boolean satisfiability (SAT) problem, enabling the use of efficient SAT solvers, provided the problem size remains manageable. If direct conversion to SAT is infeasible, the specification can instead be expressed as a first-order logic satisfiability problem, allowing the application of SMT solvers.

**User-in-the-loop:** Instead of automatically synthesizing a program, a user can provide suggestions about what different parts of the output program should look like, helping the synthesizer make decisions.

**Termination**: Enumeration-based and constraint-based algorithms terminate once all possible programs have been explored, but this can demand an intractable amount of computation. In contrast, stochastic search algorithms lack a natural termination point and can, in principle, run indefinitely, potentially producing progressively better programs over time. As a result, program synthesizers often treat these search methods as anytime algorithms, designed to return the best program discovered within a predefined computational budget. This approach ensures useful results even when the search cannot fully exhaust the program space.

**Neural Guidance**: Many search algorithms benefit from heuristic guidance, which is where machine learning often plays a role. For example, beam search requires ranking possible next states, branch-and-bound needs to estimate bounds on scores before expanding, and stochastic methods can use non-uniform distributions for random changes. Neural guidance typically involves learning heuristics to suggest what to add to a program next. Autoregressive language models, especially transformers, are widely used for this purpose, while earlier methods employed pointer networks to suggest variable re-use or graph convolutional networks, which have become less common since the rise of attention-based models. In program retrieval systems, deep feature spaces for similarity search also act as neural guidance. Some approaches bypass search entirely, generating programs in a single forward pass of a neural network. These methods can be categorized as either explicit enumeration or stochastic search with neural guidance, depending on whether the network selects the most probable choice or samples randomly.

## Program Execution
Once a program is generated, it has to be executed to produce an output. This execution process can be implemented in various ways. The most common approach is to directly execute the program. Alternatively, some languages require solving a search, optimization or a constraint-satisfying problem. This approach typically increases learning complexity as computing gradients with respect to an execution engine may not be straightforward. Here, a neural network can be used to guide search during program execution. While we can use hardcoded execution engines, it is possible to use a learned module which maps program tokens to program output. Although this learned mapping is only an approximation it can aid learning of other neural components. 

## Neural Postprocessing
After executing the program, one can include a post-processing component that refines the output to make it more realistic, e.g. improve the quality of the image.

## Learning Algorithm
A neurosymbolic model can be trained using one of two approaches: end-to-end in an unsupervised manner or modularly in a supervised fashion. In end-to-end training, the final task loss is backpropagated through all components of the model, including each decision made by the synthesizer in constructing the program. However, programs are inherently discrete due to branching factors and primitive types, making it difficult to define gradients over discrete choices.

To address this challenge, one approach is to introduce a continuous relaxation of discrete program choices. This requires defining a program executor capable of producing semantically meaningful programs. The design of such relaxations and their executors is highly domain-specific and demands expert knowledge. Nevertheless, once a continuous space over programs is established, the system becomes end-to-end differentiable.

An alternative strategy is to employ policy gradient reinforcement learning using a score function gradient estimator. This method does not require any component of the model to be differentiable, as it stochastically estimates gradients in an unbiased manner. However, these algorithms often suffer from slow convergence, frequently getting trapped in poor local optima or failing to converge at all due to high variance.

A neurosymbolic model can also be trained in a modular fashion, where each module is supervised using input-output pairs specific to that module. However, obtaining input data paired with ground-truth program annotations is often challenging. In practice, human-written programs satisfying particular task specifications have been used, but such data is difficult to acquire. Thus, an alternative approach involves generating synthetic programs and task specification pairs, for example, by randomly deriving programs from a domain-specific language (DSL) grammar. While this method allows for scalable data generation, synthetic data does not generalize well due to its limited complexity and diversity compared to real-world programs. As a result, synthetic datasets are primarily used for initialization rather than as a primary learning method.
A common approach to mitigate this limitation is bootstrapping, where a model is first pre-trained on synthetic data and then refined iteratively by training on variations of its own predictions. 

[^1] **Definition of a version space.**
A **version space** is a conceptual framework used to represent all hypotheses consistent with a set of observations or constraints. In the context of program synthesis, a version space is a compact representation of all programs that meet a given specification. Instead of enumerating each individual program, the version space describes a set of equivalent or partially equivalent programs, often using symbolic or parametric constraints. This approach helps reduce the computational burden by focusing on a concise yet comprehensive representation of viable solutions.
For example, in machine learning, a version space is often defined as the set of hypotheses that remain consistent with the training data. In program synthesis, it could mean all abstract syntax trees (ASTs) that can perform a desired task within a specific domain or under certain constraints.

Version spaces focus on the representation of all consistent programs with a specification but do not inherently store equivalence relationships.

For example, suppose you are synthesizing a program to compute the area of a rectangle. The specification requires that the program take two inputs, length and width, and output their product. A version space would represent all possible programs that satisfy this specification, like this: 
* `Hypothesis 1: area = length * width`
* `Hypothesis 2: area = width * length (commutativity)`
* `Hypothesis 3: area = sum(length repeated width times) (alternative algorithm)`

[^2] **Definition of an e-graph**.
An **e-graph** is a data structure used to compactly represent equivalence classes of expressions. Each node in an e-graph represents an **equivalence class**—a set of expressions that are considered semantically equivalent under certain rules. The edges in the graph connect these equivalence classes to indicate transformations or derivations among them.

E-graphs are especially useful in program optimization and synthesis because they allow efficient representation and manipulation of large sets of equivalent programs. By compactly storing all possible rewrites of a program under a given set of rules, e-graphs reduce redundancy in the search space, enabling faster exploration. Tools like **egg** (an e-graph library) use this structure to efficiently explore and optimize programs by systematically applying equivalence rules until an optimal or desired program is found.

E-graphs explicitly track equivalences among programs or subprograms, making them particularly powerful for tasks involving program transformation or optimization.

For example, suppose you are synthesizing a program to compute the area of a rectangle. The specification requires that the program take two inputs, length and width, and output their product. To represent equivalent expressions for optimization, we can define an initial expression: 
`area = (length + length) + (width + width)`. 

Equivalence rules:
* Rule 1: Factorization; `a + a = 2 * a`
* Rule 2: Associativity; `(a * b) * c  = a * (b * c)`

An e-graph would represent all equivalent forms of the expression using equivalence classes:
* Node for `(length + length)` and `2 * length` (equivalent by Rule 1)
* Node for `(width + width)` and `2 * width` (equivalent by Rule 1)
* Node for the full expression `(length + length) + (width + width)` and its equivalent forms like `2 * length + 2 * width`.

By applying the rules repeatedly, the e-graph would compactly store multiple equivalent ways to compute the same value. This allows an optimizer to choose the simplest or most efficient version, such as: `area = 2 * (length + width)`.