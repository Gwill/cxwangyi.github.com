# Build GCCGO 4.9 on Mac OS X

GCC 4.9 includes gccgo 4.9 that supports Go 1.2.  We can install the full GCC 4.9 package using Homebrew.

First of all, you need to `tap` the GCC 4.9 installation rule:

    brew tap homebrew/versions
    brew install gcc49

It is noticable that Homebrew only installs languages supports of C, C++, Objective-C, and Objective-C++ by default.  To make Homebrew install gccgo as well, we can edit the installation instruction file:

    emacs `$(brew --repository)/Library/Formula/gcc49.rb`

In particular, we add `gccgo` to all lines like:

    languages = %w[c c++ objc obj-c++]

So we have

    languages = %w[c c++ go objc obj-c++]

Then we can run `brew install gcc49`, which would take a very long time.
