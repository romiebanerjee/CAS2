## TL;DR

- Kolmogorov's algorithmic statistics explains an individual string by a simple finite set containing it, with no probabilistic assumptions. The best such models are uncomputable, so any practical method must search a restricted class of them.
- Two papers study the class given by symmetry. **CAS I** shows that describing strings by symmetries recovers Kolmogorov complexity, via a geometric coding theorem. **CAS II** makes orbits into models, introduces the $G$-structure function and $G$-sophistication, and gives the space of strong models coordinates: a **moduli space** with explicit moves.
- The main limit: some strings have essentially no sophistication, yet their structure is completely invisible to linear symmetry. Any space of symmetry hypotheses small enough to search is small enough to miss simple structure.
- The long-term goal is an RL agent, later LLM-guided, that searches this moduli space for strong models.

## Introduction

Algorithmic statistics goes back to Kolmogorov, who in 1974, in a talk at the Information Theory Symposium in Tallinn, proposed reformulating statistics without probabilistic assumptions. Classical statistics assumes the data were drawn from a distribution and reasons about that distribution. Kolmogorov wanted to judge how well an *individual* model fits *individual* data, on the ground of finite combinatorics and computation alone. A model is a finite set containing the data, and a good model meets two criteria: the set has a simple description, and the data is a typical element of it. The string's position inside the set is then noise. The idea has since grown into a rich theory; see Li and Vitányi's textbook (§5.5) and Vereshchagin and Shen's survey [*Algorithmic statistics: forty years later*](https://arxiv.org/abs/1607.08077).

As in classical statistics, in practice one chooses from a class of candidate models, which may not contain the best one. In algorithmic statistics this is forced, because the best models are uncomputable. I've written two papers that study the case where the candidate models come from symmetry, under the series name **CAS (Computational Algorithmic Statistics)**. The long-term goal of the series is a reinforcement-learning algorithm that searches for *strong models* in algorithmic statistics. A strong model is computed from the data by a short total program, so this is a form of **program search**. These first two papers lay the mathematical foundations for that search: they define its state space of simple symmetric partitions and the moves through it, motivate it, and prove its theoretical limits. Follow-up papers in the series move towards LLM-guided strong model search.

- **CAS I: A Geometric Coding Theorem** treats a symmetry as a *program*: a string is described by a symmetry whose only fixed point is that string. For a broad class of groups, the resulting "symmetry prior" is exactly as good as the Solomonoff prior, up to a constant.
- **CAS II: Orbits as Models** treats a symmetry as a *hypothesis*: a group's orbits sort all strings into classes, and the orbit containing your data is the model. This gives a structure function that measures **which part of the regularity of $x$ is symmetric**. Restricting models to homogeneous cells also gives the space of strong models coordinates: each symmetric hypothesis is pinned down by a type and a placement, and relabelling, restriction and meets move between hypotheses at bounded cost. This **moduli space** is the space an RL agent will navigate.

## Background: models, noise and the structure function

Kolmogorov's idea for separating signal from noise is the two-part code. Explain a string $x$ by a finite set $S$ that contains it. First pay for describing $S$, then pay $\log|S|$ bits to say where $x$ sits inside $S$. The second part is treated as noise.

$$C(x) \lesssim C(S) + \log|S|$$

The **structure function** $h_x(\alpha)$ records the smallest model you can afford at complexity budget $\alpha$. A model is a *sufficient statistic* when the two parts together cost no more than $C(x)$. The **sophistication** of $x$ is the cheapest budget at which a sufficient statistic exists: roughly, how much "real structure" $x$ has, as opposed to noise.

![A typical structure function (after CAS II, Fig. 1)](UPLOAD: cas2_structure_function.png)

The curve always stays on or above the sufficiency line. The first point where it touches the line is the minimal sufficient statistic, and its $\alpha$-coordinate is the sophistication.

Arbitrary finite sets turn out to be too generous as models: some optimal ones secretly encode halting information and can never be found from data. Vereshchagin proposed restricting to *strong* models, those computable from $x$ by a total algorithm, and Milovanov observed that these are essentially the cells of simple partitions of $\{0,1\}^n$. So underneath algorithmic statistics sits a theory of **partitions as hypotheses**: a partition classifies every string, and the model for $x$ is its cell.

