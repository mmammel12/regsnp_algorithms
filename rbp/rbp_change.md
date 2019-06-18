# rbp_change

## RBPChange Object
RBP binding score change.

### init
Goals:
* initialize the attributes

Input:
* pssm_path: the path of PSSM files
* pssm_list_fname: a list of valid PSSM files
* ms_fname: the file contains binding/nonbinding mean and sd for all PSSMs

Output:
* none

Steps:
* call RBPScore to assign rbps
* call self.parse_mean_var(ms_fname) to assign mean_var
* call self.cal_ic() to assign ics
* call self.cal_beta_params() to assign beta_params

### parse_mean_var
Goals:
* Parse pre-computed mean and variance of matching score for binding and non-binding events.

Input:
* fname: contains five columns: pssm_fname, binding_mean, binding_sd, nonbinding_mean, nonbinding_sd

Output:
* a collections.OrderedDict of tuple, each element is {pssm_fname: [binding_mean, binding_sd, nonbinding_mean, nonbinding_sd]}

Steps:
* create new ordered dictionary mean_var
* with open fname as f
  * read in header
  * loop through f
    * rstrip and split line into motif, bmean, bsd, nbmean, nbsd
    * assign bmean, bsd, nbmean, nbsd as floats to mean_var[motif]
* return mean_var

### cal_ic
Goals:
* fill ordered dictionary with calculated information content.

Input:
* background=[0.25, 0.25, 0.25, 0.25]

Output:
* ics

Steps:
* create new ordered dictionary ics
* loop through each element in self.rbps.pssms
  * call self._cal_ic(self.rbps.pssms[rbp].pssm, background) and append result to ics
* return ics

### _cal_ic
Goals:
* Calculate information content.

Input:
* pssm: 4 x m numpy ndarray of PSSM.
* background: background frequency

Output:
* ic

Steps:
* create numpy array from background
* convert numpy array to matrix
* calculate frequency with (2 ** pssm) * background
* calculate ic with numpy.sum(freq * pssm)
* return ic

### cal_beta_params
Goals:
* Calculate beta distribution parameters a, b for all PSSMs.

Input:
* none

Output:
* ordered dictionary with beta paramaters a, b for all the PSSMs

Steps:
* create new ordered dictionary beta_params
* loop through self.ics
  * calculate mode using 1 / 2 ** self.ics[rbp]
  * assign quantile as 0.005
  * calculate quantile value with mode / 10.0
  * call self._cal_beta_params(mode, quantile, qtl_val) and append to ordered dict
* return beta_params

### _cal_beta_params
Goals:
* Calculate beta distribution parameters a, b based on mode and quantile value atl_val.

Input:
* mode: set to 1/2 ** ic. For beta distribution, mode = (a - 1) / (a + b - 2).
* quantile
* qtl_val
* lower: lower end points of the interval to be searched for the root.
* upper: upper end points of the interval to be searched for the root.

Output:
* 2 beta parameters

Steps:
* define beta_prior within this method's scope
>
* use scipy.optimize.brentq to assign a. pass in reference to beta_prior function, a=lower, b=upper, args=(mode, quantile, qtl_val)
* calculate relationship between alpha, beta and mode, (a - 1) / mode - (a - 1) + 1. assign to b
* return a, b

#### beta_prior
Goals:
* needed to pass to scipy.optimize.brentq

Input:
* x
* mode
* quantile
* qtl_val

Output:
* whatever scipy.stats.beta.cdf returns

Steps:
* call and return scipy.stats.beta.cdf(qtl_val, x, (x - 1) / mode - (x - 1) + 1) - quantile

### cal_change
Goals:
* Given matching score file, calculate log2(Odds Ratio) and posterior probability of binding change.

Input:
* score_fname: output of RBP_score.cal_matching_score()
* out_fname: output file name
* keep_ncol: keep only the first keep_ncol columns

Output:
* writes to output file

Steps:
* with open score_fname as score_f, out_fname as out_f in write mode
  * read in, rstrip, and split header
  * keep on start to keep_ncol of the header
  * append to header [x + '_logOR' for x in self.rbps.pssm_list] + [x + '_pvalue' for x in self.rbps.pssm_list]
  * write header to output file
  * loop through score_f
    * rstrip and split the line to get columns
    * assign first 6 columns to chrom, pos, ref, alt, ref_seq, alt_seq
    * ref_scores = map(float, cols[keep_ncol:(keep_ncol + self.rbps.pssm_num)])
    * alt_scores = map(float, cols[(keep_ncol + self.rbps.pssm_num):])
    * assert length of ref_scores is the same as length of alt_scores
    * create empty lists: log_ors[], pvalues[]
    * loop for i, rbp in enumerate(self.rbps.pssm_list)
      * assign bmean, bsd, nbmean, nbsd from self.mean_var[rbp]
      * set a, b from self.beta_params[rbp]
      * call self._cal_prob(ref_scores[i], alt_scores[i], bmean, bsd, nbmean, nbsd, a, b) to assign pvalue
      * append pvalue to pvalues
      * call self._cal_logor(ref_scores[i], alt_scores[i], bmean, bsd, nbmean, nbsd) to assign log_or
      * append log_or to log_ors
    * write to output file

### _cal_prob
Goals:
* Calculate posterior probability of binding change when the prior follows a beta distribution specified by a, b.

Input:
* ref_score
* alt_score
* bmean
* bsd
* nbmean
* nbsd
* a
* b

Output:
* posterior probability

Steps:
* if max of ref_score and alt_score is less than (bmean - 3 * bsd)
  * return 0
* else
  * b_ref = scipy.stats.norm.cdf(ref_score, bmean, bsd)
  * b_alt = scipy.stats.norm.cdf(alt_score, bmean, bsd)
  * nb_ref = 1 - scipy.stats.norm.cdf(ref_score, nbmean, nbsd)
  * nb_alt = 1 - scipy.stats.norm.cdf(alt_score, nbmean, nbsd)
  >
  * define beta_distri within this method's scope
  >
  * pass reference to beta_distri into scipy.integrate.quad(beta_distri, 0, 1, epsrel=10 ** -10) to assign prob, err
  * return prob

#### beta_distri
Goals:
* needed to be passed into scipy.integrate.quad

Input:
* x

Output:
* something that scipy.integrate.quad can use

Steps:
* calculate and return  x * (1 - x) * (b_ref * nb_alt + nb_ref * b_alt ) / ((x * b_ref + (1 - x) * nb_ref) * (x * b_alt + (1 - x) * nb_alt)) * scipy.stats.beta.pdf(x, a, b)

### _cal_logor
Goals:
* Calculate log2(Odds Ratio) of binding change bewteen ref and alt.

Input:
* ref_score
* alt_score
* bmean
* bsd
* nbmean
* nbsd

Output:
* log base 2 of odds_alt/ odds_ref

Steps:
* b_ref = scipy.stats.norm.cdf(ref_score, bmean, bsd)
* b_alt = scipy.stats.norm.cdf(alt_score, bmean, bsd)
* nb_ref = 1 - scipy.stats.norm.cdf(ref_score, nbmean, nbsd)
* nb_alt = 1 - scipy.stats.norm.cdf(alt_score, nbmean, nbsd)
* odds_ref = b_ref / nb_ref
* odds_alt = b_alt / nb_alt
* return np.log2(odds_alt / odds_ref)