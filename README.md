# BCB546X_Unix_Assignment

## File Inspection
  * inspecting the size of 2 original files 

    ```
    du -h fang_et_al_genotypes.txt snp_position.txt
    ```
  * check how many lines in the 2 files
     
     ```
    wc -l fang_et_al_genotypes.txt snp_position.txt
     ```
  * check how many columns in the 2 files

     ```
     awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt 
     
     awk -F "\t" '{print NF; exit}' snp_position.txt
     
     ```
    
    
## Data processing     
  * 1 extract column 1 (SNP ID), column 3 (chromosome location) and column 4 (nucleotide location) from snp_positions.txt file and save these information into a new file
  
  ```
  cut -f 1,3,4 snp_position.txt > snp_column134.txt 
 
  ```
  * 2 extract maize and teosinte data from fang_et_al_genotypes.txt and save these data into 2 seperate files
  
  ```
  grep "ZMM[ILLRMR]" fang_et_al_genotypes.txt > maize_genotypes.txt
  
  grep "ZMP[BAILJA]" fang_et_al_genotypes.txt > teosinte_genotypes.txt
  ```
  * 3 when I creat the 2 genotype (maize and teosinte) files in step 2, I didn't include the first line from the original fang_et_al_genotypes.txt file. The SNP_ID information is included in this first line and will be used for later joining of snp file with genotype file. So I will extract the first line from fang_et_al_genotypes.txt and save it as a seperate file
  
  ```
  head -n 1 fang_et_al_genotypes.txt > firstline_genotypes.txt
  ```
  
  * 4 cut or copy the first line and paste it to maize and teosinte genotype files by using vi (open files),dd (cut) and P (paste)
  
  ```
  vi firstline_genotypes.txt maize_genotypes.txt teosinte_genotypes.txt
  
  dd
  
  P
  ```
  
  * 5 transpose the 2 newly created genotype files to convert columns to rows
   
   ```
   awk -f transpose.awk maize_genotypes.txt > transposed_maize.txt
   
   awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte.txt
   
   ```
   
  * 6 before joining snp file and maize and teosinte genotype files, count lines (rows) in 3 files to see if they match: 984 (snp file), 986 (maize) and 986 (teosinte), respectively

  ```
  wc -l snp_column134.txt transposed_maize.txt transposed_teosinte.txt
  ```
  
  * 7 since the lines in 3 files don't match, it may casue failure of joining files. Use "head" command to check the file and find the headers in 3 files casue the unmatched line numbers

  ```
  head -n 5 snp_column134.txt 
  
  cut -f 1 transposed_maize.txt | head -n 5
  
  cut -f 1 transposed_teosinte.txt | head -n 5
  ```
  
  * 8 use vi, and dd command to open and delete the headers in 3 files, respectly

  ```
  
  vi snp_column134.txt transposed_maize.txt transposed_teosinte.txt
  
  dd 
  
  3dd
  
  ```
 
 * 9 count the rows in 3 files again to make sure the I did everyting correctly in the last step (983 lines for every single file). Also "head" and "tail" the files to double check the information in the first column of 3 fiels match with each other for later joining.
  
  ```
  wc -l snp_column134.txt transposed_maize.txt transposed_teosinte.txt

  (head -n 5; tail -n 5) < snp_column134.txt
  
  cut -f 1 transposed_maize.txt | (head -n 5; tail -n 5)
  
  cut -f 1 transposed_teosinte.txt | (head -n 5; tail -n 5)
  ```
  
 * 10 combine snp file and maize and teosinte genotype data, respectively
  
  ```
  join -1 1 -2 1 -t $'\t' snp_column134.txt transposed_maize.txt > joined_maize.txt 
  
  join -1 1 -2 1 -t $'\t' snp_column134.txt transposed_teosinte.txt > joined_teosinte.txt
  ```
  
 * 11 from this step, same commands are used to creat sorted maize and teosinte files. So, I will use maize as the example to illustrate what I have done. 
  
 * 12 grep "unknown" and "multiple" data from joined_maize.txt file and save them into 2 seperate files. 
  
  ```
  grep "unknown" joined_maize.txt > SNP_unknownpositions_maize.txt | cut -f 1-3 | head -n 5
  
  grep "multiple" joined_maize.txt > SNP_multiplepositions_maize.txt | cut -f 1-3 | head -n 5
  ```
  
 * 13 creat maize file without "unknown" and "multiple" data and sort it based on column 2 (chromosome) and column 3 (increasing position)
 
 ```
 grep -v "unknown" joined_maize.txt | grep -v "multiple" | sort -k2, 2V -k3,3n > sort_maize.txt
 
 ```
 
 * 14 extract data from the sorted maize file according to chromosome number. I tried the "for" loop command, but it didn't work. So I kind of manually extract these data using "awk". Before doing awk, I use "sort" and "uniq -c" command to list how much lines I need to extract for each chromosome
  chromosome 1: 155
  chromosome 2: 126
  chromosome 3: 107
  chromosome 4: 88
  chromosome 5: 122
  chromosome 6: 73
  chromosome 7: 96
  chromosome 8: 62
  chromosome 9: 57
  chromosome 10: 53
 
  ```
  cut -f 2 sort_maize.txt | sort -k2,2V | uniq -c
  
  awk 'NR >= 1 && NR <= 155' sort_maize.txt > chromosome1_maize.txt
  ```
 * 15 to validate the extraction in step 14 is correct, use "wc -l" for all the 10 chromosome files to see if the number match the "sort" and "uniq -c" number
 
 ```
 wc -l chromosome*
 
 ```
 * 16 replace the "?" with "-" globally and re-sort the file based on column 2 (chromosome) and column 3 (decreasing positions)

 ```
 sed 's/?/-/g' sort_maize.txt | sort -k2,2V -k3,3nr > replace_sort_maize.txt
 
 ```
 * 17 repeat what I did in step 14-15
 
 * 18 repeat step 12-17 to sort teosinte data
 