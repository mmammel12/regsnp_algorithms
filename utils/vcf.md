# vcf

## VCF Object

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

### convert_to_txt
Goals:
* convert vcf to 4-column .txt

Input:
* output .txt file name
* pass_filter: whether filter to out variants that failed the filter, default True

Output:
* 4-column txt file

Steps:
* with open input file as vcf, output file as out_f
  * loop through input file
    * if the line does not contain '#'
      * continue
    * strip and split each line to get columns
    * assign each column to a variable
    * split alts (one of the columns)
    * split filters (one of the columns)
    * if pass_filter == False, or not filters, or filters == ['PASS']
      * loop through alts
        * if length of ref and alt == 1
          * write 4 columns to out_f (chrom, pos, ref, alt)