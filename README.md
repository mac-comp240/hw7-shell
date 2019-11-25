# Writing a Shell

In this assignment, you will create a program, `shell.c`, that implements a
command-line interface for running programs, similar to bash or zsh, which you
have been using for this course.

Your program will need to loop, reading commands from the user, and use C
methods `fork()` and `exec()` to carry out those instructions. You will need to
handle common errors without exiting the program, like a real shell.

You may work with a partner for this project, though it is not required.

This assignment is meant to streatch your coding ability--it is intentionally
challenging! You can use outside sources for help: google for example code, look
at man pages. But **you must credit your sources in the comments** and **you
must write up the code on your own**. You may **not** copy code from external
sources verbatim. A good general rule is that if you do seek external help, you
should cite your source and write up pseudocode as a comment, then adapt that
pseudocode to real code.

# Problem Description

This assignment has multiple parts that you will need to implement for full
credit, each described below. 

If you complete the first four parts successfully, you will receive an A-
(90/100) on this assignment. To receive a 100/100, you will need to complete
everything described below. I want you to prioritize--complete as much as you
can, but be willing to accept the A- if you need more time for other
end-of-semester considerations!

Each task includes a reference to the C function and/or Unix system call that
could help you implement that task. If you get stuck on a particular step, you
can try faking it and demonstrating that the rest is working. Please clearly
indicate any work that you have faked.

# Tasks

We will be supporting input/output redirection from/to files, background
execution, and pipelines. E.g.,

    foo > output
    foo < input
    foo > output < input
    foo | bar
    foo & 

You should take advantage of the PATH variable that is inherited from the
environment.

## Read user input

Inside a loop in your main(), present a prompt to the user and then read a full
line of input (up to and including the newline character). Continue looping
until EOF is reached. 

Reference: `fgets(3)`

## Run a simple process

Run a single word command line (e.g., "ls") by forking a subprocess and execing it. You may assume the word is at the start of the line and that there is no extraneous whitespace. Your main loop should wait until the process completes before continuing. 

Reference: `fork(2)`, `execvp(2)`, 1waitpid(2)`

## Built-in commands

Create the following as builtin commands:
exit - terminate your shell input loop
myinfo - print out your PID and PPID in a readable format
cd - change your working directory to $HOME
cd <dir> - change your working directory to dir


Reference: `chdir(2)`, `getenv(3)`, `getpid(2)`, `getppid(2)`

## Parse whitespace in commands

Parse a command line with whitespace into a vector or strings. So something like the string

    "foo |    bar > baz"
        
will be turned into

    args[0] = "foo";
    args[1] = "|";
    args[2] = "bar"
    args[3] = ">";
    args[4] = "baz";
    args[5] = NULL;
        
I'm told this is likely to be the trickiest part of the assignment.

For extra credit, handle parsing a command line without spaces between the symbols:  > < & | 

Reference: `strtok(3)`, `strchr(3)`, `strsep(3)`, `strpbrk(3)`

## Run multi-token commands

Take a vectorized multi-token command line and run the program specified, waiting until the program completes before displaying the next command prompt. I recommend creating a function that will do the actual program execution and return the child PID. 

Reference: `fork(2)`, `execvp(2)`, `waitpid(2)`

## Trap SIGINT

Trap SIGINT and have it terminate the currently running child process of the shell (if any) and not the shell itself. 

Reference: `signal(3)`, `kill(3)`

## Run background processes

If the last token on the command line is &, have that program run in the background instead of the foreground. The prompt should be redisplayed without waiting for the child to return. At the start of the loop, before displaying the prompt, check to see if any children need to be reaped, reap them, and display a message to the user about that fact. 

Reference: `waitpid(2)` with a pid of -1 and the `WNOHANG` option

For extra credit, add in support to trap SIGTSTP (ctrl-z) and have it change the child program that is currently running into a background process. (This might be trickier than I think it is.)

## Output redirection

Support for < and > for input and output redirection to the program. Do this by opening the requested file for read/write as needed and duping it to STDIN_FILENO or STDOUT_FILENO. 

Reference: `open(2)` or `fopen(3)`/`fileno(3)`, `dup2(2)`

## Piping

Support for command line piping with a single pipe. For example

    foo | bar
        
should have STDIN_FILENO of bar be reading from the STDOUT_FILENO of bar. You
can do this by using pipe(2) to create connected pairs of file descriptors and
then use dup2(2) to set them up appropriately in the children. Close the unused
ends (e.g., foo should close the descriptor that bar is using to read). Your
shell will need to exec both processes and wait for both of them to return. Note
that you can change the "&" in your argument vector to a NULL and just pass in
different starting locations of the same vector to your execution function. You
may want to have a separate array that keeps track of the start index of various
command segments. 

Reference: `dup2(2)`, `pipe(2)`

## Create a Makefile and README

Your Makefile should support `make` and `make clean`, and your README should
have the information described in the Deliverables section below.

## Extra Credit

Add support for setting and modifying environment variables
Support a multi-pipe sequence.
Programming Notes

Your program needs to handle input lines up to 1024 characters
You may assume/limit the number of background processes
You may assume there are no more than 100 tokens (words) on a line
You may assume there are no more than 10 command segments on the command line
Error handling

The following are possible errors that your program should be able to handle without your shell crashing. The first two should be readily handled, but for the last couple you might have some processes that run anyway depending on how you execute portions of the pipeline.

Command not found on the path
Input redirected in from a non-existant file
Input redirected in to a pipe command that is not the first command
Output redirected from a pipe command that is not the last command
handin

# Deliverables

You should deliver:
* `shell.c`, which contains the C source code for your shell
* `Makefile`, which will `make` and `make clean` your source code
* A README that contains:
    * Your name and your partner's name, if any
    * A list of all the portions of the shell you were asked to complete, and
      in which you've clearly indicated which parts you have completed and which
      parts you have not
    * Any known bugs/errors with your code

## Thanks

Thanks to the Oberlin College Computer Science Department, where I originally
completed this assignment as a student, Professor Ben Kuperman for
developing it, and Professor Roberto Hoyle for his additional modifications.
