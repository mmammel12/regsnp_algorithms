# snp

## SNP Object

### init
Goals:
* initialize all attributes

Input:
* input file path

Output:
* none

Steps:
* set input file path to attribute
* get logger

### sort
Goals:
* sort csv

Input:
* output file (path or object)

Output:
* sorted csv

Steps:
* read in csv input file using pandas
* sort csv

### is_valid
Gaols:
* check if SNV input is valid
  * contains four columns:
    * chrom
    * pos
    * ref
    * alt
  * chrom and pos are in reference genome
  * reg and alt are single nucleotide allele
* reference fasta file with .fai index
* return boolean for validity

Input:
* path to file

Output:
* boolean for valid

Steps:
* check if input file is empty
  * return false if it is empty
* open file with open as f, and pysam.FastaFile(ref_fname) as ref
* create a dictionary of references and lengths from ref
* loop through f
  * strip and split each line to get columns
  * assign each column to a variable
  * if length of cols != 4
    * invalid file format
    * return false
  * check that chrom, ref_allele from file are valid (in set lists)
    * return false if not correct
  * try to convert pos to int
    * except:
    * return false
  * check pos is valid
    * return false if not valid
  * check ref_allele/alt_allele is consistent with reference genome and not switched
    * return false if not
* return true

### switch_alleles
Goals:
* switch ref_allele and alt_allele in input file if the alt_allele matches reference genome

Input:
* input snp
* reference fasta file with .fai index
* output snp

Output:
* no return
* switches ref_allele and alt_allele
* write new snp file with switched alleles

Steps:
* open input file using with open(input file) as in_f, pysam.FastaFile as ref, open(output file in write mode) as out_f
  * loop through in_f
    * strip and split each line to get columns
    * assign each column to a variable
    * convert pos to int
    * check the ref_allele and alt_allele need to swap
      * swap them
    * write columns to out_f