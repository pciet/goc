# Go and C

*[www.github.com/pciet/goc](https://github.com/pciet/goc)*

This repository is a collection of essays on computer programming topics illustrated by source code excerpts in Go and C. The essays are intended to empower individuals or small teams by collecting and conveying practices for making practical and high quality programs used to apply computers constructively.

The information in these essays is not intended for life-critical computer applications like machine control systems or medical recordkeeping.

```
package main

import (
    "fmt"
    "os"
)

func main() {
    for i, a := range os.Args {
        fmt.Println(i, a)
    }
}
```

```
#include <stdio.h>
int main(int argc, const char** argv) {
    for (int i = 0; i < argc; i++) {
        printf("%d %s\n", i, argv[i]);
    }
}
```

If you have feedback please share on the issue tracker at [github.com/pciet/goc/issues](https://github.com/pciet/goc/issues) and I'll try to respond, thanks! The interpretations of others is an effective way to improve. Private comments have already made these essays better and I can commit changes described by our discussion.

## Essays

Each essay is the ```README.md``` file in the subdirectory listed in parenthesis:

**Function that causes a return** ([funcret](funcret/README.md))

**Grow Array** ([growarray](growarray/README.md))

Other essays are in draft form while I add features to my latest program that has both Go and C components, stay tuned. The next planned topic is about efficiently encoding network messages.

## My World

macOS is how I've been making programs, by editing my source code with ```mvim``` ([macvim-dev.github.io/macvim](https://macvim-dev.github.io/macvim)). I use the LLVM ([www.llvm.org](https://llvm.org/)) compiler included with Xcode for C, by the ```gcc``` commands in Bash scripts. I install Go ([www.golang.org](https://golang.org/)) for the ```go build``` command and other tools like ```gofmt``` which can be integrated into the editor with vim-go ([www.github.com/fatih/vim-go](https://github.com/fatih/vim-go)).

The Homebrew Internet service ([brew.sh](https://brew.sh/)) can be delightfully convenient for installing other libraries and programs, but be aware that you are responsible for calling ```brew upgrade``` and ```brew cask upgrade``` periodically.

Lately I've taken a suggestion from Go online video documentation and started editing source code in black and white without syntax highlighting like italic comments or green keywords. Theoretically, despite the visual appeal, syntax highlighting distracts from content choices that make the program readable.

I do like having line numbers so these are in my ```~/.vimrc```:

```
syntax on
colorscheme black
set number
```

The file for ```colorscheme black``` is included in this repository as [black.vim](black.vim) and is placed at ```~/.vim/colors/black.vim``` for use; it makes the line numbers different than the code text and is a place to make syntax highlighting customization if wanted.

The Terminal on my computer is customized in ```~/.bash_profile```:

```
# listing files in a directory
alias l='/bin/ls -GpF'
alias ls='/bin/ls -GpFhla'

# inspect C system library headers
# h stdlib.h
alias h='open -h'

# man pages are usually more readable than the headers
# the following adapted from www.stackoverflow.com answers
# remove line wrapping and open the page in Chrome
export MANWIDTH=5000
function man {
    args="$@"
    /usr/bin/man $args > /dev/null
    if [ $? -ne 0 ]; then
        return 1
    fi
    /usr/bin/man $args | col -b > "/tmp/man $args"
    open -a /Applications/Google\ Chrome.app "/tmp/man $args"
}
```

Additional environment variables are set for Go as described in that documentation, and I set more variables for other libraries.

These texts were typed into MacDown from [macdown.uranusjr.com](https://macdown.uranusjr.com/), installed via Homebrew's ```brew cask install macdown```.

---

### My other works that describe computer programming

**Web Application Details: Wisconsin Chess** ([youtu.be/yZn2jU8eCUo](https://www.youtube.com/watch?v=yZn2jU8eCUo))

**Go For Art** ([www.github.com/pciet/goforart](https://github.com/pciet/goforart))

---

![Arboretum](Arboretum.jpg)
*Photograph at the UCSC Arboretum in Santa Cruz, California, spring 2013.*

Thanks for reading, and have a nice day!

by Matthew Juran

[www.github.com/pciet](https://github.com/pciet)

[www.youtube.com/matthewjuran](https://youtube.com/matthewjuran)