java cProject Three: Simple World
Due: Nov. 17, 2024
I. Motivation
1. To give you experience in using arrays, pointers, structs, enums, and different I/O streams and 
writing program that takes arguments.
2. To let you have fun with an application that is extremely captivating.
II. Introduction
The simple world program we will write for this project simulates a number of creatures running 
around in a simple square world. The world is an m-by-n two-dimensional grid of squares (The 
number m represents the height of the grid and the number n represents the width of the grid.).
Each creature lives in one of the squares, faces in one of the major compass directions (north, 
east, south, or west) and belongs to a particular species, which determines how that creature 
behaves.
la ho
fl
fl ho Figure 1. A 4-by-4 grid, which contains five creatures. Two creatures belong to the species 
flytrap (whose short name is “fl”), two belong to the species hop (whose short name is “ho”), and
one belongs to the species landmine (whose short name is “la”). The direction of each creature 
is represented by the direction of the arrow.
Figure 1 shows a 4-by-4 grid populated by five creatures. Two of them belong to the species 
flytrap (whose short name is “fl”), two belong to the species hop (whose short name is “ho”), and 
one belongs to the species landmine (whose short name is “la”). The direction of each creature is 
represented by the direction of the arrow. For example, the flytrap at the top row is facing east 
and the flytrap at the bottom row is facing west.
Table 1. The list of instructions and their explanations.
hop
The creature moves forward as long as the square it is facing is empty. If 
moving forward would put the creature outside the boundaries of the grid or 
would cause it to land on top of another creature, the hop instruction does 
nothing.
left The creature turns left 90 degrees to face in a new direction.
right The creature turns right 90 degrees to face in a new direction.
infect
If the square immediately in front of this creature is occupied by a creature of 
a different species (an “enemy”), that enemy creature is infected to become 
the same as the infecting species. When a creature is infected, it keeps its 
position and orientation, but changes its internal species indicator and begins 
executing the same program as the infecting creature, starting at step 1. If the 
square immediately in front of this creature is empty, outside the grid, or 
occupied by a creature of the same species, the infect instruction does 
nothing.
ifempty n
If the square in front of the creature is inside the grid boundary and 
unoccupied, jump to step n of the program; otherwise, go on with the next 
instruction in sequence.
ifwall n
If the creature is facing the border of the grid (which we imagine as 
consisting of a huge wall) jump to step n of the program; otherwise, go on 
with the next instruction in sequence.
ifsame n
If the square the creature is facing is occupied by a creature of the same
species, jump to step n; otherwise, go on with the next instruction.
ifenemy n
If the square the creature is facing is occupied by a creature of an enemy 
species, jump to step n; otherwise, go on with the next instruction.
go n This instruction always jumps to step n, independent of any condition.
Each species has an associated program which controls how each creature of that species 
behaves. Programs are composed of a sequence of instructions. The instructions that can be 
part of a program are listed in Table 1. There are nine legal instructions in total. The last five 
instructions have an additional integer argument.
Program is an attribute associated with species. Creatures of the same species have the same 
program. However, different species have different programs. 
For example, the program of the species flytrap is composed of the following five instructions:
ifenemy 4
left
go 1
infect
go 1
The meaning of each instruction for this example is commented below:
(step 1) ifenemy 4 # If there is an enemy ahead, go to step 4
(step 2) left # Turn left
(step 3) go 1 # Go to step 1
(step 4) infect # Infect the adjacent creature
(step 5) go 1 # Go to step 1
We will simulate the behaviors of all the creatures for a user specified number of rounds. In each 
round, creatures take their turns one by one, starting from the first creature. After the first 
creature finishes its turn, the second creature begins its turn. So on and so forth. One round ends 
with the last creature finishing its turn. Then the next round begins with the first creature taking
its turn. Note that during the simulation, a creature may infect another creature so that the 
infected one changes its species. However, the simulation order of the infected creature does not 
change.
Each creature also maintains a variable called program counter which stores the index of the 
instruction it is going to execute. On each turn of a creature, it executes a number of instructions 
of its program, starting from the step indicated by the program counter. A program ordinarily 
continues with each new instruction in sequence, although this order can be changed by certain 
instructions in the program such as the if*** instructions. In each turn, a creature can execute 
any number of if*** or go instructions without relinquishing this turn. Its turn ends only when 
the creature executes one of the instructions: hop, left, right, or infect. After its turn ends, the 
creature updates the program counter to point to the next instruction, which will be executed at 
the beginning of its next turn.
Note that each creature maintains its own program counter, so that two different creatures 
belonging to the same species can have different program counters. The indices of the 
instructions start from one, i.e., the first instruction of each program is “step 1”. At the very 
beginning of the simulation process, the program counters of all the creatures are set to their first 
instructions.
III. Available Types
In completing this project, you will have the following types available to you. They are defined 
in the file world_type.h.
const unsigned int MAXSPECIES = 10; // Max number of species in the
 // world
