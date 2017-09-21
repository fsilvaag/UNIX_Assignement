#UNIX ASSIGNEMENT
##Author: Fernando Silva Aguilar

##DATA INSPECTION

###Document Information
After making a fork of the [repository](git@github.com:fsilvaag/BCB546X-Fall2017.git.) to my GitHub account I create a local copy of that repository in my remote session (ssh hpc_class). 

We have the following files:
None of the files has a header that interferes with the analysis. 

-  fang_et_al_genotypes.txt 
>>This file is in format .txt with a size of 11 M. When looking at the type of encoding, this file is an "ASCII text", with very long lines. It has  2783 lines,  2744038 words, and 11051939 characters, with 986 columns. 
>>In the column group we can find the following:

$ `sort -k3,3  fang_et_al_genotypes.txt | cut -f 3 | uniq -c`

Count  Group

     22 TRIPS
     15 ZDIPL
     17 ZLUXR
     10 ZMHUE
    290 ZMMIL
    1256 ZMMLR
     27 ZMMMR
    900 ZMPBA
     41 ZMPIL
     34 ZMPJA
     75 ZMXCH
     69 ZMXCP
      6 ZMXIL
      7 ZMXNO
      4 ZMXNT
      9 ZPERR

- snp_position.txt         
>>This file is in format .txt with a size of 84 K. When looking at the type of encoding, this file is an "ASCII text". It has  984 lines,  13198 words, and 82763 characters with 15 columns.  The column number 5 and 6 are pretty much empty with just a few lines with values. 

- transpose.awk       
>> This file is in format .awk with a size of 4.0 K. When looking at the type of encoding, this file is an "awk script, ASCII text". It has  15 lines,  46 words, and 353 characters.

- UNIX_Assignment.pdf 
>>This file is in format .pdf with a size of 56 K. When looking at the type of encoding, this file is an "pdf document",  version 1.3.
>>This file conatins the information about the homework in a readable format pdf.

- UNIX_Assignment.md
>>This file is in format .md with a size of 8.0 K. When looking at the type of encoding, this file is an "ASCII text", with very long lines.
>>This file conatins the information about the homework in a readable format in markdown.

###Commands Used:
*Size of Files = $ `du -h <file name and extension>`

*Encoding System = $ `file *` ; where * is a wildcard

*Number of lines, words, and characters = $ `wc * | sort -k1,1nr -k2,2nr -k3,3nr`

*Number of columns = $ `awk -F "\t" '{print NF; exit}' <file name and extension> or $ tali -n +5 <file> | awk -F "\t" '{print NF; exit}'; $ grep -v "^#" <file> | awk -F "\t" '{print NF; exit}' `

>Should give you the same result. The second or third option is used for files that has headers at the beginning of the file. 

*Understand the content of th file = $ `less <file> | cut -f <column or columns>`

#DATA PROCESSING

##Data Split in Groups

>To separate the data in the file fang_et_al_genotypes.txt in the three species, Teosinte, Maize, and Tripsacum we proceed to use the grep option:

$ `grep -E "(ZMPBA|ZMPIL|ZMPJA) fang_et_al_genotypes.txt > teosinte_genotypes.txt`

$ `grep -E "(ZMMIL|ZMMLR|ZMMMR) fang_et_al_genotypes.txt > teosinte_genotypes.txt`

$ `grep -v -E "(ZMMIL|ZMMLR|ZMMMR|ZMPBA|ZMPIL|ZMPJA) fang_et_al_genotypes.txt > tripsacum_genotypes.txt `

>To verify no error in the split of the files we proceed to make a count of the lines in each file:

$ `wc -l teosinte_genotypes.txt tripsacum_genotypes.txt maize_genotypes.txt`

     975 teosinte_genotypes.txt
     235 tripsacum_genotypes.txt
    1573 maize_genotypes.txt
    2783 total 
The total number of lines summing over the three files is the same as the total number of lines in the original file, which implies that the copy was correct. However, the files teosinte_genotypes.txt and maize_genotypes.txt do not have header. Then to put the header:

$ `head -n 1 fang_et_al_genotypes.txt > header.txt`

$ `cat header.txt teosinte_genotypes.txt > teosinte_genotypes_header.txt`

$ `cat header.txt maize_genotypes.txt > maize_genotypes_header.txt`

##Transposed data

Once we have the files separated by specie, we can transpose the files to get the information of each SNP:

