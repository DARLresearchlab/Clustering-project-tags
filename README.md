# Hierarchical Tag Taxonomy Generator

This project implements a **data-driven algorithm** to transform flat, co-occurring tags into a structured, multi-level hierarchy (similar to **NAICS** or **SIC** codes).

Instead of manual categorization, the system leverages **Network Science** and **Community Detection** to allow the data itself to define semantic structure and relationships.

---

## Algorithm Workflow

The classification pipeline consists of **five phases**, progressing from raw tag co-occurrence to a final **8-digit hierarchical taxonomy code**.

---

### Phase 1: Co-occurrence Network Construction

The dataset is modeled as a **weighted undirected graph**:

\[
G = (V, E)
\]

- **Nodes (V)**: Every unique tag in the dataset  
- **Edges (E)**: An edge exists between two tags if they appear together in the same project  
- **Weights (W)**: Edge weight equals the frequency of tag co-occurrence  

This representation captures both tag relationships and their relative strength.

---

### Phase 2: Community Detection via Louvain Method

To identify **Major Sectors (Level 1)**, we apply the **Louvain community detection algorithm**, which optimizes for **Modularity (Q)**.

#### Modularity Formula

$$Q = \frac{1}{2m} \sum_{ij} [A_{ij} - \frac{k_i k_j}{2m}] \delta(c_i, c_j)$$

Where:
- `A_ij`: Weight of the edge between tag *i* and *j*
- `k_i, k_j`: Sum of edge weights connected to nodes *i* and *j*
- `m`: Total weight of all edges in the network
- `δ(c_i, c_j)`: 1 if tags belong to the same community, 0 otherwise

**Outcome**:  
Tags naturally group into organic clusters (e.g., *DeFi*, *Infrastructure*) without any manual rules.

---

### Phase 3: Recursive “Hub Peeling”

Once a community is identified, hierarchical structure is extracted through an iterative **peeling process**:

1. **Identify the Hub**  
   - The tag with the highest **weighted degree** is selected as the parent for that level.
2. **Assign Code**  
   - The hub receives a new 2-digit identifier (e.g., `01`).
3. **Peel & Re-cluster**  
   - The hub is removed from the cluster.
   - Remaining tags are re-clustered to identify sub-hubs.

> This ensures that general concepts (e.g., *Layer 1*) are not overshadowed by highly frequent tags (e.g., *Ethereum*).

---

### Phase 4: Termination via Structural Rank (DM Logic)

To prevent overfitting artificial hierarchies in small or tightly connected groups, recursion is terminated using **Structural Rank**, based on **Dulmage–Mendelsohn decomposition**.

- The function `is_separable` evaluates the adjacency matrix of the tag subgraph.
- **Stopping Condition**:
  
 rank(Adj) < |V|


If the condition holds, the group is considered a cohesive block that cannot be meaningfully decomposed further.

**Result**:  
Remaining tags are assigned to the current hierarchy level, and recursion stops.

---

### Phase 5: Project Mapping & Primary Identity Assignment

Because projects often contain multiple tags, the algorithm assigns each project a single **Primary Identity**.

- **Cluster Vote**  
  - All project tags vote for their Level 1 cluster.
  - Votes are weighted by global tag degree.
- **Winning Cluster**  
  - The cluster with the highest total vote share is selected.
- **Specificity Rule**  
  - The tag with the deepest hierarchy (most non-zero digit pairs) is chosen.
- **Final Code**  
  - The project inherits the tag’s full 8-digit taxonomy code.

**Example**:
01030200

---

## Taxonomy Code Structure

Each taxonomy code consists of **8 digits**, where every 2 digits represent a deeper hierarchical level.

| Code       | Level | Meaning               | Example              |
|------------|-------|-----------------------|----------------------|
| 01000000   | 1     | Major Sector (Hub)    | DeFi                 |
| 01030000   | 2     | Sub-Sector            | Lending              |
| 01030200   | 3     | Niche                 | Undercollateralized  |
| 01030201   | 4     | Specific Detail       | Flash Loans          |

---

## How to Run

1. Set `INPUT_FILE`  
   - CSV containing project titles and tags (`Level_1`, `Level_2`, etc.)
2. Define `OUTPUT_FOLDER`
3. Run:
```bash
python taxonomy_generator.py



