# L7 - 02.19.19
Note: was in class

## Const 

```c
void strcpy(char *t, const char *s);
```
> char *t is the target
> const char *s is the source 
> 

Const tells us that this function won't modify what s points to

```c
const char *p = &c;

//NO 
*p = 'A'; // ERROR compiler, cannot change the thing 
          // that p points to     
// YES
p = &d; // WORKS - change p itself, where it points

char *q = (char *) p; // cast const p to non-const
*q = 'A'; // WORKS 
```

Const is most useful for documentation and accident catching with the compiler errors, but you can work around it 

## Pointer to Function 
```c
void main(){
  int a[5] = {1, 27, -12, 0, 5};
  qsort (&a[0], 5, sizeof(int), ________); 
}
```

What goes in place of the blanks?

qsort prototype: 
```c
void qsort(void *a, size_t num, size_t size, 
          int (*f)(const void *, const void *));
```
> a is the pointer to what we are sorting
> num is the number of elements in whatever we're sorting
> size is the size of each individual element
> and the last is a pointer to a function 
> 

```c
int (*)(const void *, const void *)
```
> The (*) indicates a poitner to a function
> int indicates the return type of that function
> There are two inputs to this function, both are const void pointers
> 

qsort implements quicksort algorithm with a starting address of an array. It knows how big they are so it can recognize chunks of elements for the operation of swapping.

qsort MOVES elements but it does not know how to COMPARE elements. That is where the (*f) comes in. The function that qsort is called with to point to is the fucntion that knows how to COMPARE the elements. 

We must write the function that compares two elements and qsort will repeatedly call it. So in calling qsort, we have to pass an argument to it that is a pointer to a comparator function. 

For example: 
```c
 qsort (&a[0], 5, sizeof(int), &compInt); 
 // where compInt is a function we implement to 
 // compare integer elements
 ```

**typedef** introduces a a synonym for a type. This is where size_t comes from. For example:
```c 
typedef int MyInt;
```

Implementing the comparator function: 
```c
int compInt(const void *x, const void *y){
  int *a = (int *) x;  // We know we're sorting int
  int *b = (int *) y;  // so we cast void* to int*
          // Now that they're casted, we can deref
  if (*a < *b) 
    return -1;
  else if (*a > *b)
    return 1; 
  else 
    return 0; 
}
```

Alternatively to assigning a and b, we could have dereferenced directly after casting as follows: 
```c
  *(int *)x - *(int *)y; 
```
 This will automatically return 0 if they're equal, a positive number if x > y, and a negative number if y > x. So the entire above compInt can be replaced by returning the above. 
 
Note: when calling qsort, you can also just call compInt without &compInt and it will automatically cast it to an address

```c
int main(){
  int(*f1)(void *);  //Declare a pointer to a funct
                     // f1 is a variable 
                     // f1's type is int(*)(void *)
  int *f2(void *);   // f2 is a protype of a funct
                     // the function takes void *
                     // and returns int *
  int(*f3[5])(void *); // f3 is an array of
        // five things that are each of type of f1
}
```

**Parsing function pointers**
```c
int foo(void *p) {... return 5;}

f1 = &foo;  // f1 is a pointer to foo

foo(NULL);   // WORKS
(*f1)(NULL); // WORKS, same as foo(NULL);
f1(NULL);    // WORKS, call without dereferencing 
             // the pointer works the same way
             
             
f3[0] = foo; // assign foo as first element of f3
fr[0](NULL); // WORKS, same as above 
```

## Complex Declarations

Strategy #1 - from outside inwards - Replace inside elements with E
```c
char (*(*x())[])()

1. char (E)()
    where E is *(*x())[]
    E is a function returning char 
2. Unhide E, *(*x())[] --> *(E)[]
    E is *x()
    E is an array of pointers 
3. Unhide E, *x()
    x is a function that returns a pointer 
    
Read it backwards:
x is a function that returns a pointer to an array
of pointers to functions returning a char 
```

Strategy #2 - counterclockwise and outward
```c
char (*(*x())[])()

1. Start at name
    "x is"
2. swirl forwards
    () "a function with unspecified parameters"
3. swirl backwards
    * "that returns a pointer to"
4. swirl forwards
    [] "an array of"
5. swirl backwards
    * "pointers to"
6. swirl fowards
  () "functions with unspecified parameters"
7. swirl backwards
    char "that return char"
    
Read it all together!
```

