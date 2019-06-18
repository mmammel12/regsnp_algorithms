# protein_feature

## ProteinFeature Object

### init
Goals:
* initialize all attributes

Input:
* db_fname
* gene_pred_fname

Output:
* none

Steps:
* fetch db using db_fname
* create SpineD, PTM, Pfam objects passing in db
* create GenePredExt object passing in path to gene_pred_fname

### calculate_protein_feature
Goals:
* read in BedTool file and use to calculate protein features then write csv to output file

Input:
* bfname: input BedTool file name
* out_fname: output file name

Output:
* writes to output file

Steps:
* with open bfname as bed_f, out_fname as out_f in write mode
  * call self._build_header() to build the header
  * write header to out_f
  * loop through lines in bed_f
    * rstrip and split columns
    * assign estart as int(cols[8])
    * assign eend as int(cols[9])
      * would love a comment on these to explain why they are 8 and 9
    * get transcript_id using regex search
    * if transcript_id is in self.gene_pred.transcripts
      * set pstart and pend using self.gene_pred.get_protein_coord()
      * set disorder, ss, asa using self.spined.cal_spined()
      * set ptm using self.ptm.cal_ptm()
      * set pfam using self.pfam.cal_pfam()
    * write line to output file

### _build_header
Goals:
* build header for output file

Input:
* none

Output:
* header string list

Steps:
* create the header list and return it