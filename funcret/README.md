# Function that causes a return

*by Matthew Juran*

![Neary Lagoon](Path.jpg)
*Photograph at Neary Lagoon in Santa Cruz, California, summer 2013.*

This essay is part of a series about programming contemporary electronic computers with the Go or C languages. It shows how I handle errors uncovered by the program that need human intervention. My perspective is from solo computer program development of networked computer games and social worlds at an early stage, and I hope what I've learned can apply to the variety of computer applications you might be making.

---

Often the computer information paths lit by your program need a thing. If that thing isn't right then we call the problem an error. Some errors show that the source code of the program has a mistake, or the computer itself isn't configured right yet for the program. Programmers might isolate a mistake that caused an error by changing the program to show diagnostic text with ```fmt.Println``` or ```printf```, but having these prints as part of the regular program can save time that's needed for new features and polish instead.

Concatenating error strings into something like a simplified stack trace has been my favorite error indication approach in C. When the program encounters an error what I then see is a precise take of “while trying this a specific step failed because of the following reason”:

```
init-failed: loadConfig-failed: ReadFileLen-failed: fopen (can't open)-config.xml
```

A function called in ```main``` that indicates my errors is followed by behavior tailored to how the program is used. During program development any error is printed:

*main.c*

```
#include "error.h"
#include "wr.h"

int main(int argc, const char** argv) {
    Error err = Run("Distant Wrapping Planes");
    if (err != NO_ERROR) {
        PrintError(err);
        return 1;
    }
    return 0;
}
```

Theoretically an error trace is better than just stating the error reason (like ```can't open file```) because less investigation into the program is necessary to understand the problem.

This source code is my latest C implementation of the idea:

*error.h*

```
#ifndef ERROR_H_SEEN
#define ERROR_H_SEEN

#include <stddef.h>

// An error is a null terminated C string.
typedef const char* Error;

#define NO_ERROR NULL

// Input strings must all be null terminated.

// NewError makes copies of the input strings. You free the returned allocation.
// The error is formatted as "symbol-message".
Error NewError(const char* symbol, const char* message);

// PrependError will put the new error message in front of a.
// PrependError frees a but allocates the return error, which you free.
// The error is formatted as "symbol-message:symbol_a-message_a".
Error PrependError(Error a, const char* symbol, const char* message);

// Writes to stderr.
void PrintError(Error a);

#define NewErrorReturn(condition, symbol, message) if ((condition) != 0) return NewError(symbol, message)
#define ErrorReturn(err, symbol, message) if (err != NO_ERROR) return PrependError(err, symbol, message)

extern const char* FAILED_STRING;

#endif
```

*error.c*

```
#include "error.h"
#include "panic.h"

#include <stdio.h>
#include <stdlib.h>
#include <strings.h>

const char* FAILED_STRING = "failed";

Error NewError(const char* symbol, const char* message) {
    if (symbol == NULL) Panic("null symbol");
    if (message == NULL) Panic("null message");

    size_t ns = strlen(symbol) + strlen(message);
    if (ns == 0) Panic("zero length");

    // 1 for null termination, 1 for the added formatting character
    char* m = malloc(sizeof(char)*(ns+1+1));
    if (m == NULL) Panic("malloc failed");

    int c = snprintf(m, ns+1+1, "%s-%s", symbol, message);
    if (c != (ns+1)) Panic("snprintf wrote unexpected char count");

    return (Error)m;
}

Error PrependError(Error a, const char* symbol, const char* message) {
    if (a == NULL) Panic("null a");
    if (symbol == NULL) Panic("null symbol");
    if (message == NULL) Panic("null message");

    size_t ns = strlen(a) + strlen(symbol) + strlen(message);
    if (ns == 0) Panic("zero length");

    // 1 for null termination, 3 for the added formatting characters
    char* m = malloc(sizeof(char)*(ns+1+3));
    if (m == NULL) Panic("malloc failed");

    int c = snprintf(m, ns+1+3, "%s-%s: %s", symbol, message, a);
    if (c != (ns+3)) Panic("snprintf wrote unexpected char count");

    free((void*)a);

    return (Error)m;
}

void PrintError(Error e) {
    fprintf(stderr, "%s\n", (const char*)e);
}
```

If the error should never happen but then does I’ll panic instead. Unlike Go which has a ```recover``` mechanism for ```panic``` like you might see with exception error handling, my C version only exits.

