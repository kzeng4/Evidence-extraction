```text
Algorithm: Multi-Section Paragraph-Gated CONSORT Extraction

Input:
    Article A represented as article → paragraph → sentence JSON
    Optimized or base section predictor C
    Enabled section extractors {E_s}: currently Introduction and Methods
    Gold paragraph section labels G and gold item-sentence pairs Y for optimization

Output:
    Merged CONSORT item-sentence relations Ŷ = {(item, sid)}

1. Predict paragraph sections once:
       Ĝ ← C(A)

2. For each enabled section s:
       Initialize routed paragraph set P_s ← ∅
       For each paragraph p_i:
           true_has_s ← s ∈ G_i
           pred_has_s ← s ∈ Ĝ_i

           If true_has_s and pred_has_s:
               Add p_i to P_s
           If true_has_s and not pred_has_s:
               Add paragraph-specific feedback to the section predictor
           If not true_has_s and pred_has_s:
               Add over-routing feedback to the section predictor

3. For each enabled section s:
       Run section-specific extractor:
           Ŷ_s ← E_s(P_s)
       Generate section-specific extraction feedback using only paragraphs
       that passed the target-section gate.

4. Merge all section outputs:
       Ŷ ← Deduplicate( ⋃_s Ŷ_s ) using item-sid pairs

5. Evaluate final extraction:
       TP ← Ŷ ∩ Y
       FP ← Ŷ \ Y
       FN ← Y \ Ŷ
       Compute precision, recall, and F1 over merged item-sid pairs.

6. Optimize with separated feedback:
       Routing errors update the section predictor.
       Section-specific extraction errors update the corresponding section extractor/prototype.
