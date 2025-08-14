Pedigree simulation with offspring genome generation

For simulation of each new generation you need both parents or at least one of them. These parents called founders and there are two types of founders: explicit and implicit ones. Explicit founders - individuals that do not have known parents. Implicit founders are a pair of individuals only one individual in which has known parents.
Let's consider you want to simulate children and grandchildren in a family.
In this case you need VCF file which must include only biallelic SNPs of all founders.

1.Install an environment with ped_sim.yml file. It's better to use bash terminal instead of zsh (somehow the error occurs if you do otherwise). In addition, take into account that libflate library needs a version 1.0 which exists only for osx-64
You can install conda osx-64 environment by doing this:
conda create -n intel_env
  conda activate intel_env
  conda config --env --set subdir osx-64
  conda install python

Bear in mind that this wrapper works only with SLiM 4.0.1 version
2.Filter your VCF file to get only biallelic SNPs with these commands:
bcftools norm -m+ first_family_biall.vcf  > first_family_biall2.vcf #this helps to group all multiallelic variants written in several lines into one line
bcftools view -m2 -M2 --types=snps first_family_biall2.vcf>first_family_biall3.vcf #save only SNPs
*OR just run the filtration step in the next step (3)

3. It is necessary to add field AA (Ancestral allele) in INFO in your VCF file (it will be just nucleotide in reference genome at this position, to determine actual AA you need to run multiple alignment of nucleotide sequences of human and its relatives genomes)
 python run_ped_sim.py -t filter_vcf -v test_data/lwk_1kg_toydata.vcf
It produces file which name ends with 'slim_fil'

4. Build pedigree structure.
Input: table with information about sibship distribution across generation you're interested in
	Columns: year/generation number mean sd
Output: pedigree structure in nx format (each line - one edge between parent and a child inside a directed graph)

5. Create file with founders in txt format
No header!
Columns: id_in_vcf_file id_in_pedigree_structure_nx_file (sep=' ')

6. You probably want Nucleotide specific simulation (in this case your mutations will be aligned to reference genome) and want to use recombination map (only one map is possible for both sexes in this version of SLiM(4.0.1))
For nucleotide specific simulation you'll need exactly the same reference genome as was used in variant calling
Example of code:
python py_ped_sim_init/run_ped_sim.py -t sim_genomes_exact -n gen_ped.nx -e founders.txt -v first_family_biall_parents_slim_fil.vcf.gz -f Homo_sapiens.GRCh38.dna.chromosome.1.fa -rm recomb_pos_mat.tsv -o first_family_simul 
-t option to choose which mode to use to run simulation
-n path to your pedigree in nx format
-e this option is used if you want to assign founder status to particular individuals, otherwise the founders status will be assigned independently from those individuals whose SNPs are in vcf file
-v path to vcf file with SNPs of individuals
-f path to reference genome
-rm path to map with recombination rates

Format of map with recombination rates:
Start postion of genomic element involved in recombination Mutation rate
1 0 #this is first line if you have recombination position in 1-based coordinate system
Probably you will have mutation rate in cMperMb, bear in mind that this number then will be multiplied by e-8

In the input of a simulation you are to get vcf files of your offspring and their parents


WORK WITH MULTIPLE CHROMOSOMES
As SLim=4.0 version can't work with multiple chromosomes on its own (SLiM=5.0 can), you shouls preprocess your fasta and vcf files.
To preform this follow steps in multiple_chr_simul.ipynb.

You can run zero step of merging fasta files in bash (To use multiple chromosomes and SLiM 4.0 version you should merge your fasta file with reference for your chromosomes and vcf files for these chromosomes):
Merge fasta files: 
cd d:\data
cat *.fasta.txt > d:\combined.fasta

Your vcf files must be sorted, bgzipped and indexed with tabix (you can do it in ipynb script)






  
