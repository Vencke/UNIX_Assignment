#UNIX Assignment

##Data Inspection

###Attributes of `fang_et_al_genotypes`

```
head fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
cut -f 1-10 fang_et_al_genotypes.txt | head | column -t
wc -l fang_et_al_genotypes.txt
du -h fang_et_al_genotypes.txt
file fang_et_al_genotypes.txt
cut -f 4-986 fang_et_al_genotypes.txt | tail -n +2 | grep [^ATCG/] | less
```
General commands/ points for me to remember: 
* vi to edit md 
* i for insert
* x to delete character 
* dd to delete line
* esc, :wq to save and exit
* esc, just :w to save without exiting 
* I can have two terminal windows open, and edit the md in one and do the commands in the other one

By inspecting this file I learned that:

* With the head command I learned that the file has 986 columns, with the wc -l command I learned that the file has 2783 rows
* With the cut head column pipe I could look at the first 10 fields and head gave me the first 10 lines, column -t made it look like a table  
* du -h gave the size in human readable format --> 6.7M 
* With file, I checked for ASCII characters and it has ASCII text, with very long lines
* With the pipe cut, tail, grep, less, I inspected only the SNP data (starting in the 4th column in the second row) for missing data (anything that is not A, T, C or G), with less I could see that missing data is marked with ?/? 
 

###Attributes of `snp_position.txt`

```
head snp_position.txt | awk -F "\t" '{print NF; exit}'
wc -l snp_position.txt
head snp_position.txt | cut -f 1-10 | column -t
du -h snp_position.txt
file snp_position.txt
```

By inspecting this file I learned that:

* With the head and awk command I learned that the file has 15 columns and the wc -l 984 rows 
* With the head cut column pipe I could look at the first 10 fields and head gave me the first 10 lines, column -t made it look like a table
* du -h gave me the human readable size --> 49K
* With file I checked for ASCII characters and the file is encoded in ASCII characters 
 

##Data Processing


```
cut -f 1,3,4 snp_position.txt > snp_formatted_position.txt
(head -n 1 snp_formatted_position.txt && tail -n +2 snp_formatted_position.txt | sort -k1,1) > sorted_snp_formatted_position.txt
``` 
From the SNP position file I only need the SNPID, Chromosome and Position Column, so I cut it and saved it into a new file
Keeping the header, I am sorting based on the first column (for merge, later I can sort for Chromosome) and saved it into a new file 

```
head snp_formatted_position.txt | column -t
tail snp_formatted_position.txt | column -t
```

With head and tail I visually checked if the beginning and end look sorted


###Maize Data

Since the Groups for maize (ZMMIL, ZMMLR, ZMMMR) are in the third column, I sorted for the third column, cut that column and counted how many times the group names appeared in the column, for ZMMIL, ZMMLR and ZMMMR it was 290, 1256 and 27 respectively. 

```
sort -k3 fang_et_al_genotypes.txt | cut -f 3 | uniq -c | column -t
```

Now I can extract the desired maize groups and skip column 2 and 3, if we skip column 2 and 3 we have 984 remaining columns which matches the 984 rows in the snp position file, I just need to transpose it and save it to a new file 

```
awk -F "\t" '$3 ~ /ZMMIL|ZMMLR|ZMMMR|Group/' fang_et_al_genotypes.txt | cut -f 1,4-986 > maize_genotypes.txt
awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
```

To make sure its sorted the same way, I sort it like the snp file

```
(head -n 1 transposed_maize_genotypes.txt && tail -n +2 transposed_maize_genotypes.txt | sort -k1,1) > sorted_transposed_maize_genotypes.txt
``` 

To check if evertyhing workd, I looked at the files with:

```
head maize_genotypes.txt | cut -f 1-10 | column -t
head maize_genotypes.txt | awk -F "\t" '{print NF; exit}'
```
--> 984 

```
head transposed_maize_genotypes.txt | cut -f 1-10 | column -t
wc -l transposed_maize_genotypes.txt
```
--> 984

```
head sorted_transposed_maize_genotypes.txt | cut -f 1-10 | column -t
wc -l sorted_transposed_maize_genotypes.txt
```
--> 984 

```
join --header -1 1 -2 1 -t $'\t' sorted_snp_formatted_position.txt sorted_transposed_maize_genotypes.txt > joined_maize_snp_and_genotypes.txt

```
I merged the sorted files starting in the first column of both files

To remove the unknown and multiples from the file:

```
awk '$3 !~ /unknown|multiple/' joined_maize_snp_and_genotypes.txt > joined_maize_only_numeric_position.txt
```
I created a directory called maize to save all the 22 files

```
mkdir maize
```

And used a loop to create the files for each chromosome in increasing order with ?/? for missing values and decreasing order with -/- for missing values

```
for i in {1..10}; do (head -n 1 joined_maize_only_numeric_position.txt && awk '$2 == '$i'' joined_maize_only_numeric_position.txt | sort -k3,3n) > maize/maize_chr"$i"_increasing.txt; done 
```

