# TFforge

TFforge (Transcription factor forward genomics) is a method to assiciate transciption factor binding site divergence in putative regulatory elements with pehnotypic changes between species [1].

# Requirements:
* Python3 with Bio and rpy2 (requires R)
* Stubb_2.1 [2]
* tree_doctor [3]. A linux 64 bit binary is included. The source code is available at https://github.com/CshlSiepelLab/phast/ in src/util/

# Installation:

`git clone https://github.com/hillerlab/TFforge`

Download Stubb (http://www.sinhalab.net/software) and newmat11 (http://www.robertnz.net/download.html)
```
cp /path/to/stubb_2.1.tar.gz /path/to/newmat11.tar.gz .
tar -xvf stubb_2.1.tar.gz
tar -xvf newmat11.tar.gz -C stubb_2.1/lib/newmat/

# TFforge uses a slightly modified version of Stubb
cd stubb_2.1/
patch -p1 < ../TFforge/stubb.patch
cd lib/newmat/
gmake -f nm_gnu.mak
cd ../../
make
export PATH=$PATH:`pwd`/bin
cd ../TFforge/
export PATH=$PATH:`pwd`
```

# Example of screening 20 simulated CREs
```
# this directory contains a minimal example of simulated CREs
cd example

# Create a joblist file 'alljobs_simulation' containing the branch_scoring jobs
TFforge.py data/tree_simulation.nwk motifs.ls data/species_lost_simulation.txt elements_simul.ls \
  --windowsize 200 --scorefile scores_simulation -bg=background/

# Run job list as batch or in parallel 
bash alljobs_scores_simulation > scores_simulation

# Run the association test
TFforge_statistics.py data/tree_simulation.nwk motifs.ls data/species_lost_simulation.txt \
  elements_simul.ls --scorefile scores_simulation
# This generates a file 'significant_elements_simulation' that alphabetically lists the elements, their P-value and the number of branches
```

# General workflow
## Input data
- species tree in newick format
- motif files in wtmx format (see example/data/ACA1.wtmx as example) which is: 
> \>*motif_name*	*motif_length*	[optional fields]  
> *pos1_freqA*	*pos1_freqC*	*pos1_freqG*	*pos1_freqT*  
> *pos2_freqA*	*pos2_freqC*	*pos2_freqG*	*pos2_freqT*  
> ..  
> *posN_freqA*	*posN_freqC*	*posN_freqG*	*posN_freqT*  
> \<  
- motif list: path of wtmx files of one TF per line (see example/motifs.ls)
- Phenotype-loss species list: one species per line (see example/data/species_lost_simulation.txt)
- CRE fastafiles: each file contains the sequence for every (ancestral or extant) species
- CRE list: path of fastafile of one CRE per line

## Step 1: Branch score computation
Generate the TFforge branch_scoring commands for all CREs and TFs.
```
TFforge.py <tree> <motif_list> <lost_species_list> <element_list>
```
This creates for every CRE and every TF a TFforge_branch_scoring.py job. Each line in alljobs_\<scorefile\> consists a single job. Each job is completely independent of any other job, thus each job can be run in parallel to others.

Execute that alljobs file. Either sequentially via
```
bash alljobs_simulation > scores_simulation
```
or run it in parallel by using a compute cluster.
Every job returns a line in the following format:  
> *motif_file*	*CRE_file*	(*branch_start*>*branch_end*:*branch_score*	)*	<  

which should be concatenated into a file called "\<scorefile\>".

## Step 2: Association test
```
TFforge_statistics.py <tree> <motif_list> <lost_species_list> <element_list>
```
TFforge_statistics.py classifies branches into trait-loss and trait-preserving and assesses for every TF the significance the association of this classification with the branch scores of all CREs.

## Common Parameters
#### TFforge.py
```
--scorefile <name>
Name file <name> instead of "scores"
--windowsize/-w <n>
Scoring window used in sequence scoring
--background/-bg <folder>
Background used for sequence scoring. Either a file or a folder structure with backgrounds for different GC contents
--scrCrrIter <n>
Stubb score is corrected with the average score of <number> of shuffled sequences. Default is 10. 0 turns the score correction off
```
#### TFforge_statistics.py
```
--scorefile <name>
Use <name> as scorefile instead of "scores"
--filterspecies <comma separated list>
Exclude species from analyses
--elements <file>
Analyse only the elements specified <file>
```

## Special parameters
```
--verbose/-v
--debug/-d
```

####TFforge.py
```
--no_ancestral_filter
Turn of ancestral score filtering
--no_branch_filter
Turn of branch score filtering
--no_fixed_TP
Do not fix transition probabilites while computing branch scores
--filter_branch_threshold <x>
Threshold for branch filter; By default branches are filtered if start and end node score are below 0
--filter_GC_change <x>
Filter branches with a GC content change above <x>
--filter_length_change <x>
Filter branches with a relative length change above <x>
```
 
# References
[1] Langer BE, Hiller M. TFforge utilizes large-scale binding site divergence to identify transcriptional regulators involved in phenotypic differences. 

[2] Sinha S, van Nimwegen E, Siggia ED. A Probabilistic Method to Detect Regulatory Modules. Bioinformatics, 19(S1), 2003

[3] Hubisz MJ, Pollard KS, Siepel A. PHAST and RPHAST: Phylogenetic Analysis with Space/Time Models. Briefings in Bioinfomatics 12(1):41-51, 2011.
