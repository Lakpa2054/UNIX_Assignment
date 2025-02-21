# UNIX Assignment

## Data Inspection

### Attributes of `fang_et_al_genotypes`

```
wc fang_et_al_genotypes.tx
wc -l fang_et_al_genotypes.tx #gave me only the lines 2783
du -h fang_et_al_genotypes.txt
awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
```


By inspecting this file I learned that:
1.	The .txt file contains 2783 lines, 2744038 words, and 11051939 number of bytes in the files. 
2.	This gave me only the number of lines is 2783 
3.	File size is 6.7M 
4.	Number of columns is 986



### Attributes of `snp_position.txt`

```
wc snp_position.txt
wc -l snp_position.txt
du -h snp_position.txt
awk -F "\t" '{print NF; exit}' snp_position.txt
```


By inspecting this file I learned that:
1.	The snp_position.txt file contains 984 lines, 13198 words, and 82763 number of bytes in the files. 
2.	This gave me only the number of lines is 984
3.	File size is 49K 
4.	Number of columns is 15 


## Data Processing

### Maize Data

```
1.	tail -n +3 transposed_genotypes.txt > Transposed_1header.txt
2.	sed 's/Group/SNP_ID/g' Transposed_1header.txt > transposed_Group.txt 
3.	tail -n +6 Transposed_1header.txt | awk -F "\t" '{print NF; exit}'
4.	tail -n +6 transposed_Group.txt | awk -F "\t" '{print NF; exit}'
5.	join -1 1 -2 1 -t $'\t' snp_position.txt transposed_Group.txt > transjoin.txt
6.	awk '{for(i=1;i<=NF;i++) if ($i == "ZMMIL") print "Found at column " i}' transjoin.txt
7.	awk '{for(i=1;i<=NF;i++) if ($i == "ZMPJA") print "Found at column " i}' transjoin.txt
8.	awk '{for(i=1;i<=NF;i++) if ($i == "ZMMMR") print "Found at column " i}' transjoin.txt
9.	awk '{for(i=1;i<=NF;i++) if($i == "ZMMMR") count++} END{print count}' transjoin.txt
10.	awk '{for(i=1;i<=NF;i++) if($i == "ZMMLR") count++} END{print count}' transjoin.txt
11.	awk '{for(i=1;i<=NF;i++) if($i == "ZMMIL") count++} END{print count}' transjoin.txt
12.	cut -d $'\t' -f1,3,4,2508-2797,1225-2480,2481-2507 transjoin.txt > maize.txt
13.	(head -n 1 maize.txt && tail -n +2 maize.txt | sort -k3,3n) > maize_sorted.txt
14.	grep -v "^#" maize_sorted.txt | cut -f2 | sort | uniq -c
15.	mkdir Maize_Increasing Maize_Decreasing 
16.	for i in {1..10}; do
{ head -n 1 maize_sorted.txt && awk -v chr="$i" '$2 == chr' maize_sorted.txt; } > "Maize_chromo${i}.txt"
 mv "Maize_chromo${i}.txt" Maize_Increasing/
done
17.	{ head -n 1 maize_sorted.txt && awk '$2 == "multiple"' maize_sorted.txt; } > Maize_chrM.txt && { head -n 1 maize_sorted.txt && awk '$2 == "unknown"' maize_sorted.txt; } > Maize_chrU.txt
18.	Mv Maize_chrU.txt  Maize_ChrM.txt Maize_Increasing/
19.	sed 's/?/-/g' maize_sorted.txt > maize-sorted.txt
20.	mv maize-sorted.txt Maize_Decreasing
21.	cd Maize_Decreasing
22.	(head -n 1 maize-sorted.txt && tail -n +2 maize-sorted.txt | sort -k3,3nr) > MDsorted.txt
23.	out_file="Maize_chromo_Decreasing_sorted${i}.txt"
    { head -n 1 MDsorted.txt && awk -v chr="$i" '$2 == chr' MDsorted.txt; } > "$out_file"
   echo "Chromosome $i: $(wc -l < "$out_file") lines"
```