## Structs 

Structs are a way of gouping or aggregating things. When you declare a struct, you declare a new type. Thist must go at the top of the file that uses it or in a header file.

```c
struct Pt{    // Declare a struct - a point (x,y)
  double x;
  double y;
};

int main(){
  struct Pt p1; // Create p1 of type Pt
  
  p1.x = 1.0; // Assign the x component
  p1.y = 2.0; /// Assign the y component 
  struct Pt *q1 = &p1; // A pointer to the struct
}
```
p1 is a 16 byte structure on the stack. The size of a struct object is the size of its component parts, in this case two doubles which are each 8 bytes. 

@startuml
object q1
object p1
p1 : x = 1.0
p1 : y = 2.0 
q1 --> p1: points to
@enduml

p1 is declared on the stack as successive variables. q1 points to the first one. Say 1.0 is at adress 2000 and 2.0 at adress 2008. The struct goes from 2000 to 2016

What is the difference between &p1 and &p1.x? They address the same thing, but a different type. 
> &p1.x is of type double *
> &p1.x + 1 --> 2008, jump to next double
> &p1 + 1 --> 2016, jump to next struct
> Will not work: q1 = &p1.x because q1 is declared as a pointer to a struct 

Dereferncing a pointer to a struct:
```c
  (*q1).x = 3.0; // You access the struct directly,
                // so the dot notation works 
  q1 -> x = 4.0; // the -> is how to access the
                // components of a struct by
                // its pointer
```
Allocating an array of structs:
```c
struct pt *p = malloc(sizeof(struct pt)*5);
p[2].x = 100; // VALID SYNTAX
            // p[2] = *(p +2)
            // This will assign 100 to the 
            // third struct's x field
(p + 2) -> y = 200; // VALID SYNTAX 
```

> sizeof( p) is 8, it's a pointer!
> sizeof(*p) is 16, it's a struct of size 16


# L7/9 - 02.21.19 
Note: missed class, notes from girl from mesorah 

## Passing by value 

```c
// DOES NOT WORK, PASS BY REFERENCE
void transpose(struct pt p) {
  swap(&p.x, &p.y);

}
// p is a local variable in transpose 
// this takes the values of p given in main

int main(){
  struct pt p = {1.0, 2.0}
  transpose(p);
}
```


```c
// WORKS, PASS BY VALUE
void transpose(struct pt *p) {
  swap(&p -> x, &p -> y);

}
// p is a local variable in transpose 
// this takes the values of p given in main

int main(){
  struct pt p = {1.0, 2.0}
  transpose(&p);
}
```

In both cases, the & is there in the call to swap because swap takes a pointer. So regardless the and will be there. The difference is in the first version which does not work, we are passing a struct itself to swap. Whereas in the second version, we are passing a pointer to a struct by passing &p. The fact that p is used in both transpose and main should not be confused. In the workign second version, transpose's p is a pointer to main's p, which is a struct.

## Linked List in C

```c
struct Node{
  struct Node *next;
  int val; 
};
```
The above declares a type of struct, a Node, which forms a linked list by having two fields: a next field and a value itself. 

**Self-referential structure** The declaration of a node with a node as one of its fields works because the field is a pointer a struct. You cannot put a struct itself as  field. A pointer is 8 bytes, so the compiler can identify the size of the struct which is 12 bytes total, but it will most probably be allocated 16 bytes, as a multiple of 8.

What is the significance of allocating in multiples of 8? if we assign these things to be an array, they each have to occupy address multiples of 8 apart in order to use a pointer to move from one element to the next (pointers are of size 8). 


### create2Nodes()

```c
struct Node *create2Nodes(){
  struct Node *n1 = malloc(sizeof(struct Node));
  n1 -> next = NULL;
  n1 -> val = 100; 
  struct Node *n2 = malloc(sizeof(struct Node));
  n2 -> next = n1;
  n2 -> val = 200; 
  return n2
}

int main(){
  struct Node *head = NULL;
  head = create2Nodes();
  printf("head n2 %d--> n1 %d", 
        head->val, head->next->val);
  free(head->next);
  free(head); 
}
```
Prints: 
> head n2 200 --> n1 100
@startuml
object null
object head
head --> null: points to
@enduml

@startuml
object node1
node1 : val = 100
object node2
object head
head --> node2: points to
object null
node2 : val = 300
node2 --> node1: points to
node1 --> null: points to
@enduml


