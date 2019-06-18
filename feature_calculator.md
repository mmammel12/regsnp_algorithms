# feature_calculator

## FeatureCalculator Object

### init
Goals:
* initialize all attributes

Input:
* settings
* ifname
* out_dir
* iformat

Output:
* none

Steps:
* settings, iformat, out_dir, iformat (default 'txt') passed in as arguments
* db_dir from settings
* get logger

### calculate_feature
> this method is responsible for too much, should be refactored

Goals:
* create tmp directory in output directory
* convert vcf to txt (if necessary)
* check input format
* sort input
* switch alleles
* annotate SNVs
* calculate distance to closest protein coding exons and extract intronic SNVs
* extract flanking sequence
* calculate RBP binding change
* calculate protein structural features (disorder, secondary structure, ASA, Pfam, PTM)
* calculate junction score
* calculate phylop conservation score

Input:
* none passed in
* uses object attributes

Output:
* input file as .txt (if started as .vcf)
* 

Steps:
* create tmp directory in out_dir
* check if input file format is .vcf
  * if yes, convert to .txt
>
* pass input file to .txt using SNP
* get path for hg19/hg19.fa as ref_name
* check if ref_name is valid SNP, raise RuntimeError if not correct
>
* use SNP to sort tmp
>
* use SNP to switch alleles
  * pass in path to tmp/snp.sorted, ref_name, path to tmp/snp.switched
>
* create new Annovar object
  * arguments:
    * path to annovar
    * path to annovar/humandb
* call Annovar.annotate(2 args)
  * arguments:
    * path to tmp/snp.switched
    * path to tmp/snp
>
* create new ClosestExon object
  * argument:
    * path to db/hg19_ensGene_exon.bed
* call ClosestExon.get_closest_exon(2 args)
  * arguments:
    * path to tmp/snp.intronic
    * path to tmp/snp.distance
* check size of  tmp/snp.distance
  * if <= 0, raise RuntimeError
>
* create new FlankingSeq object
  * arguments:
    * ref_name
    * 20
      * magic number, don't know why it was chosen
* call FlankingSeq.fetch_flanking_seq(2 args)
  * arguments:
    * path to tmp/snp.distance
    * path to tmp/snp.seq
* call FlankingSeq.fetch_flanking_seq(3 args)
  * arguments:
    * path to tmp/snp.distance
    * path to tmp/snp.fa
    * otype='fasta'
* close FlankingSeq
>
* create new RBPChange object
  * arguments:
    * path to db/motif/pwm
    * path to db/motif/pwm_valid.txt
    * path to db/motif/binding_score_mean_sd.txt
* call RBPChange.rbps.cal_matching_score(2 args)
  * arguments:
    * path to tmp/snp.seq
    * path to tmp/snp.rbp_score
* call rbp.cal_change(2 args)
  * arguments
    * path to tmp/snp.rbp_score
    * path to tmp/snp.rbp_change
>
* create new ProteinFeature object
  * arguments:
    * path to db/ensembl.db
    * path to db/hg19_ensGene.txt
* call ProteinFeature.calculate_protein_feature(2 args)
  * arguments:
    * path to tmp/snp.distance
    * path to tmp.snp.protein_feature
>
* create new JunctionStrength object
  * arguments:
    * path to db/motif/donorsite.pssm
    * path to db/motif/acceptorsite.pssm
* call JunctionStrength.cal_junction_strength(3 args)
  * arguments:
    * ref_name
    * path to tmp/snp.distance
    * path to tmp/snp.junc
>
* create new Phylop object
  * argument:
    * path to db/phylop/hg19.100way.phyloP100way.bw
* call Phylop.calculate(2 args)
  * arguments:
    * path to tmp/snp.distance
    * path to tmp/snp.phylop
* close Phylop
>
* call self._merge_features()

### _merge_features
Goals:
* merge all the features calculated in calculate_feature()

Input:
* none passed in, uses object attributes

Output:
* csv with the results

Steps:
* read in csv tmp/snp.rbp_change with pandas
  * drop columns in place
    * #chrom
    * pos
    * ref
    * alt
    * ref_seq
    * alt_seq
>
* read in csv tmp/snp.protein_feature with pandas
  * drop columns in place
    * #chrom_snp
    * start_snp
    * end_snp
    * ref
    * alt
    * feature
    * gene_id
    * chrom
    * start
    * end
    * score
>
* read in csv temp/snp.junc using pandas
  * only read in columns:
    * aic
    * dic
    * aic_change
    * dic_change
>
* read in csv tmp/snp.phylop
  * only read in columns:
    * phylop1
    * phylop3
    * phylop7
>
* concatenate previously read in csv files
* create out_dir/snp.features.txt