Here is my brief description of what this code does
1.	Tail -n +3 print the line 3 onward instead of printing the default last few lines. It skipped the whole row of Sample_ID and JG_OTU, redirected the output of the tail command of transposed_genotypes.txt into new file called Transposed_1header.txt.
2.	stream editor (sed) substitute all “Group” with “SNP_ID” globally in Transposed_1header.txt and redirect the modified output to transposed_Group.txt
3.	tail -n +3 skips the first 5 lines and output the rest of the file, -F”\t” tells awk to use tab(\t) as the column separator and print the number of fields (column). 
4.	Same as 3. 
5.	Join command merge two files based on their first column (SNP ID) of the first line using the tab delimiter and saved the output in transjoin.txt 
6.	Awk read each line, for(i=1;i<=NF;i++) goes through each column in the current row of the file until i=1. If it finds “ZIMML” in any of the column, it printed as “Found at column” followed by the column number i. 
7.	Awk read each line, for(i=1;i<=NF;i++) goes through each column in the current row of the file until i=1. If it finds “ZMPJA” in any of the column, it printed as “Found at column” followed by the column number i. 
8.	Awk read each line, for(i=1;i<=NF;i++) goes through each column in the current row of the file until i=1. If it finds “ZMMR” in any of the column, it printed as “Found at column” followed by the column number i. 
9.	It prints the final count of how many times “ZMMMR” appeared across all rows in the file. 
10.	Same as 9 searches for “ZMMLR”. 
11.	Same as 9 searches for “ZMMIL”. 
12.	Cut command select extract columns (f1,3,4,2508-2797,1225-2480,2481-2507) and saved into a new file called maize.txt. 
13.	It extracts the first line the header, && ensure second part execute only when header is extracted. Tail -n +2 extracts all lines except the first from maize.txt. The rest of the files excluding the first line pipes it into sort -k3, 3n. It then sorts data numerically (n) by the third column (k3,3) in ascending order. It redirects the combined output (header + sorted data) into a new file called maize_sorted.txt. 
14.	Excludes the lines that start with #, extract lines from second column, sort in ascending order, count how many times each unique values appear in the second column. 
15.	Makes two working directory 
16.	Loops from i=1 to i=10, extract the first line from the line, (awk -v chr=”I”) sets the chr variable in awk to the current value of i ($2 == chr’) check if value in second column is equal to chr (current value of i). The filtered rows (awk going line by line) along with header are saved in Maize_chromo{i}.txt. All files are then moved into Maize_Increasing directory. 
17.	Extract the header and filter rows based on multiple and unknown from column 2. 
18.	Move both files into working directory. 
19.	t reads the maize_sorted.txt file, replaces all “?” with “-“globally and saved into maize-sortex.txt. 
20.	Move maize-sorted file into Maize_Decreasing 
21.	Changing directory 
22.	Same as 13 but sort in decreasing order from MDsorted.txt file. 
23.	For each chromosome ($i from 1 to 10), the script filters the sorted data (MDsorted.txt) to include any rows where the second column matches the chromosomes number and writes the result into new file named Maize_Decreasing_sorted{i}.txt. echo command prints the number of lines in each new file using the wc -l command, which count the number of lines in the file 



### Teosinte Data

```
1.	awk '{for(i=1;i<=NF;i++) if ($i == "ZMPBA") print "Found at column " i}' transjoin.txt
2.	awk '{for(i=1;i<=NF;i++) if ($i == "ZMPJA") print "Found at column " i}' transjoin.txt
3.	awk '{for(i=1;i<=NF;i++) if ($i == "ZMPIL") print "Found at column " i}' transjoin.txt
4.	cut -d $'\t' -f1,3,4,89-988,1178-1218,989-1022 transjoin.txt  > Teosinte.txt
5.	(head -n 1 Teosinte.txt && tail -n +2 Teosinte.txt | sort -k3,3n) > Teosinte_sorted.txt
6.	grep -v "^#" Teosinte_sorted.txt | cut -f2 | sort | uniq -c
7.	{ head -n 1 Teosinte_sorted.txt && awk -v chr="$i" '$2 == chr' Teosinte_sorted.txt; } > "Teosinte_chromo${i}.txt"
done
8.	mv Teosinte_chromo*.txt Teosinte/
9.	{ head -n 1 Teosinte_sorted.txt && awk '$2 == "multiple"' Teosinte_sorted.txt; } > Teosinte_chromoM.txt && \
{ head -n 1 Teosinte_sorted.txt && awk '$2 == "unknown"' Teosinte_sorted.txt; } > Teosinte_chromoU.txt
10.	mv Teosinte_chromoM.txt Teosinte_chromoU.txt Teosinte/
11.	sed 's/?/-/g' Teosinte_sorted.txt > Teosinte-sorted.txt
12.(head -n 1 Teosinte-sorted.txt && tail -n +2 Teosinte-sorted.txt | sort -k3,3nr) > TDsorted.txt
13. for i in {1..10}; do
{ head -n 1 TDsorted && awk -v chr="$i" '$2 == chr' TDsorted; } > "Teosinte_chromo${i}.txt"
done
```


Here is my brief description of what this code does

1. This awk command loops through every field in each line ($i represents each column) of transjoin.txt to check if the value in the column is “ZMPBA”. If it finds this string, it prints the message "Found at column" followed by the column number (i).
2. This works similarly to the previous awk command but looks for the string “ZMPJA” instead.
3. This awk command searches for the string “ZMPIL” in all columns of transjoin.txt and prints the column number where it finds the string.
4. Extracts important data (from the specified columns) into a new file called Teosinte.txt.
5. Sorts the data in Teosinte.txt based on the 3rd column in numerical ascending order and retains the header in the sorted file.
6. This helps to count how many times each unique value appears in the 2nd column of Teosinte_sorted.txt, excluding any comment lines.
7. This separates the data into different files based on chromosomes 1 through 10.
8. This moves all files starting with Teosinte_chromo into the directory Teosinte/.
9. The first block extracts rows where the 2nd column is “multiple” and saves them 
to Teosinte_chromoM.txt.The second block extracts rows where the 2nd column is “unknown” and saves them to Teosinte_chromoU.txt.
10. This moves the files Teosinte_chromoM.txt and Teosinte_chromoU.txt into the Teosinte/ directory for organization.
11. The sed command performs a substitution (s/?/-/g), replacing all occurrences of ? with - globally throughout the Teosinte_sorted.txt file.
12. This sorts the data by the 3rd column in descending order while keeping the header intact.
13. Similar to step 7, but working with the TDsorted.txt file. It extracts rows for chromosomes 1 to 10 and saves them into separate files, e.g., Teosinte_chromo1.txt, Teosinte_chromo2.txt, etc.