That raises the question the CAS series asks. Which partitions are *structured*? The most natural answer in mathematics and physics is symmetry: two strings are indistinguishable if some transformation in a group maps one to the other.

## CAS I: symmetries as programs

CAS I asks the lossless question. Take a symmetry to be a computable bijection of binary strings. If a symmetry has $x$ as its *only* fixed point, it pins $x$ down, so it can serve as a description of $x$. Now pick a symmetry program at random from a group $G$. How likely is it to isolate $x$?

Call that probability the **symmetry prior** $m_G(x)$. The main theorem is a geometric analogue of the coding theorem:

$$-\log_2 m_G(x) = K(x) + O(1)$$

It holds whenever $G$ is **fix-retractable**: there is a computable way to pick, for every string, a symmetry in $G$ that isolates it. Equivalently, the symmetry prior dominates the Solomonoff prior and is a universal lower semi-computable semimeasure.

To be upfront about credit: the underlying equivalence is due to Trejo, Kreinovich and Longpré, who showed that describing a string by a symmetry with that unique fixed point costs exactly its Kolmogorov complexity. CAS I recasts their result in the language of algorithmic probability and makes the group a parameter. That change of viewpoint buys two things:

- Symmetry becomes a *prior* that sits inside Solomonoff induction and can be compared with other priors.
- Each group gives its own complexity measure, and fix-retractability is exactly the condition under which that family collapses onto $K$. It isolates the point where group structure *could* make symmetry complexity differ from Kolmogorov complexity.

The limitation is also instructive. In the full group of computable bijections, every string is equally isolable: isolator sets of any two strings are conjugate. So CAS I measures how *cheaply $x$ can be singled out*, not how *symmetric $x$ is*. Isolators are lossless descriptions; they recover $K(x)$ and say nothing about structure. That is what CAS II fixes.

## CAS II: orbits as models

CAS II takes the lossy view. A group $U$ acting on strings sorts them into orbits, and the orbit containing $x$ is the model. The position of $x$ inside the orbit is noise. By orbit–stabiliser, the noise term is set by how much of the group fixes $x$:

$$\log|U\cdot x| = \log|U| - \log|\mathrm{stab}_U(x)| = -\log \Pr_{u\in U}[ux = x]$$

That is the precise sense in which symmetric objects are simple. A symmetry hypothesis explains $x$ well when it is cheap to describe and $x$ is fixed by a large part of it.

**The framework.** Different groups can have the same orbits, so the group is really a *certificate* that a partition comes from symmetry. A classical Galois connection between subgroups and partitions gives, for any ambient group $G$, a lattice of *symmetric partitions*. Each has a canonical certificate that costs exactly as much as the partition, and hypotheses combine by joins and meets. A set is a cell of some symmetric partition exactly when its setwise stabiliser acts transitively on it; call such sets *$G$-homogeneous*.

**Two new invariants: the $G$-structure function and $G$-sophistication.** A *$G$-model* for $x$ is a pair $(p, B)$: a symmetric partition $p$ together with its cell $B$ that contains $x$. The paper's central definitions restrict Kolmogorov's structure function and sophistication to these models:

$$\begin{aligned} h^G_x(\alpha) &= \min\{\log|B| : (p,B)\ \text{a } G\text{-model for } x,\ C(p,B)\le\alpha\} \\ \mathrm{soph}^G(x) &= \min\{\alpha : h^G_x(\alpha)+\alpha \le C(x)+O(\log n)\} \end{aligned}$$

There is also a strong version $h^G_{x,\varepsilon}$, which additionally requires the hypothesis itself to be cheap, $C(p) \le \varepsilon$. That is the version a search procedure can actually use, since a cheap $p$ maps $x$ to its cell by a short total program.

A few facts make these invariants usable:

