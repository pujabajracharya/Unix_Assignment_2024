## Puja's UNIX Assignment 2024
# Submission Deadline: 16th Feb, 2024 By 11:59pm

## Data Inspection

### Attributes of `fang_et_al_genotypes`
### Using cat command to print the content to the screen
```
$ cat fang_et_al_genotypes.txt
```
### Using head to see the first few lines of the file and tail to inspect the end of the file
```
$ head -n 3 fang_et_al_genotypes.txt
$ tail -n 4 fang_et_al_genotypes.txt
```
### Using less to highlight text matches 
```
$ less fang_et_al_genotypes.txt
# then press / and enter PZA00013
```
### Using wc command to know the number of lines, words and bytes in the file
```
$ wc fang_et_al_genotypes.txt
```
### Inspecting the file size using du command also with -h "human readable" option
```
du -h fang_et_al_genotypes.txt
```
### Using the cut command to extract specific columns and also tidying things up with column. Here I extract Sample_ID, abph1.20,     abph1.22
```
$ grep -v "^#" fang_et_al_genotypes.txt | cut -f 1,4,5 | head -n 5
$ grep -v "^#" fang_et_al_genotypes.txt | cut -f1-8 | head -n3
```
### Inspecting the number of columns in fang_et_al_genotypes.txt with awk
```
$ tail -n +6 fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
```
By inspecting this file fang_et_al_genotypes.txt I learned that:

1. fang_et_al_genotypes.txt is a published SNP dataset which includes genotypic information for SNPs in maize, teosinte, and Tripsacum.
2. These genotypes likely posseses variations at specific SNP positions among the individuals
3. The rows contain the data on each SNP, including SNP ID, various markers and genotype information for different samples. The dataset including wild maize and Tripsacum indicates broader genetic study.


### Attributes of `snp_position.txt`
### Using cat command to print the content to the screen
```
$ cat snp_position.txt
```
### Using head to see the first few lines of the file and tail to inspect the end of the file
```
$ head -n 3 snp_position.txt
$ tail -n 4 snp_position.txt
```
### Using less to highlight text matches 
```
$ less snp_position.txt
# then press / and enter PZA00013
```
### Using wc command to know the number of lines, words and bytes in the file
```
wc snp_position.txt
```
### Using awk to inspect the number of columns in a file
```
$ awk -F "\t" '{print NF; exit}' snp_position.txt 
```
### Using cut command to extract specific column. Here I am extracting cdv_marker_id
```
$ cut -f 2 snp_position.txt | head -n 3
```
By inspecting this file snp_position.txt I learned that:

1. snp_position.txt contains additional information about the position of SNPs that are genotyped in fang_et_al_genotypes.txt. These two file are related by the 'SNP_ID' column.
2. Some of the key columns includes 'SNP_ID' (corresponding to the same in fang_et_al_genotypes.txt), 'Chromosome' (giving the information where the SNP is located), 'Position' (Nucleotide position of the SNP on the chromosome) 
3. Other columns like alt_pos, gene, cdv_map_feature.name, count_cmf, count_gene etc provides additional details anout the SNPs. Here the genes associated with these SNPs are mentioned in the cdv_map_feature.name column.



## Data Processing

