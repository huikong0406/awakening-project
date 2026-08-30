

# Layer 0 Specification: Entry-Boundary & Population Manifest

This document formalizes the **Entry-Boundary Check** required for deterministic execution graphs and automated approval workflows.

## Problem Statement: The Empty-Set Trap

A deterministic process graph (predefined nodes, explicit routing conditions, and automated archiving) guarantees **execution integrity**: 
> *If an payload enters Node 1, it will be routed according to deterministic rules.*

However, deterministic execution graphs are **structurally blind to input population drift**. 

If an upstream collector fails, drops records, or returns an empty set ($N = 0$), every node condition will execute without throwing errors, pass physical tests, and produce a **clean audit trail over a hollow or missing population**.

[ Upstream Collector ] ──( Silent Failure: 0 rows )──> [ Node 1: Submit ]
│
▼
[ Node 8: Archive (PASS) ]
⭐ Clean Audit Trail / Zero Data


## Mandatory Verification Metric: `eligible_seen`

To prevent "verification theater" on starved inputs, any deterministic workflow must decouple **Process Audit** from **Population Audit**.

Every entry point must log and verify the following invariant:

$$ coverage\_gap = eligible\_seen - population\_size $$

Where:
* **`eligible_seen`**: The total count of entities matching the selection query at the source database/collector level before filtering or routing.
* **`population_size`**: The number of entities that actually arrived at Node 1 of the workflow.

### Formal Rules:
1. **$coverage\_gap = 0$**: Complete coverage. Process proceeds with execution.
2. **$coverage\_gap > 0$**: Partial drop. Workflow MUST trigger an `INCONCLUSIVE` or `POPULATION_DRIFT` state.
3. **`eligible_seen` $= 0$ while expected $> 0$**: Upstream collector fault. Execution MUST NOT yield an automated `PASS`.

## Real-World Case Study

> **Production Bug (Tom Jones / OWP Thread):**  
> 1,106 records matched the selection criteria at the database level, but only 193 were delivered to the execution pipeline. Worst-case incident: 39 matched, 0 delivered.  
> 
> The routing logic was 100% deterministic. The execution logs were completely clean. Without tracking `eligible_seen` at the boundary, the system generated a perfect audit trail of silent drop failures.

## Two Core System Questions

Any verifiable system must answer two distinct questions independently:
1. **Did the process run correctly?** *(Handled by Node Routing & Conditions)*
2. **Did the right population reach the process at all?** *(Handled by Population Manifest & `eligible_seen`)*

---
*Contributed in the spirit of open protocol design. Concepts derived from OWP community
