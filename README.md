# README

### Unix Assignment by Timothy Nessel

#### Setting up the Enviornment 

After logging in to hpc-class `ssh tnessel@hpc-class.its.iastate.edu`  
I navigated to the class Unix assignment
`cd BCB546X-Fall2018/assignments/UNIX_Assignment/`

(Midway through completion, I moved the enviornment to the independant git repository from which it is currently being accessed)

#### Data Inspection

To get some metadata on these files such as permissions, file size, etc., I used the follow command:  
`ls -l -h *`  
To view the number of lines in each file, I use the following:
`wc -l *.txt`

To view the files, I could `cat` them, or to just get a sample of their format, I could use the command:
`head filename.txt` or `tail filename.txt` with an optional distinction of output lines `n -5`

To view the number of columns for each file, I will use the following command:  
`awk -F "\t" '{print NF; exit}' inputfile.txt`
This is using the language `awk` , distinguishes tab as our field separator, then executing the command: print 'number of fields' then 'exit' for our data file.

#### Data Processing


To select only the samples with the desired maize groups, I utilized this command:
`grep -w -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > maize_genotypes.txt`
Generating _maize\_genotypes.txt_

To maintain the important information from the header of fang_et_al_genotypes.txt; I extracted that and added it back on to our maize matches (which also could have been done with a long winded use of >>):
`head -n 1 fang_et_al_genotypes.txt > fang_head.txt`
`cat fang_head.txt maize_genotypes.txt > maize_genotypes_h.txt`
Generating _maize\_genotypes\_h.txt_

I then transposed the file as per the recommendation of the instructor:
`awk -f transpose.awk maize_genotypes_h.txt > maize_genotypes_transposed.txt`
Generating _maize\_genotypes\_transposed.txt_

Next, I sorted both data files by their common column (SNP\_ID/Sample\_ID) using the following command:
`sort -k1,1 fileinput.txt > sortedfileinput.txt`  
Generating _maize\_genotypes\_transposed\_sorted.txt_  
Generating _snp\_sorted.txt_ 

I checked these intermediate files to ensure that their first column was in fact their common one:
`cut -f 1 maize_genotypes_transposed_sorted.txt | head -n 5`
`cut -f 1 snp_sorted.txt | head -n 5`

Now that both files were sorted by their common column, I joined them with: 
`join -1 1 -2 1 -a 2 snp_sorted.txt maize_genotypes_transposed_sorted.txt > maize_full.txt`
Generating _maize\_full.txt_

To select the columns that were relevant for future use, I check the total columns using the previously mentioned command and filtered this file into a new file only containing SNP_ID, Chromosome, Position, Genotype:
`cut -f 1,3,4,14-1586 -d ' ' maize_full.txt > maize_filtered.txt`
Generating _maize\_filtered.txt_

Following the same procedure; I generated the sample set for teosinte:
`grep -w -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > maize_genotypes.txt`
`cat fang_head.txt teosinte_genotypes.txt > teosinte_genotypes_h.txt`
`awk -f transpose.awk teosinte_genotypes_h.txt > teosinte_genotypes_transposed.txt`
`sort -k1,1 teosinte_genotypes_transposed.txt > teosinte_genotypes_transposed_sorted.txt`
`join -1 1 -2 1 -a 2 snp_sorted.txt teosinte_genotypes_transposed_sorted.txt > teosinte_full.txt`
`cut -f 1,3,4,14-988 -d ' ' teosinte_full.txt > teosinte_filtered.txt`
Generating _teosinte\_filtered.txt_


##### Generating the files

######Maize

For the first 10 files, I manually typed out this script for all 10 chromosomes:
`awk '$2 = (CHROMOSOME#)' maize_filtered.txt | sort -k 3 -n > maize_(CHROMOSOME#).txt`
These first 10 files have the format _maize\_#.txt_

I really wanted to use this command to loop this script by chromosome number, but I couldnt get the syntax to work:
`awk '{for (s = 1; s <= 10; s++) $2 = $s maize_filtered.txt | sort -k3,1 > maize_$s.txt}`

For the next 10 files, I used reverse sort and sed to generate the different arrangement:
`awk '$2 = (CHROMOSOME#)' maize_filtered.txt |sort -r -n -k3 | sed 's/?/-/g' > maize_reverse_(CHROMOSOME#).txt`
These first 10 files have the format _maize\_reverse\_#.txt_

To generate the unknown chromosome file:
`awk '$2 = "unknown"' maize_filtered.txt  >  maize_unknown.txt`
being sure to include "unknown" in quotes, as it is a string not a number
Generating _maize\_unknown.txt_

Similarily for the multiple chromosome file:
`awk '$2 = "multiple"' maize_filtered.txt  >  maize_multiple.txt`
Generating _maize\_multiple.txt_

It is worth saying that it was useful to use:
`cut -f 2 -d ' ' maize_filtered.txt | sort | uniq -c`
to get a indication of the number and terminology used in these chromosome fields. Inspecting these files afterwards also helps confirm my commands. 

######Teosinte

I generated the 10 teosinte files:
`awk '$2 = (CHROMOSOME#)' teosinte_filtered.txt | sort -k 3 -n > teosinte_(CHROMOSOME#).txt`
These first 10 files have the format _teosinte\_#.txt_

For the next 10 files, I used reverse sort and sed to generate the different arrangement:
`awk '$2 = (CHROMOSOME#)' teosinte_filtered.txt |sort -r -n -k3 | sed 's/?/-/g' > teosinte_reverse_(CHROMOSOME#).txt`
These first 10 files have the format _teosinte\_reverse\_#.txt_

To generate the unknown chromosome file:
`awk '$2 = "unknown"' teosinte_filtered.txt  >  teosinte_unknown.txt`

For the multiple chromosome file:
`awk '$2 = "multiple"' teosinte_filtered.txt  >  teosinte_multiple.txt`

######Headers
I added headers to all existing created files with the following command
`echo -e "SNP_ID Chromosome Position Genotype" | cat - targetfile.txt > /tmp/out && mv /tmp/out targetfile.txt`
This combines the new header and the existing file in a temporary output; then overwrites the original file

######Unknown and Multiple Positions
To create files for maize and teosinte with unknown positions, I used the following command:
`awk '$3 = "unknown"' teosinte_filtered.txt  >  teosinte_unknown_positions.txt`
`awk '$3 = "unknown"' maize_filtered.txt  >  maize_unknown_positions.txt`

To create files for maize and teosinte with multiple positions, I used the following command to search for both 'multiple' and anything specifying 'Chr' (which appears when multiple chromosome positions are specified):

`awk '$3 == "multiple" || $3 ~ /Chr/' teosinte_filtered.txt > teosinte_multiple_positions.txt`
`awk '$3 == "multiple" || $3 ~ /Chr/' maize_filtered.txt > maize_multiple_positions.txt`

#####Git

Most of this work was done in the class directory on hpc-class. In the final week, I copied the files from the class directory to a new directory in my home directory, and initialized it with git. From that point forward, the files have been updated and version controlled with git. 