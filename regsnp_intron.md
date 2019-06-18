# regsnp_intron

## main
Goals:
* start the program
* set db + annovar + annovar/humandb paths
* pull in input file
* set output directory

Input:
* settings.json for paths to db, annovar, annovar/humandb
* -f, --force (optional, command line) to override outpul directory
* --iformant ['txt' or 'vcf'] (optional, command line) set input file type, txt is default
* ifname is the input file path
* out_dir is the directory to be made to hold output files

Output:
* makes the output directory
* all output is handled through object method calls

Steps:
* add all command line arguments
* check if out_dir exists already
  * require -f or --force if it does exist
  * make the directory if it does not exist or if -f/--force
* set up logger
* load settings
* try:
  * create new FeatureCalculator
      s:
      * settings file
      * ifname
      * out_dir
      * iformat (defaulted to 'txt')
  * call FeatureCalculator.calculate_feature()
  * create new Predictor
    * arguments:
      * on_ss_imp.pkl path
      * off_ss_imp.pkl path
      * on_ss_clf.pkl path
      * off_ss_clf.pkl path
      * on_ss_fpr_tpr.txt path
      * off_ss_fpr_tpr.txt path
  * call Predictor.predict(2 args)
    * arguments:
      * snp.features.txt (created by ?????????? and placed in out_dir)
      * snp.prediction (created by ?????????? and placed in out_dir)
* except RuntimeError:
  * shutil.rmtree(out_dir, ignore_errors=True)