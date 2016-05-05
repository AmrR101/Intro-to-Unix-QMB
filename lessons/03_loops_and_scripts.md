---
title: "The Shell: Loops & Scripts"
author: "Bob Freeman, Mary Piper, Radhika Khetani"
date: "Thursday, May 5, 2016"
---

Approximate time: 60 minutes

## Learning Objectives

* Learn how to operate on multiple files 
* Capture previous commands into a script to re-run later
* Abstract your script for flexibility
* Write a series of scripts, that are increasingly more flexible, to automate your workflow

Now that you've been using quite a number of commands to interrogate your data, 
wouldn't it be great if you could do this for each set of data that comes in, without having to manually re-type the commands?
Welcome to the beauty and purpose of shell scripts, looping and lopping with shell scripts! Read on!

## Shell scripts

We are finally ready to see what makes the shell such a powerful programming environment. We are going to take the commands we repeat frequently and save them in files so that we can re-run all those operations again later by typing a single command. For historical reasons, a bunch of commands saved in a file is usually called a shell script, but make no mistake: these are actually small programs.

Shell scripts are text files that contain commands we want to run over and over again, and usually have the extension `.sh`. Let's write a shell script that tells us what our current working directory is and then lists the contents of the directory. First open a new file using nano:

	$ nano listing.sh
	
Then type in the following lines in the `listing.sh` file:

	echo "Your current working directory is:"
	pwd
	echo "These are the contents of this directory:"
	ls -l 

Close nano and save the file. Now let's run the new script we have created. To run a shell script you usually use the `bash` or `sh` command.

	$ sh listing.sh
	
> Did it work like you expected?
> 
> Were the `echo` commands helpful in letting you know what came next?

This is a very simple shell script. In this session and in upcoming sessions, we will be learning how to write more complex ones, and use the power of scripts to make our lives much easier.

## Loops and bash variables

Another powerful concept in the Unix shell is the concept of "Loops".

Looping is a concept shared by several programming languages, and its implementation in bash is very similar to other languages. Let's dive right in!

`$ cd ~/unix_workshop/raw_fastq`

The structure or the syntax of (*for*) loops in bash is as follows:

```
$ for (variable_name) in (list)
> do
>   (command $variable_name) 
> done
```

where the ***variable_name*** defines (or initializes) a variable (see below) that takes the value of every member of the specified ***list*** one at a time. The loop, using the value in the variable then runs through the commands indicated between the `do` and `done` one at a time. *This syntax/structure is virtually set in stone.* 

For example:

```
$ ls -l  *fq		# list in long form all files ending in .fq

$ for var in *fq
> do
> 	echo $var
>   wc -l $var
> done
```

####What does this loop do? 
Most simply, it writes to the terminal (`echo`) the name of the file and the number of lines (`wc -l`) for all files that end in `.fq` in the current directory.

In this case the list of files is specified using the asterisk wildcard: `*.fq`, i.e. all files that end in `.fq`. Then, we execute 2 commands between the `do` and `done`. With a loop, we execute these commands for each file at a time. Once the commands are executed for one file, the loop then executes the same commands on the next file in the list. 

Essentially, **the number of loops == the number of items in the list**, in our case that is 6 times since we have 6 files in `~/unix_workshop/raw_fastq` that end in `.fq`. This is done by changing the value of the `var` variable 6 times. 

####What is a "Variable"?
Just like loops, *variable* is a common concept shared by many programming languages. Variables are essentially a symbolic/temporary name for, or reference to, information. Variables are analogous to "buckets", where information can be maintained and referenced, and modified without too much hassle. 

Extending the bucket analogy: the bucket has a name associated with it, i.e. the name of the variable (`var` in the above example), and when referring to the information in the bucket, we use the name of the bucket, and do not directly refer to the actual data stored in it.

In the example above, we defined a variable or a 'bucket' called "var". We put the file names (values) inside it, one at a time.

#####Using variables
You may have noticed in the `for` statement we define the variable with the name **var**. But in the loop, we explicitly use a "$" in front of it (`$var`). Why? 

Well, in the former, we're setting the value, while in the latter, we're retrieving the value. This is standard shell notation (syntax) for defining and using variables. Don't forget the `$` when you want to retrieve the value of a variable! 

Of course, `var` is a useless variable name. But it doesn't matter what variable name we use and we can make it something more intuitive:

