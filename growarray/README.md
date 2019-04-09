# Grow Array

*by Matthew Juran*

![Slats](Slats.jpg)
*Photograph at Neary Lagoon in Santa Cruz, California, spring 2013.*

For me much of the apparant value in computer programming languages beyond C is in variable length memory things like strings and sets. In this essay I show how I've brought part of the Go slice idea for variable length collections back to C.

Before the main part of the essay, which is a relatively advanced programming topic, I'm going to try to describe pointers for a wider audience.

---

I've been told that in the modern past computers were people that calculated hard math problems as teams. Perhaps the thought of computers computing without electronics can help illustrate pointers.

The math circuits of the contemporary computer machines today have been applied to tasks that would have been impossible for people computers: printing presses that don't need to print, instantaneous worldwide communication by anyone, music without records, the art galleries of the world in every home, one useful map that shows every ocean and street. The pointer is a tool to solve math problems fast for those kinds of new things.

Memory is part of the computer circuit. Computer memory might be thought of like papers with intermediate numerical results that human computers use to start their own calculations, or the formulas used to get to a result; a program is held in electronic memory while the computer runs it, and the program likely uses that memory for iterative math and sharing results too:

```
Memory     Memory
Address    Content
 0             25    A memory address is for
 1             50    a number in memory.
 2              1
 3              0
 4            100
 5          65535
 6            'a'    Things like letters can
 7            'b'    be encoded as numbers.
 8            'c'
 9            'd'
10            'e'
11            'f'
12            'g'
13            add    Programs are a sequence
14       subtract    of numbers that tell the
15       multiply    computer what to do with
16         divide    other numbers in memory.
17       move mem
...
```

When an electronic computer is said to have more than a GB of memory that means there are billions of memory addresses available. If a computer is said to operate at GHz that means the programs can do a sequence that reaches toward billions of calculations every second. Bigger specifications tends to mean bigger ideas of what to do with the display, speakers, ports, and network that wouldn't be possible with less memory or a slower operating speed.

Anyway, a pointer is a memory address held as memory content in a contemporary computer:

```
Memory     Memory
Address    Content
...
10,000     10,050
...
10,050          1
...
10,100     move content pointed to 
           by 10,000 to 10,020
...
```

In this above example when the program at memory address 10,100 happens then the content of 10,020 will be 1. Pointers can be prominent in programming computers not just for languages like Go or C because they can be part of how the computer's commands operate, like how this example has memory addressing as part of the command.

Functions, implemented with "jump the program to a pointer and keep running" and "return to where the jump was from" commands, are one of the main uses of pointers. A function is a program that is reused within a program, and often additional memory input (arguments) for the function shouldn't be unnecessarily copied. Other tradeoffs are made by giving the function a pointer instead of the content, like the pointed to memory can be anywhere in memory without having to change the program to use exact memory addresses.