## Maize Data
### Extracting only the header from fang_et_al_genotypes file 
```
awk 'NR==1' fang_et_al_genotypes.txt > header.txt
```
This awk command extracts the first line (header) from the file fang_et_al_genotypes.txt and save it to the new file.
### Selecting all lines from fang_et_al_genotypes.txt where the value in the third column is "ZMMIL" 
```
awk '$3 == "ZMMIL"' fang_et_al_genotypes.txt > maize_ZMMIL_genotypes.txt
```
### Selecting all lines from fang_et_al_genotypes.txt where the value in the third column is "ZMMLR" 
```
awk '$3 == "ZMMLR"' fang_et_al_genotypes.txt > maize_ZMMLR_genotypes.txt
```
### Selecting all lines from fang_et_al_genotypes.txt where the value in the third column is "ZMMMR" 
```
awk '$3 == "ZMMMR"' fang_et_al_genotypes.txt > maize_ZMMMR_genotypes.txt
```
Using awk command to select all the lines from the file fang_et_al_genotypes.txt where the third column ($3) is equal to ZMMIL, ZMMLR, ZMMMR.
### Combining all these four files using cat
```
cat header.txt maize_ZMMIL_genotypes.txt maize_ZMMLR_genotypes.txt maize_ZMMMR_genotypes.txt > combined_maize_genotypes.txt
```
cat command is used to link all the four files together.
### Transpose this combined file
```
awk -f transpose.awk combined_maize_genotypes.txt > transposed_combined_maize_genotypes.txt
```
transpose.awk is used to switch the rows and the columns within the combined file.
### Removing the header from the transposed file
```
tail -n +2 transposed_combined_maize_genotypes.txt > transposed_combined_maize_genotypes_no_header.txt
```
tail command is used here to remove the first line (header) from the trasposed file
### Sorting this new no header transposed file
```
sort -k1,1 transposed_combined_maize_genotypes_no_header.txt > sorted_transposed_combined_maize_genotypes.txt 
```
sort command is used here to sort the transposed combined no header file based on the first column.
## For snp_position.txt file
### Extracting the first, third and the fourth column from this snp_position.txt file
```
cut -f1,3,4 snp_position.txt > snp_position_cut.txt
```
Using cut to extract specific columns (1,3,4) from the snp_position.txt file
### Removing the header from this file
```
tail -n +2 snp_position_cut.txt > snp_position_cut_no_header.txt
```
tail command is used to exclude the first line (header) from the file
### Sort this snp_position_cut_no_header.txt file
```
sort -k1,1 snp_cut_no_header.txt > sorted_snp_position.txt
```
sort command is used here to sort file based on the first column.
### Joining sorted maize file and sorted snp position file
```
join -1 1 -2 1 -t $'\t' sorted_snp_position.txt sorted_transposed_combined_maize_genotypes.txt > maize_joined_data.txt
```
Join command is used to join two sorted files based on the first field (column) in both files. The -t $'\t sets the field separator to a tab
### Creating 10 files with increasing chromosome order
```
for i in {1..10}; do awk -v chr="$i" '$2 == chr { print $0 }' maize_joined_data.txt | sort -k3n | sed 's/\t/?/g' > maize_chr${i}_increasing.txt; done
```
This awk command extract lines from the file maize_joined_data.txt where the second column (chromosome) is equal to the current value $i. The -v chr="$i" passes the shell variable '$i to awk as a variable char
The sort -k3n sort the extracted lines based on the third column (position).
sed is used to replace the tab character to ?

### Creating 10 files with decreasing chromosome order and replacing "-" with "?"
```
for i in {1..10}; do awk -v chr="$i" '$2 == chr { print $0 }' maize_joined_data.txt | sort -k3nr | sed 's/\t/-/g' > maize_chr${i}_decreasing.txt; sed -i 's/-/?/g' maize_chr${i}_decreasing.txt; done
```
This code creates 10 output files each containing the data for a specific chromosome sorted by position and the sed to replace "-" by "?"
### Creating one file with all SNP with unknown positions
```
grep '?' maize_joined_data.txt > maize_unknown_position.txt
```
This code redirect the line containing "?" to the new file maize_unknown_position.txt
### Creating 1 file with all multiple position
```
awk 'NF > 3' maize_joined_data.txt > maize_multiple_position.txt
```
Extracts the lines where the number of field is greater than 3
### Moving all Maize 22 files in Maize.data 
```
mv maize_chr{1..10}_{increasing,decreasing}.txt Maize.data/
```
Moving all the maize file to the new directory Maize.data


## Teosinte Data
### Extracting only the header from fang_et_al_genotypes file 
```
awk 'NR==1' fang_et_al_genotypes.txt > teosinte_header.txt
```
This awk command extracts the first line (header) from the file fang_et_al_genotypes.txt and save it to the new file.

