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

```

By inspecting this file I learned that:

* point 1
* point 2
* point 3

##Data Processing

###Maize Data

```
here is my snippet of code used for data processing
```

Here is my brief description of what this code does


###Teosinte Data

```
here is my snippet of code used for data processing
```

Here is my brief description of what this code does
