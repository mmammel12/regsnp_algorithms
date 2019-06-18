# gene_pred_ext

## GenePredExt Object

### init
Goals:
* initialize all attributes

Input:
* fname: input file name

Output:
* none

Steps:
* assing path to fname
* get logger
* assign transcripts using self.extract_transcripts()

### parse_file
Goals:
* parse a file and get the columns as a generator

Input:
* none

Output:
* generators containing the columns of each line of self.fname

Steps:
* with open self.fname as f
  * loop through lines in f
    * if line does not contain '#'
      * exit loop
    * rstrip and split line to get columns
    * yield columns

### extract_transcripts
Goals:
* extract the transcripts from the generators created with self.parse_file

Input:
* none

Output:
* transcripts dict

Steps:
* use self.parse_file to get generators, store in gene_pred
* create new defualtdict called transcripts
* loop through all generators at the same time
  * if it's a protein coding gene
    * convert txStart, txEnd, cdsStart, cdsEnd, exonCount to ints
    * rstrip and split exonStarts, exonEnds, exonFrames and convert to ints
    * transcripts[transcript_id]['transcript_id'] = transcript_id
    * transcripts[transcript_id]['gene_id'] = gene_id
    * transcripts[transcript_id]['chrom'] = chrom
    * transcripts[transcript_id]['strand'] = strand
    * transcripts[transcript_id]['txStart'] = txStart
    * transcripts[transcript_id]['txEnd'] = txEnd
    * transcripts[transcript_id]['cdsStart'] = cdsStart
    * transcripts[transcript_id]['cdsEnd'] = cdsEnd
    * transcripts[transcript_id]['exonCount'] = exonCount
    * transcripts[transcript_id]['exons'] = zip(exonStarts, exonEnds)
    * transcripts[transcript_id]['length'] = sum(x[1] - x[0] for x in zip(exonStarts, exonEnds))
    * transcripts[transcript_id]['exonFrames'] = exonFrames
* log total number of protein coding transcripts
* return transcripts

### get_protein_coord
Goals:
* get the start and end points of a protein

Input:
* transcript_id: transcript id.
* gstart: genomic start of exon, 0-based.
* gend: genomic end of exon, 1-based.

Output:
* start and end of protein coordinates if valid

Steps:
* initialize pstart and pend as None
* if transcript_id is in self.transcripts
  * assign transcript from self.transcripts[transcript_id]
  * assign strand, cdsStart, cdsEnd, exons from ther corresponding elements in transcript
  >
  * assign g_cds_len (the CDS length of query exon) to 0
  * g_cds_start = cdsStart if gstart < cdsStart < gend else gstart
  * g_cds_end = cdsEnd if gstart < cdsEnd < gend else gend
  * g_cds_len = g_cds_end - g_cds_start
  >
  * set offset to 0
  * if gstart < cdsEnd and gend > cdsStart
    * if strand == '+'
      * loop through elements in exons, estart, eend as sentries
        * cstart = estart
        * cend = eend
        * if cdsStart is past eend
          * continue
        * if estart <= cdsStart < eend
          * set cstart = cdsStart
        * if gstart != estart or gend != eend
          * set offset += cend - cstart
        * else
          * exit loop
      * pstart = offset
      * pend = offset + g_cds_len
    * if strand == '-'
    > could be changed to elif
      * loop through exons reversed, estart, eend as sentries
        * if cdsEnd < estart
          * continue
        * cstart = estart
        * cend = eend
        * if estart < cdsEnd <= eend
          * cend = cdsEnd
        * if gstart != estart or gend != eend
          * offset += cend - cstart
        * else
          * break
      * pstart = offset
      * pend = offset + g_cds_len
    * pstart /= 3
    * pend /= 3
    > more magic numbers...
  * else
    * log debug exon is out of the cds region of transcript
  * return pstart, pend