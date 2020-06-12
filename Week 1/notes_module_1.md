Linux Tools for Developers
==========================

by The Linux Foundation

# Week 1
#
## Title: Essential Command Line Tools

#### Tools Used to List, Create, Delate and Rename Files and Directories

* **ls**  -->  List files
* **cat**  -->  Type out (concatenate) the files
* **rm**  -->  Remove files
* **mv**  -->  Rename (move) files
* **mkdir**  -->  Create directories
* **rmdir**  -->  Remove directories
* **file**  -->  Show file types
* **ln**  -->  Create symbolic and hard links
* **tail**  -->  Look at the tail end of the file
* **head**  -->  Look at the beginning of the file
* **less**  -->  Look at the file, one screenful at a time
* **more**  -->  Look at the file, one screenful at a time
* **touch**  -->  Either create an empty file, or update the file modification time
* **wc**  -->  Count lines, words, and bytes in a file


#### Finding Files: find and locate

##### find

* The **find** command line utility provides an extremely powerful and flexible method for locating files based on their properties, including name
* It does not search the interior of files for patterns, etc.; that is more the province of **grep** and its variations
* Syntax
	```bash
	$ find [location] [criteria] [actions]
	```
	* where there are three classes of arguments, each of which may be omitted
		* If no location is given, the current directory (.) is assumed
		* if no criteria are given, all files are displayed
		* if no actions are given, only a listing of the names is given.
* Example
	1. Sample cmd
		```bash
		$ find /etc -name "*.conf"
		```
		* will print out the names of all files in the /etc directory and its descendants, recursively, that end in .conf
	1. Sample cmd
		```bash
		$ find . -name "*~" -exec rm {} ’;’
		```
		* where {} is a fill in for the files to be operated on, and ’;’ indicates the end of the command
	1. Sample cmd
		```bash
		$ find . -name "*~" | xargs rm
		``` 
		* Performs same action as above command
	1. Sample cmd
		```bash
		$ for names in $(find . -name "*~" ) ; do rm $names ; done
		```
		* Performs same action as above command

##### locate

* Another method of locating files is provided by the **locate** command, which searches your entire filesystem (except for paths which have been excluded) and works off a database that is updated periodically with **updatedb**. Thus, it is very fast.
	* Example
		```bash
		$ locate .conf
		```
		* will find all files on your system that have .conf in them
* **locate** will only find files that were already in existence the last time the database was updated. On most systems, this is done by a background cron job, usually daily. To force an update, you need to do 
	```bash
	$ sudo updatedb
	```

##### grep Command

* **grep** is a workhorse command line utility whose basic job is to search files for patterns and print out matches according to specified options.
* Its name stands for **g**lobal **r**egular **e**xpression **p**rint, which points out that it can do more than just match simple strings
	* it can work with more complicated regular expressions which can contain wildcards and other special attributes
* Sample
	```bash
	$ grep pig file
	$ grep -i -e pig -e dog -r .
	
	# print all lines that start with "dog"
	$ grep "^dog" file
	
	# print all lines that end with "dog"
	$ grep "dog$" file
	
	# print all lines that end with "dog"
	$ grep d[a-p] file

	# restrict to those that do not use the tcp protocol, while printing out the line number
	$ grep -n ftp /etc/services | grep -v tcp
	```

* **grep** has many options; some of the most important are
	* **`-i`**  -->  Ignore case
	* **`-v`**  -->  Invert match
	* **`-n`**  -->  Print line number
	* **`-H`**  -->  Print filename
	* **`-a`**  -->  Treat binary files as text
	* **`-I`**  -->  Ignore binary files
	* **`-r`**  -->  Recurse through subdirectories
	* **`-l`**  -->  Print out names of all files that contain matches
	* **`-L`**  -->  Print out names of all files that do not contain matches
	* **`-c`**  -->  Print out number of matching lines only
	* **`-e`**  -->  Use the following pattern; useful for multiple strings and special characters

##### sed Command

* **sed** stands for **S**tream **ED**itor
* Its job is to make substitutions and other modifications in files and in streamed output.
* Any of the following methods will change all first instances of the string "pig" with "cow" for each line of file, and put the results in **newfile**
	```bash
	$ sed s/pig/cow/ file > newfile
	$ sed s/pig/cow/ < file > newfile
	$ cat file | sed s/pig/cow/ > newfile
	```
