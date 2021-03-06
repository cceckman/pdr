PDR: GDB Tutorial
=================

[Go up to the Tutorials table of contents page](../index.html)

This tutorial is meant to get you used to using the GNU debugger, gdb.  As you read through the first part of the tutorial, you are not expected to remember everything -- there is a reference list at the end of this tutorial, and is also contained on the [GDB command summary](../../docs/gdb_summary.html) page.  This tutorial will guide you through the process of using those commands.  Gdb is a command-line debugger -- we may see graphical debuggers later in the semester.

------------------------------------------------------------

Part I: GDB Tutorial
--------------------

### What is a Debugger? ###

A debugger is a utility program that allows you to run a program under development while controlling its execution and examining the internal values of variables.  We think of a program running "inside" a debugger.  The debugger allows us to control the execution of the program by pausing its execution and then resuming it.  While paused, we can find out where we are in the program, what values variables have, reset the values of variables, etc.  If a program crashes, the debugger can tell you exactly *where* the program crashsed (something that Java does naturally, but C++ does not).  The principles and commands described in this document are specific to the gdb debuggers for clang++ under UNIX, but every debugger has similar commands.

All computer scientists should learn the basics of debugging and how to use a debugger.  It will save you literally hours of time when finding and fixing problems in your programs.  The few minutes of investment you put into learning how to use a debugger will pay off tremendously in a matter of weeks.  Work smart!

### Compiling for Debugging ###

Programs normally have to be compiled with a special option to allow debugging to take place.  On UNIX, the option for clang++ is the `-g` option.  For example:

```
clang++ -Wall -g -o prog1 prog1.cpp 
```

We also include the `--Wall` option, which lists warnings (the 'all' is to list all warnings).  Note that this option leads to executable files that are larger and slower, so you may not want to use it for final distributions or time-critical programs.  But you can always remove the debugging information from an executable without recompiling it with the `strip` command.  See the man page for `strip`.

The -g option causes the compiler to include information about the source file (the .cpp file) that is needed for debugging as part of the executable file.  So when you run the debugger, you specify the executable file (not the source file) as the input to the debugger.

### How to Start Using gdb on our UNIX Systems ###

The GNU C++ compiler, clang++, has an accompanying debugger: gdb.  To run the command-line version, compile your program as described above, and then type:

```
gdb prog1
```