const unsigned int MAXPROGRAM = 40; // Max size of a species program
const unsigned int MAXCREATURES = 50; // Max number of creatures in
 // the world
const unsigned int MAXHEIGHT = 20; // Max height of the grid
const unsigned int MAXWIDTH = 20; // Max width of the grid
struct point_t
{
 int r;
 int c;
};
/*
// Type: point_t
// ------------
// This type is used to represent a point in the grid.
// Its component r corresponds to the row number; its component
// c corresponds to the column number.
*/
enum direction_t { EAST, SOUTH, WEST, NORTH };
/*
// Type: direction_t
// ----------------
// This type is used to represent direction, which can take on
// one of the four values: East, South, West, and North.
*/
const std::string directName[] = {"east", "south", "west", "north"};
// An array of strings representing the direction name.
const std::string directShortName[] = {"e", "s", "w", "n"};
// An array of strings representing the short names for directions.
enum opcode_t {HOP, LEFT, RIGHT, INFECT, IFEMPTY, IFENEMY,
 IFSAME, IFWALL, GO};
/*
// Type: opcode_t
// -------------
// The type opcode_t is an enumeration of all of the legal
// command names.
*/
const std::string opName[] = {"hop", "left", "right", "infect",
 "ifempty", "ifenemy", "ifsame", "ifwall", "go"};
