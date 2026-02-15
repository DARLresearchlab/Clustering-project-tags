# Hierarchical Tag Taxonomy Generator

This project implements a **data-driven algorithm** that transforms flat, co-occurring tags into a structured, multi-level hierarchy—analogous to industrial classification systems such as **NAICS** or **SIC**.

Rather than relying on manual categorization or heuristic rules, the system leverages **network science** and **community detection** to allow semantic structure and hierarchy to emerge directly from the data.

---

## Overview

The classification pipeline consists of **five sequential phases**, progressing from raw tag co-occurrence to a final **8-digit hierarchical taxonomy code**.

1. Co-occurrence Network Construction  
2. Community Detection (Louvain)  
3. Recursive Hub Peeling  
4. Structural Rank Termination (DM Logic)  
5. Project Mapping & Primary Identity Assignment  

Each phase incrementally refines structure while preserving interpretability and scalability.

---

# Algorithm Workflow

---

## Phase 1 — Co-occurrence Network Construction

We model the dataset as a **weighted, undirected graph**:

```
G = (V, E)
```

- **Nodes (V)** represent unique tags  
- **Edges (E)** connect two tags if they appear together  
- **Weights (W)** represent co-occurrence frequency  

This network representation captures both structural relationships and relationship strength.

---

## Phase 2 — Community Detection via Louvain Method

To identify **Major Sectors (Level 1)**, we apply the **Louvain community detection algorithm**.

Louvain optimizes for **Modularity (Q)**:

```
Q = (1 / 2m) * Σ_ij [ A_ij - (k_i k_j / 2m) ] δ(c_i, c_j)
```

Where:

- `A_ij` = weight between tags *i* and *j*  
- `k_i`, `k_j` = weighted degrees  
- `m` = total edge weight  
- `δ(c_i, c_j)` = 1 if same community, else 0  

### How Louvain Works

The method alternates between two stages:

**1. Local Movement Phase**

- Each tag starts in its own community  
- Tags move to neighboring communities if modularity increases  
- Stronger co-occurrence → stronger clustering pull  

**2. Community Aggregation Phase**

- Communities collapse into super-nodes  
- Edge weights are aggregated  
- Local movement repeats on reduced graph  

Implementation:python  partition = community_louvain.best_partition(sub, weight="weight", random_state=42)

**Why Louvain?**

- No predefined number of clusters required  
- Fully data-driven  
- Adapts naturally as new data is added  
- Preserves cross-domain relationships  

---

## Phase 3 — Recursive Hub Peeling

Once Level 1 communities are identified, hierarchical depth is extracted via **hub peeling**.

**Step 1 — Identify the Hub**

- Select the tag with highest weighted degree  
- This tag becomes the parent at that level  

**Step 2 — Assign Code**

- Parent receives a new 2-digit segment (e.g., `01`)  

**Step 3 — Peel and Re-cluster**

- Remove the parent from the network  
- Re-run clustering on remaining tags  
- Identify next-level hubs  

This process repeats recursively.

> This prevents highly frequent tags (e.g., Ethereum) from obscuring broader structural concepts.

---

## Phase 4 — Termination via Structural Rank (DM Logic)

To avoid artificial hierarchies in tightly connected groups, recursion terminates using **Structural Rank**, derived from **Dulmage–Mendelsohn decomposition**.

At each recursion step:

- Compute adjacency matrix of remaining subgraph  
- Evaluate separability using `is_separable()`  

**Stopping Condition**

```
rank(Adj) < |V|
```

If true:

- The group is structurally inseparable  
- All remaining tags are assigned at current level  
- Recursion stops for that branch  

---

## Phase 5 — Project Mapping & Primary Identity Assignment

Projects often contain multiple tags spanning different branches.

The algorithm assigns a single **Primary Identity** using:

### 1. Cluster Voting

- Each tag votes for its Level 1 cluster  
- Votes weighted by global weighted degree  

### 2. Winning Cluster Selection

- Cluster with highest total vote share wins  

### 3. Specificity Rule

- Within winning cluster, choose tag with deepest hierarchy  
- Depth = number of non-zero 2-digit segments  

### 4. Final Assignment

- Project inherits full 8-digit taxonomy code  

Example:

```
01030200
```

---

# Taxonomy Code Structure

Each taxonomy code consists of **8 digits**.

Every 2-digit segment represents a deeper hierarchical level.

| Code       | Level | Meaning        | Example                 |
|------------|-------|---------------|-------------------------|
| 01000000   | 1     | Major Sector  | DeFi                    |
| 01030000   | 2     | Sub-Sector    | Lending                 |
| 01030200   | 3     | Niche         | Undercollateralized     |
| 01030201   | 4     | Specific      | Flash Loans             |

---

# Design Principles

- Fully data-driven  
- No manual taxonomy seeding  
- Scalable to large networks  
- Interpretable hierarchical codes  
- Robust structural stopping criteria  
- Evolves naturally with new data  

---

# Output

The system produces:

- Hierarchical tag tree  
- 8-digit taxonomy codes  
- Project → primary classification mapping  
- Sector-level aggregation capability  

---

# Summary

This project converts flat tag co-occurrence into a structured, recursive, interpretable taxonomy using:

- Network modeling  
- Louvain modularity optimization  
- Recursive hub peeling  
- Structural rank termination  
- Weighted identity assignment  

The result is a scalable, data-native classification system capable of evolving with the dataset itself.


