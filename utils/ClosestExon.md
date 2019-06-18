# closest_exon

## ClosestExon Object

### init
Goals:
* initialize the attribute

Input:
* efname

Output:
* none

Steps:
* set efname

### get_closest_exon
Goals:
* get the closest exon

Input:
* input SNV file name in BED-like format
* output file name
* column index of start position from file b in output. '-1' indicates no closest exon found. (default 8)
* column index of distance in output. '0' indicates overlap. (default 8)

Output:
* annotates the closest exon
* filters out the SNVs that overlap with exons and SNVs without exons

Steps:
* pass input file to BedTool, store result as in_f
* .sort and .closest on in_f, pass back to BedTool with .filter and .moveto(output file)