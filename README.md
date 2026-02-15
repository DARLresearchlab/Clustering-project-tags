# Hierarchical Tag Taxonomy Generator

This project implements a **data-driven algorithm** that transforms flat, co-occurring tags into a structured, multi-level hierarchy—analogous to industrial classification systems such as **NAICS** or **SIC**.

Rather than relying on manual categorization or heuristic rules, the system leverages **network science** and **community detection** to allow semantic structure and hierarchy to emerge directly from the data.

---

## Algorithm Workflow

The classification pipeline consists of **five sequential phases**, progressing from raw tag co-occurrence to a final **8-digit hierarchical taxonomy code**. Each phase incrementally refines structure while preserving interpretability and scalability.

---

### Phase 1: Co-occurrence Network Construction

We begin by modeling the dataset as a **weighted, undirected graph**:

\[
G = (V, E)
\]

- **Nodes (V)** represent unique tags.
- **Edges (E)** connect two tags if they appear together in the same project.
- **Weights (W)** represent how frequently two tags co-occur across all projects.

This network representation captures both the structure and strength of relationships between tags, forming the foundation for all subsequent analysis.

---

### Phase 2: Community Detection via Louvain Method

To identify **Major Sectors (Level 1)**, we apply the **Louvain community detection algorithm** to the full tag network.  
Louvain optimizes for **Modularity (Q)**, which measures how densely connected nodes are within a community relative to a random null model.

#### Modularity Formula

$$
Q = \frac{1}{2m} \sum_{ij} \left[ A_{ij} - \frac{k_i k_j}{2m} \right] \delta(c_i, c_j)
$$

Where:
- `A_ij` is the weight of the edge between tags *i* and *j*
- `k_i, k_j` are the weighted degrees of nodes *i* and *j*
- `m` is the total weight of all edges in the network
- `δ(c_i, c_j)` equals 1 if *i* and *j* belong to the same community, and 0 otherwise

**Outcome**  
Tags naturally partition into coherent, high-level clusters (e.g., *DeFi*, *Infrastructure*) without any manual intervention.

---

#### How Louvain Works in This Implementation

In this system, the Louvain method is used as a **greedy, data-driven mechanism** to discover macro-level structure directly from tag co-occurrence patterns.

At a high level, Louvain operates through two repeating internal stages:

1. **Local Movement Phase**  
   - Each tag initially forms its own community.
   - Tags are repeatedly moved between neighboring communities if the move increases global modularity.
   - Because edge weights represent co-occurrence frequency, tags that frequently appear together exert stronger “pull” toward the same community.

2. **Community Aggregation Phase**  
   - Once no further local improvements are possible, each detected community is collapsed into a “super-node.”
   - Edge weights between communities are aggregated.
   - The local movement phase is repeated on this reduced graph.

These two stages iterate until modularity can no longer be improved.

In the code, this process is executed via:

```python
partition = community_louvain.best_partition(sub, weight='weight', random_state=42)

**Why Louvain Is Appropriate Here**
   - Tags frequently co-occur across many projects → strong intra-community density
   - Cross-domain tags remain connected but do not dominate clustering
   - No predefined number of sectors is required
   - Structure adapts naturally as new data is added

As a result, Level-1 communities correspond to emergent functional or economic sectors, rather than manually imposed categories

---

### Phase 3: Recursive “Hub Peeling”

Once Level 1 communities are identified, hierarchical depth is extracted through an iterative **hub peeling** process.

1. **Identify the Hub**  
   - Within each community, the tag with the highest **weighted degree** is selected as the parent for that level.
2. **Assign Code**  
   - The parent tag receives a new 2-digit identifier (e.g., `01`).
3. **Peel and Re-cluster**  
   - The parent is removed from the network.
   - The remaining tags are re-clustered to identify sub-hubs at the next hierarchical depth.

This process is repeated recursively, revealing increasingly fine-grained structure.

> This design prevents highly frequent tags (e.g., *Ethereum*) from obscuring more general concepts (e.g., *Layer 1*), even when they frequently co-occur.

---

### Phase 4: Termination via Structural Rank (DM Logic)

To avoid creating artificial hierarchies in small or tightly connected tag groups, recursion is terminated using **Structural Rank**, derived from **Dulmage–Mendelsohn decomposition**.

- At each recursion step, the adjacency matrix of the remaining tag subgraph is evaluated using the function `is_separable`.
- **Stopping Condition**:rank(Adj) < |V|


If this condition holds, the group is treated as a cohesive block that cannot be meaningfully decomposed further.

**Result**  
All remaining tags are assigned to the current hierarchy level, and recursion stops for that branch.

---

### Phase 5: Project Mapping and Primary Identity Assignment

Because individual projects often contain multiple tags spanning different hierarchy branches, the algorithm assigns each project a single **Primary Identity**.

- **Cluster Voting**  
  - Each project’s tags vote for their corresponding Level 1 clusters.
  - Votes are weighted by each tag’s global weighted degree.
- **Winning Cluster Selection**  
  - The cluster with the highest total vote share is selected.
- **Specificity Rule**  
  - Within the winning cluster, the tag with the deepest hierarchy (i.e., the most non-zero digit pairs) is chosen.
- **Final Assignment**  
  - The project inherits the full 8-digit taxonomy code of that tag.

**Example**
01030200

---

## Taxonomy Code Structure

Each taxonomy code consists of **8 digits**, where every 2-digit segment represents a deeper hierarchical level.

| Code       | Level | Meaning               | Example              |
|------------|-------|-----------------------|----------------------|
| 01000000   | 1     | Major Sector (Hub)    | DeFi                 |
| 01030000   | 2     | Sub-Sector            | Lending              |
| 01030200   | 3     | Niche                 | Undercollateralized  |
| 01030201   | 4     | Specific Detail       | Flash Loans          |