// An array of strings representing the command name.
struct instruction_t
{
 opcode_t op;
 unsigned int address;
};
/*
// Type: instruction_t
// ------------------
// The type instruction_t is used to represent an instruction
// and consists of a pair of an operation code and an integer.
// For some operation code, the integer stores the address of
// the instruction it jumps to. The address is optional.
*/
struct species_t
{
 std::string name;
 unsigned int programSize;
 instruction_t program[MAXPROGRAM];
};
/*
// Type: species_t
// ------------------
// The type species_t is used to represent a species
// and consists of a string, an unsigned int, and an array
// of instruction_t. The string gives the name of the
// species. The unsigned int gives the number of instructions
// in the program of the species. The array stores all the
// instructions in the program according to their sequence.
*/
struct creature_t
{
 point_t location;
 direction_t direction;
 species_t *species;
 unsigned int programID;
};
/*
// Type: creature_t
// ------------------
// The type creature_t is used to represent a creature.
// It consists of a point_t, a direction_t, a pointer to 
// species_t and an unsigned int. The point_t gives the location of
// the species. The direction_t gives the direction of the species.
// The pointer to species_t points to the species the creature belongs
// to. The programID gives the index of the instruction to be
// executed in the instruction_t array of the species.
*/
struct grid_t
{
 unsigned int height;
 unsigned int width;
 creature_t *squares[MAXHEIGHT][MAXWIDTH];
};
/*
// Type: grid_t
// ------------------
// The type grid_t consists of the height and the width of the grid
// and a two-dimensional array of pointers to creature_t. If there is
// a creature at the point (r, c) in the grid, then squares[r][c]
// stores a pointer to that creature. If point (r, c) is not occupied
// by any creature, then squares[r][c] is a NULL pointer.
*/
struct world_t
{
 unsigned int numSpecies;
 species_t species[MAXSPECIES];
 unsigned int numCreatures;
 creature_t creatures[MAXCREATURES];
 grid_t grid;
};
/*
// Type: world_t
// --------------
// This type consists of two unsigned ints, an array of species_t,
// an array of creature_t, and a grid_t object. The first unsigned
// int numSpecies specifies the number of species in the creature
// world. The second unsigned int numCreatures specifies the number
// of creatures in the world. All the species are stored in the array 
// species and all the creatures are stored in the array creatures.
// The grid is given in the object grid.
*/
IV. File Input
All the species, the programs for all the species, and the initial layout of the creature world are 
stored in files and these files will be read by your program to set up the simulation environment. 
Note: when you read files, you must use input file stream ifstream. Otherwise, since the 
files are read-only on our online judge, you may fail to read the files.
As we described before, each species has an associated program. The program for each species is 
stored in a separate file whose name is just the name of that species. For example, the program 
for the species flytrap is stored in a file called flytrap.
A file describing a program contains all the instructions of that program in order. Each line lists
just one instruction. The first line lists the first instruction; the second line lists the second 
instruction; so on and so forth. Each instruction is one of the nine legal instructions described in 
Table 1. The program ends with the end of file or a blank line. Comments may appear after the 
blank line or at the end of each instruction line. For example, the program file for the flytrap 
species looks like:
ifenemy 4 If there is an enemy, go to step 4.
left If no enemy, turn left.
go 1
infect
go 1
The flytrap sits in one place and spins.
It infects anything which comes in front.
Flytraps do well when they clump.
Note that in writing functions for reading these program files, you should handle the comments 
correctly, which means that you should ignore these comments when setting up the program for a 
species.
Since there are many species, we stored all of their program files in a directory.
To help you get all the species and their program files, we also have a file telling the directory 
where the program files are stored and listing all the species. We call this file a species summary. 
The first line of this file shows the directory where all of the program files are stored. The next 
lines list all the species, with one species per line. For example, the following is a species 
summary file:
creatures
flytrap
hop
landmine
From this file, we can learn that the program files are stored in the directory called creatures
located relative to the current working directory (i.e., where the program locates). We have three 
species to simulate, which are flytrap, hop, and landmine. By first reading the species summary 
file, you will know where to find the program file for each species.
Finally, there is a file describing the initial state of the creature world. We call it a world file. The 
first line of this file gives the height of the two-dimensional grid (i.e., the number of rows) and 
the second line gives the width of the grid (i.e., the number of columns). The remaining lines of 
this file describe all the creatures to simulate and their initial directions and locations, with one 
creature per line. Each of these lines has the following format:
   
where  is one of the species from the species summary file, 
 describes the initial direction and is one of the strings “east”, 
“south”, “west”, and “north”.  describes the initial row location of the 
creature. We use the convention that the top-most row of the grid is row 0 and the row 
number increases from top to bottom.  describes the initial column 
location of the creature. We use the convention that the left-most column of the grid is column 
0 and the column number increases from left to right. An example of a world file looks like:
4
4
hop east 2 0
flytrap east 2 2
It says that the size of the grid is 4-by-4 and there are two creatures in the world. The first 
creature belongs to the species hop. It faces east and lives at point (2, 0) initially. The second 
creature belongs to the species flytrap. It faces east and lives at point (2, 2) initially.
In the simulation, the order on the creatures to simulate is important. This order is 
determined by the order that these creatures appear in the world file.
V. Program Arguments
Your program will obtain the names of the species summary file and the world file via program 
arguments. Additionally, your program will be told the number of rounds to simulate and 
whether it should print the simulation result verbosely or not.
The expected order of arguments is:
   [v|verbose]
