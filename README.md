# TFforge

TFforge (Transcription factor forward genomics) is a method to assiciate transciption factor binding site divergence in putative regulatory elements with pehnotypic changes between species.

# Requirements:
Python3 with Bio and rpy2 (requires R)
Stubb_2.1 (see below)
tree_doctor (binary included; source code from https://github.com/CshlSiepelLab/phast/ in src/util/)

# Preparation:
Download Stubb (http://www.sinhalab.net/software) and newmat11 (http://www.robertnz.net/download.html)
```
git clone https://github.com/hillerlab/TFforge
tar -xvf stubb_2.1.tar.gz
tar -xvf newmat11.tar.gz -C stubb_2.1/lib/newmat/
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

# Example of analyzing 20 simulated CREs
```
# this directory contains a minimal example of simulated CREs
cd example

# Create a file of branch_scoring jobs
TFforge.py data/tree_simulation.nwk motifs.ls data/species_lost_simulation.txt elements_simul.ls --windowsize 200 --add_suffix _simulation -bg=background/
# Run job list as batch or in parallel 
./alljobs_simulation > scores_simulation
# Execute association test
TFforge_statistics.py data/tree_simulation.nwk motifs.ls data/species_lost_simulation.txt elements_simul.ls --add_suffix _simulation
```

# General workflow
## Input data
- species tree in newick format
- motif files in wtmx format (see example/data/ACA1.wtmx as example)
- motif list: path of wtmx files of one TF per line 
- Phenotype-loss species list: one species per line
- CRE fastafiles: each file contains the sequence for every (ancestral or extant) species
- CRE list: path of fastafile of one CRE per line

## Step 1: Branch score computation
Generate the TFforge branch_scoring commands for all CREs.
```
TFforge.py <tree> <motif_list> <lost_species_list> <element_list>
```
This creates for every CRE a TFforge_branch_scoring.py job. Each line in alljobs<suffix> consists a single job. Each job is completely independent of any other job, thus each job can be run in parallel to others.

Execute that alljobs file. Either sequentially by doing
```
./alljobs_simulation > scores_simulation
```
or run it in parallel by using a compute cluster.
Every job returns a line in the following format:
motif_file	CRE_file	(branch_start>branch_end:branch_score	)*	<
which should be concatenated into a file called "scores<suffix>".

## Step 2: Association test
```
TFforge_statistics.py <tree> <motif_list> <lost_species_list> <element_list>
```
TFforge_statistics.py classifies branches into trait-loss and trait-preserving and assesses the significance the association of this classification with the branch scores. 

## Common Parameters
### TFforge.py
--add_suffix <suffix>
Appends suffix to every generated file
--windowsize/-w <n>
Scoring window used in sequence scoring
--background/-bg <folder>
Background used for sequence scoring. Either a file or a folder structure with backgrounds for different GC contents
--scrCrrIter <n>
Number score correction iterations. 0 turns the score correction off

### TFforge_statistics.py
--add_suffix <suffix>
Appends suffix to every generated file
--filterspecies <comma separated list>
Exclude species from analyses
--elements <file>
Analyse only the elements specified <file>


### Expert parameters
--verbose/-v
--debug/-d

TFforge.py
--no_ancestral_filter
Turn of ancestral score filtering
--no_fixed_TP
Do not fix transition probabilites while computing branch scores
--filter_branch_threshold <x>
Filter branches if start and end node are below <x>
--filter_branches <file>
Like --filter_branch_threshold but with motif specific branch thresholds from <file>
--filter_GC_change <x>
Filter branches with a GC content change above <x>
--filter_length_change <x>
Filter branches with a relative length change above <x>