```bash
$ for filename in *.fq
> do
>   echo $filename
> done
```
In the long run, it's best to use a name that will help point out its function, so your future self will understand what you are thinking now.

Now that we understand the concept of looping, let's put that to work:

```bash
$ for filename in *.fq
> do 
>   echo $filename >> ../other/number-of-badreads.txt
>   grep -B1 -A2 NNNNNNNNNN $filename | wc -l >> ../other/number-of-badreads.txt
>   grep -B1 -A2 NNNNNNNNNN $filename > ../other/$filename.badreads.fastq 
> done
```
Now we have used the for loop along with the `>>` redirection symbol to populate one file with all the bad reads in our data set.

Pretty simple and cool, huh?

## Automating with Scripts

Now that you've learned how to use loops and variables, let's put this processing power to work. Imagine, if you will, a series of commands that would do the following for us each time we get a new data set:

- Use for loop to iterate over each FASTQ file
- Dump out bad reads into a new file
- Get the count of the number of bad reads
- And after all the FASTQ files are processed, we generate one summary file of the bad read counts

You might not realize this, but this is something that you now know how to do. Let's get started...

Move to our sample data directory and use `nano` to create our new script file:

`$ cd ~/unix_workshop/raw_fastq`

`$ nano generate_bad_reads_summary.sh`

We always want to start our scripts with a shebang line: 

`#!/bin/bash`

This line is the absolute path to the Bash interpreter. The shebang line ensures that the bash shell interprets the script even if it is executed using a different shell.

After the shebang line, we enter the commands we want to execute. First we want to move into our `raw_fastq` directory:

```
$ cd ~/unix_workshop/raw_fastq
```

And now we loop over all the FASTQs:

```bash
for filename in ~/unix_workshop/raw_fastq/*.fq;
```

and we execute the commands for each loop:

```bash
do
  # tell us what file we're working on
  echo $filename;
  
  # grab all the bad read records into new file
  grep -B1 -A2 NNNNNNNNNN $filename > $filename-badreads.fastq;
``` 
  
We'll also get the counts of these reads and put that in a new file, using the count flag of `grep`:

```bash
# grab the # of bad reads from our badreads file
grep -cH NNNNNNNNNN $filename-badreads.fastq > $filename-badreads.counts;
done
```

If you've noticed, we slipped a new `grep` flag `-H` in there. This flag will report the filename along with the match string. This won't matter within each file, but it will matter when we generate the summary:

```bash
# grab all our bad read info to a summary file
cat *.counts > bad-reads.count.summary
```

And now, as a best practice of capturing all of our work into a running summary log:

```bash
# and add this summary to our run log
cat bad-reads.count.summary >> ../runlog.txt
```

You're script should look like:

```bash
#!/bin/bash

cd ~/unix_workshop/raw_fastq

for filename in ~/unix_workshop/raw_fastq/*.fq; do 
echo $filename;
grep -B1 -A2 NNNNNNNNNN $filename > $filename-badreads.fastq;
grep -cH NNNNNNNNNN $filename-badreads.fastq > $filename-badreads.counts;
done

cat *.counts > bad-reads.count.summary

cat bad-reads.count.summary >> ../runlog.txt

```

Exit out of `nano`, and voila! You now have a script you can use to assess the quality of all your new datasets. Your finished script, complete with comments, should look like the following:

```bash
#!/bin/bash 

# enter directory with raw FASTQs
cd ~/unix_workshop/raw_fastq

# count bad reads for each FASTQ file in our directory
for filename in ~/unix_workshop/raw_fastq/*.fq; do 
  echo $filename; 

  # grab all the bad read records
  grep -B1 -A2 NNNNNNNNNN $filename > $filename-badreads.fastq;

  # grab the # of bad reads from our bad reads file
  grep -cH NNNNNNNNNN $filename-badreads.fastq > $filename-badreads.counts;
done

# add all our bad read info to a summary file
cat *.counts > bad-reads.count.summary

# and add this summary to our run log
cat bad-reads.count.summary >> ../runlog.tx

```

To run this script, we simply enter the following command:

```bash
$ bash generate_bad_reads_summary.sh
```

To keep your data organized, let's move all of the bad read files out of our `raw_fastq` directory into the `other` directory

`$ mv ~/unix_workshop/raw_fastq/*bad* ~/unix_workshop/other`


---
*The materials used in this lesson was derived from work that is Copyright © Data Carpentry (http://datacarpentry.org/). 
All Data Carpentry instructional material is made available under the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0).*

