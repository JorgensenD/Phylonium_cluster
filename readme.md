## Phylonium on HPC
#### David Jorgensen 
#### 2020/05/01

### Build from git
[Phylonium](https://github.com/EvolBioInf/phylonium) needs to be compiled from the github repository. As we do not have sudo rights on the cluster, this requires a mix of conda and installed modules. Once GCC in Anaconda is updated to 8.2.0 this should no longer be necessary. The following should be run line-by-line in the shell, answering yes to dependencies and installs.

```bash 
module load anaconda3/personal

conda create -n phylonium_build
source activate phylonium-build

conda install gsl automake git gxx_linux-64

#this lib is the reason we need to use conda
conda install -c bioconda libdivsufsort

#need to export the most recent gcc version installed on the cluster
export CC=/apps/gcc/8.2.0/bin/gcc

#clone the github repo and build
git clone https://github.com/evolbioinf/phylonium

#this makes a new subdir you need to work from
cd phylonium
autoreconf -fi -Im4
./configure
make

source deactivate phylonium-build
```
After this build is complete you should have an executable 'phylonium' in /phylonium/src/ which can be run outside of this conda environment

### Format of sequences
The program requires each sequence used to be in a separate fasta file. This could be carried out with `awk` in bash more efficiently but I have used the following R code.

```R
require(ape)

# IMPORTANT: set wd where you want - about to spam 10k+ sequence files!!

seqs <- read.FASTA("./your_input_multifasta.fas")

# cant have / or | in filenames
labs <- gsub("\\/", "_",gsub("\\|", "_", labels(seqs)))
names(seqs) <- labs

# progress bar for loop
pb = txtProgressBar(min = 0, max = length(seqs), initial = 0, style=3) 

for(i in 1:length(seqs)){
write.FASTA(seqs[i], paste0(names(seqs)[i], ".fas"))
  setTxtProgressBar(pb,i)
}
```

### Shell script for qsub
As phylonium is installed locally you will need to copy the directory to `$TEMPDIR` on the compute node to use the command. This script copies and runs on all `.fas` files in the current directory. Something similar to the following should be saved as a `.sh` or `.pbs` shell script.
```bash
#PBS -S /bin/bash
#PBS -N matrix
#PBS -l select=1:ncpus=4:mem=5gb,walltime=01:00:00
#PBS -j oe

cp $PBS_O_WORKDIR/*.fas $TMPDIR
cp $HOME/phylonium/ -r $TMPDIR

./phylonium/src/phylonium *.fas > distmat.out

cp distmat.out $PBS_O_WORKDIR
```






