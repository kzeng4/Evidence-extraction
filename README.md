```text
Algorithm: Paragraph-Gated Section-Specific CONSORT Extraction

Input:
    Article A = {p1, ..., pn}, where each paragraph pi contains sentence IDs
    Target section q = Methods
    Optimized section predictor C
    Methods-specific extractor E_q
    Gold section labels G and gold item-sentence pairs Y

Output:
    Predicted CONSORT item-sentence pairs Ŷ

1:  Initialize selected paragraph set p ← ∅
2:  Initialize section feedback F_sec ← ∅
3:  Initialize extraction feedback F_ext ← ∅

4:  Predict paragraph sections Ĝ ← C(A)

5:  for each paragraph p_i ∈ A do
6:      true_has_section ← q ∈ G_i
7:      pred_has_section ← q ∈ Ĝ_i

8:      if true_has_section and not pred_has_section then
9:          Add p_i to F_sec as a missed Methods-routing error
10:         continue

11:     else if not true_has_section and pred_has_section then
12:         Add p_i to F_sec as an over-routing error
13:         continue

14:     else if true_has_section and pred_has_section then
15:         Add p_i to P_q
16:     end if
17: end for

18: Run Methods extractor Ŷ_q ← E_q(P_q)

19: Evaluate extraction using item-sentence pairs (item, sid)
20: TP ← predicted pairs that match gold pairs
21: FP ← predicted pairs not in gold
22: FN_ext ← gold pairs in correctly routed paragraphs but not predicted
23: FN_sec ← gold pairs blocked by Step-1 section errors

24: if F_sec is not empty then
25:     Generate feedback for the section predictor
26: end if

27: if FP or FN_ext is not empty then
28:     Generate Methods-specific feedback for the extractor
29:     Update mutable Methods prototype rules
30: end if

31: Compute precision, recall, and F1
32: return Ŷ_q, F_sec, F_ext
```
