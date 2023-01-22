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
- Create lots of different strands with random sequence and simulate interaction.
- Take top 1% of interacting strands and place mismatches at each position of the toehold and simulate the interaction again.

The problem with this is, that the random strand could create a different secondary strucutre to form a basepair with the "mismatch" which is not possible with non-natural bases, however it should give us an idea, as I filtered the best matches before which tend to not alter their overall interaction with the invader strand.



The code I use to create random strands:

```python
import random
Bases = ["A","C","G","T"]
with open('N50.txt', 'a') as fp:
    for j in range(0,100000):
        for i in range(0,50):
            fp.write(str(random.choice(Bases)))
        fp.write("\n")
    fp.close()
```

The code I use for analysis:

```python
# Header
from nupack import *
import numpy as np
config.parallelism = True

Model1 = Model(material='dna04', ensemble='some-nupack3', celsius=25, sodium=0.05, magnesium=0.0125) #Define model used for NUPACK calculations
```


```python
# Import strands
Strands_list = []
Interfering = open("N50.txt")
for line in Interfering:
    Strands_list.append(line.rstrip())
Interfering.close()
```


```python
# Define strands/complexes and tasks 
F2upper_Strand = Strand('TCTCTCATCTACTACAACCACAAACATACCAACAA', name='F2upper')
Interfering_free_energy_list = []
results = open("Results.txt", 'a')
for i in range(len(Strands_list)):
    Interfering_Strand = Strand(Strands_list[i], name ='Interfering_Strand')
    Interfering_F2upper_Complex = Complex([Interfering_Strand, F2upper_Strand], name='Interfering_F2upper_Complex')
    Interfering_Complex = Complex([Interfering_Strand], name='Interfering_Complex')
    F2upper_Complex = Complex([F2upper_Strand], name='F2upper_Complex')
    InteractionTube = Tube({F2upper_Strand: 10e-9, Interfering_Strand: 1e-6}, complexes=SetSpec(include=[Interfering_F2upper_Complex, Interfering_Complex, F2upper_Complex]), name='InteractionTube')
    my_results = tube_analysis([InteractionTube], model=Model1, compute=['mfe'])
    ddG = my_results[Interfering_F2upper_Complex].free_energy - my_results[F2upper_Complex].free_energy - my_results[Interfering_Complex].free_energy
    results.write(str(Interfering_Strand) + '\n' +str(my_results[Interfering_F2upper_Complex].free_energy) + '\n' +str(ddG) +'\n'+ str(my_results[Interfering_F2upper_Complex].mfe[0].structure) +'\n'+'\n')
results.close()
```


```python
# Create one list only containing the ddG and one for the secondary structure
with open("Results.txt") as Results:
    lines = Results.readlines()
ddGList = lines[2::5]
structuresList = lines[3::5]

ddG = open("ddG.txt", 'a')
structures = open("structures.txt", 'a')

for i in ddGList:
    ddG.write(str(i))
ddG.close()

for i in structuresList:
    structures.write(str(i))
structures.close()
```