* More Examples
	1. the s stands for substitute. If you want to change all instances, you have to add the g (global) qualifier
	```bash
	$ sed s/pig/cow/g file > newfile
	```
	1. The / characters are used to delimit the new and old strings. You can choose to use another character, as in:
	```bash
	$ sed s:pig:cow:g file > newfile
	```
	1. suppose you want to replace all back slashes with forward slashes:
	```bash
	$ sed s/’\\’/’\/’/g file > newfile
	```
	1. If you want to make multiple simultaneous substitutions, you need to use the -e option, as in
	```bash
	$ sed -e s/"pig"/"cow"/g -e s/"dog"/"cat"/g < file > newfile
	```
	1. If you have a lot of commands, you can put them in a file and apply the -f option, as in
	```bash
	$ cat scriptfile

	s/pig/cow/g
	s/dog/cat/g
	s/frog/toad/g

	$ sed -f scriptfile < file > newfile
	```

#
## Title: File and Text Manipulation Utilities

#### Command Line Tools for Manipulating Text Files

> tail prints the last few lines of each named file and displays it on standard output

* The basic Linux command line utilities are
	1. **`cat`** --> simply takes whatever is in a file and displays it on the screen or adds it to another file
	1. **`echo`** --> lets you type something, and then it's echoed to the screen, or added to a file
	1. **`head`** --> show the beginning of files
	1. **`tail`** --> show the end of files
	1. **`sed`** -->  stands for stream editor that's used to substitute strings within files
	1. **`awk`** --> it can do a lot of things in any file that has a series of fields in it
 
 
##### cat

* **cat** is short for **concatenate** and is one of the most frequently used Linux command line utilities
* It is often used to read and print files, as well as for simply viewing file contents
* Syntax
	```bash
	$ cat <filename>
	```
* For example, **cat readme.txt** will display the contents of readme.txt on the terminal
* The main purpose of cat is often to combine (concatenate) multiple files together

##### tac

* The **tac** command (cat spelled backwards) prints the lines of a file in **reverse order**. Each line remains the same, but the order of lines is inverted
* The syntax of **tac** is exactly the same as for cat, as in
* Syntax
	```bash
	$ tac file
	$ tac file1 file2 > newfile
	```
* Some utilities of **cat** command
	* **`cat file1 file2`**  -->  Concatenate multiple files and display the output; i.e. the entire content of the first file is followed by that of the second file
	* **`cat file1 file2 > newfile`**  -->  Combine multiple files and save the output into a new file
	* **`cat file >> existingfile`**  -->  Append a file to the end of an existing file
	* **`cat > file`**  -->  Any subsequent lines typed will go into the file, until Ctrl-D is typed
	* **`cat >> file`**  -->  Any subsequent lines are appended to the file, until Ctrl-D is typed

##### echo

* **echo** displays (echoes) text
* Syntax
	```bash
	$ echo string
	```
* **echo** can be used to 
	* display a string on standard output (i.e. the terminal)
	* to place in a new file (using the > operator)
	* append to an already existing file (using the >> operator)
* The `-e` option, along with the following switches, is used to enable special character sequences, such as the newline character or horizontal tab
	* `\n` represents newline
	* `\t` represents horizontal tab
* **echo** is particularly useful for viewing the values of environment variables (built-in shell variables)
* Example
	* **`echo $USERNAME`** will print the name of the user who has logged into the current terminal
* The following lists echo commands and their usage
	* **`echo string > newfile`**  -->  The specified string is placed in a new file
	* **`echo string >> existingfile`**  -->  The specified string is appended to the end of an already existing file
	* **`echo $variable`**  -->  The contents of the specified environment variable are displayed

#### Working with Large Files

##### less

* One can use **less** to view the contents of a large file, scrolling up and down page by page, without the system having to place the entire file in memory before starting; this is much faster than using a text editor
* Viewing somefile can be done by typing either of the two following commands
	```bash
	$ less somefile
	$ cat somefile | less
	```
	* Both commands mentioned above give same output
* By default, man pages are sent through the less command
* Older **more** utility which has the same basic function but fewer capabilities, i.e. less is more!

##### head

