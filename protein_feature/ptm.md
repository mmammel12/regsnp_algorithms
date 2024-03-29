# ptm

## PTM Object

### init
Goals:
* initialize all attributes

Input:
* db: sqlite database contains PTM information generated by SpineD.

Output:
* none

Steps:
* assign db
* get logger

### query_ptm_info
Goals:
* create query_info list using ptm_info

Input:
* transcript_id
* pstart
* pend
* ptm_info

Output:
* query_info list

Steps:
* create empty list query_info
* loop through ptm_info
  * assign trans_id, uniprot_id, position, modification from current element
  * if pstart <= position <= pend
    * append record onto query_info
* return query_info

### cal_ptm
Goals:
* calculate and return ptm value

Input:
* transcript_id
* pstart
* pend

Output:
* ptm, either a float or 'NA'

Steps:
* get ptm_info from self.db.query_ptm(transcript_id)
* initialize ptm as 'NA'
* if pstart and pend are not None
  * if ptm_info is not None
    * use self.query_ptm_info() to assign query_result
    * if query_result is not None
      * assing ptm as float(len(query_result)) / (pend - pstart + 1) * 100
    * else
      * ptm = 0.0
      * log debug 'region does not contain PTM' followed by region info
  * else
    * ptm = 0.0
* return ptm