Every time you malloc something, you have to subsequently free it. Free it in the proper order so that you do not lose a reference to the next thing that you have to free. 

Head is a pointer to a node, n2 (which is a node)
An empty linked list is a head pointer that points to null, like on the left before enterring the function create2Nodes()

|Command| Stack | Heap |
| ------|----|-------|
|struct Node *head = NULL | head [ ] | |
|*enter create2Nodes()* | | |
|struct Node *n1 = malloc... | n1 [ ]| |
|n1 -> next = NULL| |next [ ] |
| n1 -> val = 100| | val = 100|
|struct Node *n1 = malloc... | n2 [ ]| |
|n1 -> next = NULL| |next [ ] points to next[ ] above|
| n1 -> val = 100| | val = 200|
|*function return to main* |Cleared | Maintained|
|head = create2Nodes()|head[ ] | points to next #2| 
|free(head->next) |  |Remove memory of next, val #1 | 
|free(head) | | Remove memory of next, val #2| 

You can use nodes themselves in a linked list as a while loop header.
```c
Node *n = head;
while(n){
  printf("%d-->", n -> val);
  n = n -> next; 
}
```
# L9 - I/O
## Standard I/O and redirection 

The keyboard is the **standard input.** Standard text stream is a series of lines: a series of characters followed by a new line character. You can read one character at a time from standard input (with getchar()). 

```c
int getchar(void) // Returns a char or EOF
```

The terminal screen is **standard output.**

**Standard error** is to output a character stream onto the terminal 

**Redirection**: 
> **executable > filename** redirects output into the named file, rather than on the terminal screen 
> **executable < filename** inputs the required values for the executable from the filename 
> Redirections can be combined without differentiation as to order of input/output
>  **executable < filename1 > filename2** run the executable with input from filename1 and put the output into filename2
>  **executable 2> filename** redirects the output into filename 
>  **executable >> filename** The output of the executable will be appended to whatever exists in filename, rather than overwriting everything 
>  **executable >> filename 2>&1** redirect the error to the same place as the output

## Pipeline

Pipeline feeds output of one program to the input of another 
> **executable < filename | executable** 
The following code will print any lines that the pattern hex is found 
```c
$./a.out < myin | grep hex 
```
echo prints whatever you write so:
```c
echo 555 | ./a.out
```
The above code will input 555 into the program of a.out 

# L09 02.21.19

Note: overlaps with 2/21
Note: missed class, making up from Serena notes and textbook

## Standard I/O/Error

**scanf()** reads user input from theyboard and parses that characters, recognizing it as either an integer, double, string, etc.

Every program has some notion of standard input and standard output.
**standard input** where the typical input comes from  - keyboard
**standard output** where the typical output comes from - screen

>Programs associated with stdin/stdoout:
> - **scanf( )** always reads from standard input
> - **printf( )** always reads from standard output
> These are both **variadic functions** which means they take a variable number of arguments(they can have 0 or 20 or 100 or 437 arguments)

By default, when a program starts up, standard input is connected to the keyboard, and standard output to the screen 
**standard error** is also connected to the terminals creen. We can send output to standard error

>Example of standard error: valgring prints its own message, these messages actually go to standard error, a different channel than standard output. 
>
> Both standard output and error go to the terminal screen int his case, although they can go to separate places 
> 
Three standard I/O channels to start with:
> standard input - connected to keybord
> standard output - connected to terminal output
> standard error - connect to terminal output 

## Redirection

**It is possible to redirect output using the > shell operator**
> Output can be redirected to a particular file
> Either appends or overwrites the output of an existing file with that name


```c
./a.out > myout
```
> The output goes to the file "myout", not to erminal screen
> The input is still taken from the terminal, if there is input to take
> This overwrites the existing "myout" file, and creates a new file
> 

```c
cat myout
```
> Dumps the output of ./a.out onto terminal screen
> 
```c
./a.out >> myout
```
> This appends the output to the contents of myout, instead of replacing 
> 

```c
cat myout
```
> The 'cat' command reads a file byte by byte and sends output to standard output and then it apepars in the terminal 
> 
```c
cat a.out
```
> Outputs lots of random non-human readable bytes
> 

**It is also possible to redirect input using the < shell operator**

Assume there is a file "myin", which has nothing other than the integer "678" in its text.

