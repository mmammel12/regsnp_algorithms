# phylop

## Phylop Object

### init
Goals:
* initialize all attributes

Input:
* bw_fname: Phylop 100way bigwig file name.

Output:
* none

Steps:
* assign path to bw_fname to bw_handle
* assign return from BigWigFile(self.bw_handle) to bw

### get
Goals:
* use numpy to get the mean

Input:
* chrom: chr1, chr2, etc.
* param start: 0-based.
* param end: 1-based.
* param flanking: length of flanking sequence on each side. default 0

Output:
* mean based on provided input

Steps:
* return np.nanmean(self.bw.get_as_array(chrom, start - flanking, end + flanking))

### calculate
Goals:
* pass specific parts of chrom to self.get and write the result to a file

Input:
* fname: SNP BED.
* param out_fname: output file.

Output:
* writes to output file

Steps:
* with open fname as bed_f, out_fname as out_f in write mode
  * write header using self._build_header()
  * loop through lines in bed_f
    * rstrip and split to get columns
    * assign first 3 columns to chrom, start, end
    * convert start, end to ints
    * create list with self.get(chrom, start, end), self.get(chrom, start, end, 3), self.get(chrom, start, end, 7) as the elements
    * write columns and list to output file

### close
Goals:
* close self.bw_handle

Input:
* none

Output:
* none

Steps:
* close self.bw_handle

### _build_header
Goals:
* build the header for the output file created with calculate()

Input:
* none

Output:
* string list header

Steps:
* create string list header and return it