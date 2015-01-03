# Install and Configure Go

This short note is about how to install Go compiler and configure Emacs to be an IDE for Go.

## Install Prerequisits

On Debian/Ubuntu Linux:

    sudo apt-get install gcc mercurial emacs

where mercurial will be used to retreive Go source code, gcc will be used to build Go from source code, and emacs will be configured as the IDE of Go language.

On Mac OS X, we need to install Homebrew and Xcode.  This would give you gcc and mercurial.  Then you need to install emacs:

    brew install emacs --HEAD --use-git-head --cocoa --srgb

## Build from Source Code

    cd ~/
    export PATH=~/go/bin:$PATH
    hg clone -u release https://code.google.com/p/go
    cd go/src/
    ./all.bash

You might want to put the following line into your `~/.profile` file:

    export PATH=~/go/bin:$PATH

## Build Other Tools

    mkdir -p /tmp/gotools
    export GOPATH=/tmp/gotools
    go get code.google.com/p/go.tools/cmd/...
    go get github.com/nsf/gocode
    go get code.google.com/p/rog-go/exp/cmd/godef
    mv /tmp/gotools/bin/* ~/go/bin/

## Configure Emacs

To download and compile Emacs plugins for Go:

    mkdir -p ~/.emacs.d/lisp
    cd ~/.emacs.d/lisp
    wget https://raw.githubusercontent.com/dominikh/go-mode.el/master/go-mode.el
    emacs -batch -f batch-byte-compile go-mode.el
    wget https://raw.githubusercontent.com/sah/emacs/master/go-mode-load.el

Update `go-mode-load.el` according to your `go-mode.el` file:

    emacs ~/.emacs.d/lisp/go-mode-load.el

Move the cursor to the end of the S-expression denoted as "update this file" and press `C-x C-e`.  The `go-mode-load.el` file's content will change.  Then you can type `C-x C-s` to save the changed content.

Then edit your `~/.emacs` file and add the following content:

    ((show-paren-mode 1)
    (add-to-list 'load-path "~/.emacs.d/lisp" t)
    (require 'go-mode-load)
    (add-hook 'before-save-hook 'gofmt-before-save)
    (add-hook 'go-mode-hook
          (lambda ()
            (setq-default)
            (setq tab-width 4)
            (setq standard-indent 4)
            (setq indent-tabs-mode nil)
            (local-set-key (kbd "C-c C-r") 'go-remove-unused-imports)
            (local-set-key (kbd "C-c i") 'go-goto-imports)))

If you want more sophisticated configurations like auto-completion and on-the-fly syntax checking, please refer to [this post](http://dominik.honnef.co/posts/2013/03/writing_go_in_emacs/) for more details.
