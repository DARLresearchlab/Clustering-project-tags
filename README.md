## Hierarchical Tag Construction Methodology

This project builds a multi-level hierarchy of tags based on their co-occurrence patterns across projects. The hierarchy is derived entirely from network structure and community detection, without relying on arbitrary thresholds or manual labeling.

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
- **Objective Root Selection**: Within each detected community (e.g., an *Infrastructure* cluster), the tag with the highest weighted degree is designated as the **Level 1 parent**.
- **Result**: The data naturally decomposes into multiple independent communities. In our case, this process resulted in **9 distinct hierarchies**, each with its own Level 1 root tag.

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

A project containing both would receive the hierarchy code:

This encoding preserves both hierarchical depth and parent-child relationships.

---

### Step 5: Handling Temporal or Repetitive Tags

Temporal or repeated concepts (e.g., treatments across different time periods) are treated as distinct tags.

- Tags such as *Treatment A (Q1)* and *Treatment A (Q2)* are modeled independently.
- If these tags exhibit different co-occurrence patterns, the algorithm will naturally place them in different sub-clusters or hierarchy levels.
- This preserves time-sensitive structure without requiring manual temporal ordering or rules.

---