- **They sit above the classical ones.** Always $h_x \le h^G_x \le h^G_{x,\varepsilon}$ and $\mathrm{soph}(x) \le \mathrm{soph}^G(x)$. So the gap $\mathrm{soph}^G(x) - \mathrm{soph}(x)$ measures exactly the structure in $x$ that $G$-symmetry misses.
- **Cells suffice.** $h^G_x$ is just Kolmogorov's structure function restricted to $G$-homogeneous sets, because naming such a cell already names a canonical hypothesis.
- **Groups and partitions agree.** Equivalently, $h^G_x(\alpha)$ is the smallest orbit size $\log|U\cdot x|$ over subgroups $U \le G$ that cost at most $\alpha$, so one can search over groups or over partitions.
- **Noise is a fixed-point probability.** For a $G$-model with certificate $U$, $|B| = 1/\Pr_{u\in U}[ux = x]$, and almost every string in $B$ is typical there.

**The search space.** This is where the RL framing becomes concrete. The *states* are symmetric partitions, with coordinates given by a group's type (a Burnside-ring element) and its placement (a permutation). A linear hypothesis is fixed by its type up to $n^2$ bits. The *moves* each have a bounded cost: relabelling by $g$ adds at most $C(g)$, a meet with a second hypothesis $q$ costs at most $C(q) + O(\log n)$ and shrinks the cell of $x$, and restriction to a subgroup refines partitions cell by cell via the Mackey formula. The *objective* is the strong $G$-structure function: find a cheap hypothesis whose cell of $x$ is small.

**Results, by choice of ambient group:**

- **All permutations of $\{0,1\}^n$.** Every partition is symmetric, so symmetry adds nothing: $h^{\mathrm{Sym}}_x = h_x$ and $\mathrm{soph}^{\mathrm{Sym}}(x) = \mathrm{soph}(x)$ for every string. Cells recover all Kolmogorov models, and cells of cheap partitions recover exactly the strong models. For complex models, almost all the information sits in *where* the symmetry is placed, not in its abstract type.
- **Linear symmetry, $\mathrm{GL}(n,2)$.** The cells are the *linearly homogeneous* sets, those whose XOR dependencies look the same from every point. For every nonzero $x$, the linear structure function lies in a band between $C(x) - \alpha$ and $n - \alpha$, and both edges are attained.
- **The maximal gap theorem.** There are stochastic, normal strings with $\mathrm{soph}(x) \approx 0$ but $\mathrm{soph}^{\mathrm{GL}}(x) \approx C(x)$. Their regularity is simple and even computable from $x$ by a short total program, yet no linear symmetry captures any of it.

![The structure functions of a typical normal string; in this figure κ = C(x) (after CAS II)](UPLOAD: cas2_symmetry_gap.png)

The shaded region is the symmetry gap: budget the best models don't need but linear symmetry does. The maximal gap theorem says this region can stretch all the way up to the ceiling $n - \alpha$.

This is the headline result of CAS II. It is a negative one, and I think it should interest people who think about inductive bias: *any space of symmetry hypotheses small enough to search is small enough to miss simple structure.* There are strings with essentially zero sophistication whose structure is completely invisible to linear symmetry.

The maximal gap proof is a counting argument. $\mathrm{GL}(n,2)$ has at most $2^{n^4}$ subgroups, so there are at most about $2^{n^4+n}$ candidate cells. A simple random-looking set $S$ can be chosen so that no cell intersects it much more than chance predicts, and typical elements of $S$ are then invisible to every cheap linear-symmetric model. Nothing in this is special to $\mathrm{GL}$: it applies to any family of $2^{\mathrm{poly}(n)}$ decidable cells.

There's an HTML edition of CAS II with interactive figures of cells, orbits, types and placements on the 3- and 4-cubes.

## Why LessWrong readers might care

**Symmetry is an inductive bias, and inductive biases have blind spots you can prove.** Equivariant architectures, conservation-law priors and "look for invariances" heuristics all bet that data's structure is symmetric. CAS II gives one way to quantify that bet. The gap $\mathrm{soph}^G(x) - \mathrm{soph}(x)$ measures how much simple structure a symmetry-based learner is structurally unable to see. The maximal gap theorem says this gap can be as large as possible, for strings that are otherwise as well-behaved as strings get.

