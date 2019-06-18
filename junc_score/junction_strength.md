# junction_strength

## JunctionStrength Object

### init
Goals:
* initialize all attributes

Input:
* donor_ic
* acceptor_ic

Output:
* none

Steps:
* assign donor_ic_matrix using self.parse_matrix(donor_ic)
* assign acceptor_ic_matrix using self.parse_matrix(acceptor_ic)

### parse_matrix
Goals:
* parse a matrix and retunr the transpose

Input:
* fname: file name

Output:
* transpose of the matrix

Steps:
* create new empty list matrix
* with open fname as f
  * read in the header
  * loop through the lines of f
    * rstrip and split to get columns
    * append columns index 1 to end as floats to matrix
* convert list to matrix using numpy
* return transposed matrix

### cal_donor_ic
Goals:
* calculate a score for the donor sequence based on a matrix

Input:
* seq

Output:
* score

Steps:
* assert sequence length is the same as self.donor_ic_matrix.shape[1]
* convert seq to uppercase
* create letter dictionary {'G': 0, 'A': 1, 'C': 2, 'T': 3}
* set score to 0
* for i, v in enumerate(seq)
  * score += self.donor_ic_matrix[letter[v], i]
* return score

### cal_acceptor_ic
Goals:
* calculate a score for the acceptor sequence

Input:
* seq

Output:
* score

Steps:
* assert sequence length is the same as self.acceptor_ic_matrix.shape[1]
* convert seq to uppercase
* create letter dictionary {'G': 0, 'A': 1, 'C': 2, 'T': 3}
* set score to 0
* for i, v in enumerate(seq)
  * score += self.acceptor_ic_matrix[letter[v], i]
* return score

### cal_junction_strength
this method is doing way too much, needs to be refactored

Goals:
* calculate junction strengths and write to file, lots of other stuff along the way

Input:
* ref_fname
* bed_fname
* out_fname

Output:
* writes to output file

Steps:
* with open bed_fname as bed_f, out_fname as out_f write mode
  * call self._build_header() to build the header then write it to the output file
  * get the reference file using pysam.Fastafile
  * loop through lines in bed_f
    * rstrip and split to get columns
    * assign values in cols to chrom_snp, start_snp, end_snp, ref_snp, alt_snp, feature, gene_id, chrom, start, end, transcript_id, score, strand, distance
    * convert start_snp, end_snp, start, end, distance to ints
    > Lots of magic numbers incoming, this needs a fix
    * if strand == "+"
      * astart = start - 13 acceptor: 3' end of intron
      * aend = start + 1
      * dstart = end - 3  donor: 5' end of intron
      * dend = end + 7
    * elif strand == "-"
      * astart = end - 1
      * aend = end + 13
      * dstart = start - 7
      * dend = start + 3
    * aseq = ref.fetch(chrom, astart, aend).lower()
    * dseq = ref.fetch(chrom, dstart, dend).lower()
    >
    * set aseq, dseq to None
    * if -13 <= distance < 0 (snp in intron region of acceptor site)
      * assert aseq[(end_snp - astart - 1)]
      * aseq_snp = aseq[:(end_snp - astart - 1)] + alt_snp + aseq[(end_snp - astart):]
      * assert len(aseq) == len(aseq_snp), 'Unequal length: {0}, {1}'.format(len(aseq), len(aseq_snp))
    * if 0 < distance <= 7  # snp in intron region of donor site
      * assert dseq[(end_snp - dstart - 1)]
      * dseq_snp = dseq[:(end_snp - dstart - 1)] + alt_snp + dseq[(end_snp - dstart):]
      * assert len(dseq) == len(dseq_snp), 'Unequal length: {0}, {1}'.format(len(dseq), len(dseq_snp))
    >
    * if strand == "-"
      * reverse aseq, dseq, aseq_snp, dseq_snp using self.reverse_complement()
    >
    * aic = 'NA'
    * dic = 'NA'
    * aic_change = 'NA'
    * dic_change = 'NA'
    > not sure what prev 4 lines are doing, a comment would really help
    >
    * if distance < 0 and aseq.find('N') == -1:
      * aic = self.cal_acceptor_ic(aseq)
    * if distance > 0 and dseq.find('N') == -1:
      * dic = self.cal_donor_ic(dseq)
    * if aseq_snp:
      * if aseq_snp.find('N') == -1:
        * aic_change = self.cal_acceptor_ic(aseq_snp) - aic
    * if dseq_snp:
      * if dseq_snp.find('N') == -1:
        * dic_change = self.cal_donor_ic(dseq_snp) - dic
    * out_f.write('\t'.join(map(str, cols + [astart, aend, aseq, dstart, dend, dseq, aic, dic, aic_change, dic_change])) + '\n')
    > not sure about this chunk of if statements either
    >
    * close ref

### reverse_complement
Goals:
* reverse a string sequence

Input:
* seq

Output:
* seq reversed

Steps:
* if seq
  * reverse the sequence
  * return seq
* return none

### _build_header
Goals:
* build header to be written to output file

Input:
* none

Output:
* header string list

Steps:
* build header string array then return it