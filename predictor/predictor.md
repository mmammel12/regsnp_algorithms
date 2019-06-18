# predictor

## Predictor Object

### init
Goals:
* initialize all attributes

Input:
* on_ss_imp python object
* off_ss_imp python object
* on_ss_clf python object
* off_ss_clf python object
* on_ss_fpr_tpr csv
* off_ss_fpr_tpr csv

Output:
* none

Steps:
* joblib.load the python objects
* pandas.read_csv the 2 csv files
* get logger

### predict
Goals:
* Predict diseasing-causing probability of all intronic SNVs

Input:
* ifname: m x n input file. m is the number of SNVs, n is the number of features.
* ofname: output file prefix.

Output:
* writes to output file
* insert into on/off_ss files

Steps:
* use pandas to read in the input file into data list
* separate data into data_on_ss and data_off_ss
* if data_on_ss is not empty
  * call self._predict_on_ss(data_on_ss) to get preds and scores
  * call self._cal_fpr_tpr(scores, True) to get fpr, tpr, disease
  * data_on_ss.insert(4, 'splicing_site', ['on'] * len(preds))
  * data_on_ss.insert(4, 'fpr', fpr)
  * data_on_ss.insert(4, 'tpr', tpr)
  * data_on_ss.insert(4, 'prob', scores[:, 1])
  * data_on_ss.insert(4, 'disease', disease)
* else
  * data_on_ss.insert(4, 'splicing_site', '')
  * data_on_ss.insert(4, 'fpr', 0.0)
  * data_on_ss.insert(4, 'tpr', 0.0)
  * data_on_ss.insert(4, 'prob', 0.0)
  * data_on_ss.insert(4, 'disease', '')
* if data_off_ss is not empty
  * call self._predict_off_ss(data_off_ss) to get preds and scores
  * call self._cal_fpr_tpr(scores, False) to get fpr, tpr, disease
  * data_off_ss.insert(4, 'splicing_site', ['off'] * len(preds))
  * data_off_ss.insert(4, 'fpr', fpr)
  * data_off_ss.insert(4, 'tpr', tpr)
  * data_off_ss.insert(4, 'prob', scores[:, 1])
  * data_off_ss.insert(4, 'disease', disease)
* else
  * data_off_ss.insert(4, 'splicing_site', '')
  * data_off_ss.insert(4, 'fpr', 0.0)
  * data_off_ss.insert(4, 'tpr', 0.0)
  * data_off_ss.insert(4, 'prob', 0.0)
  * data_off_ss.insert(4, 'disease', '')
* pandas concatenate [data_on_ss, data_off_ss], sorted by ['#chrom', 'pos'] onto result
* convert result to csv form
* create records dict from results data
* json.dump({"data": records}, open(ofname + '.json', 'w'))

### _predict_on_ss
Goals:
* Predict diseasing-causing probability of on-splicing-site SNVs

Input:
* data: m x n pandas data frame. m is the number of SNVs, n is the number of features.

Output:
* preds
* scores

Steps:
* drop ['#chrom', 'pos', 'ref', 'alt', 'name', 'strand', 'distance'] from data
* data = self.on_ss_imp.transform(data)
* preds = self.on_ss_clf.predict(data)
* scores = self.on_ss_clf.predict_proba(data)
* return preds, scores

### _predict_off_ss
Goals:
* Predict diseasing-causing probability of off-splicing-site SNVs

Input:
* data: m x n pandas data frame. m is the number of SNVs, n is the number of features.

Output:
* preds
* scores

Steps:
* drop ['#chrom', 'pos', 'ref', 'alt', 'name', 'strand', 'distance', 'aic_change', 'dic_change'] from data
* data = self.off_ss_imp.transform(data)
* preds = self.off_ss_clf.predict(data)
* scores = self.off_ss_clf.predict_proba(data)
* return preds, scores

### _cal_fpr_tpr
Goals:
* determine fpr, tpr, and disease information

Input:
* scores: predicted scores
* on_ss: boolean indicate whether it's on_ss or off_ss

Output:
* fpr
* tpr
* disease

Steps:
* set fpr_tpr to None
* if on_ss
  * fpr_tpr = self.on_ss_fpr_tpr
* else
  * fpr_tpr = self.off_ss_fpr_tpr
* fpr = [fpr_tpr.fpr[(abs(x - fpr_tpr.pvalue)).idxmin()] for x in scores[:,1]]
* tpr = [fpr_tpr.tpr[(abs(x - fpr_tpr.pvalue)).idxmin()] for x in scores[:,1]]
* set cutoff list to [0.0, 0.05, 0.1, 1.1] 
* set disease_category list to ['D', 'PD', 'B']
> more magic numbers, would like to see these defined somewhere with a comment to explain
>
* use numpy to digitize disease based on fpr, cutoff
* disease = [disease_category[i - 1] for i in disease]
* return fpr, tpr, disease