**The searchability trade-off is general.** The slogan from CAS II is that *a coordinate system in which the space of symmetry hypotheses is small enough to search is small enough to miss simple structure.* For the full permutation group, search is complete but the hypotheses stop being meaningfully symmetric; you are searching arbitrary finite sets under another name. For matrix groups, search is feasible but blind. I suspect this pattern, completeness versus tractability of a structured hypothesis class, shows up well beyond symmetry.

**Failure to find a pattern is not evidence of randomness.** If a symmetric search finds no good model, the right conclusion is "$x$ has little symmetric structure", not "$x$ is random". The blind strings are stochastic and normal, with simple, computable sufficient statistics. I'd argue a symmetry-based method should report *how much* of the structure it explains, as a deficiency against a compressor's estimate of $C(x)$, rather than claiming to report "the structure".

**The common failure is blindness, not strangeness.** Strange, non-stochastic strings are rare. But a symmetric family has only $2^{\mathrm{poly}(n)}$ cells, so blindness to symmetry is plausibly common among ordinary stochastic strings. In practice the gap between symmetric and arbitrary models, not the gap between computable and uncomputable ones, looks like the dominant limitation.

**Where this is going.** Later papers in the series develop learned search over orbit models, first with reinforcement learning and then guided by LLMs. CAS II is meant to make that search well posed: types and placements give the state space, relabelling, restriction and meets give the moves, and the symmetric structure function gives the objective.

## Open problems and how to engage

Both papers end with open problems. The first few bear directly on the search programme; the rest are about the theory itself.

1. **Navigation.** Use the coordinates (type and placement) and the moves (restriction, relabelling, meets) to search the space of symmetric partitions for good models of a given string. Which moves are cheap, and how does the structure function change along them? This is the problem later papers attack with reinforcement learning. (CAS II)
2. **Halving by refinement.** Does every cell of a linear-symmetric partition admit a cheap refinement, by a meet with a cheap hypothesis or a restriction whose Mackey decomposition splits the cell of $x$? In search terms: does the agent always have a move that makes progress? If not, the linear structure function can plateau. (CAS II)
3. **Rigidity.** Is there a string with a good linearly homogeneous model and a good strong model, but no cheap symmetric partition? That would be a gap caused by the rigidity of the space of symmetric partitions rather than by computability, so the search space itself would hide good models. (CAS II)
4. **Other ambient groups.** Each ambient group gives a different search space. What do the affine group, coordinate permutations, or polynomial automorphisms see, and what is each blind to? Is there a group with a better trade-off between searchability and coverage? (CAS II)
5. **A coding theorem without a section.** When $G$ is not fix-retractable, does the symmetry prior still match the shortest isolator, to the extent that isolator sets are stochastic? Or is there an r.e. group where the two drift apart without bound? (CAS I)
6. **Explicit blind strings.** The maximal gap theorem is non-constructive. Can we write down explicit stochastic strings with $\mathrm{soph}^{\mathrm{GL}}(x) \approx C(x)$? Natural candidates are pairs $(y, f(y))$ for simple $f$ whose graph has neither large Walsh coefficients nor large linear symmetry groups. (CAS II)
7. **Realizability.** Which pairs of ordinary and linear-symmetric structure functions actually occur? The conjecture is that essentially every compatible pair does, so every value of $\mathrm{soph}^{\mathrm{GL}}(x)$ between $\mathrm{soph}(x)$ and $C(x)$ is attained. (CAS II)

I'm also interested in pushback on the framing. Is symmetry the right restricted class for strong-model search, or should an agent search a different hypothesis space? And if you work on RL for program synthesis or on LLM-guided search, I'd like to hear what a good state space and move set look like from your side.

**Links**

- CAS I: A Geometric Coding Theorem — [arXiv:2607.13796](https://arxiv.org/abs/2607.13796)
- CAS II: Orbits as Models. Kolmogorov's Structure Function under Symmetry — [interactive HTML edition](https://romiebanerjee.github.io/CAS2/index.html) · [arXiv:2609.40290](https://arxiv.org/pdf/2609.40290)