Assuming your executable (created with clang's `-o` option) was "prog1".  If you didn't use the -o option, then you'll type:

```
gdb a.out
```

The following sections describe the important types of things you can do with gdb, organized by "category" of activity.  These activities and operations will have been implemented in other debuggers you may have used, such as the debuggers that come with various IDEs.

### Executing your Program ###

Once in gdb, use the `run` command to start your program running.  It will run until it completes, until it crashes, or until it reaches a breakpoint that you set (more on this later) -- and it will pause for input, of course.  Once it finishes, you're still in gdb, so you can run it again from the beginning.

If your program requires command-line arguments, you can give them after the run command.  If you would normally run the program on the command line by entering:

```
prog1 100 test1.dat
```

In the debugger, you would enter: 

```
run 100 test1.dat
```

### Where Am I? Where Did It Crash? ###

Under UNIX, one of the most frustrating things about running C or C++ programs is that they normally give little useful information when they crash -- usually they just say, 'segmentation fault'.  Part of the reason is that by default the executable file doesn't include information about the source code that is needed to print an error message (like the line number).

But when you run a program inside a debugger, you can easily see what the current line is when a program crashes.  Type `list` to see the current and surrounding lines.

More usefully, you can see a list of the function calls that led you to this point in your program.  Your program may have died deep inside a function that is called many times in your program, and you need to know which sequence of nested functions calls led to the failure.  In the command-line mode, type `backtrace` or `bt` to show this list.  IMPORTANT: this command is one of the most important and useful debugging commands you'll see in this lesson.

While we're talking about reaching a point in a sequence of nested function calls, sometimes in gdb you will need to understand the concept of frames.  When a program stops, you can examine local variables, view lines of code, etc.  that are local to that function.  If you need to move up to the function that called this one, you need to move up to the higher frame using the `up` command to debug there.  The `down` command moves you back down a frame towards where you started.  The up and down commands let you move up and down the calling stack (of nested function calls) so you can issue debug commands about the function that's "active" at each level.

### Making your Program Pause: Breakpoints ###

One of the most fundamental things you want to do while debugging is make the program pause at a particular line or at the start of a function.  These locations in a program where execution pauses are called "breakpoints."  IMPORTANT: You must choose a line of code that actually executes something: not a comment, for example.

In gdb you can set breakpoints by typing either `break` or `b` followed by information on where you want the program to pause.  After the `b` command, you can put either:

- a function name (e.g., `b GetAverage`)
- a line number (e.g., `b 23`)
- either of the above preceded by a file name (e.g., `b debug.cpp:23` or `b debug.cpp:GetAverage`)

Here, the GetAverage() function doesn't start on line 23 (it starts on line 44) -- the breakpoint on line 23 is for the cout statement in the main() function.  We could have also set a breakpoint at the beginning on the GetAverage() function by calling `b 44`.

At any time you can see information about all the breakpoints that have been defined by entering `info break`.  You can remove a breakpoint using the `delete` command (or just `d`).  You can delete all breakpoints (`d`) or a specific one (`d 1`).  Or you can type `clear` followed by the function name or line number (i.e.  `clear GetAverage`, `clear 44`, etc., to remove break-points at those positions).

Breakpoints stick around until you delete them.  This is handy if you put a breakpoint inside a function that is called more than once or if you put one inside a loop.  You can set a temporary breakpoint with the `tbreak` command; the program pauses the first time, but after it pauses there, that breakpoint is cleared.

### Controlling Execution After a Breakpoint ###

After you make your program pause, you may want to execute it line-by-line to see what it does next.  There are two commands that make a program execute the next line and then pause again: `next` and `step`.

The difference between these two is how they behave when the program reaches a function call.  The `step` command steps into that function; in other words, you see the debugging session move into the called function.  The `next` command steps over that function call, and you see the current line as the one after the function call.  Both are useful, depending on what level of detail you need.

Sometimes after you've hit a break point and are doing line-by-line execution, you want to resume normal execution until the next breakpoint is reached (or the program completes).  The command to do this is `continue`.  A useful variant on this is the `finish` command which finishes executing the current function and then pauses.

You can use the abbreviations `s`, `n` and `c` for the common commands described in this section.

### Displaying Variables and Expressions ###

Another thing you often want to do when the program pauses is to see what value a variable or an expression has.  To do this, just type `print` or `p` followed by the variable name or expression.  If the variable or expression is a pointer or an address, you can print the value that this address references using the `print *` command (i.e.  `print *foo`).  In addition, you can enter `info locals` to see all the local variables (and their values) displayed.

It is often handy to have the debugger automatically display one or more variable values at all times so you could watch how they change. You do this with the `display <var>` command, and gdb will display that variable's value each time the program execution hits a breakpoint.  You can use 'display' more than once to show multiple variables.

To see info on all variables chosen for display, just enter `display`.  To remove a variable from the automatic display list, use the `undisplay` command followed by the display variable's numeric-id (entering `display` shows the variables' numeric ids).

### Changing a Variable's Value ###

If you see that a variable has the wrong value, and you'd like to change that value in mid-stream before continuing execution, you can do this easily.  Enter `set` followed by the variable, an equals symbol (`=`), and the value or expression.  It's just like a C++ assignment statement but without the semi-colon at the end.  For example:

```
set x=5
```

The expression can be any C++ expression, including a function call.  So a statement like this is legal:

```
set y=countNegValues(list, num)
```

Assuming you have a countNegValues() method defined, of course.  This would execute your function and then set the variable y to be whatever your function returns.