*panic.h*

```
#ifndef PANIC_H_SEEN
#define PANIC_H_SEEN

// Prints the message and stack trace to stderr then exits immediately with code 1.
// Use -g with gcc to generate symbols for the trace.
void Panic(const char* message);

#endif
```

In ```Panic``` the exact function names and containing program of the stack calls are printed:

```
null socket
0   wr               Panic
1   wr               Connect
2   wr               Run
3   wr               main
4   libdyld.dylib    start
5   ???              0x0
```

The actual output of the ```execinfo.h``` functions used to implement ```Panic``` can be improved with source code line numbers which is useful when a function calls another in multiple places. Line numbers are part of the Go ```panic``` output by default.

Testing parts of programs by writing short programs that use the part can save time. Before moving onto showing how I use the above source code, this is the program I wrote as a quick test of the ```Error``` functions and macros:

*tests/error.c*

```
#include "../error.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

Error returnsError1() {
    NewErrorReturn(false, "bad", "return");
    NewErrorReturn(true, "good", "return");
}

Error returnsError2() {
    Error err = NewError("returnsError2", FAILED_STRING);
    ErrorReturn(NO_ERROR, "returnsError2", "bad");
    ErrorReturn(err, "returnsError2", "good");
    return NO_ERROR;
}

int main(int argc, const char* argv[]) {
    Error t = NewError("main", "testing for errors");
    if (strcmp("main-testing for errors", (const char*)t) != 0) {
        printf("NewError returned unexpected string: %s\n", t);
        return 1;
    }

    Error t2 = PrependError(t, "main", "testing combining errors");
    if (strcmp("main-testing combining errors: main-testing for errors", (const char*)t2) != 0) {
        printf("PrependError returned unexpected string: \"%s\"\n", t2);
        return 1;
    }

    PrintError(t2);
    free((void*)t2);

    Error r = NewError("good", "return");
    Error ret = returnsError1();
    if (strcmp((const char*)r, (const char*)ret) != 0) {
        printf("returnsError1 returned unexpected string: %s\n", ret);
        return 1;
    }

    PrintError(r);
    PrintError(ret);

    free((void*)r);
    free((void*)ret);

    r = NewError("returnsError2", FAILED_STRING);
    r = PrependError(r, "returnsError2", "good");
    ret = returnsError2();
    if (strcmp((const char*)r, (const char*)ret) != 0) {
        printf("returnsError2 returned unexpected string: %s\n", ret);
        return 1;
    }

    PrintError(r);
    PrintError(ret);

    free((void*)r);
    free((void*)ret);

    return 0;
}
```

This test at least gives a small amount of confidence that will be verified by implementing the rest of the program.

So the idea is that with this above ```Error``` and ```Panic``` stuff in my source code toolbox I'm able to more quickly improve or change my programs.

---

A theory is less time is taken to add or understand error checks than it takes to isolate mistakes that cause ambiguous symptoms when errors aren't checked for. Go idiomatic error handling is this idea, done by having error checks after every function call that can signal error conditions:

*excerpt of listen.go*

```
func ListenAndServe(port int) error {
	c, err := net.Listen("tcp", fmt.Sprintf(":%d", port))
	if err != nil {
		return err
	}
	defer c.Close()
	for {
		conn, err := c.Accept()
		if err != nil {
			return err
		}
		go ListenToConnection(conn)
	}
}
```

*main.go*

```
package main

import "fmt"

const (
	Debug bool = true
	port  int  = 5000
)

func main() {
	err := ListenAndServe(port)
	if err != nil {
		if Debug {
			fmt.Println(err.Error())
		}
	}
}
```

If you've taken a shortcut by not checking errors and your program isn't working right then adding error checks is a good step toward finding the problem.

So far with Go I've used the simpler error reason approach, but writing this essay has convinced me to spend the time to add the error trace like I've done in C.

## When to start

There's a balance to find between error checking and getting the job done. For where I'm at with C it would be nice to add ```snprintf``` formatting to the error string, or even better like how it is in Go:

```
func ConcatStringsAndInt(a, b string, i int) string {
    return fmt.Sprint(a, "-", b, " ", i)
}
```

but I have a long list of other tasks to do that hypothetically might not be significantly helped by variadic error messages. Just my C ```Error``` functions and macros above save significant time for many kinds of source code mistakes.

