# PÇYS Calculation Specification v1.0

**Specification ID:** `PCYS-CALC-SPEC-1.0`  
**Canonical kernel:** `pcys.kernel.attainment.CalculationKernel`  
**Kernel version:** `ATT-KERNEL-1.0`

This specification defines the numerical calculation contract. The kernel is deliberately independent of PyQt6, SQLite, Excel/PDF, filesystem state and GUI workflows.

## 1. Input domain

For student `i`, question `j`, program outcome `k` and assessment component `c`:

- `q_ij`: raw question score.
- `q_j^max`: question normalization maximum. In the current legacy-compatible method this is the observed class maximum, with zero replaced by 1.
- `w_j`: question percentage weight used by the assessment mapping.
- `m_jk`: question-to-program-outcome mapping coefficient.
- `a_c`: assessment-component weight in the course grade/PÇ aggregation.

All official attainment outputs are expressed on a 0–100 scale.

## 2. Question normalization — `ATT-001`

\[
q_{ij}^{norm}=\frac{q_{ij}}{q_j^{max}}
\]

A zero normalization maximum is replaced by 1 to prevent division by zero.

## 3. Component-level program-outcome attainment — `ATT-002`

\[
PC_{ick}=100\frac{\sum_j q_{ij}^{norm} w_j m_{jk}}
{\sum_j w_jm_{jk}}
\]

If the denominator is zero, the component contribution for that PÇ is zero and the outcome is considered unmeasured in that component.

## 4. Course PÇ aggregation — `ATT-003`

\[
PC_{ik}=\sum_c a_c PC_{ick}
\]

Component weights are not silently renormalized. A total other than 1 is a health-check finding.

## 5. Course grade — `GRADE-001`

For each component:

\[
G_{ic}=100\frac{\sum_j q_{ij}}{M_c}
\]

where `M_c` is the configured component full score. Course grade:

\[
G_i=\sum_c a_c G_{ic}
\]

## 6. Resit replacement — `RESIT-001`

If a student's resit question-score sum is greater than zero, the resit vector replaces the final-exam vector for that student. Questions present in the final but absent from the resit are treated as zero.

## 7. PÇ threshold — `THR-001`

\[
T_k=\frac{L\,A\,D_k}{P_k}
\]

where:

- `L`: semester lecture-result parameter,
- `A`: course AKTS,
- `D_k`: weighted course contribution (`PÇ_Ders`),
- `P_k`: catalog `Total_PÇ`.

If `P_k <= 0` or `D_k <= 0`, the threshold is zero and the configuration/health layer must flag the condition as appropriate.

## 8. Decision contract

For a measured outcome:

- attainment `>= threshold` → successful,
- attainment `< threshold` → insufficient.

An outcome with zero measured course contribution is **out of scope**, not automatically successful.

## 9. Reproducibility contract

Each official analysis may be assigned a deterministic Result ID computed from:

1. course definition hash,
2. semester-data hash,
3. rule-pack hash,
4. method-registry bundle hash,
5. software version,
6. numerical result hash.

The same complete material must produce the same Result ID.

## 10. Numerical tolerance

Reference-vector comparisons use an absolute tolerance of `1e-10` unless a test explicitly documents a looser tolerance.

## 11. Compatibility policy

Any modification that changes a reference vector requires:

- a new method identifier/version,
- an explicit specification revision,
- migration/release notes,
- preservation of old locked snapshot interpretation.