Sometimes you want to actually execute a function "by hand" while the program is stopped, even if this function isn't what would normally be called at this point in the code.  You can do this using the `set` command as shown above, or by making the function call the argument to the print command.  For example, you could type:

```
print initQueue(&myQueue)
```

And the function would be called right now while the program is paused.  This works even if the function returns void.

### A few final commands ###

If you find the problem while using the debugger, you may want to exit gdb (by entering `quit`), recompile your source code, and restart gdb.  Be sure to use the `--g` option when recompiling!

The `start` command works just like `run` -- it starts up the execution of the program.  However, the `start` command will break at the beginning of the main() method, even if you did not enter a breakpoint there.  This is useful for then stepping through the main() method.

### Final Remarks ###

The best way to learn these commands is to try them out.  Even if you don't use a debugger often, you should make sure you know the basics of breakpoints, single-line execution, and printing a variable's value.  These commands, along with the 'backtrace' command, will be enough for you to solve most of your problems.

Again, a programmer must know how to use a debugger just like an accountant must know how to use a spreadsheet program or a calculator.  Your professors and your boss will expect it of you.  Remember this before you go see your instructor about a run-time bug next time!  The time you spend now to learn how to use a debugger will save you hours in the future.

------------------------------------------------------------

Part II: GDB Lab Exercise
-------------------------

### The Source Code ###

We will be using the [debug.cpp](debug.cpp.html) ([src](debug.cpp)) source code.  There are a few errors in the code, but don't fix them!  We'll use the debugger to find them.
 
### Running the Debugger ###

After you enter the code (remember: if you spot the errors do not correct them -- we will use the debugger to find them!), compile the code.  If you plan on using the debugger, you have to specifically tell clang++ to include debugging information.  To do that, enter the --g flag.  For example, enter:

```
clang++ -Wall -g -o lab2 debug.cpp
```

The `-g` flag will include debugging information.  The `-o lab2` flag will cause the output executable to be `lab2`.  The '-Wall' flag is a new one -- it will include all warnings about your code (errors are still reported without the flag; this includes warnings as well).

There are no compiler or linker errors (or warnings!), so if you get any you will need to find and fix them.  We should now be ready to go.

First just run the program, and enter the numbers 2, 4, 6, 8, and 10 -- we'll be using those five numbers throughout the debugging of the program.  The first thing that you should see is that these are not the numbers that the program displays back to you! What happened?  What's wrong?  Let's use the debugger to find out.

The first feature we will use is the ability to set a breakpoint.

### Setting up Breakpoints ###

Remember that a breakpoint is any point in the executable code where the execution stops, allowing the programmer to see what is going on inside the program as it runs.  Breakpoints allow you to run through portions of the code where there are no problems, so that you can spend your time focusing on the areas that need to be fixed.

To set up a breakpoint, you enter the 'break' command, and where you want the breakpoint to be.  There are 3 ways to specify breakpoints, by entering:

- a function name (e.g., `b GetAverage`)
- a line number (e.g., `b 23`)
- either of the above preceded by a file name (e.g., `b debug.cpp:23` or `b debug.cpp:GetAverage`)

If we knew where the problems were, we could skip over some lines, but since we don't, put a breakpoint on the first line of the code, the cout statement.  You probably want to set the breakpoint based on the line number in the code -- you can use the Emacs command "M-x line-number-mode" to have Emacs display line numbers.  Enter `break x`, where x is the line of the first cout statement in the main() method.  Now we need to run the program -- to do this, enter `run`.  Gdb should start running, then should pause and display the following:

```
Breakpoint 1, main () at debug.cpp:23
23          cout << "Enter five numbers: " << endl;
(gdb)
```

Gdb is stating that it hit a breakpoint, on line 23 of debug.cpp, and displays the line of code.  There are a number of commands we can enter at this point (try them all):

