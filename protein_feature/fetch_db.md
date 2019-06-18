# fetch_db

## DB Object

### init
Goals:
* initialize all attributes

Input:
* fname: sqlite3 db file name

Output:
* none

Steps:
* set fname as path to sqlite3 db

### query_structure
Goals:
* get structure info from db, format and return it

Input:
* transcript_id

Output:
* structure_info

Steps:
* create connection to db
* assign row_factory
* create cursor
* create tuple for transcript_id to use in query
* execute sql query to select all from structure with matching transcript_id
* fetchall and store in r
> those lines could all be refactored into method that accepts a table name string since it is used in all 3 methods
* assert len(r) == 1 or r == []
* create new empty dict for structure_info
* if r
  * fill structure info
  * key, values:
    * 'transcript_id': r[0]['transcript_id']
    * 'aa': r[0]['aa'].split(',')
    * 'beta_sheet': map(float, r[0]['beta_sheet'].split(','))
    * 'random_coil': map(float, r[0]['random_coil'].split(','))
    * 'alpha_helix': map(float, r[0]['alpha_helix'].split(','))
    * 'asa': map(float, r[0]['asa'].split(','))
    * 'disorder': map(float, r[0]['disorder'].split(','))
    * 'length': len(structure_info['aa'])
* close connection
* return structure_info

### query_pfam
Goals:
* get pfam_info from db, format and return it

Input:
* transcript_id

Output:
* pfam_info

Steps:
* create connection to db
* assign row_factory
* create cursor
* create tuple for transcript_id to use in query
* execute sql query to select all from pfam with matching transcript_id
* fetchall and store in r
>
* create empty list pfam_info
* if r
  * loop through records in r
    * append (record['transcript_id'], record['start'], record['end'], record['family'], record['name'], record['clan']) to pfam_info
* close connection
* return pfam_info

### query_ptm
Goals:
* query the database for ptm, format and return it

Input:
* transcript_id

Output:
* ptm_info

Steps:
* create connection to db
* assign row_factory
* create cursor
* create tuple for transcript_id to use in query
* execute sql query to select all from ptm with matching transcript_id
* fetchall and store in r
>
* create empty list ptm_info
* if r
  * loop through records in r
    * append (record['transcript_id'], record['uniprot_id'], record['position'], record['modification']) to ptm_info
* close connection
* return ptm_info