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

The problem with this is, that the random strand could create a different secondary strucutre to form a basepair with the "mismatch" which is not possible with non-natural bases, however it should give us an idea. However, it is not trival for me, how I can simulate interactions without having this problem. (Just deleting the base is also not a good option)



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

The following table shows how many bases of the toehold are occluded for the original sequence of F2 (upper) and the strands where I introduced another base at a certain position of the toehold. I simulated interaction with 100,000 strands of a random pool of length 50 (N50) and took the 1,000 strands with the strongest interaction for further analysis.

The code I use to count the occluded bases of the toehold:

```python
with open("structures.txt") as structures:
    lines = structures.readlines()
    
occluded = []

for i in lines:
    toe = str(i[81:86])
    occluded.append(toe.count(')'))
structures.close()
```


The position of the mismatch is counted begging from the 3' end.
Even though this is definitely not the best way for simulating the interaction with a non-natural base at a certain position, it supports the naive approach to place the non-natural base in the middle of the toehold (position 3) as this lowers the amount of random pool strands that bind 4 or 5 bases (in reality 5 is no longer possible with the non natural base). There is also an argument for positon 4, as it shows the highest proportion of strands with a completely free toehold. I actually do not see why this is the case and have to go into more detail.

|          | Zero | One | Two | Three | Four | Five |
| -------- | ---- | --- | --- | ----- | ---- | ---- |
| Original | 645  | 29  | 83  | 161   | 41   | 41   |
| MM pos 1 | 646  | 28  | 92  | 190   | 15   | 29   |
| MM pos 2 | 654  | 35  | 70  | 179   | 62   | 0    |
| MM pos 3 | 756  | 41  | 115 | 60    | 21   | 7    |
| MM pos 4 | 824  | 0   | 85  | 43    | 33   | 15   |
| MM pos 5 | 766  | 76  | 64  | 52    | 37   | 5    |

![image](https://user-images.githubusercontent.com/110489104/213924436-8632ff0e-01d6-4f46-8463-bfbcb5752843.png)

I think that we should first try one version of the F2 strand to directly invade the reporter (maybe position 3) and how it behaves with the random pool (different concentrations, lengths and pre-equilibrated, as well as a direct start after mixing everything together) as it is much easier to analayze a one-step reaction. However, we would have to get the reporter with a non-natural base, too.
I guess for the whole cascade it would be best, to use the non-natural base in the step which takes the longest, as the random strands have "more time to interact with the invader" and find good matches. Using non-natural bases in each step could be very expensive I guess

## Experimental Considerations
### Nonnatural Base-pair to use:
    1. Based on what what the Marchand Lab can/will do
    2. isoC/isoG pair (orthogonak hydrogen bonding pattern to standard base pairs)
        - https://academic.oup.com/nar/article/32/6/1937/1111542
        - https://www.idtdna.com/site/Catalog/Modifications/Product/2039