- `bt`: shows a list of the function calls that got us to this point (we are only in the main() method at this point, so it's not all that interesting)
- `list`: shows a listing of the source code where the breakpoint occurred
- `info locals` shows the current variables, and their values.  Note that the variables have not been initialized, so they have strange values!
- `p nCount` will print the current value in nCount
- `p nValues` shows all the values in the array nValues

### Examining Data ###

One of the most powerful features of the debugger is the ability to look at the state of the variables as the program executes.  This way the programmer can see if the variables are changing the way that they are intended to change, and to see if the program is doing the things that were intended.

The `info locals` command will show all the local variables of the current scope of execution.  Which variables are displayed will change as the program executes, always showing the most recently defined variables, values returned from functions, and changed or referenced variables.  When our test program hits the breakpoint, two variables are shown: nCount and nValues.  nCount will be some random integer which reflects the contents of that memory location at the beginning of the program.  The nValues variable looks different -- it's an array, so the entire contents of the array are shown.

### Stepping through the Code ###

Gdb allows you to step through the code in two different ways.  You can execute one line at a time, stepping into each function call, or you can run the functions without tracing their execution.  We will look at examples of both.

First, let's see what happens when we start the program.  You should now be at the breakpoint from above -- if not, restart the program (`run`) -- it will break at the breakpoint you entered before.  Entering `info locals` shows that the nCount variable is filled with a random number.  Let's step into the loop and see what happens.

First, we'll start by stepping OVER commands.

### Single Stepping through your code ###

Enter `n` (or `next`) -- this steps OVER the next command.  This stepped over the `cout` command -- if we had entered `s`, it would have started showing the execution of the cout function call, which is not what we want.  Note after we entered the `n`, it showed the output ("Enter five numbers:") to the screen.  We were stopped BEFORE the line executed, so by single stepping, we caused the computer to execute that one line.  Nothing else has changed (`info locals`, `bt`, etc.  are the same), so let's press `n` again.

Now the cout statement inside the for loop is the current line.  Also notice that now the nCount variable (via `info locals` or `print nCount`) has a value - it is zero, because that's where our for loop begins.  Press `n` again.  The prompt for the next number is displayed, since the cout statement has executed.  Step through another line of code.

The code is now stopped on the `cin` statement.  You will need to enter a value and press enter.

You should see these changes:

- The number you typed shows after the prompt.
- 'info locals' shows that the 2 was entered into the array at index 1 (not 0!)

Pressing `n` again will take us to the beginning of the loop; pressing it again will increment the value of nCount in the watch windows.  Let's single step through another pass through the loop.  Step over button twice more, entering successive values when prompted (2, 4, 6, 8, 10).  See what happens to the variables.

This shows you one of the errors - the data is all going into nValues\[1\].  Go to the source window, and correct the line.  Exit gdb (`quit`), recompile the program, and start up gdb again (`gdb lab2`).  Set your breakpoint, run the code, and make sure that it is getting the input correctly.  Try entering `display nValues` after the first breakpoint -- it will always display the contents of the nValues array each time the program pauses.

When you are satisfied that the input is working, you can `continue` (or `c`), and the program will run until the next breakpoint, or the end of the program.

There appears to be a couple more errors in the code.  Let's address the problem with the average value.  To do this we'll need to use the step into ability of the debugger.

Place a breakpoint on the line that the GetAverage() function is called (the cout line in the main() method, not the GetAverage() method itself).  Remove the breakpoint from the beginning of the source code: `clear <line>`, where <line> is the line of code where the breakpoint is at, or `delete 1`, where 1 is the breakpoint number.  You can see the list of breakpoints by entering `info break`.  Once you have only one breakpoint set up at line that GetAverage() is called, run your code (`run`).  It will run normally (we'll enter the same values: 2, 4, 6, 8, 10) until it hits the breakpoint.

Press the Step Into button (`s`).  Execution of the program now passes to the first line of the function GetAverage().  Entering `bt` will show the series of function calls that got us to this point.  We can now use the step over command (`n`) to step through the function and identify the errors.

A word of caution: using the step into command at the wrong time may cause the debugger to load and display either assembly language or unfamiliar code.  Don't worry; all you've done is stepped into the code for a standard function or operator such as the insertion operator.  To exit this code, enter `c` for continue (which will continue execution until the next breakpoint), or `finish`, which will execute until the current function ends.

Does the printing of the maximum value work?  If so, great!  If not, you get to figure that one out on your own...

### Explore on your own ###

There are still a few errors; try tracing through the function and see what you can fix!  Remember that there are a few different types of breakpoints.  Rather than entering the line number, you can enter a breakpoint by specifying the function call -- for example, you can enter 'break GetAverage' to debug the GetAverage function, rather than trying to figure out which line the function starts on.

### Finishing up ###

When you are finished debugging the code with gdb -- and it works correctly -- you should submit the debug.cpp file to inlab2.  Remember to put your identifying information at the top.

------------------------------------------------------------

Part III: Summary of gdb commands
---------------------------------

These commands are also listed on the [GDB command summary](../../docs/gdb_summary.html) page.

Assembly-specific commands

- `stepi` (or `si`): step one MACHINE instruction (i.e. assembly instruction), instead of one C++ instruction (which is what 'step' does)
- `info registers`: display the values in the registers
- `set disassembly-flavor intel`: set the assembly output format to what we are used to in class (and what we are programming in)
- `disassemble`: like list, but displays the lines of assembly code currently being executed.
- `disassemble (function)`: prints the assembly code for the supplied function (up until the next label)

Program execution

- `run`: starts a program execution, and continues until it exits, crashes, or hits a breakpoint
- `start`: starts a program execution, and breaks when it enters the main() function
- `bt`: prints a back trace, which is the list of function calls that got to the current point
- `list`: shows the lines of source code before and after the point at which the program paused
- `up`: move up the back trace function stack list
- `down`: move down the back trace function stack list
- `step` (or just '`s`'): step INTO the next line of code to execute
- `next` (or just '`n`'): step OVER the next line of code to execute
- `continue` (or just '`c`'): continue execution
- `finish`: finishes executing the current function and then pauses
- `quit`: exits gdb

Breakpoints

- `b (pos)` (or `break (pos)`): set a breakpoint at (pos).  A breakpoint can be a function name (e.g., `b GetMax`), a line number (e.g., `b 22`), or either of the above preceded by a file name (e.g., `b lab2.cpp:22` or `b lab2.cpp:GetMax`)
- `tbreak (pos)`: set a temporary breakpoint (only breaks the first time)
- `info break`: show breakpoints
- `delete` (or just '`d`'): deletes all breakpoints
- `delete (num)`: delete the breakpoint indicated by (num)
- `clear (pos)`: clear a breakpoint, where (pos) is either a function name or line number

Examining data

- `print (var)` (or '`p`'): print the value in the given variable
- `print *`: print the destination of a pointer
- `x/(format) (var/address)`: format controls how the memory should be displayed, and consists of (up to) 3 components: a numeric count of how many elements to display; a single-character format, indicating how to interpret and display each element -- e.g. a few of the flags are `x/x` displays in hex, `x/d` displays in signed decimals, `x/c` displays in characters, `x/i` displays in instructions, and `x/s` displays in C strings; and a single-character size, indicating the size of each element to display -- e.g. b, h, w, and g, for one-, two-, four-, and eight-byte blocks, respectively. You can have multiple at a time, e.g. `x/30x (var/address)` will display 30 elements in hexidecimal from the provided `var/address` OR if no `var/address` is provided, from the top of the stack.
- `info locals`: display all the local variables and their values
- `display (var)`: always display the value in (var) whenever the program pauses
- `display`: show the variables that have been entered with 'display' and their numeric IDs
- `undisplay (num)`: stop displaying the variable with numeric ID num
- `print function_call(params)`: execute the function, and print the result
- `set variable (var) = (value)`: set the variable (var) to the value (value)

<!--- - examine (var): look at top of stack -->

Note: if you have any question about any of the above commands, simply type `help (command)` and it will describe the command and provide the full list of flags.
