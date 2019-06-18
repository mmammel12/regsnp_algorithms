# annovar

## Annovar Object

### init
Goals:
* initialize the attributes

Input:
* annovar path
* db path

Output:
* none

Steps:
* set annovar path
* set db path

### annotate
Goals:
* prepare input file
* run commmand arguments on the input and output files
* write annotated data to output file

Input:
* input file
* output file

Output:
* writes to output file

Steps:
* call self._prepare_input(2 args)
  * arguments:
    * input file name
    * output file name + '.avinput'
* run command arguments
* with open output file name + '.hg19_multianno.txt' as in_f, and out_fname + '.intronic' in write mode as out_f
  * read in the header
  * loop through each line of in_f
    * strip and split each line to get columns
    * if statement, use regular expressions to search
      * if True, write to output file

### _prepare_input
Goals:
* prepare the input for annotation

Input:
* input file
* output file

Output:
* prepared output file

Steps:
* get path to input and output files
* with open input file as in_f, output file as out_f in write mode
  * loop through lines in in_f
    * strip and split each line, assign to:
      * chrom
      * pos
      * ref
      * alt
    * write to output file with those 4 columns
* return output file