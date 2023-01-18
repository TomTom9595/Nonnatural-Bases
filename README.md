# Nonnatural Bases Project

## Used Sequences

without mismatches:

| Name        | Sequence    |
| ----------- | ----------- |
|X1           | CCATCACCCTCACCTTCCCTAAACCTCTCT  |
|F0 (upper)  | ACCCTCACCTTCCCTAAACCTCTCTCATCTACTAC |
|F0 (lower)  | AGATGAGAGAGGTTTAGGGAAGGTGAGGGTGATGG |
|F1 (upper)  | TCCCTAAACCTCTCTCATCTACTACAACCACAAAC |
|F1 (lower)  | TGGTTGTAGTAGATGAGAGAGGTTTAGGGAAGGTG |
|F2 (upper)  | TCTCTCATCTACTACAACCACAAACATACCAACAA |
|F2 (lower)  | GGTATGTTTGTGGTTGTAGTAGATGAGAGAGGTTT |
|R (upper)  | ACTACAACCACAAACATACCAACAA |
|R (lower)  | TTGTTGGTATGTTTGTGGTTGTAGTAGATG |

with mismatches:

| Name        | Sequence    |
| ----------- | ----------- |
|X1           | CCATCACCCTCACCTTCCCTAAACCTCTCT  |
|F0 mm (upper)  |ACCCTCATCTTCCCTAACCCTCTCTCAACTACTAC|
|F0 mm (lower)  |AGTTGAGAGAGGTTTAGGGAAGGTGAGGGTGATGG|
|F1 mm (upper)  |TCCCTAATCCTCTCTCACCTACTACAAACACAAAC|
|F1 mm (lower)  |TGTTTGTAGTAGATGAGAGAGGGTTAGGGAAGATG|
|F2 mm (upper)  |TCTCTCATCTACTACAACCACAAACATACCAACAA|
|F2 mm (lower)  |GGTATGTTTGTGTTTGTAGTAGGTGAGAGAGGATT|
|R (upper)  | ACTACAACCACAAACATACCAACAA |
|R (lower)  | TTGTTGGTATGTTTGTGGTTGTAGTAGATG |

## Placement of non natural bases
To figure out where we should place the nonnatural bases I am doing the following:
- Create lots of different strands with mostly random sequence but all including a complementary domain to the toehold and simulate interaction
- Take top 1% of interacting strands and place mismatches at each position of the toehold and simulate the interaction again

With this I would like to see the difference in ddG depending on the mismatch / non natural base position