```
for i in {1..10}; do (head -n 1 joined_maize_only_numeric_position.txt && awk '$2 == '$i'' joined_maize_only_numeric_position.txt | sort -k 3nr | sed 's/?/-/g') > maize/maize_chr"$i"_decreasing.txt; done
```

I cd;ed into the folder and checked visually some increasing files for ?/? and some decreasing files for -/- and I could see it worked

```
head maize_chr5_increasing.txt | cut -f 1-10 | column -t
head maize_chr5_decreasing.txt | cut -f 1-10 | column -t
head maize_chr1_increasing.txt | cut -f 1-10 | column -t
head maize_chr1_decreasing.txt | cut -f 1-10 | column -t
```

For the unknown and multiple position, I used this:

```
awk '$3 ~ /unknown|Position/' joined_maize_snp_and_genotypes.txt > maize/maize_snp_with_unknown_position.txt
awk '$3 ~ /multiple|Position/' joined_maize_snp_and_genotypes.txt > maize/maize_snp_with_multiple_position.txt
```

Again, I visually checked the unknwon and multiple file and it looked good

```
head maize_snp_with_multiple_position.txt | cut -f 1-10 | column -t
head maize_snp_with_unknown_position.txt | cut -f 1-10 | column -t
```



###Teosinte Data
Since the Groups for teosinte (ZMPBA, ZMPIL, ZMPJA) are also in the third column, I sorted for the third column, cut that column and counted how many times the group names appeared in the column, for ZMPBA, ZMPIL and ZMPJA, it was 900, 41 and 34 respectively.

```                                            
sort -k3 fang_et_al_genotypes.txt | cut -f 3 | uniq -c | column -t
```

Now I can extract the desired teosinte groups and skip column 2 and 3, transpose it, sort it and save each step to a new file

```
awk -F "\t" '$3 ~ /ZMPBA|ZMPIL|ZMPJA|Group/' fang_et_al_genotypes.txt | cut -f 1,4-986 > teosinte_genotypes.txt
awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
heag transposed_teosinte_genotypes.txt | cut -f 1-10 | column -t
(head -n 1 transposed_teosinte_genotypes.txt && tail -n +2 transposed_teosinte_genotypes.txt | sort -k1,1) > sorted_transposed_teosinte_genotypes.txt
```

look at the files:

```
head teosinte_genotypes.txt | cut -f 1-10 | column -t
tail teosinte_genotypes.txt | cut -f 1-10 | column -t
```

merged the files like for maize:

```
join --header -1 1 -2 1 -t $'\t' sorted_snp_formatted_position.txt sorted_transposed_teosinte_genotypes.txt > joined_teosinte_snp_and_genotypes.txt
```

excluded unknown and multiples

```
awk '$3 !~ /unknown|multiple/' joined_teosinte_snp_and_genotypes.txt > joined_teosinte_only_numeric_position.txt
```

make the folder for teosinte

```
mkdir teosinte
```


used a loop again to create the files for each chromosome in increasing order with ?/? for missing values and decreasing order with -/- for missing values
and also made the files for unkown and multiple position

```
for i in {1..10}; do (head -n 1 joined_teosinte_only_numeric_position.txt && awk '$2 == '$i'' joined_teosinte_only_numeric_position.txt | sort -k3,3n) > teosinte/teosinte_chr"$i"_increasing.txt; done
for i in {1..10}; do (head -n 1 joined_teosinte_only_numeric_position.txt && awk '$2 == '$i'' joined_teosinte_only_numeric_position.txt | sort -k3,3nr | sed 's/?/-/g') > teosinte/teosinte_chr"$i"_decreasing.txt; done
awk '$3 ~ /unknown|Position/' joined_teosinte_snp_and_genotypes.txt > teosinte/teosinte_snp_with_unknown_position.txt
awk '$3 ~ /multiple|Position/' joined_teosinte_snp_and_genotypes.txt > teosinte/teosinte_snp_with_multiple_position.txt
```

I cd;ed into the folder and checked visually some increasing files for ?/? and some decreasing files for -/- and I could see it worked

```
head teosinte_chr5_increasing.txt | cut -f 1-10 | column -t
head teosinte_chr5_decreasing.txt | cut -f 1-10 | column -t
head teosinte_chr1_increasing.txt | cut -f 1-10 | column -t
head teosinte_chr1_decreasing.txt | cut -f 1-10 | column -t
```

Finally, a quick visual check for the unknwon and multiple files and it looked good

```
head teosinte_snp_with_multiple_position.txt | cut -f 1-10 | column -t
head teosinte_snp_with_unknown_position.txt | cut -f 1-10 | column -t
```

Final check how many files are in the teosinte and maize folders:

```
ls -1 teosinte | grep -v / | wc -l
ls -1 maize | grep -v / | wc -l
```

Yay, 22 in each!