If later an exact error needs to be checked for, like if attempting an automatic retry of a network connection, then my idea is the ```Error``` type can be changed to a ```struct``` that signals specific errors:

```
typedef struct {
    const char* Trace;
    int Code;
} Error;
```

```
...
    Error err = Connect(addr, port);
    while (err.Code == CONNECT_REFUSED_ERROR) {
        err = RetryConnect(addr, port);
    }
    ErrorReturn(err, "Connect", FAILED_STRING);
...
```

This change in Go is similar because an ```error``` is any type that has the ```Error() string``` method:

```
type Err struct {
    Trace string
    Code  int
}

func (an Err) Error() string {
    return fmt.Sprintf("code %#x: %v", an.Code, an.Trace)
}
```

## Func that is a return

For C the preprocessor adjusts source code at the start of compilation to a more precise but less readable state (for example it removes comments). My C program's source code became condensed because of using preprocessor macros for the "if error" checks contained in ```ErrorReturn``` and ```NewErrorReturn```:

*excerpt from wr.c*

```
...
    Error err = Connect("127.0.0.1", 5000);
    ErrorReturn(err, "Connect", FAILED_STRING);
...
    int rc = SDL_SetRelativeMouseMode(true);
    NewErrorReturn(rc != 0, "SDL_SetRelativeMouseMode", SDL_GetError());

    while (loop(&err));
    ErrorReturn(err, "loop", FAILED_STRING);

    err = Disconnect();
    ErrorReturn(err, "Disconnect", FAILED_STRING);
...
```

*connect.c*

```
#include "connect.h"

#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/errno.h>
#include <string.h>
#include <unistd.h>

static int soc;

// hostString is the IP address in string form.
Error Connect(const char* hostString, int port) {
    soc = socket(PF_INET, SOCK_STREAM, 0);
    NewErrorReturn(soc == -1, "socket", strerror(errno));

    struct sockaddr_in t = {
        .sin_family = AF_INET,
        .sin_port = htons(port)
    };

    int err = inet_pton(AF_INET, hostString, &(t.sin_addr));
    NewErrorReturn(err == 0, "inet_pton", "unparseable host address string");
    NewErrorReturn(err == -1, "inet_pton", strerror(errno));

    err = connect(soc, (struct sockaddr*)&t, sizeof(t));
    NewErrorReturn(err != 0, "connect", strerror(errno));

    return NO_ERROR;
}

Error SendMessage(const void* m, size_t len) {
    int c = send(soc, m, len, 0);
    NewErrorReturn(c == -1, "send", strerror(errno));
    NewErrorReturn(c != len, "send", "failed to send entire message");

    return NO_ERROR;
}

Error Disconnect() {
    int err = shutdown(soc, SHUT_RDWR);
    NewErrorReturn(err == -1, "shutdown", strerror(errno));

    err = close(soc);
    NewErrorReturn(err == -1, "close", strerror(errno));

    return NO_ERROR;
}
```

Besides this condensation, the ```#if```-```#else```-```#endif``` blocks of the preprocessor will be useful to make a Windows implementation of these network message functions later. In Go such varying implementation is done in separate files which are picked between by the compiler depending on build tag comments in the source code.

I like that C can have what looks like a function cause a return of the calling function which is impossible without the preprocessor adding the ```return``` keyword. I've tried to hint at this nontypical behavior by including "Return" in the macro names, but the world of nontypical behaviors enabled by macros might be why they're not in Go.

## Summary

In going back to C after a few years of Go I found the preprocessor macro concept to be surprisingly useful for making this error logic readable and condensed, a concept that isn't matched by Go, but C string manipulation doesn't seem to usually be as beautiful and immediately useful as the Go ```string``` type and associated standard library functions. There could be pride in wrangling C string manipulation to the same level as baseline Go but it takes time that I hypothesize should be spent elsewhere for my program.

To summarize my approach to errors that need my intervention: have error responses inline with the function sequence by using the ```if``` keyword, if a function has an error signal then check for it, and use an error trace string instead of just the error reason to minimize time to mistake isolation. Add improvements as needed for the program.

From my C source code in ```error.h``` and ```error.c``` the ```__FILE__``` and ```__LINE__``` predefined preprocessor macros might be a first addition to make, along with variadic formatted error strings.

*This essay is part of a series at [www.github.com/pciet/goc](../README.md)*