$ `awk -f transpose.awk teosinte_genotypes_header.txt > transposed_teosinte_genotypes.txt`

$ `awk -f transpose.awk maize_genotypes_header.txt > transposed_maize_genotypes.txt`

$ `awk -f transpose.awk tripsacum_genotypes.txt > transposed_tripsacum_genotypes.txt`

In the file snp_position.txt we have identified that the column 3 is for chromosome, the position of the chromosome is the column 4, while the column 1 (SNP_ID) is the common column with the transposed files. Then, we proceed to create a new file with only the information required from the snp_position.txt file:

$ `cut -f 1,3,4 snp_position.txt > snp_ID_chro_pos.txt`

Now we can order the files transposed_teosinte_genotypes.txt, transposed_tripsacum_genotypes.txt, and transposed_maize_genotypes.txt, by snp_id. However, we have to remove the header since the sort procedure do not distinguish from the values and the header. We proceed as follows:

$ `tail -n +2 transposed_teosinte_genotypes.txt | sort -k1,1V > transposed_teosinte_genotypes_sorted_noheader.txt`

$ `tail -n +2 transposed_maize_genotypes.txt | sort -k1,1V > transposed_maize_genotypes_sorted_noheader.txt`

$ `tail -n +2 transposed_tripsacum_genotypes.txt | sort -k1,1V > transposed_tripsacum_genotypes_sorted_noheader.txt`

$ `sort -k1,1V snp_ID_chro_pos.txt > snp_ID_chro_pos_sorted.txt`

To keep the headers we remove:

$ `head -n 1 transposed_teosinte_genotypes.txt > hader_transposed_teosinte.txt`

$ `head -n 1 transposed_maize_genotypes.txt > hader_transposed_maize.txt`

$ `head -n 1 transposed_tripsacum_genotypes.txt > hader_transposed_tripsacum.txt`

$ `head -n 1 snp_ID_chro_pos.txt > header_snp_ID_chro_pos.txt`


Now that we have sorted the files by snp_ID, we can proceed to join them with the information of the SNP position, and chromosome: 

$ `join -11 -21 snp_ID_chro_pos_sorted.txt  transposed_teosinte_genotypes_sorted_noheader.txt -t $'\t' | sort -k2,2n -k3,3n > teosinte_all_join.txt`

$ `join -11 -21 snp_ID_chro_pos_sorted.txt  transposed_maize_genotypes_sorted_noheader.txt -t $'\t' | sort -k2,2n -k3,3n > maize_all_join.txt`

$ `join -11 -21 snp_ID_chro_pos_sorted.txt  transposed_tripsacum_genotypes_sorted_noheader.txt -t $'\t' | sort -k2,2n -k3,3n > tripsacum_all_join.txt`


The new joined files do not have header, then we have to concatenate the header of each file and the header of the file of snp ID, chromosome and position. The following command is to merge the two files headers:

$ `paste header_snp_ID_chro_pos.txt  hader_transposed_maize.txt > header_transposed_maize_join.txt`

$ `paste header_snp_ID_chro_pos.txt hader_transposed_teosinte.txt > header_transposed_teosinte_join.txt`

$ `paste header_snp_ID_chro_pos.txt hader_transposed_tripsacum.txt > header_transposed_tripsacum_join.txt`

Now we put the header to the joined files:

$ `cat header_transposed_tripsacum_join.txt tripsacum_all_join.txt > tripsacum_all_join_header.txt`

$ `cat header_transposed_teosinte_join.txt teosinte_all_join.txt > teosinte_all_join_header.txt`

$ `cat header_transposed_maize_join.txt maize_all_join.txt > maize_all_join_header.txt`

In order to create the new files (10 one per chromosome) we can use the function awk as follows:

$ `awk 'NR==1{h=$0; next}!seen[$2]++{f="teosinte_"$2"_header.txt"; print h > f} {print >> f}' teosinte_all_join_header.txt`

$ `awk 'NR==1{h=$0; next}!seen[$2]++{f="tripsacum_"$2"_header.txt"; print h > f} {print >> f}' tripsacum_all_join_header.txt`

$ `awk 'NR==1{h=$0; next}!seen[$2]++{f="maize_"$2"_header.txt";print h > f} {print >> f}' maize_all_join_header.txt`

In this function the first term NR++1{h=$0; next} is used to keep the header of the original file in the new files. This files has already been ordered by position values in an increasing way and with the values for missing data "?", as in the original file.