Variables are memory addresses that can have their content changed or used by the program. In Go and C a ```*``` character is used to state that a variable is a pointer (the variable's address has a pointer for content), and the same asterisk character is also used to get the content of memory pointed to by the pointer variable. The ```&``` character is to get the pointer to a variable and can be thought of as "address of".

Here's an example of a C function with and without a pointer input argument:

```
// In AddFive, 'to' is copied to a new memory address
// when the function is called, then another memory 
// address is used to return the function result 
// back to the caller. The memory used for 'to' and
// the return is then forgotten by the program.
int AddFive(int to) {
    return to + 5;
}

// In AddFiveTo, the memory content pointed to by 'to'
// is directly changed and persists in the program.
void AddFiveTo(int* to) {
    // set the content of 'to' to the original 
    // content of 'to' plus 5
    *to = *to + 5;
}

int main(int argc, const char** argv) {
    // this program has memory for this number
    int number = 10;
    
    // number is copied for AddFive, then the result
    // of the function is written back into the content
    // of the number variable.
    number = AddFive(number);
    
    // AddFiveTo does the same thing as AddFive but
    // is used differently.
    // "Add five to the content pointed to by &number."
    AddFiveTo(&number);

    // number now equals 20
}
```

The above program excerpt could be this in Go:

```
func AddFive(to int) int {
    return to + 5
}

func AddFiveTo(to *int) {
    *to = *to + 5
}

func main() {
    number := 10
    number = AddFive(number)
    AddFiveTo(&number)
    // number now equals 20
}
```

In terms of a historic computer department of people, perhaps a pointer can be thought of as one computer walking down the hall to get the result of another computer's calculation for input into their own calculations. The pointed to math result can be reused by writing it down on a new sheet of paper instead of taking the paper with the original calculations. It's not an exact analogy for pointers but might help in thinking how new computing memory is used.

For new computers pointers also represent arrays of things by pointing to the start of the array in memory which is the topic for the rest of this essay.

## No repeats

A theory is that part of making a high quality computer program is avoiding repeated source code. Source code needs careful thought and practices to get right, and a repeated thing gets in the way because a mistake in one has to be fixed in every repeat, or a new feature has to be copy-pasted to each.

The type systems of programming languages abstract useful kinds of numbers from the specifications of the underlying computer memory, but if a functionality is needed for varying types then the type systems can become a roadblock. Often this is worked around by using other features of the programming language instead of repeating similar source code for each type.

Besides repeating, this essay is about straightforward collections of numbers in computer memory, a useful computer programming tool. Go and C have the array which is variables of a type listed consecutively in memory with one overall variable that is a pointer to the start of the array. An array index offsets from the start to read a specific element in the array. Types help keep indexing about array elements instead of specific memory addresses which vary depending on the type. For example, if the array is made of ```char``` then each index refers to a byte in memory, but if it's made of ```int64_t``` then each index refers to eight bytes in memory.

Go slices simplify challenges found with arrays by having more information than just the held type and start pointer, like the length of the slice is part of the slice variable which means only the slice argument is needed to a function instead of both the array pointer and array length. An example use is that strings can be represented without the C style of null termination and the array walk needed to determine the string length.

The following set type that transparently allocates the right amount of memory illustrates a use of slices in Go:

*modified excerpt from set.go*

```
// the empty interface can have any type assigned to it
type Item interface{}

// a set is a slice of items which can be any length
type Set []Item

func (a Set) Add(an Item) Set {
	return append(a, an)
}

func (a Set) Combine(with ...Set) Set {
	l := len(a)
	for _, s := range with {
		l += len(s)
	}
	out := make(Set, l)
	i := 0
	for _, item := range a {
		out[i] = item
		i++
	}
	for _, s := range with {
		for _, item := range s {
			out[i] = item
			i++
		}
	}
	return out
}
```

In this example from the package [github.com/pciet/unordered]() a set of anything is implemented as a slice. If the capacity of the slice is less than what's being added to it with ```append``` then the array pointed to by the slice will be reallocated to a larger size in ```append```, which is why the slice should be assigned back to itself when the ```Add``` method is used:

```
func main() {
    // capacity set to 1 Item, length is 0
    s := make(Set, 0, 1)
    
    // now len is 1, cap is 1
    s = s.Add("abc")
    
    // backing array reallocated in the Add
    // len is 2, cap is increased
    s = s.Add("def")
    
    // should print:
    // abc
    // def
    for _, e := range s {
        fmt.Println(e)
    }
}
```

A way to reduce repeating source code because of varying types in Go is the ```interface{}``` which can have variables of any type assigned to it. In C this is similarly done with ```void*``` which is a pointer without knowing the memory length of what's pointed at. In both cases a cast to another type is necessary for use so the program knows the memory width of the variable.

The ```Combine``` method might look like this in C:

```
// "Uniform" means each element of the set should be the same type
typedef struct {
    int len;
    size_t elem; // number of bytes for each element
    void* content;
} UniformSet;

void CombineSets(UniformSet* s, int len, UniformSet* out);
```

Notably, array arguments like ```s``` for ```CombineSets``` in C are assumed to be arrays by the programmer. Array indexing can be used on any pointer variable, but a correct program must stay within the bounds of allocated memory which is why array length is also an argument. Another consideration is that the memory allocated for ```out``` in the function will have to be freed later; in Go the garbage collector means the programmer doesn't have to usually be concerned in the same way with freeing allocated memory.

I believe an experienced C programmer will understand the nuances of the pointer arguments for ```CombineSets``` without explanation, especially if they were looking at the implementation. ```s``` is an input argument that is an array of ```UniformSet``` with ```len``` for a length, and ```out``` is one ```UniformSet``` that will be allocated in the function and filled with the contents of the sets in ```s```.

The previous Go implementation of ```Combine``` is close to what it could look like in C, but there are shortcuts like the ```...``` for a variable number of inputs, ```:=``` that infers the type of a variable, ```range``` for slice element iteration, ```len``` for getting the length of the array pointed to by the slice, and ```make``` for handling the acutal memory size of ```out``` by having the compiler infer the allocation size. In C ```malloc``` requires using the ```sizeof``` function on types to manually calculate the exact memory size needed for the new allocation.

Part of the usefulness of the slice type is subslicing, which is changing the pointer or length values but in context of the abstracted array element indexing. I've implemented a subset of the slice functionality in C, without subslicing:

*grow_array.h*

```
#ifndef GROW_ARRAY_H_SEEN
#define GROW_ARRAY_H_SEEN

#include <stdlib.h>

// Inspired by the slice type from the Go programming language.

typedef struct {
    void* array;
    size_t elem; // size of each element
    int len; // number of elements in the array
    int cap; // total number of elements that fit into the current allocation (always >= len)
} GrowArray;

// When the array is appended a new backing array may be allocated and the original allocation freed. 
// To keep the grow array struct valid the var should be reassigned with every function call.
//
//   #define ARRAY_LENGTH 3
//   int in[ARRAY_LENGTH] = {0, 1, 2};
//   GrowArray a = GA_New((void*)in, ARRAY_LENGTH, sizeof(int));
//   a = GA_AppendArray(a, (void*)in, ARRAY_LENGTH);
//   for (int i = 0; i < a.len; i++) {
//       printf("[%d] %d\n", i, ((int*)a.array)[i]);
//   }
//   GA_Free(a);
//
// Expected Output:
// [0] 0
// [1] 1
// [2] 2
// [3] 0
// [4] 1
// [5] 2

GrowArray GA_New(void* array, int len, size_t elem);
GrowArray GA_Append(GrowArray a, GrowArray with);
GrowArray GA_AppendArray(GrowArray a, void* array, int len);
void GA_Free(GrowArray a);

#endif
```

*grow_array.c*

```
#include "grow_array.h"
#include "panic.h"

#include <string.h>

#define INIT_CAP 12

GrowArray GA_New(void* a, int len, size_t elem) {
    if (elem == 0) Panic("0 byte size");
    if (len == 0) {
        return (GrowArray){
            .array = malloc(elem*INIT_CAP),
            .elem = elem,
            .len = 0,
            .cap = INIT_CAP
        };
    }
    if (a == NULL) Panic("null array");
    GrowArray r = {
        .array = malloc(elem*len*2),
        .elem = elem,
        .len = len,
        .cap = len*2
    };
    memcpy(r.array, a, elem*len);
    return r;
}

GrowArray GA_Append(GrowArray a, GrowArray with) {
    if (a.elem != with.elem) Panic("mismatched element size");
    if (with.len <= (a.cap-a.len)) {
        memcpy(a.array+(a.len*a.elem), with.array, a.elem*with.len);
        a.len = a.len + with.len;
        return a;
    }
    // otherwise a isn't big enough
    int newCap = (a.cap*2)+with.len;
    GrowArray n = {
        .array = malloc(a.elem*newCap),
        .elem = a.elem,
        .len = a.len + with.len,
        .cap = newCap
    };
    memcpy(n.array, a.array, a.len*a.elem);
    memcpy(n.array+(a.len*a.elem), with.array, with.len*a.elem);
    GA_Free(a);
    return n;
}

GrowArray GA_AppendArray(GrowArray a, void* array, int len) {
    // ideally this would be improved to 
    // avoid unnecessary memory copying
    GrowArray w = GA_New(array, len, a.elem);
    a = GA_Append(a, w);
    GA_Free(w);
    return a;
}

void GA_Free(GrowArray a) {
    free(a.array);
}
```

Using those ```memcpy``` calls was satisfyingly simple even if doing it wrong can lead to mistake symptoms that seem impossible.

The task that led to this generic C array type was repeating ```GA_Append``` for multiple types in my source code and then having to write a test for it. Instead of powering through further with copy-paste I was able to not repeat that relatively complicated function because of careful ```void*``` memory-aware programming.

Here's my first pass of the test program for ```GrowArray```:

*tests/grow_array.c*

```
#include "../grow_array.h"

#include <stdio.h>

int main(int argc, const char** argv) {
    #define ARRAY_SIZE 5
    int array[ARRAY_SIZE] = {5, 4, 3, 2, 1};
    
    GrowArray ga = GA_New((void*)array, ARRAY_SIZE, sizeof(int));
    if (ga.array == NULL) {
        printf("null array field\n");
        return 1;
    }
    if ((ga.len <= 0) || (ga.cap <= 0)) {
        printf("len %d or cap %d invalid\n", ga.len, ga.cap);
        return 1;
    }
    if (ga.elem != sizeof(int)) {
        printf("elem %zu not equal to sizeof(int) %lu\n", ga.elem, sizeof(int));
        return 1;
    }
    if (ga.len != ARRAY_SIZE) {
        printf("len %d not equal to ARRAY_SIZE %d\n", ga.len, ARRAY_SIZE);
        return 1;
    }
    if (ga.len > ga.cap) {
        printf("len %d greater than cap %d\n", ga.len, ga.cap);
        return 1;
    }
    for (int i = 0; i < ARRAY_SIZE; i++) {
        if (array[i] != ((int*)ga.array)[i]) {
            printf("input array [%d] = %d not equal to grow array %d\n", i, array[i], ((int*)ga.array)[i]);
            return 1;
        }
    }

    ga = GA_Append(ga, ga);
    if (ga.array == NULL) {
        printf("null array field after append\n");
        return 1;
    }
    if ((ga.len <= 0) || (ga.cap <= 0)) {
        printf("len %d or cap %d invalid\n", ga.len, ga.cap);
        return 1;
    }
    if (ga.elem != sizeof(int)) {
        printf("elem %zu not equal to sizeof(int) %lu\n", ga.elem, sizeof(int));
        return 1;
    }
    if (ga.len != (ARRAY_SIZE*2)) {
        printf("len %d not equal to %d\n", ga.len, ARRAY_SIZE*2);
        return 1;
    }
    if (ga.len > ga.cap) {
        printf("len %d greater than cap %d after append\n", ga.len, ga.cap);
        return 1;
    }
    int comparison[ARRAY_SIZE*2] = {5, 4, 3, 2, 1, 5, 4, 3, 2, 1};
    for (int i = 0; i < (ARRAY_SIZE*2); i++) {
        if (comparison[i] != ((int*)ga.array)[i]) {
            printf("comparison array [%d] = %d not equal to grow array %d\n", i, comparison[i], ((int*)ga.array)[i]);
            return 1;
        }
    }

    ga = GA_AppendArray(ga, array, ARRAY_SIZE);
    if (ga.array == NULL) {
        printf("null array field after array append\n");
        return 1;
    }
    if ((ga.len <= 0) || (ga.cap <= 0)) {
        printf("len %d or cap %d invalid\n", ga.len, ga.cap);
        return 1;
    }
    if (ga.elem != sizeof(int)) {
        printf("elem %zu not equal to sizeof(int) %lu\n", ga.elem, sizeof(int));
        return 1;
    }
    if (ga.len != (ARRAY_SIZE*3)) {
        printf("len %d not equal to %d\n", ga.len, ARRAY_SIZE*3);
        return 1;
    }
    if (ga.len > ga.cap) {
        printf("len %d greater than cap %d after array append\n", ga.len, ga.cap);
        return 1;
    }
    int comparison2[ARRAY_SIZE*3] = {5, 4, 3, 2, 1, 5, 4, 3, 2, 1, 5, 4, 3, 2, 1};
    for (int i = 0; i < (ARRAY_SIZE*3); i++) {
        if (comparison2[i] != ((int*)ga.array)[i]) {
            printf("comparison array 2 [%d] = %d not equal to grow array %d\n", i, comparison2[i], ((int*)ga.array)[i]);
            return 1;
        }
    }

    GA_Free(ga);

    return 0;
}
```

This test program could be improved by reducing repeating and adding more test cases, but there's also a balance to find here with getting something working fast that's going to be tested during program development anyway.

A metaphor is that sometimes there's fish that are bigger, tastier, and easier to catch and prepare, or similarly there's kinds of food plants that grow better in your environment and are easier to harvest and perserve anyway. My experience is that it's not always easy to spend your limited time wisely but trying is the only way to improve. Try to avoid repeating, try to use good practices like writing test programs, and try to pace your time, but in the modern world good fishing doesn't always have catching.

## Generic type safety

The ```GrowArray``` type can be used in a type safe way by making a wrapper type that does the cast to ```void*``` behind a function that requires the held type for input. Set type safety for avoiding ```interface{}``` works similarly in Go.

*geometry.h*

```
#ifndef GEOMETRY_H_SEEN
#define GEOMETRY_H_SEEN

#include "point.h"
#include "grow_array.h"

#define RECTANGLE_TOP_LEFT 0
#define RECTANGLE_TOP_RIGHT 1
#define RECTANGLE_BOTTOM_LEFT 2
#define RECTANGLE_BOTTOM_RIGHT 3

typedef Point Rectangle[4];

typedef GrowArray RectangleGrowArray;

RectangleGrowArray RGA_Empty();
RectangleGrowArray RGA_Append(RectangleGrowArray a, RectangleGrowArray with);
RectangleGrowArray RGA_AppendArray(RectangleGrowArray a, Rectangle* array, int len);
RectangleGrowArray RGA_AppendElement(RectangleGrowArray a, Rectangle element);
Rectangle* RGA_Array(RectangleGrowArray a);
void RGA_Free(RectangleGrowArray a);

#endif
```

*geometry.c*

```
#include "geometry.h"

RectangleGrowArray RGA_Empty() {
    return GA_New(NULL, 0, sizeof(Rectangle));
}

RectangleGrowArray RGA_Append(RectangleGrowArray a, RectangleGrowArray with) {
    return GA_Append(a, with);
}

RectangleGrowArray RGA_AppendArray(RectangleGrowArray a, Rectangle* array, int len) {
    return GA_AppendArray(a, array, len);
}

RectangleGrowArray RGA_AppendElement(RectangleGrowArray a, Rectangle element) {
    return GA_AppendArray(a, &element, 1);
}

Rectangle* RGA_Array(RectangleGrowArray a) {
    return (Rectangle*)a.array;
}

void RGA_Free(RectangleGrowArray a) {
    GA_Free(a);
}
```

The documentation for package ```unordered``` shows how I would do this in Go:

```
// example for unordered.EqualSet
type Coordinate struct {
    X int
    Y int
}

// satisfies unordered.Comparable
func (a Coordinate) Equal(to unordered.Comparable) bool {
    if a != to.(Coordinate) {
        return false
    }
    return true
}

type CoordinateSet unordered.EqualSet

func (a CoordinateSet) Add(the Coordinate) CoordinateSet {
    return CoordinateSet(unordered.EqualSet(a).Add(the))
}
...

// Iteration requires a type assertion even with a wrapper:
...
    for _, coord := range set {
        if coord.(Coordinate).X == 1 {
            return false
        }
    }
...
```

## Using unordered sets

Here's an example from my source code written in 2017:

*excerpt from board_moves.go*

```
func (b Board) SurroundingPoints(from Point) PointSet {
	set := make(PointSet, 0, 8)
	for i := -1; i <= 1; i++ {
		for j := -1; j <= 1; j++ {
			if (i == 0) && (j == 0) {
				continue
			}
			f := i + int(from.File)
			r := j + int(from.Rank)
			if (f < 0) || (f >= 8) {
				continue
			}
			if (r < 0) || (r >= 8) {
				continue
			}
			set = set.Add(b.Points[IndexFromFileAndRank(uint8(f), uint8(r))])
		}
	}
	return set
}
```

In my take on chess part of calculating the available moves a player chooses from for a turn is making a list of squares (points) that surround pieces. A set with a maximum of eight squares is returned by ```SurroundingPoints``` with squares that leave the board not included, and the relative addresses of the surroundings (```-1, -1```, ```0, -1```, and so on) are converted to real board addresses as an index from 0 to 63 (chess is played on an 8x8 board).

*excerpt from board_moves.go*

```
		for _, point := range b.SurroundingPoints(the) {
			if point.Piece != nil {
				if (point.Orientation == the.Orientation) && point.Rallies {
					rallied = true
					break
				}
			}
		}
```

This is one use of ```SurroundingPoints```, where if a piece has a friendly piece adjacent to it that rallies then this piece is rallied. The slice use and ```range``` keyword removes the concern of array length (between three and eight) of the array that's made by ```SurroundingPoints``` which to me makes more readable source code and hypothetically faster time for me to finish the program. Deallocation from ```make``` isn't an immediate concern in Go because of the garbage collection memory management.

## Using grow array

My C program has a visual component where a scene is made of vertices that are drawn by the computer's graphics circuit. A list of vertices is transferred to graphics memory and then when the program requests the display be updated each three vertices in the list is drawn as a triangle with a color. I put together the list by adding vertices to a ```VertexTriangleGrowArray```.

*excerpt of vertex.h*

```
#include "geometry.h"
#include "color.h"
#include "grow_array.h"

typedef struct {
    Point point;
    Color color;
} Vertex;

typedef Vertex VertexTriangle[3];

typedef GrowArray VertexTriangleGrowArray;

#define VT_IN_RECTANGLE_COUNT 2

// followed by GrowArray wrapper functions for the VTGA
...
```

*excerpts from vertex.c*

```
VertexTriangleGrowArray VTGA_AppendRectangle(VertexTriangleGrowArray a, Rectangle r, Color c) {
    VertexTriangle t[VT_IN_RECTANGLE_COUNT];
    VTA_ForRectangle(r, c, t);
    return VTGA_AppendArray(a, t, VT_IN_RECTANGLE_COUNT);
}
```

```
void VTA_ForRectangle(Rectangle a, Color c, VertexTriangle* out_array) {
    out_array[0][0] = (Vertex){PointAssign(a[RECTANGLE_TOP_LEFT]), c};
    out_array[0][1] = (Vertex){PointAssign(a[RECTANGLE_TOP_RIGHT]), c};
    out_array[0][2] = (Vertex){PointAssign(a[RECTANGLE_BOTTOM_LEFT]), c};

    out_array[1][0] = (Vertex){PointAssign(a[RECTANGLE_BOTTOM_RIGHT]), c};
    out_array[1][1] = (Vertex){PointAssign(a[RECTANGLE_TOP_RIGHT]), c};
    out_array[1][2] = (Vertex){PointAssign(a[RECTANGLE_BOTTOM_LEFT]), c};
}
```

*excerpt from point.h*

```
...
// Constant sized arrays aren't allowed 
// to be directly assigned in C.
#define PointAssign(p) {p[0], p[1], p[2]}
...
```

Defining a cube is done by calling ```VTGA_AppendRectangle``` six times to append to the triangle list that will be moved to the graphics memory. ```VTA_ForRectangle``` converts the rectangle parameters into a regular array of triangles made of three vertices each, with the output array already allocated before the function is called.

The scene is constructed without knowing the bounds of how many vertices will be needed, so ```GrowArray``` will keep up with the allocation. ```VTGA_Free``` will need to be called later in the program when it's done with the array.

## Summary

For this essay I first described pointers because programming with slices and arrays can need a good understanding of contemporary computer memory.

Performance considerations were skipped. Computer memory can have caches and operating system details (like how exactly ```make``` and ```malloc``` allocate memory) that if considered can improve your program's performance by orders of magnitude, but a theory is that source code performance should only be part of your design when you have clear profiling or other data to reference that shows what should be improved. It's possible that a use of ```GrowArray``` might need tuning (like allowing a guess for the initial allocation size like Go does with ```make```) or have to be replaced with a more performance-minded concept that minimizes memory reallocation. ```unordered.Set``` has some performance improvements already but by using ```interface{}``` the array is of pointers instead of the actual variable memories which might be too many memory interactions.

Despite package ```unordered``` and ```GrowArray``` having different use cases they both use the transparently growing array concept that slices in Go bring with ```append```. The ```interface{}``` and ```void*``` types can help reduce source code repeating that can come with an implementation needed for varying types.

The Go type system seems to be more helpful for memory arithmetic by skipping the manual calculations that have to be done in C for ```malloc``` by multiplying the number of array indices by the size of the type.

Knowledge will improve your computer applications and the efforts toward them; memory is one part of knowledge that is a big win to know.

*This essay is part of a series at [www.github.com/pciet/goc](../README.md)*