## Hierarchical Tag Construction Methodology

This project builds a multi-level hierarchy of tags based on their co-occurrence patterns across projects. 
---

### Step 1: Compute the Weighted Degree of Tags

We first construct a weighted network of tags, where:
- Nodes represent tags
- Edge weights represent how frequently two tags co-occur across projects

The **weighted degree** of a tag is defined as the sum of all edge weights connected to that tag.

- **Example**: If *DeFi* appears across 50 projects alongside many other tags, while *Staking* appears in only 5 projects, *DeFi* will have a much higher weighted degree.
- **Significance**: Tags with the highest weighted degrees across the entire network are considered the most influential. These tags are selected as **Level 1 root tags**, as they act as primary hubs of information flow.

---

### Step 2: Global Community Detection (Louvain Method)

Instead of arbitrarily selecting a fixed number of top tags, we apply the **Louvain community detection algorithm** to the full tag network.

- **Modularity Optimization**: Louvain identifies groups of tags that are more densely connected to each other than to the rest of the network.
- ### Modularity Formula
The Louvain algorithm optimizes for **Modularity ($Q$)**, which measures the density of connections within communities compared to a random distribution:

$$Q = \frac{1}{2m} \sum_{ij} [A_{ij} - \frac{k_i k_j}{2m}] \delta(c_i, c_j)$$

Where:
* $A_{ij}$: The actual weight of the connection between Tag $i$ and Tag $j$.
* $k_i, k_j$: The sums of the weights of the edges attached to nodes $i$ and $j$ respectively.
* $m$: The total weight of all edges in the network.
* $\delta(c_i, c_j)$: The Kronecker delta function (1 if $i$ and $j$ are in the same community, 0 otherwise).
- **Objective Root Selection**: Within each detected community (e.g., an *Infrastructure* cluster), the tag with the highest weighted degree is designated as the **Level 1 parent**.
- **Result**: The data naturally decomposes into multiple independent communities. In our case, this process resulted in **8 distinct hierarchies**, each with its own Level 1 root tag.

---

### Step 3: Iterative “Peeling” and Re-Clustering

Once Level 1 root tags are identified, we iteratively extract deeper levels of the hierarchy through a peeling process.

1. **Remove**: All Level 1 root tags are temporarily removed from the network.
2. **Re-Cluster**: The remaining tags are re-analyzed without dominant root tags overshadowing smaller structures.
3. **Recalculate Degrees**: Weighted degrees are recomputed for the reduced network.
4. **Assign Level 2**: The highest-degree tags within the new sub-communities are assigned as **Level 2 parents**.

This process is repeated iteratively to uncover deeper hierarchical layers.

---

### Step 4: Lineage Tracking and Hierarchy Recombination

The algorithm tracks the lineage of every tag throughout the peeling process.

- **Sequence Tracking**: Each tag is assigned:
  - A **Level** (iteration number at which it was peeled)
  - An **ID** (its rank within that level)
- **Hierarchy Encoding**: These identifiers are concatenated to form a fixed-length hierarchical code.

**Example**:
- *Infra* peeled at Level 1 with ID `01`
- *Layer 1* peeled at Level 2 with ID `03`

A project containing only these two tags would receive the hierarchy code:01030000


