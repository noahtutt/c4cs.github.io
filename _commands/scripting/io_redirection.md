---

I/O Redirection
---------------

The I/O redirection operators `>`, `>>`, `<`, and `|`  allow us to read and write input and output from files, and pass output from one program to another.

~~~ bash
$ echo "I love bash!" > message.txt
$ cowsay -w < message.txt
$ echo "I love bash so much!" >> message.txt
$ cosway -w < message.txt
$ cat message | grep "bash"
~~~

<!--more-->

### Basic Usage

#### `>` operator
Puts the output from the program on the left of the operator into the file named on the right of the operator. Will overwrite all of current contents of the file, and will create the file if it doesn't exist yet.

~~~ bash
$ echo "Text in a file" > message.txt
$ cat message.txt
Text in a file
~~~

#### `>>` operator
Appends the output from the program on the left of the operator to the contents of the file named on the right of the operator. Will not overwrite any of the current contents of the file, and will create the file if it doesn't already exist.

~~~ bash
$ echo "Text in a file" >> message.txt
$ echo "More text in a file" >> message.txt
$ cat message.txt 
Text in a file
More text in a file
~~~

#### `<` operator
Feeds the contents of the file on the right into the input of the program on the left. Will not modify any of the contents of the file, and will throw an error if it doesnt already exist.

~~~ bash 
$ echo "I love bash" > message.txt
$ cowsay -w < message.txt
 _____________
< I love bash >
 -------------
        \   ^__^
         \  (OO)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
~~~

#### `|` operator 
Feeds the output of the command on the left into the input of the command on the right. 

~~~ bash
$ echo "Pipes are fun!" | grep -c "Pipe"
1  
~~~

### Advanced Examples 

#### Stream Selection
By default, the `>` and `>>` operators all operate on the standard output stream `stdout`, and leave the standard error stream `stderr` untouched. If we want to redirect just the error messages from a program, we can do so by selecting stream 2:

~~~ bash
$ git commit 
fatal: Not a git repository (or any of the parent directories): .git
$ git commit > git_errors.txt
$ cat git_errors.txt # will generate no output
$ git commit 2> git_errors.txt
$ cat git_errors.txt
fatal: Not a git repository (or any of the parent directories): .git
~~~

#### Changing the Input of `|`: Crossing the Streams
We can't directly change the input source of the `|` operator as we did for `>` and `>>`, it always reads from the default source of `stdout`. If we want to redirect `stderr` only, we can first point `stderr` at `stdout` and then point `stdout` at nothing:

~~~ bash
$ git commit 
fatal: Not a git repository (or any of the parent directories): .git
$ git commit 2>&1 1>/dev/null | grep -c "fatal"
1
~~~

#### Combining the Streams
If we want the output of both streams to be redirected into a single file, we can do that with `&>`

~~~ bash
$ echo '#include <iostream>
> int main() {
> std::cout << "This is stdout text\n";
> std::cerr << "This is stderr text\n";
> }' > prog.cpp # a simple program that prints to both streams
$ g++ prog.cpp -o prog
$ ./prog 2>/dev/null 1> output.txt
$ cat output.txt
This is stdout text
$ ./prog 1>/dev/null 2> output.txt
$ cat output.txt
This is stderr text
$ ./prog &> output.txt
$ cat output.txt
This is stdout text
This is stderr text
~~~

#### Pipe Chaining
The `|` operator allows us to solve complex problems in a simple and expressive manner by piping output between many different programs. To demonstrate this, we'll build up a pipeline to solve the following problem: given the text of a `git log`, how many unique authors have commited to this branch? For the purposes of this exercise, we'll define a unique author to be a unique email address. We'll build our pipeline one command at a time, starting with generating the `git log`:

~~~ bash
$ git log
~~~

First, we need to isolate all the emails in this file. `grep` makes this pretty straightword. Every email
contains an @ sign and is bracketed by the < and > characters. This gives us the regular expression `<.*@.*>` (note that we're assuming no one types an email in this format into a commit message, which is true of the repository for this website). Combine this with the `-o` flag to print the matching strings only:

~~~ bash
$ git log | grep -o "<.*@.*>"
~~~ 

With each email now isolated, we need to group all identical emails together. `sort` does this easily:

~~~ bash
$ git log | grep -o "<.*@.*>" | sort
~~~

We're almost there. `uniq` will allow us to remove all the duplicate emails:

~~~ bash
$ git log | grep -o "<.*@.*>" | sort | uniq
~~~

Now we have a list of every unique contributor to the project. All that's left to do is count them up with `wc` (the `-l` flag counts the number of lines only):

~~~ bash
$ git log | grep -o "<.*@.*>" | sort | uniq | wc -l
~~~
