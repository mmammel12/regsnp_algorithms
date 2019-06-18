# seq

## FlankingSeq Object

### init
Goals:
* initialize all attributes

Input:
* ref file path
* length
* logger

Output:
* none

Steps:
* set ref file path to attribute
* set length to attribute
* get logger

### fetch_flanking_seq
Goals:
* write to out_f

Input:
* input file name
* output file name
* otype='tab'

Output:
* csv with columns:
  * #chrom
  * pos
  * ref
  * alt
  * ref_seq
  * alt_seq

Steps:
* with open input file as in_f, output file (write mode) as out_f
  * check if otype=='tab'
    * if yes, write '\t'.join(['#chrom', 'pos', 'ref', 'alt', 'ref_seq', 'alt_seq']) + '\n'
  * loop through lines in in_f
    * strip and split each line to get columns
    * assign first 5 columns and cols[12] to variables
    * convert pos to int
    * call self._fetch_seq(4 args), returns 2 variables
      * arguments:
        * chrom
        * pos
        * ref
        * alt
      * return vars
        * seq
        * seq_alt
    * assert strand (cols[12]) == '+' or '-'
    * if strand == '-'
      * call self._reverse_complement(1 arg)
        * called twice passing in seq, then seq_alt
    * if otype == 'tab'
      * write non-fasta style data
    * elif otype == 'fasta'
      * write fasta style data

### _fetch_seq
> line 1 of this method is:
> pos = pos
> wut.gif

Goals:
* return a sequence and alt sequence

Input:
* chrom
* pos
* ref_allele
* alt_allele

Output:
* seq
* alt_seq

Steps:
* store start and end index
* get sequence based on start and end from chrom
* if the input reference allele does not match the reference fasta
  * log an error
* get seq_alt based on seq[:self.length] + alt_allele + seq[self.legth + 1:]
* return seq, seq_alt

### _reverse_complement
Goals:
* reverse a sequence

Input:
* sequence

Output:
* reversed sequence

Steps:
* reverse the sequence string

### close
Goals:
* close the reference

Input:
* none

Output:
* none

Steps:
* close the reference