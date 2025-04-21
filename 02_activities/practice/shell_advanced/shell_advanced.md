#!/bin/bash

### Task 1: Setup your environment
#1. Download their undergraduate data package: [dataset.zip](dataset.zip?raw=1)
curl -Lo dataset.zip https://github.com/NTDuongLe/shell/blob/main/02_activities/practice/shell_advanced/dataset.zip

#2. Unzip it into your directory
unzip -q dataset.zip

#3. Change directory into the folder containing the data package contents
cd ./dataset

#4. To make sure we're keeping good records this time, print the current path to your working directory
pwd

#5. Make a new folder named `tidied_data`
mkdir tidied_data

### Task 2: Taking inventory
#1. List the contents of the data folder 
    #DL: does the instruction mean 'dataset' folder?
ls ./dataset

#2. List all the EEG files and write this list to a text file in the tidied_data folder named `eeg_inventory.txt`. 
##*Hint*: Consider the `ls` and `head` commands, and either the `>` file redirection operator or `|` pipe operator and the `tee` command
ls ./dataset/*eeg* | xargs -n 1 basename > ./tidied_data/eeg_inventory.txt
## Preview the **first 8 lines** of this file using the `head` command, does this look right?
cat | head -n 8 eeg_inventory.txt

### Task 3: Taking inventory, part 2
#1. Let's double check: does the naming convention of the EEG files match `eeg_[subjectid]_[session].edf`?
    #* List the contents of the folder and inspect it visually!
        #DL: does the instruction mean inspect manually or via command? 
            #possible approach using commands: 
                # count number of lines in egg_inventory_txt: wc -l ./tidied_data/eeg_inventory.txt
                # list all the files matching the name format to a check_name_format.txt file: 
                    # ls ./dataset | grep -E '^eeg_[0-9]+_[0-9]+\.EDF$' > ./dataset/check_name_format.txt
                # count number of lines in check_name_format.txt: wc -l ./dataset/check_name_format.txt
                # if the line counts of both files match, the file name matches the required format


#2. Based on the `eeg_inventory.txt` file and the naming convention, generate a list of subject numbers and write it a file named `subject_ids_from_eeg.txt`. 
    #* Preview the **first 8 lines** of this file.
    #* *Hint*: Consider the `cut` command
cut -d '_' -f 2 ./tidied_data/eeg_inventory.txt > ./tidied_data/subject_ids_from_eeg.txt
cat | head -n 8 ./tidied_data/subject_ids_from_eeg.txt

#3. Sort the `subject_ids_from_eeg.txt` file and write the output to `subject_ids_from_eeg_sorted.txt`. 
    #* Preview the **first 8 lines** of this file.
sort -n ./tidied_data/subject_ids_from_eeg.txt > ./tidied_data/subject_ids_from_eeg_sorted.txt
cat | head -n 8 ./tidied_data/subject_ids_from_eeg_sorted.txt

#4. Let's double check: does each subject have multiple EEG files? Are their subject IDs duplicated? 
    #*Hint*: Look through the whole with the `less` command
        #DL: yes, there are duplicate subject IDs
cat ./tidied_data/subject_ids_from_eeg_sorted.txt | less

#5. Create a unique list of subject IDs and write the output to `subject_ids.txt`. 
    #* *Hint*: Consider the `unique` command 
        #DL: 'unique' command not found error. Is it typo or is it the alias set up on the instructor's shell app?
    #* Preview the **first 8 lines** of this file.
uniq ./tidied_data/subject_ids_from_eeg_sorted.txt > ./tidied_data/subject_ids.txt
cat | head -n 8 ./tidied_data/subject_ids.txt

### Task 4: The life changing magic of tidying up
#1. For each subject:
    #1. create a folder named after the subject ID in the `tidied_data` directory
        #* *Hint*: Consider a `for` loop
            # DL: only works if .txt file contains one entry per line without spaces in the names, if space is present in a line treated as 2 entries
for id in $(cat ./tidied_data/subject_ids.txt); do
  mkdir -p "./tidied_data/$id"
done
        #DL: check number of folders created that match with number of subject_id in .txt file
find ./tidied_data/ -mindepth 1 -type d | wc -l

    #2. Move all files relating to that subject into their respective directories (eg. `tidied_data/[subjectid]/eeg_[subjectid]_[session].edf`)
        #* *Hint*: Consider the `mv` command
for id in $(cat ./tidied_data/subject_ids.txt); do
    mv ./dataset/*$id* ./tidied_data/$id
done

#2. Notice that all the notes files have not been named consistently. Rename all the note files to `[subjectid]_notes.txt` within each subject folder.
    #* *Hint*: Consider the `mv` command inside a `for` loop or using `find ... -exec mv ...`
for id in $(cat ./tidied_data/subject_ids.txt); do
    mv ./tidied_data/$id/*$id*_*notes*.txt ./tidied_data/$id/${id}_notes.txt 
done
        #DL: approach in case there are >1 note file for the same subject ID
for id in $(cat ./tidied_data/subject_ids.txt); do
    cat ./tidied_data/$id/*$id*_*notes*.txt > ./tidied_data/$id/${id}_notes.txt && rm ./tidied_data/$id/*$id*_*notes*_*.txt
done     
        #DL: test for some folders in a separate test folder before applying to all 
mkdir test
cp -r ./tidied_data/{1013,1117,1122} ./test

            # Check if code works before removing file, if echo message does not appear meaning does not work properly. 
            # Be extremely careful with variable, bash treat $id_notes as a variable and not just $id as what I expected. 
                # So when $id_notes is not defined, bash will create a "(blank).txt" file with cat command
                # Troubleshoot: ${id}

for id in {1013,1117,1122}; do
    cat ./test/$id/*$id*_*notes*_*.txt > ./test/$id/${id}_notes.txt && echo "Would remove: rm ./test/$id/*$id*_*notes*_*.txt"
done 
            # Remove echo once sure that code works
for id in {1013,1117,1122}; do
    cat ./test/$id/*$id*_*notes*_*.txt > ./test/$id/${id}_notes.txt && rm ./test/$id/*$id*_*notes*_*.txt
done 

### Task 5: Checking our work
#1. Confirm that you've copied all the files over to the `tidied_data` directory
    #1. Count the number of files copied:
        #1. Generate a list of all the files within all the directories in `tidied_data` 
            #* *Hint*: Consider the `find` command and look for files with the `-type` option
                #DL: move all the .txt files that are not in the original data set (due to above tasks) to outside of tidied_data folder before generate file_list
find ./tidied_data -type f > file_list_after_tidy.txt
        #2. Count the number of lines in the file list
            #* *Hint*: Consider the `wc` command. How do we make it count lines?
wc -l .file_list_after_tidy.txt

    #2. Count the number of files in the original folder, using the same strategy as above
        #DL: since the all the files have been moved from dataset to tidied_data, have to unzip again
unzip -q dataset.zip
find ./dataset -type f > file_list_before_tidy.txt
wc -l .file_list_before_tidy.txt
        
    #2. Do the file counts match?
    #* *Hint*: Consider using variables, an `if` statement, and `-eq`
a=$(wc -l < file_list_before_tidy.txt)
b=$(wc -l < file_list_after_tidy.txt)

if [ "$a" -eq "$b" ]; then
    echo "File count match and complete file tidy-up process!"
fi