* **head** reads the first few lines of each named file (10 by default) and displays it on standard output
* You can give a different number of lines in an option
* Example
	* If you want to print the first 5 lines from grub.cfg, use the following command
	```bash
	$ head –n 5 grub.cfg
	```

##### tail

* **tail** prints the last few lines of each named file and displays it on standard output
	* By default, it displays the last 10 lines
	* We can give a different number of lines as an option
* **tail** is especially useful when you are troubleshooting any issue using log files, as you probably want to see the most recent lines of output
* Example
	1. To display the last 15 lines of somefile.log, use the following command
		```bash
		$ tail -n 15 somefile.log
		```
	1. To continually monitor new output in a growing log file
		```bash
		# This command will continuously display any new lines of output in somefile.log as soon as they appear
		# Thus, it enables you to monitor any current activity that is being reported and recorded
		$ tail -f somefile.log
		```

##### Viewing Compressed Files

* For many commonly-used file and text manipulation programs, there is also a version especially designed to work directly with compressed files
* These associated utilities have the letter **z** prefixed to their name
* For example, we have utility programs such as **zcat**, **zless**, **zdiff** and **zgrep**
* Here is a list of some **z** family commands
	* **`$ zcat compressed-file.txt.gz`**  -->  To view a compressed file
	* **`$ zless somefile.gz or $ zmore somefile.gz`**  -->  To page through a compressed file
	* **`$ zgrep -i less somefile.gz`**  -->  To search inside a compressed file
	* **`$ zdiff file1.txt.gz file2.txt.gz`**  -->  To compare two compressed files
* **Note** that if you run **zless** on an uncompressed file, it will still work and ignore the decompression stage
* There are also equivalent utility programs for other compression methods besides **gzip**
	* For example, 
		* **bzcat** and **bzless** associated with **bzip2**
		* **xzcat** and **xzless** associated with **xz**

#### Introduction to sed and awk

> **awk** was created at Bell Labs in the 1970s.

* **sed** is a stream editor, it takes an input file or an input stream that's in a pipe
	* It puts it in, what's called, a working stream, where is manipulated and the output stream is the resolve
	* In that manipulation phase you can substitute strings, you can eliminate certain strings, and do various kinds of operations
	* it's a **filter**
* **awk** is really not a filter, awk is an **extraction device**
	* Its name just comes from the name of the three authors and the initials of their last names
	* In a very logical way it is used to extract some of the contents specially a files which have columns. You work on a column basis, extract the contents and do various operations on them

##### sed Command Syntax and Basic Operations

* We can invoke sed using commands like those listed below
	* **`sed -e command <filename>`**  -->  Specify editing commands at the command line, operate on file and put the output on standard out (e.g. the terminal)
	* **`sed -f scriptfile <filename>`**  -->  Specify a scriptfile containing sed commands, operate on file and put output on standard out
* The `-e` command option allows you to specify multiple editing commands simultaneously at the command line
	* It is unnecessary if you only have one operation invoked
* Some basic operations, where pattern is the current string and replace_string is the new string
	* **`sed s/pattern/replace_string/ file`**  -->  Substitute first string occurrence in every line
	* **`sed s/pattern/replace_string/g file`**  -->  Substitute all string occurrences in every line
	* **`sed 1,3s/pattern/replace_string/g file`**  -->  Substitute all string occurrences in a range of lines
	* **`sed -i s/pattern/replace_string/g file`**  -->  Save changes for string substitution in the same file
* Sample
	1. Command will replace all occurrences of pattern with replace_string in file1 and move the contents to file2
		* The contents of file2 can be viewed with **cat file2**
	```bash
	$ sed s/pattern/replace_string/g file1 > file2
	```
	1. To convert **01/02/… to JAN/FEB/…**
	```bash
	$ sed -e 's/01/JAN/' -e 's/02/FEB/' -e 's/03/MAR/' -e 's/04/APR/' -e 's/05/MAY/' \
    -e 's/06/JUN/' -e 's/07/JUL/' -e 's/08/AUG/' -e 's/09/SEP/' -e 's/10/OCT/' \
    -e 's/11/NOV/' -e 's/12/DEC/'
    ```

##### awk Command Syntax and Basic Operations

