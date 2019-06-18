# rbp_score

## RBPScore Object
Store a collection of PSSM files. Given a file contains the input sequences, calculate the matching scores for all the PSSMs.

### init
Goals:
* initialize the attributes

Input:
* pssm_path: the path of PSSM files
* pssm_list_fname: a list of valid PSSM files

Output:
* none

Steps:
* assign pssm_path
* call self.parse_pssm_list(pssm_list_fname) to assign pssm_list
* assign length of pssm_list to pssm_num
* call self.generate_pssms(self.pssm_path, self.pssm_list) to assign pssms

### parse_pssm_list
Goals:
* parse the list of valid PSSM file.

Input:
* fname: each row contains a valid PSSM file name.

Output:
* a list of PSSM file name.

Steps:
* create empty list, pssm_list
* with open fname as f
  * loop through f
    * rstrip line and append to pssm_list
* return pssm_list

### generate_pssms
Goals:
* create a PSSM ordered dictionary

Input:
* pssm_path: the path of PSSM files.
* pssm_list: a list of valid PSSM files.

Output:
* a collections.OrderedDict of RBPs.

Steps:
* create collections.OrderedDict()
* loop through pssm_list
  * fill the dictionary with RBP objects
* return dictionary

### cal_matching_score
Goals:
* given a sequence file that contains reference sequences and alternative sequences, calculate the max matching score of each RBP for each sequence.

Input:
* seq_fname: a file whose first six columns are: chrom, pos, ref, alt, ref_seq (the center is the SNP), alt_seq (the center is the SNP).
* out_fname: output file name
* keep_ncol: keep only the first keep_ncol columns

Output:
* writes to output file

Steps:
* with open seq_fname as seq_f, out_fname as out_f in write mode
  * read in the header, save from start to keep_ncol\
  * append to header each element of self.pssm_list + '_ref' and each element of self.pssm_list + '_alt'
  * write header to output fil
  * loop through seq_f
    * strip and split line to get columns
    * chrom, pos, ref, alt, ref_seq, alt_seq = cols[:keep_ncol]
    * create empty lists: ref_scores, alt_scores
    * loop through self.pssms
      * append self.pssms[pssm].match(ref_seq) to ref_scores
      * append self.pssms[pssm].match(alt_seq) to alt_scores
    * write to out_f