### Selecting all lines from fang_et_al_genotypes.txt where the value in the third column is "ZMPBA" 
```
awk '$3 == "ZMPBA"' fang_et_al_genotypes.txt > teosinte_ZMPBA_genotypes.txt
```
### Selecting all lines from fang_et_al_genotypes.txt where the value in the third column is "ZMPIL" 
```
awk '$3 == "ZMPIL"' fang_et_al_genotypes.txt > teosinte_ZMPIL_genotypes.txt
```
### Selecting all lines from fang_et_al_genotypes.txt where the value in the third column is "ZMPJA"
```
awk '$3 == "ZMPJA"' fang_et_al_genotypes.txt > teosinte_ZMPJA_genotypes.txt
```
Using awk command to select all the lines from the file fang_et_al_genotypes.txt where the third column ($3) is equal to ZMPBA, ZMPIL, ZMPJA.
### Combining all these four files using cat
```
cat teosinte_header.txt teosinte_ZMPBA_genotypes.txt teosinte_ZMPIL_genotypes.txt teosinte_ZMPJA_genotypes.txt > combined_teosinte_genotypes.txt
```
cat command is used to link all the four files together.
### Transpose this combined file
```
awk -f transpose.awk combined_teosinte_genotypes.txt > transposed_combined_teosinte_genotypes.txt
```
transpose.awk is used to switch the rows and the columns within the combined file.
### Removing the header from the transposed file
```
tail -n +2 transposed_combined_teosinte_genotypes.txt > transposed_combined_teosinte_genotypes_no_header.txt
```
tail command is used here to remove the first line (header) from the trasposed file
### Sorting this new no header transposed file
```
sort -k1,1 transposed_combined_teosinte_genotypes_no_header.txt > sorted_transposed_combined_teosinte_genotypes.txt 
```
sort command is used here to sort the transposed combined no header file based on the first column.
### Now join the sorted snp_position file and the sorted teosinte file
```
join -1 1 -2 1 -t $'\t' sorted_snp_position.txt sorted_transposed_combined_teosinte_genotypes.txt > teosinte_joined_data.txt
```
Join command is used to join two sorted files based on the first field (column) in both files. The -t $'\t sets the field separator to a tab
### Creating 10 files with increasing chromosome order
```
for i in {1..10}; do awk -v chr="$i" '$2 == chr { print $0 }' teosinte_joined_data.txt | sort -k3n | sed 's/\t/?/g' > teosinte_chr${i}_increasing.txt; done
```
This awk command extract lines from the file maize_joined_data.txt where the second column (chromosome) is equal to the current value $i. The -v chr="$i" passes the shell variable '$i to awk as a variable char
The sort -k3n sort the extracted lines based on the third column (position).
sed is used to replace the tab character to ?
### Creating 10 files with decreasing chromosome order and replacing "-" with "?"
```
for i in {1..10}; do awk -v chr="$i" '$2 == chr { print $0 }' teosinte_joined_data.txt | sort -k3nr | sed 's/\t/-/g' > teosinte_chr${i}_decreasing.txt; sed -i 's/-/?/g' teosinte_chr${i}_decreasing.txt; done
```
This code creates 10 output files each containing the data for a specific chromosome sorted by position and the sed to replace "-" by "?"
### Creating one file with all SNP with unknown positions
```
grep '?' teosinte_joined_data.txt > teosinte_unknown_position.txt
```
This code redirect the line containing "?" to the new file teosinte_unknown_position.txt
### Creating 1 file with all multiple position
```
awk 'NF > 3' teosinte_joined_data.txt > teosinte_multiple_position.txt
```
Extracts the lines where the number of field is greater than 3
### Moving all Teosinte 22 files in Teosinte.data
```
mv teosinte_chr{1..10}_{increasing,decreasing}.txt Teosinte.data/
```
Moving all the teosinte file to the new directory Teosinte.data




