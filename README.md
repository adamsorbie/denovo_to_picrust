# denovo_to_picrust-
Pipeline for using de-novo picked otus with PICRUSt 

Requirements: 

QIIME 1/QIIME 2 
R 
PICRUSt 

Input files: 

De-novo picked OTU table 
FASTA file with sequence for each OTU 

## Part One - Map sequences against greengenes database 

1. For this we will use the qiime1 script pick_closed_reference_otus.py, for those using qiime2 you can use the vsearch plugin with cluster-features-closed-reference.
```
pick_closed_reference_otus.py -i merged_otus.tab -o qiime_output

``` 