* As with sed, short awk commands can be specified directly at the command line, but a more complex script can be saved in a file that you can specify using the -f option
	* **`awk 'command' file`**  -->  Specify a command directly at the command line
	* **`awk -f scriptfile file`**  -->  Specify a file that contains the script to be executed
* The input file is read one line at a time, and, for each line, awk matches the given pattern in the given order and performs the requested action
* The `-F` option allows you to specify a particular **field separator character**
* For example
	* the `/etc/passwd` file uses **:** to separate the fields, so the `-F:` option is used with the `/etc/passwd` file.
* The command/action in awk needs to be surrounded with apostrophes (or single-quote ('))
	* **`awk '{ print $0 }' /etc/passwd`**  -->  Print entire file
	* **`awk -F: '{ print $1 }' /etc/passwd`**  -->  Print first field (column) of every line, separated by a space
	* **`awk -F: '{ print $1 $7 }' /etc/passwd`**  -->  Print first and seventh field of every line

#### File Manipulation Utilities

> uniq removes duplicate consecutive lines in a text file and is useful for simplifying the text display

* Linux provides several file manipulation utilities that you can use while working with text files, including
	* **`sort`** --> It sorts the file. It has many options
	* **`uniq`** --> It'll look at a file and remove duplicate lines and replace them with single lines
	* **`paste`** --> work with files that have columns in them and let you reorganize them into a new file, which joins the contents of the files, lil different from join
	* **`join`** --> work with files that have columns in them and let you reorganize them into a new file, which joins the contents of the files, lil different from paste
	* **`split`** --> lets you split up a file into smaller pieces

##### sort

* **sort** is used to rearrange the lines of a text file either in ascending or descending order, according to a sort key
* You can also sort by particular fields of a file
* The **default** sort key is the order of the **ASCII characters (i.e. essentially alphabetically)**
* **sort** can be used as follows
	* **`sort <filename>`**  -->  Sort the lines in the specified file, according to the characters at the beginning of each line
	* **`cat file1 file2 | sort`**  -->  Combine the two files, then sort the lines and display the output on the terminal
	* **`sort -r <filename>`**  -->  Sort the lines in reverse order
	* **`sort -k 3 <filename>`**  -->  Sort the lines by the 3rd field on each line instead of the beginning
* When used with the `-u` option, **sort** checks for unique values after sorting the records (lines)
	* It is equivalent to running uniq (which we shall discuss) on the output of **sort**

##### uniq

* **uniq** removes duplicate consecutive lines in a text file and is useful for simplifying the text display
* Because **uniq** requires that the **duplicate entries** must be **consecutive**, one often runs **sort** first and then pipes the output into **uniq**
	* if sort is used with the -u option, it can do all this in one step
* Sample
	1. To remove duplicate entries from multiple files at once
	```bash
	$ sort file1 file2 | uniq > file3
	# OR
	$ sort -u file1 file2 > file3
	```
	1. To count the number of duplicate entries
	```bash
	$ uniq -c filename
	```

#### Introduction to paste, join, and split

> The different columns are identified based on delimiters, such as a blank space, a tab, or an Enter
#
> **split** breaks up a file into 1000 line segments by default

##### Using paste
	
* **paste** can be used to combine fields (such as name or phone number) from different files, as well as combine lines from multiple files
* For example
	* line one from file1 can be combined with line one of file2, line two from file1 can be combined with line two of file2, and so on
* Sample Commands
	1. To paste contents from two files
	```bash
	$ paste file1 file2
	```
	1. The syntax to use a different delimiter is
	```bash
	$ paste -d, file1 file2
	```
	* Common delimiters are 'space', 'tab', '|', 'comma', etc.

##### Using join

* To combine two files on a common field, at the command prompt type **join file1 file2** and press the **Enter** key
* For example
	* the common field (i.e. it contains the same values) among the phonebook and cities files is the phone number

##### Using split

* Sample Commands
	1. We will apply split to an American-English dictionary file of over 99,000 lines
	```bash
	$ wc -l american-english
	99171 american-english
	```
	* where we have used wc to report on the number of lines in the file
	1. split the American-English file into 100 equal-sized segments named **dictionaryxx**
	```bash
	$ split american-english dictionary
	```

##### Regular Expressions and Search Patterns

* **Regular expressions** are text strings used for **matching a specific pattern**, or to search for a specific location, such as the start or end of a line or a word
* **Regular expressions** can contain both normal characters, as well as so-called **meta-characters**, such as `*` and `$`
* Many **text editors** and **utilities** such as **vi**, **sed**, **awk**, **find** and **grep** work extensively with regular expressions
* Some of the popular computer languages that use regular expressions include Perl, Python and Ruby
* These **regular expressions** are different from the wildcards (or meta-characters) used in filename matching in command shells such as **bash**
* The list below shows search patterns and their usage
	* **`.(dot)`**  -->  Match any single character
	* **`a|z`**  -->  Match a or z
	* **`$`**  -->  Match end of string
	* **`^`**  -->  Match beginning of string
	* **`*`**  -->  Match preceding item 0 or more times
* For example
	* consider the following sentence: _The quick brown fox jumped over the lazy dog._
		* Some of the patterns that can be applied to this sentence are
			* **`a..`**  -->  matches azy
			* **`b.|j.`**  -->  matches both br and ju
			* **`..$`**  -->  matches og
			* **`l.*`**  -->  matches lazy dog
			* **`l.*y`**  -->  matches lazy
			* **`the.*`**  -->  matches the whole sentence

##### grep

* **grep** is extensively used as a primary text searching tool
* It scans files for specified patterns and can be used with regular expressions, as well as simple strings
* Sample usage is mentioned below
	* **`grep [pattern] <filename>`**  -->  Search for a pattern in a file and print all matching lines
	* **`grep -v [pattern] <filename>`**  -->  Print all lines that do not match the pattern
	* **`grep [0-9] <filename>`**  -->  Print the lines that contain the numbers 0 through 9
	* **`grep -C 3 [pattern] <filename>`**  -->  Print context of lines (specified number of lines above and below the pattern) for matching the pattern; here, the number of lines is specified as 3

##### strings

* **strings** is used to extract all printable character strings found in the file or files given as arguments
* It is useful in locating human-readable content embedded in binary files; for text files, you can just use **grep**
* For example
	* To search for the string my_string in a spreadsheet
	```bash
	$ strings book1.xls | grep my_string
	```

##### tr

* The **tr** utility is used to translate specified characters into other characters or to delete them
* Syntax
	```bash
	$ tr [options] set1 [set2]
	```
* **tr** requires at least one argument and accepts a maximum of two
	* The first designated **set1** in the example lists the characters in the text to be replaced or removed
	* The second, **set2**, lists the characters that are to be substituted for the characters listed in the first argument
	* Sometimes, these **sets** need to be surrounded by **apostrophes (or single-quotes ('))** in order to have the shell ignore that they mean something special to the shell
	* It is usually **safe** (and may be required) to use the **single-quotes** around each of the **sets**
* For example
	1. Suppose you have a file named city containing several lines of text in mixed case
		* To translate all lower case characters to upper case, at the command prompt type `cat city | tr a-z A-Z` and press the **Enter** key
* Sample Commands
	* **`$ tr abcdefghijklmnopqrstuvwxyz ABCDEFGHIJKLMNOPQRSTUVWXYZ`**  -->  Convert lower case to upper case
	* **`$ tr '{}' '()' < inputfile > outputfile`**  -->  Translate braces into parenthesis
	* **`$ echo "This is for testing" | tr [:space:] '\t'`**  -->  Translate white-space to tabs
	* **`$ echo "This is for testing" | tr -s [:space:]`**  -->  Squeeze repetition of characters using -s
	* **`$ echo "the geek stuff" | tr -d 't'`**  -->  Delete specified characters using -d option
	* **`$ echo "my username is 432234" | tr -cd [:digit:]`**  -->  Complement the sets using -c option
	* **`$ tr -cd [:print:] < file.txt`**  -->  Remove all non-printable characters from a file
	* **`$ tr -s '\n' ' ' < file.txt`**  -->  Join all the lines in a file into a single line

##### tee

* **tee** takes the output from any command, and, while sending it to standard output, it also saves it to a file
* In other words, it **tees** the output stream from the command:
	* one stream is displayed on the standard output
	* the other is saved to a file
* For example
	* To list the contents of a directory on the screen and save the output to a file, at the command prompt type `ls -l | tee` newfile and press the **Enter** key

##### wc

* **wc (word count)** counts the number of lines, words, and characters in a file or list of files
* Options for **wc** are mentioned below
	* **`–l`**  -->  Displays the number of lines
	* **`-c`**  -->  Displays the number of bytes
	* **`-w`**  -->  Displays the number of words
* By **default**, all three of these options are active.
* For example
	* To print only the number of lines contained in a file, type `wc -l filename` and press the **Enter** key

##### cut

* **cut** is used for manipulating column-based files and is designed to extract specific columns
* The **default** column separator is the **tab** character
	* A different delimiter can be given as a command option
* For example
	* To display the third column delimited by a blank space, at the command prompt type `ls -l | cut -d" " -f3` and press the **Enter** key

#
## Title: Bash Scripting

#### Script Basics

> `$@` preserves the argument grouping
> `$#` gives the number of arguments
> `$0` is the special environment variables that is the command name

* If **`test.sh`** is
	```bash
	#!/bin/bash
	echo 0 = $0
	echo 1 = $1
	echo ’*’ = $*
	```
	* the output of `./foobar.sh a b c d e` is:
		```bash
		0 = ./foobar
		1=a 
		*=abcde
		```
* Inside the script, the command **shift n** shifts the arguments n times (to the left)
* There are two ways to include a script file inside another script:
	* **. file**
	* **source file**
* There are a number of options that can be used for debugging purposes
	* where the **set** command is used inside the script (with a + sign behavior is reversed)
	* the second form, giving an option to bash, is invoked when running the script from the command line
	* **`set -n (bash -n)`**  -->  just checks for syntax
	* **`set -x (bash -x)`**  -->  echos all commands after running them
	* **`set -v (bash -v)`**  -->  echos all commands before running them
	* **`set -u (bash -u)`**  -->  causes the shell to treat using unset variables as an error
	* **`set -e (bash -e)`**  -->  causes the script to exit immediately upon any non-zero exit status


##### Conditionals

* **bash** permits control structures like
	```bash
	if condition
	then
	   statements
	else
	   statements
	fi
	```
* There is also an **elif** statement
* Note that the variable `$?` holds the exit status of the previous command
* Sample
	```bash
	if [[ -f file.c ]] ; then ... ; fi
	if [-ffile.c] ;then...;fi 
	if test -f file.c ; then ... ; fi
	```
* **Remember to put spaces around the `[ ]` brackets**
* The first form with double brackets is preferred over the second form with single brackets, which is now considered deprecated
	* will produce a syntax error if VAR is empty
	```bash
	$ if [ $VAR == "" ]
	```
	* to avoid above mentioned error, we can write code as
	```bash
	$ if [ "$VAR" == "" ]
	``` 
* You will often see the `&&` and `||` operators (AND and OR, as in C) used in a compact shorthand
	* Sample
	```bash
	$ make && make modules_install && make install
	$ [[ -f /etc/foo.conf ]] || echo ’default config’ >/etc/foo.conf
	```
* The `&&s` (ANDs) in the first statement say stop as soon as one of the commands fails; it is often preferable to using the `;` operator
* The `||’` (ORs) in the second statement says stop as soon as one of the commands succeeds
* The && operator can be used to do compact conditionals
	* Example
	```bash
	$ [[ $STRING == mystring ]] && echo mystring is "$STRING"
	```
	* Example equivalent to above mentioned
	```bash
	if [[ $STRING == mystring ]] ; then
	    echo mystring is "$STRING"
	fi
	```

###### File Conditionals

* There are many conditionals which can be used for various logical tests
* Doing man 1 test will enumerate these tests
* Grouped by category we find
	* **`-e file`**  --> file exists?
	* **`-d file`**  --> file is a directory?
	* **`-f file`**  --> file is a regular file?
	* **`-s file`**  --> file has non-zero size?
	* **`-g file`**  --> file has sgid set?
	* **`-u file`**  --> file has suid set?
	* **`-r file`**  --> file is readable?
	* **`-w file`**  --> file is writeable?
	* **`-x file`**  --> file is executable?

###### String Comparisons

* If you use **single square brackets** or the test syntax, be sure to **enclose environment variables in quotes** in the following
	* **`string`**  -->  string not empty?
	* **`string1 == string2`**  -->  string1 and string2 same?
	* **`string1 != string2`**  -->  string1 and string2 differ?
	* **`-n string`**  -->  string not null?
	* **`-z string`**  -->  string null?

###### Arithmetic Comparisons

* Syntax
	```bash
	$ exp1 -op exp2
	```
	* where the operation (-op) can be
		* `-eq`, `-ne`, `-gt`, `-ge`, `-lt`, `-le`
* **Note** that any condition can be negated by use of !

###### case

* This construct is similar to the switch construct used in C code
* Example
	```bash
	#!/bin/sh
	echo "Do you want to destroy your entire file system?"
	read response
	# Case Statement
	case "$response" in
	   "yes")              echo "I hope you know what you are doing!" ;;
	   "no" )              echo "You have some comon sense!" ;;
	   "y" | "Y" | "YES" ) echo "I hope you know what you are doing!" ;
	                       echo ’I am going to type: " rm -rf /"’;;
	   "n" | "N" | "NO" )  echo "You have some comon sense!" ;;
	   *   )               echo "You have to give an answer!" ;;
	esac
	exit 0
	```
* **Note** - `read` command, reads user input into an **environment variable**

##### Loops

* **bash** permits several looping structures. 
* Examples
	* **for** Loop
		* Sample
			```bash
			for file in $(find . -name "*.o")
			do
			    echo "I am removing file: $file"
			    rm -f "$file"
			done
			```
			* above loop condition is equivalent to the ones mentioned below
				```bash
				$ find . -name "*.o" -exec rm {} ’;’
				# OR
				# showing use of the xargs utility
				$ find . -name "*.o" | xargs rm
				```
	* **while** Loop
		* Sample
			```bash
			#!/bin/sh

			ntry_max=4 ; ntry=0 ; password=’ ’

			while [[ $ntry -lt $ntry_max ]] ; do
			        
			   ntry=$(( $ntry + 1 ))
			   echo -n ’Give password:  ’
			   read password
			   if  [[ $password == "linux" ]] ; then
			       echo "Congratulations: You gave the right password on try $ntry!"
			       exit 0 
			   fi
			   echo "You failed on try $ntry; try again!"
			done

			echo "you failed $ntry_max times; giving up"
			exit -1
			```
	* **until** Loop
		* Sample
			```bash
			#!/bin/sh

			ntry_max=4 ; ntry=0 ; password=’ ’

			until [[ $ntry -ge $ntry_max ]] ; do
			        
			   ntry=$(( $ntry + 1 ))
			   echo -n ’Give password:  ’
			   read password
			   if [[ $password == "linux" ]] ; then
			       echo "Congratulations: You gave the right password on try $ntry!"
			       exit 0 
			   fi
			   echo "You failed on try $ntry; try again!"

			done

			echo "you failed $ntry_max times; giving up"
			exit -1
			```

##### Functions

* Functions must always be defined before they are used, because scripts are interpreted, not compiled
* Example
	```bash
	#!/bin/sh

	test_fun1(){
	    var=FUN_VAR
	    shift
	    echo "   PARS after fun shift:   $0 $1 $2 $3 $4 $5"
	}

	var=MAIN_VAR
	echo ’ ’
	echo "BEFORE FUN MAIN, VAR=$var"
	echo "   PARS starting in main:  $0 $1 $2 $3 $4 $5"
	     
	test_fun1 "$@"
	echo "   PARS after fun in main: $0 $1 $2 $3 $4 $5"
	echo "AFTER FUN MAIN, VAR=$var"

	exit 0
	```
* Sometimes you will see an (older) method of declaring functions, which explicitly includes a function keyword, like
	```bash
	function fun_foobar(){
	   statements
	}
	# OR
	# Old declaration without the parentheses
	function fun_foobar{
	   statements
	}
	```
* Old syntax will work fine in **bash** scripts, but is not designed for the original **Bourne shell**, **sh**
* In the case where a function name is used which collides with an alias, this method will still work
* Use of the **function** keyword is not often used in new scripts


> **ps** gives information about running processes

```ruby
* Functions (subprograms) are useful in bash scripts because
	* They eliminate the need to retype the same set of commands more than once
	* They make things easier to read and comprehend
	* It is better not to have to call another script to get things done

* How would you get the value of a variable named VAR into a script?
	* read VAR
```