To generate the files with SNPs ordered based on decreasing position values and with missing data encoded by "-" symbol, we create a copy of the joined files and replace the "?" symbol for "-", using sed command:

$ `cp teosinte_all_join_header.txt rev_teosinte_all_join_header.txt`

$ `cp tripsacum_all_join_header.txt rev_tripsacum_all_join_header.txt`

$ `cp maize_all_join_header.txt rev_maize_all_join_header.txt`

The command [sed](https://www.computerhope.com/unix/used.htm) **`"Stream Editor"`** allow us to filter and transform text 

$ `sed 's/?/-/g' rev_teosinte_all_join_header.txt`

$ `sed 's/?/-/g' rev_tripsacum_all_join_header.txt`

$ `sed 's/?/-/g' rev_maize_all_join_header.txt`

Now we remove the header from the files in order to sort them by position in decreasing order and chromosome in increasing order.

$ `head -n 1 rev_teosinte_all_join_header.txt > header_rev_teosinte_all_join.txt`

$ `head -n 1 rev_tripsacum_all_join_header.txt > header_rev_tripsacum_all_join.txt`

$ `head -n 1 rev_maize_all_join_header.txt > header_rev_maize_all_join.txt`

$ `tail -n +2 rev_teosinte_all_join_header.txt | sort -k2,2n -k3,3nr > sorted_rev_teosinte_all_join.txt`

$ `tail -n +2 rev_tripsacum_all_join_header.txt | sort -k2,2n -k3,3nr > sorted_rev_tripsacum_all_join.txt`

$ `tail -n +2 rev_maize_all_join_header.txt | sort -k2,2n -k3,3nr > sorted_rev_maize_all_join.txt`

Now, since we have the files ordered by chromosome (increasing) and position (decreasing), we proceed to put again the headers.

$ `cat sorted_rev_maize_all_join.txt  header_rev_maize_all_join.txt >sorted_rev_maize_all_join_head.txt`

$ `cat sorted_rev_tripsacum_all_join.txt  header_rev_tripsacum_all_join.txt >sorted_rev_tripsacum_all_join_head.txt`

$ `cat sorted_rev_teosinte_all_join.txt  header_rev_teosinte_all_join.txt >sorted_rev_teosinte_all_join_head.txt`

$ `sed 's/?/-/g' sorted_rev_maize_all_join_head.txt > sorted_rev_maize_all_join_symbol-.txt`

$ `sed 's/?/-/g' sorted_rev_tripsacum_all_join_head.txt > sorted_rev_tripsacum_all_join_symbol-.txt`

$ `sed 's/?/-/g' sorted_rev_teosinte_all_join_head.txt > sorted_rev_teosinte_all_join_symbol-.txt`


Finally, with the awk command we create again the files per chromosome:

$ `awk 'NR==1{h=$0; next}!seen[$2]++{f="teosinte_"$2"_header_rev.txt"; print h > f} {print >> f}' sorted_rev_teosinte_all_join_symbol-.txt`

$ `awk 'NR==1{h=$0; next}!seen[$2]++{f="tripsacum_"$2"_header_rev.txt"; print h > f} {print >> f}' sorted_rev_tripsacum_all_join_symbol-.txt`

$ `awk 'NR==1{h=$0; next}!seen[$2]++{f="maize_"$2"_header_rev.txt";print h > f} {print >> f}' sorted_rev_maize_all_join_symbol-.txt`

**The files named as `teosinte_chr#_header.txt` and `tripsacum_chr#_header.txt` have the SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?**

**The files named as `teosinte_chr#_header_rev.txt` and `tripsacum_chr#_header_rev.txt` have the SNPs ordered based on decreasing position values and with missing data encoded by this symbol: -**


Finally to create the files for "unknown" and "multiple" positions we make a simple query with the creation of an output, as follows:

$ `grep "unknown" teosinte_all_join_header.txt > unk_teosinte_all_join.txt`

$ `grep "unknown" maize_all_join_header.txt > unk_maize_all_join.txt`

$ `grep "unknown" tripsacum_all_join_header.txt > unk_tripsacum_all_join.txt`


$ `grep "multiple" teosinte_all_join_header.txt > mult_teosinte_all_join.txt`

$ `grep "multiple" maize_all_join_header.txt > mult_maize_all_join.txt`

$ `grep "multiple" tripsacum_all_join_header.txt > mult_tripsacum_all_join.txt`
