# plinkLiftOver
Quick liftOver for PLINK data

## 1. Software
  - Conda
    - plink2
    - liftOver

## 2. Code
#Prepare conda environment \n
conda create --name liftOver; conda activate liftOver; conda install -c bioconda plink2; conda install -c bioconda ucsc-liftover \n
#download liftOver chain file from https://hgdownload.cse.ucsc.edu/goldenpath/hg19/liftOver/
wget https://hgdownload.cse.ucsc.edu/goldenpath/hg19/liftOver/hg19ToHg38.over.chain.gz

#liftOver hg19 to hg38
pl=YouPlinkName
awk '{$2=$1":"$4":"$6":"$5; print $0}' $pl.bim > temp; mv temp $pl.bim
plink2 --bfile $pl --rm-dup force-first --maf 0.01 --snps-only just-acgt --output-chr M --set-hh-missing --max-alleles 2 --make-bed --out $pl.filtered
awk 'OFS="\t"{print "chr"$1, $4-1, $4, $2}' $pl.filtered.bim > $pl.filtered.hg19.bed
liftOver $pl.filtered.hg19.bed hg19ToHg38.over.chain $pl.filtered.hg38.bed unMapped
plink2 --bfile $pl.filtered --extract <(cut -f4  $pl.filtered.hg38.bed) --update-map <(awk '{print $4,$3}' $pl.filtered.hg38.bed) --make-bed --out $pl.filtered.hg38