```c
cat myin
```
> Displays 678 on the screen
> 
```c
./a.out < myin >> myout
```
> Input is read from myin and the output of ./a.out is appended to myout
> 

Differentiate between standard I/O and file I/O:

>The program takes it input from myin and writes it output to myout, this is standard I/O not file I/O
>
>The shell program is doing the I/O redirection - you are telling the shell program to run a.out, but before that, rearrange the program's (a.out) standard input and output so that it comes from myin and goes to myout
>
>The file (a.out) does not know where theinput is ocming from or where the output will be sent to 
>
> It is the shell that interprets the command as standard I/O before the program is run 
> 
> The mechanism of standard I/O has been modified to come from and go to files instead of the keyboard or terminal, respectively 
> 
> To read from or write to more than one file, you need to do file I/O: opening files in the program usi ng the file name and reading the file. File I/O entails opening and reading files from within a program
> 

Note: the shell interprets the command before any command line arguments to a program are considered. So '< myin >> myout' are not interpreted as command line arguments to a.out but rather as part of a command for standard I/O

Let's say that within a.out there was 2 scanf( )'s. The standard I/O will not switch back to the keyboard for the second one. So the second scanf( ) would fail because it is coming from a file (myin). The return value (of scanf( ) #2) would be 0 and you would have some garbage value for the second input. 

In general, scanf() keeps reading until it sees whitespace (newline character, tab, space)

Underlying UNIX philosopy: treat everything as a file. Even the speaker on your computer, for an example. There is a special file representing that device and you write bytes to it that are outputted as sound.

```c
./a.out < myin >> myout 2>&1
```
> Redirect standard error to standard output 
> Redirect standard output to myout (in append mode) and redirect "2" (standard error) to where "1" (standard output) goes 
>


### Pipeline - redirect input from one program to come from output of another

```c
echo 5
// Prints:
5
```
> echo will print out whatever the commmand line arguments are
> It sends its input to standard output
> A newling character is also sent to the end 

```c
echo | ./a.out > myout
```
The **| operator**  is used to connect the standard output of a program to (using a direct pipeline) to the standard input of another program
> In the above example, the output of echo becomes the input of a.out

Philosophy of UNIX: you have small programs that can be stitched together to do what you want

**Pipeline example** - the w command tells you who's logged inr ight now, and other info 

Once logged into clac:
```c
w

// PRINTS:
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
dfj2106  pts/0    207.237.164.217  14Mar19 13days  0.64s  0.23s vim mdb-lookup.
flq2101  pts/2    tmux(1217).%0    07Mar19 37:18m 21.42s 20.91s vim mdb-lookup-
dfj2106  pts/3    207.237.164.217  14Mar19 13days  0.09s  0.03s vim mdb.h
htd2109  pts/4    tmux(24670).%0   04Mar19 22days  2.82s  2.77s vim mylist.c
.
.
.
ds3439   pts/108  160.39.29.16     13:13    4:51   0.08s  0.08s -bash
jk4102   pts/110  160.39.130.161   13:25    2:47   0.16s  0.16s -bash
ob2267   pts/112  160.39.162.232   13:29    4:19   0.07s  0.07s -bash
hcc2134  pts/114  160.39.249.61    13:30   43.00s  0.07s  0.07s -bash
nelson   pts/118  tmux(30877).%3   04Mar19 22days  0.51s  0.44s vim mylist.c

```

```c
w | tail -n 4

// PRINTS:
jk4102   pts/110  160.39.130.161   13:25    3:57   0.16s  0.16s -bash
ob2267   pts/112  160.39.162.232   13:29    5:29   0.07s  0.07s -bash
hcc2134  pts/114  160.39.249.61    13:30    1:53   0.07s  0.07s -bash
nelson   pts/118  tmux(30877).%3   04Mar19 22days  0.51s  0.44s vim mylist.c
```
> tail takes input and outputs the last n lines (default is 10). Here, we are feeding the output of w (as the first 4 and last 4 lines are showed above), and putting all of that list of users into the input for tail. We are then telling tail to output (print to terminal) only the last 4 lines. As demonstrated above 
> 

```c
w | tail -n +3

//PRINTS all of the output of w 
// without the first two liines

```
> Skips the first two lines
> 

```c
w | tail -n + 3 | grep vim
```
> grep looks for the characters "vim" in a given input and outputs wherever they are found. This will print all lines of w where vim appears, with "vim" in red, and skipping the first two lines
> 

```c
w | tail -n +3 | grep vim | cut -d " " -f 1

// PRINTS
dfj2106
flq2101
dfj2106
htd2109
.
.
.
```

> cut removes sections of lines, delimitted by space " " so this will print out each line of w, minus the top 2, where vim appears, but it will only print from the first character until the first space, i.e. the first field of the "table", i.e. the UNIs
> 

```c
w | tail -n +3 | grep vim | cut -d " " -f 1 | 
  sed 's/$/@columbia.edu/'
  
  // PRINTS:
dfj2106@columbia.edu
flq2101@columbia.edu
dfj2106@columbia.edu
htd2109@columbia.edu
.
.
.
```
> sed substitutes, /$ is the end of the line$
> 
```c
w | tail -n +3 | grep vim | cut -d " " -f 1 | 
  sed 's/$/@columbia.edu/' | sort > vimusers.txt
``````
> Sorts (sort) the output of UNIs (cut -d " " -f 1) appended by @columbia.edu (sed 's/$/@columbia.edu/)from all users logged in (w) minus the first two (tail -n +3) who are currently running something that contains "vim" (grep vim) and puts the output in a file called vimusers.txt
> 
> After running the above command, there is a file called vimusers.txt which has the list of emails of users in alphabetical order. 
> 

### printf( )

**The printf( ) prototype:**

```c
int printf(const char *format, ...); 
```
printf( ) takes char * (a string) and the arguments (if any) that come after depends on the format within the char * :
> %d - int
> %u - unsigned int
> %ld - long
> %lu - unsigned long
> %s - string (char * )
> %f - double
> %p - memory address (void * ), a generic pointer 

Functionality of printf( ):
> Keeps printing, and if it encounters the % sign, then it knows that the next character will determine the type printed from the next argument in the list
> 
> Grabs bytes on the stack (not allocated) as it encounters type specifiers (4 bytes for int, etc.)
> 
> Normally arguments are put ont he stack (fixed parameter list), but for printf( ), this is determined at runtime because we do not know in advance how many parameters there will be 
> 

### scanf( )

**The scanf( ) prototype:**

```c
int scanf(const char *format, ...);
```
scanf( ) takes a char * (a pointer to a char). It reads chars from standard input, interprets them according to the format specified, and stores results. Each argument must be a pointer, indicate where to store the corrresponding input. It returns the number of items found. 

Example: 
```c
// Read 25 Dec 1998

int day, year;
char monthName[20];
scanf("%d %s %d\n", &day, monthName, &year);
// &day and &year turns a variable into a pointer
// monthName is already a pointer
```

**The sscanf( ) prototype:**

```c
int sscanf(const char *input_string, const char *format, ...)
```
> Instead of reading from standard input, it reads from a string 
> Going from string to int
> Equivalent to Integer.parseInt( ) in Java
> 
> format - a string that contains one or more of: whitespace characters, non-whitespace characters, format specifiers
> 
Example using sscanf( ):
```c
int main () {
   int day, year;
   char weekday[20], month[20], dtm[100];

   strcpy( dtm, "Saturday March 25 1989" );
   sscanf( dtm, "%s %s %d  %d", weekday, month, &day, &year );

   printf("%s %d, %d = %s\n", month, day, year, weekday );
    
   return(0);
}
```

**The sprintf prototype**

```c
int sprintf(char *str, const char *format, ...)
```

> Instead of printing to standard output (screen), it would, for example, write to memory pointed to by output_buffer
> This is equivalent to Java's toString because it can take any types of arguments (int/double) and incorporate them into a string 
> 
Example using sprintf( ):
```c
int main () {
   char str[80];

   sprintf(str, "Value of Pi = %f", M_PI);
   puts(str);
   
   return(0);
}
// M_PI is an internal variable of math.h
// PRINTS: 
// Value of Pi = 3.141593
```
**The snprintf prototype**
```c
int snprintf(char *output_buffer, size_t size, const char * format, ...);
```
> This is a safer version of sprintf. It prevents you from writing past what is allocated in output_buffer if you specify the size. 
> 

### More on the I/O channels

There is only 1 input (standard input) and 2 output (standard output and erorr) channels. This is limitted.

For example, you cannot read from two files and output to five files with redirection. 










