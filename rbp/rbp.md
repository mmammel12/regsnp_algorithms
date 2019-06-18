# rbp

## RBP Object
Store PSSM of single RBP. Given a sequence, calculate the max matching score.

### init
Goals:
* initialize the attribute

Input:
* pssm file name (pssm_fname)

Output:
* none

Steps:
* assign pssm file name
* assign pssm numpy ndarray. A 4 x m matrix where m is the motif length. Row 0,1,2,3 correspond to A,C,G,T.
* assign motif length

### parse_pssm
Goals:
* parse PSSM matrix file

Input:
* fname, a file contains a 4 x m PSSM matrix. Each row corresponds to A,C,G,T, and m is the motif length.

Output:
* a 4 x m numpy ndarray.

Steps:
* use numpy to get a matrix from input file (.txt)
* return matrix

### match
Goals:
* calculate the max matching score.

Input:
* seq DNA sequence. Should be the same strand as RNA. The length should be longer than RBP motif.
* cover_center boolean to indicate the motif must cover the center of sequence.

Output:
* the max matching score.

Steps:
* convert seq to uppercase
* get length of seq
* if cover_center
  * assert sequence length should be longer than 2 * motif length
  * assert sequence should be up_flanking + mid + down_flanking, the length should be even.
  * get center index, seq_length // 2
  * assign seq to back half of seq
  * assign new seq_len
* assert sequence length is longer than motif length.
* set max_score float('-inf')
* loop for i in range(0, seq_len - self.len + 1)
  * assign subsequence as seq[i:(i + self.len)]
  * call self._match(sub_seq) to get score for the subsequence
  * assign new max score if score is higher than prev max_score
* return max_score

### _match
Goals:
* calculate the matching score

Input:
* seq: input DNA sequence. Should be the same strand as RNA. The length should be equal to RBP motif.
* pssm: a 4 x m numpy ndarray represents PSSM. Each row corresponds to A,C,G,T.

Output:
* the matching score

Steps:
* get length of seq
* assert sequence length should equal to motif length
* create dictionary (letters) for the row index of each letter in PSSM {'A':0, 'C':1, 'G':2, 'T':3}
* set score to 0
* if not pssm
  * pssm = self.pssm
* loop for i, v in enumerate(seq)
  * if v in letters
    * current_score = pssm[letters[v], i]
  * else
    * current score = 0.0
  add current_score to score
* return score