The first three arguments are mandatory. They give the name of the species summary file, the 
name of the world file, and the number of simulation rounds, respectively. The fourth argument 
is optional. If the fourth argument is the string “v” or the string “verbose”, your program should 
print the simulation result verbosely, which will be explained later. Otherwise, if it is omitted or 
is any other string, your program should print the result concisely, which will also be explained 
later. If there are more than four arguments, your program should just read the first four and 
ignore th代 写program、c/c++，Java
代做程序编程语言e remaining.
Suppose that you program is called p3. It may be invoked by typing in a terminal:
./p3 species world 10 v
Then your program should read the species summary from the file called “species” and the world 
file from the file called “world”. The number of simulation rounds is 10. Your program should 
print the simulation information verbosely.
VI. Error Checking and Error Message
Your program should check for errors before it starts to simulate the moves of the creatures. If 
any error happens, your program should issue an error message and then exit. If there are no 
errors happening, then the initial state of the creature world is legal and your program can start 
simulating the creature world.
We require you to do the following error checking and print the error message in exactly the 
same way as described below. Note that some of the output error message has two lines and each 
error message should be ended with a newline character. All error messages should be sent to 
the standard output stream cout; none to the standard error stream cerr.
1. Check whether the number of arguments is less than three. If it is less than three, then one of 
the mandatory arguments is missing. You should print the following error message:
Error: Missing arguments!
Usage: ./p3    [v|verbose]
2. Check whether the value  supplied by the user is negative. If it is negative, you 
should print the following error message:
Error: Number of simulation rounds is negative!
3. Check whether file open is successful. If opening species summary file, world file, or any 
species program file fails (for example, the file to be opened does not exist), print the following 
error message:
Error: Cannot open file !
where  should be replaced with the name of the file that fails to be opened. If that 
file is not in the same directory as your program, you need to include its path in the 
. As you may know, there are multiple ways to specify a path. For us, the path 
name should be specified in the most basic way, i.e., “/” (not 
“.//”, “//filename”, etc.). Once you find a file that cannot be 
opened, issue the above error message and terminate your program.
4. Check whether the number of species listed in the species summary file exceeds the maximal 
number of species MAXSPECIES. If so, print the following error message:
Error: Too many species!
Maximal number of species is .
where  should be replaced with the maximal number of species set by your 
program.
5. Check whether the number of instructions for a species exceeds the maximal size of a species 
program MAXPROGRAM. If so, print the following error message:
Error: Too many instructions for species !
Maximal number of instructions is .
where  should be replaced with the name of the species whose program has 
more instructions than the maximal number allowed and  should be replaced 
with the maximal size of a species program set by your program.
6. Check whether the species program file contains illegal instructions. We only allow nine 
instructions as listed in Table 1. Your program needs to check whether the instruction name is 
one of the nine legal instruction names listed in the string array opName (which is defined in 
Section III). If an instruction name is not recognized, you should print the following error 
message:
Error: Instruction  is not recognized!
where  should be replaced with the name of the unrecognized 
instruction. You can assume that for any recognized instruction, it is given in the correct format.
Thus, you don’t need to check whether an integer is appended after the instruction name or not.
If there are multiple unrecognized instruction names, you only need to print out the first one and 
then terminate the program.
7. Check whether the number of creatures listed in the world file exceeds the maximal number of 
creatures MAXCREATURES. If so, print the following error message:
Error: Too many creatures!
Maximal number of creatures is .
where  should be replaced with the maximal number of creatures allowed
by your program.
8. Check whether each creature in the world file belongs to one of the species listed in the 
species summary file. If the species for a creature is not recognized, print the following error 
message:
Error: Species  not found!
where  should be replaced with the unrecognized species. If there are 
multiple unrecognized species, you only need to print out the first one and then terminate the 
program.
9. Check whether the direction string for each creature in the world file is one of the strings in 
the array directName (which is defined in Section III). If the direction string is not recognized, 
print the following error message:
Error: Direction  is not recognized!
where  should be replaced with the unrecognized direction name. If 
there are multiple unrecognized direction names, you only need to print out the first one and then 
terminate the program.
10. Check whether the grid height given by the world file is legal. A legal grid height is at least 
ONE and less than or equal to a maximal value MAXHEIGHT. If the grid height is illegal, print 
the following error message:
Error: The grid height is illegal!
11. Check whether the grid width given by the world file is legal. A legal grid width is at least 
ONE and less than or equal to a maximal value MAXWIDTH. If the grid width is illegal, print the 
following error message:
Error: The grid width is illegal!
12. Check whether each creature is inside the boundary of the grid. If any creature is outside the 
boundary, print the following error message:
Error: Creature (   ) is out of bound!
The grid size is -by-.
where  should be replaced with the species the creature belongs to,  be 
replaced with the direction the creature is facing,  be replaced with the row location of the 
creature,  be replaced with the column location of the creature,  be replaced with
the height of the grid, and  be replaced with the width of the grid. Here, we use the 
four-tuple (   ) to identify the creature. For example, if given 
the following world file:
3
3
flytrap east 0 0
hop south 3 2
food west 2 1
then Creature (hop south 3 2) is outside the boundary. Then, the error message should be:
Error: Creature (hop south 3 2) is out of bound!
The grid size is 3-by-3.
If there are multiple creatures outside the boundary, you only need to print out the first one and 
then terminate the program.
13. Check whether each square in the grid is occupied by at most one creature. If any square is 
occupied by more than one creature, print the following error message:
Error: Creature (   ) overlaps with creature 
(   )!
where ( ) identifies the square which is occupied by more than one creature, the first
four-tuple (   ) identifies the second creature in order that 
occupies the square, and the second four-tuple (   ) identifies the 
first creature in order that occupies the square. Once you find two creatures occupying the
same square, you issue the above error message and then terminate the program.
Since you may implement the error checking in different order and in the case that there is
more than one error, the first error message printed out may be different. Therefore, we 
will only test your error checking using test cases containing just one error.
You can also assume that except the above errors, there are no other errors. 
You can assume that the species program file is non-empty and a file without errors 5 and 
6 above could be executed for infinite rounds.
VII. Simulation Output
Once all of the above error checkings on the initial state of the creature world are passed, you
can start simulating the creature world. You should print to the standard output the simulation 
information, either in a verbose mode or in a concise mode, depending on whether an 
additional argument “v” or “verbose” is provided by the user.
In the verbose mode, you first print the initial state of the world. In doing so, you begin with 
printing the string “Initial state” followed by a newline. Then you graphically show the
layout of the initial grid using just characters. Each square takes a four-character field in your 
terminal. Adjacent squares on the same row are separated by one space. If a square in the grid is 
not occupied by any creature, the field for that square is filled with FOUR “_”. If a square is 
occupied by a creature, then the first two characters of the field for that square are the first two 
letters of the name of the species the creature belongs to. (We assume that all the species names 
contain at least two characters and no two species names have the identical first two characters.) 
The third character in the field is a “_” and the fourth character is the first character of the 
direction the creature faces, i.e., “e” for “east”, “s” for “south”, “w” for “west”, and “n” for 
“north”.
For example, suppose a world file looks like
4
4
hop east 2 0
flytrap east 2 2
Then the layout of the initial grid is printed as
____ ____ ____ ____ 
____ ____ ____ ____ 
ho_e ____ fl_e ____ 
____ ____ ____ ____
Note that there is a space at the end of each line.
After printing the initial layout, we begin the simulation from the first round to the last round 
specified by the user. In the i-th simulation round, you first print “Round ” followed by the 
newline. For example, in the first round, you should first print
Round 1
During each simulation round, you simulate the moves of all the creatures in turn. When starting 
simulating a creature, you announce that this creature takes action by printing
Creature (   ) takes action:
followed by a newline. In the above output, the four-tuple (   )
shows the state of the creature right before it takes the action, where  should be 
replaced with the species the creature belongs to,  be replaced with the direction the 
creature is facing,  be replaced with the row location of the creature, and  be replaced 
with the column location of the creature.
After this, you print the sequence of instructions that the creature executes for its turn. This 
sequence may include any number of if*** and go instructions and end with one of the hop, left, 
right, and infect instruction. You should print the sequence of instructions the creature executes
in order, with one instruction per line. The output format for an instruction is:
Instruction :  [GOTO_STEP]
where  should be replaced with the number of that instruction in the program (the 
number starts from 1),  should be replaced with the name of the instruction, 
and [GOTO_STEP] is the number in an if*** or a go instruction and is optional.
After printing the last instruction of the creature under consideration, you should print the 
updated layout of the grid, using the same rule as you print the initial layout.
Now let’s look at an example. Suppose that the program for the species hop is
hop
go 1
and the program for the species flytrap is
ifenemy 4 If there is an enemy, go to step 4.
left If no enemy, turn left.
go 1
infect
go 1
Then, given the following world file
4
4
hop east 2 0
flytrap east 2 2
the simulation information for the first round is printed as
Round 1
Creature (hop east 2 0) takes action:
Instruction 1: hop
____ ____ ____ ____ 
____ ____ ____ ____ 
____ ho_e fl_e ____ 
____ ____ ____ ____ 
Creature (flytrap east 2 2) takes action:
Instruction 1: ifenemy 4
Instruction 2: left
____ ____ ____ ____ 
____ ____ ____ ____ 
____ ho_e fl_n ____ 
____ ____ ____ ____
The simulation information for the second round is printed as
Round 2
Creature (hop east 2 1) takes action:
Instruction 2: go 1
Instruction 1: hop
____ ____ ____ ____ 
____ ____ ____ ____ 
____ ho_e fl_n ____ 
____ ____ ____ ____ 
Creature (flytrap north 2 2) takes action:
Instruction 3: go 1
Instruction 1: ifenemy 4
Instruction 2: left
____ ____ ____ ____ 
____ ____ ____ ____ 
____ ho_e fl_w ____ 
____ ____ ____ ____
In the concise mode, you print the initial state of the world in the same way as in the verbose 
mode. When printing the simulation information for the i-th round, you first print “Round ” 
followed by the newline. Then you print the final action of each creature in turn, with one 
creature per line. The format is:
Creature (   ) takes action: 
Same as in the verbose mode, the four-tuple (   ) shows the 
state of the creature right before it takes the action.  should be replaced with
the last instruction the creature executes for its turn, which is one of the hop, left, right, and
infect instruction.
After printing the final actions for all the creatures, you print the updated layout at the end of 
this round.
For the same world file as above:
4
4
hop east 2 0
flytrap east 2 2
In the concise mode, the simulation information for the first round is printed as
Round 1
Creature (hop east 2 0) takes action: hop
Creature (flytrap east 2 2) takes action: left
____ ____ ____ ____ 
____ ____ ____ ____ 
____ ho_e fl_n ____ 
____ ____ ____ ____
The simulation information for the second round is printed as
Round 2
Creature (hop east 2 1) takes action: hop
Creature (flytrap north 2 2) takes action: left
____ ____ ____ ____ 
____ ____ ____ ____ 
____ ho_e fl_w ____ 
____ ____ ____ ____
There are no blank lines in the output for both the verbose and concise mode.
VIII. Source Code Files and Compiling
There is a source code file located in the Project-3-Related-Files.zip from our 
Canvas Resources:
world_type.h: The header file that defines a number of types for you to use.
You should copy this file into your working directory. DO NOT modify it! 
You need to write three other source code files. The first file is named as simulation.h, 
which contains the declarations for all the functions you write, just like the p2.h in our project 
two. The second file is named as simulation.cpp, which contains all the implementations of 
those functions declared in the simulation.h. The third file is named as p3.cpp, which 
contains only the main function. After you have written these files, you can type the following
command in the terminal to compile the program:
g++ -Wall -o p3 p3.cpp simulation.cpp
This will generate a program called p3 in your working directory. In order to ensure that the 
online judge compiles your program successfully, you should name you source code files exactly 
like how they are specified above.
IX. Implementation Requirements and Restrictions
1. When writing your code, you may use the following standard header files: , 
, , , , , and . No 
other header files can be included.
2. You may not define any global variables yourself. You can only use the global constant ints 
and string arrays defined in world_type.h.
3. Pass large structs by reference rather than value. Where appropriate, pass const references /
pointers-to-const. Do not pass lots of little arguments when you can pass an appropriate, larger 
structure instead.
4. All required output should be sent to the standard output stream; none to the standard error 
stream.
5. You should strive not to duplicate identical or nearly‐identical code, and instead collect such 
cod         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
