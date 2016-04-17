---
layout: post
title: Common Vim Commands
tags:
- osx
- vim
comments: true
description: "common vim commands on OS X"
keywords: "Vim, commands"
---

* toc
{:toc}
---


{{site.url}}
This blog lists some vim commands that are used frequently. However, some quite basic commands such as *hjkl* are not included. These commands work on Mac OS X. In the end of this post I put some sites about learning vim.

Type `:help cmd` in Vim to get more details of the command `cmd`.

##Text Edit

###Copy, Paste and Select

{% highlight vim  %}
p       " paste from buffer
:r !pbpaste " paste from clipboard. Note: it works on OS X, not sure whether it works on other systems.

yw      " copy from the current position to the next word.
nyw     " copy n words from current position, including the following white spaces. n can be any positive integer.
ye      " copy from the current position to the end of the  word.
nye     " copy n words from current position, NOT including the following white spaces. n can be any positive integer.
yiw     " copy the whole word under cursor. It does not require the cursor at the beginning of the word.

yy      " copy current line
y^      " copy from the current  position to the beginning of the line
y$      " copy from the current position to the end of the line.
y{      " copy from the current position to the beginning of the paragraph.
y}      " copy from the current position to the end of the paragraph.

:m,n y  " copy m-th line to n-th line
:'a,'b y    " copy from mark a to mark b

:m,n !pbcopy    " copy m-th line to n-th line to clipboard. The copied lines are deleted from the source text
:m,nw !pbcopy   " copy m-th line to n-th line to clipboard. The copied lines are kept in the source text
{% endhighlight %}

Replace 'y' with the following characters to change behaviors.

|--|--|
|character | behavior |
|---|----|
|d| delete|
|c| change|

###Replace and Substitute

{% highlight vim %}
r       " replace the current character
R       " turn to replace mode

s       " substitute the current character and turn into insert mode
S       " substitute current line
{% endhighlight %}

Advanced usages can be found in <a href="#searchandsubstitute">Search and Substitute</a>.

###Undo and Redo

{% highlight vim %}
u       " undo last change
U       " return the last line which was modified to its original state

Ctrl-r  " redo 
{% endhighlight %}

*Change* is defined as all the input between two vim commands. *U* is not actually a true "undo" command as it does not actually navigate undo history like *u* and *CTRL-r*. This means that (somewhat confusingly) *U* is itself undo-able with *u*; it creates a new change to reverse previous changes.

###Indent Code Block

{% highlight vim %}
=       " indent the current line
=%      " indent the current braces {...}

G=gg    " auto indent entire document, which is equal to gg=G
n==     " indent n lines from the current line
=nj     " indent the current line and the following n lines
=nk     " indent the current line and the previous n lines
{% endhighlight %}

*=* can also be combined with many jump commands listed below.

##Locate and Jump

{% highlight vim %}

w       " go to the beginning of the next word
e       " go to the end of the current word

nG      " go to n-th line
gg      " go to the beginning of the file
G       " go to the end of the file

L       " go to the bottom of the screen
H       " go to the top of the screen

ma      " set mark 'a'. You can change 'a' to any other characters
'a      " go to mark 'a'

'.      " go to the beginning of the line where the last modification occurs
`.      " go to the exact position where the last modification occurs

%       " match brackets ()[]{}
*       " go to the next same word
#       " go to the previous same word
^       " go to the beginning of the current line, not including leading whitespaces
0       " go to the beginning of the current line
$       " go to the end of the current line
(       " go to the beginning of the current sentence
)       " go to the end of the current sentence
{       " go to the beginning of the current paragraph
}       " go to the end of the current paragraph
{% endhighlight %}

<a id="searchandsubstitute"></a>

##Search and Substitute

{% highlight vim %}
/foo        " search forward for 'foo'
/\<foo\>    " search forward for 'foo', but only match the whole word
/foo\c      " search case-insensitively
?foo        " search backward for 'foo'
/^foo.*others   " line beginning with 'foo' and followed by 'others'. Regex expressions work here.

:%s/foo/ /gn    " count occurrences of 'foo'

:%s/foo/others /igc " substitute 'foo' with others, case-insensitively(i), globally(g) and asking for confirmation(c) 
:s/foo/others /igc  " substitute 'foo' with others in the current line

{% endhighlight %}

##Global Operation 

For details, refer to <a href="http://vim.wikia.com/wiki/Power_of_g">this page</a>.

Brief explanation of `:g`:
{% highlight vim %}
:[range]g/pattern/cmd      
{% endhighlight %}

This acts on the specified *[range]* (default whole file), by executing the Ex command *cmd* for each line matching *pattern* (an Ex command is one tarting with a colon such as `:d` for delete). Before executing *cmd*, "." is set to the current line.

{% highlight vim %}
:g/one\|two/        " list lines containing 'one' or 'two'
:g/foo/d            " delete all lines containing 'foo'
:v/foo/d            " delete all lines not containing 'foo', the same as :g!/foo/d. 'v' means inverse
{% endhighlight %}

##File Operations

{% highlight vim %}
:x      " save and quit, the same as :wq
:qa     " quit all files
:saveas newfile    " save to 'newfile'
{% endhighlight %}

##Window

{% highlight vim %}
:e filename     " edit another file
:split filename " open file in a new horizontal window
:vsplit filename    " open file in a new vertical window

Ctrl-w h        " move to the file on the left. 'h' can be replaced by 'j', 'k' and 'l'
Ctrl-w =        " make all windows equal size
Ctrl-w +        " increase height
Ctrl-w -        " decrease height
Ctrl-w _        " maximize height
Ctrl-w >        " increase width
Ctrl-w <        " decrease width
Ctrl-w |        " maximize width
:res +n         " increase height by n lines
:vertical res +n    " increase width by n characters

:hide           " close current window
:only       " keep only this window open
{% endhighlight %}

##Configuration

{% highlight vim %}
:set nu     " show line numbers
:set relativenumber " set relative number
:set hlsearch   " highlight search
:set autoindent " auto indent
:set smartindent    " smart indent
:set tabstop=4  " set tabstop = 4 
:set softtabstop=4
:set shiftwidth=4   " set indent width = 4
:set expandtab  " replace tab with whitespaces when indent
:filetype on    " auto detect file type
:syntax on      
:setlocal spell spelllang=en_us " auto spell check
{% endhighlight %}
----
Some helpful sites:

- [vim tips and tricks](http://www.cs.oberlin.edu/~kuperman/help/vim/home.html)
- [best vim tips](http://vim.wikia.com/wiki/Best_Vim_Tips)

