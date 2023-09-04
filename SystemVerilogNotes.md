# System Verilog
## Standard Data Types and Literals
### Verilog Data Types are:
reg, logic and integer
### System Verilog Data Types are:
bit, byte, shortint, int, logic(which replaces reg) and longint
### Bit vs logic:
A bit is a 2 state variable while logic is a 4 state variable, this means that a bit cannot represent high impedance and it cannot represent unknown state. If there is a design where we wish to know if there is a high impedence or unknown state this cannot be demonstrated with a bit but can be with a logic variable.
### Port Rules:
Port rules are much the same with verilog with the exception of the higher module does not necessarily need a net to be connected to a port. Any data type can be used and system verilog will treat it as a net if required. A net is essentially just a wire that cannot store any data but can connect one point to another.
### Error Multiple drivers:
A variable can have multiple drivers if it is of type wire but cannot have multiple drivers if it is of type logic.
### Always Comb:
This will interpret this block as combinational logic, it will write its own sensitivity list and timing control. Using this block only one of these blocks can be a driver for a single given variable.
### Unsized Literal:
This can be used to populated an array entirely with one value. To fill a logic array with 1's we can do this as follows:
``` sv
logic [5:0] databus;
databus = '1;  // 111111
```
### Time Literals:
This is essentially the sleep function in C. It is simply done by suffixing a number with a unit of seconds. The delay can also be used in steps which is some value unique to the design which will remain constant in any given bitfile. Each of these can be called as follows:
``` sv
#42ps       x <= 0;
#3.14s      x <= 1;
#1step      x <= 2;
```
To alter the amount of time delayed with a step1 command we can use the timeunit and timeprecision keywords as follows:
``` sv
timeunit 1ns;
timeprecision 100ps;

// This is functionally equivalent to:

timeunit 1ns/100ps;
```
## Procedural Statements and Procedural Blocks
### Named Block:
Names can be attached to blocks to easily differentiate between a number of nested blocks. This can be done for a module, begin, task, function, fork, primitive and interface. This can be does as follows:
``` sv
intial begin : block_a
    initial begin : block_b
    end
end
```
In system verilog unlike verilog unamed blocks are able to declare local variables, in verilog only named blocks can declare local variables.
### Loops:
In system verilog a variable can be declared inside the for loop such as it can in C, the syntax is also the same as C. The foreach keyword can be used in verilog to iterate through an array. This can be dones as follows:
``` sv
arr [7:0] [2:0];

foreach (arr [k,l]) begin
    // Loop body
end

// This is functionally equivalent to:

for (int k = 7; k >= 0 ; k = k - 1) begin
    for (int l = 2; l >= 0; l = l - 1) begin
        // Loop body
    end
end
```
The while loop also exists and can be used much the same way as is it in C. Break and continue work the same way that they do in C. In the below example the loop contents will only be executed on the positive edge of the clock signal.
``` sv
do
    @(posedge clk)
    count = count + 1;
while (enable);
```
### Case:
There is a priority list based on order of occurrence. A parallel case statement is when case statements are non overlapping so prioritisation is not required. A fullcase implies a default statement, this default case is for the purpose of avoiding forming a latch aswell as whatever other function it may provide. A priority case keyword tells the simulation tool that this case should be a full case (don't infer latch). If a branch is ever not reached then the simulation tool will give an error, this means that this keyword can be used to detect if there is ever a run of the case statement where a branch is not entered. This can also be used with an if for the same purpose. This is used as follows:
``` sv
priority case (1'b1)
    en_a    :   op = a;
    en_b    :   op = b;
    en_c    :   op = c;
endcase
``` 
A unique case is a case that is a full_case and parallel_case, this means that no cases overlap and that one branch should always be entered. If either of these conditions are not met the simulation tool will give an error. This again can also be used with an if for the same purpose. This is used as follows:
``` sv
unique case (1'b1)
    en_a    :   op = a;
    en_b    :   op = b;
    en_c    :   op = c;
endcase
```
### if and only if:
if and only if can be used with 'iff', this can be added to a sensitivity list to prevent the list from being entered unless a condition is entered. It can be used as follows:
``` sv
always @(posedge clk iff (gate == 1)) begin
    // Block body
end
```
### Example Hardware Element Models:
A combinational logic element can be created using the following block of system verilog
``` sv
always @(a or b or sel) begin
    if (sel ==1)    op3 = a;
    else            op3 = b;        
end 
```
A Latch can be modeled using the following always block:
``` sv
always @(sel or a) begin
    if (sel)    op2 <= a;
end
```
A register that is edge sensitive can be modelled with the followed always block:
``` sv
always @(posedge clk or posedge rst) begin
    if (rst)    op1 <= 8'h00;
    else        op1 <= data_in;
end
```
### always blocks:
always comb does not contain any timing or event controls, there cnanot be two always comb blocks that assign to some same variable. It is automatically trigerred at time 0 after always and intial blocks, it is also sensitive to changes in any input to a called function.     
always_latch is very similar to an always comb block but will issue a warning if the block does not infer latched logic.  
always_ff is very similar to an always comb block too but will issue a warning if the block does not infer registered logic.
## Operators
### Pre and Post-Increment/Decrement Operators:
These work the same as they do in C, this means the following lines are all valid in sytem verilog:
``` sv
i++;
b = 1;
a = b++;
a = --b;
```
### Equivalence Operators:
== This is logical equality, it will return true if the numbers are fully the same and will return false if there is any difference where one number has a 0 where another number has a 1. However if there is a high impedance or an unknown number, the result of the comparison will also be unknown.
=== This is an exact comparison where each number has to be exactly equal in order for the comparison to result true.
==? This is wildcard equivalence, if there are any characters that are of unknown or high impedance they will simply be ignored and only the positions where each number has a set value will be taken into account.
### inside Operator:
This is used to check whether a value is any one of a list of values, it can be used as follows:
``` sv
if (a inside {bar[1], bar[2], bar[0], 0, 1});
```
The operator can be used in combination with the case statement and can be used to create a case as follows:
``` sv
case(a) inside
    0, 2        : $display("0 or 2");
    [3:5]       : $display("3, 4 or 5");
    1, [6,7]    : $display("1, 6 or 7");
    default     : $display("value>7");
endcase
```
### Assignment Patterns:
A number of assignment patterns are given as follows:
``` sv
arr1 = '{0, 1, 2, 3};       // 0   ->   0
                            // 1   ->   1
                            // 2   ->   2
                            // 3   ->   3

arr1 = '{3:1, default:0};   // 0   ->   0
                            // 1   ->   0
                            // 2   ->   0
                            // 3   ->   1

reg [15:0] mem [0:1023] = '{default:0}; // Entire memory bank is initialized to 0                            
```
## User-Defined Data Types and Structures:
### typedef:
typedef is used to create a type name and set it to be a certain primitive type of a certain size, this can now be used as a legal type.
### Enumerated Type:
This is a type that has user defined values stored, for example:
``` sv
typedef enum {idle, start, pause, done} state_t;
state_t state;
state = idle;
state = start;
state = pause;
state = done;
```
Enumerated type values are by default just 1, 2, 3, 4..... However, it is possible to set the values that each name represents, we can expand from the following code snippet with this:
``` sv
typedef enum {  idle = 1,
                start = 2,
                pause = 4,
                done = 8 } state_t;
```
Range notation can be used to make the type declaration faster to write. This can be demonstrated with the following two code snippets which are funcitonally equivalent:
``` sv
typedef enum {s[2]} seq_t;
```
``` sv
typedef enum {s0, s1} seq_t;
```
The base type of the enumerated type can be set as shown in the following code snippet:
``` sv
typedef enum bit[1:0] { idle = 1,
                        start = 2,
                        pause = 4,
                        done = 8 } state_t;
```
The following are the access methods for the Enumerated Type:

| Method  |         Description                     |
| :------:|:---------------------------------------:|
| first() | Returns the first value                 |   
| last()  | Returns the last value                  |  
| next(N) | Returns next Nth value from current     |               
| prev(N) | Returns previous Nth value form current |                   
| num()   | Returns number of values                |    
| name()  | Returns string of equivalent value      |              
### Structures:
Declared yusing the struct keyword as is done in C.
``` sv
typedef struct {
    logic id, par;
    logic [3:0] addr;
    logic [7:0] data;
} frame_t;

frame_t f1;
frame_t two_frames[1:0];
```
The elements of this struct can then be read individually using the following syntax:
``` sv
data_read = f1.data;
```
Using the same syntax we can also set data individually:
``` sv
f1.data = 1'b1;
```
There is then also a number of ways to assign to multiple variables at once and they work as follows:
``` sv
// This will assign to the members of the struct in the order in which they are declared
f1 = '{1'b1, 1'b1, 4'h0, 8'hff};

// This will assign to the members of the struct based on the name of the member
f1 = '{id:0, par:1, addr:0, data:0};

// This is how one would assign to a struct array
two_frame = '{'{0,0,0,255}, '{1,1,1,0}};
```
### Packed Structure
This allows a structure to be accessed like a normal array. All nested features must also be packed. This structured can then be accessed with the notation used for structures but can also be accessed with the notation used for arrays.
``` sv
// Both of the following work
addrbuf = f1.addr;
addrbuf = f1[11:8];
```
### Packages:
Packages can be created like modules with a package and endpackage keyword, they can be used like header files in C, however they are accessed like namespaces in tcl. This would be necessary to create an interface where two different variables are of the same type and can assign to one another, because even if they were identical assignments between the two would not work.
## Hierarchy and Connectivity
### Connectivity
Explicit named ports work the same as verilog but implicit ordered ports can be used where signals are connected by order of decleration and port connections. For the port connections of both of the following types they can be combined with explicity name connections in one instantiation
#### .name connection
This can be used to connect a port when the name and size of the signals are the same, this can be shown by the two following functionally equivalent module instansiations
``` sv
// This is the .name port connection
count c4 (.clk, .rst, .ld, .data, .cnt, .val);
// This is the traditional functionally equivalent explicitly named port connection
count c5 (.clk(clk), .rst(rst), .ld(ld), .data(data), .val(val));
```
#### .* connection
For a module instansiation a number of equivalent .name compatible can be connected at once using the * as the name argument.
``` sv
// This is the .name port connection
count c4 (.clk, .rst, .ld, .data, .cnt, .val);
// This is the functionally equivalent .* port connection
count c5 (.*);
```
