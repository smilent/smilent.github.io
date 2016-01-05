---
layout: post
title: Common Terminal Commands
tags:
- osx 
comments: true
---

* toc goes here
{:toc}

---

This blog only lists ***SOME*** arguments of a command.
<br><br>

##Directory

###dirs
<pre> dirs [-clpv] [+N | -N] </pre>

Display the list of currently remembered directories. Directories are added to the list with the `pushd` command; the `popd` command removes directories from the list. Click [here](https://www.gnu.org/software/bash/manual/html_node/Directory-Stack-Builtins.html) for more details.
<br>

<dt markdown="1">`-c`</dt>
<dd> Clears the directory stack by deleting all of the elements.</dd>

<dt markdown="1">`-l`</dt>
<dd> Produces a listing using full pathnames; the default listing format uses a tilde to denote the home directory. </dd>

<dt markdown="1">**`-p`**</dt>
<dd markdown="1"> **Causes `dirs` to print the directory stack with one entry per line.** 

<dt markdown="1">**`-v`**</dt>
<dd markdown="1">**Causes `dirs` to print the directory stack with one entry per line, prefixing each entry with its index in the stack.**

<dt markdown="1">`popd`</dt>
<pre> popd [-n] [+N |-N] </pre>

Remove the top entry from the directory stack, and `cd` to the new top directory. When no arguments are given, `popd` removes the top directory from the stack and performs a `cd` to the new top directory. The elements are numbered from 0 starting at the first directory listed with `dirs`; that is, `popd` is equivalent to `popd +0`.

<dt markdown="1">`pushd`</dt>
<pre> pushd [-n] [+N | -N | dir] </pre>

Save the current directory on the top of the directory stack and then `cd` to `dir`. With no arguments, `pushd` exchanges the top two directories

<dt markdown="1">`pushd`</dt>
<pre> pushd [-n] [+N | -N | dir] </pre>

Save the current directory on the top of the directory stack and then `cd` to `dir`. <B>With no arguments, `pushd` exchanges the top two directories.</B>

<dt markdown="1"><code>+<i>N</i></code></dt>
<dd markdown="1">
Brings the *N*th directory(counting from the left of the list printed by `dirs`, starting with zero) to the top of the list by rotating the stack.
</dd>

---

## Search and Replace

###find 

Click [here](https://www.digitalocean.com/community/tutorials/how-to-use-find-and-locate-to-search-for-files-on-a-linux-vps) for more examples.

{% highlight bash linenos %}
# Print out a list of all the files with name "foo"
find / -name "foo" -print

# Print out a list of all the files with name "foo", 
# but the search is case insensitive
find / -iname "foo" -print

# Print out a list of all the files whose names do not end in .c.
find / \! -name "*.c" -print

# Print out a list of all the files owned by user ``wnj'' that are 
# newer than the file ttt.
find / -newer ttt -user wnj -print

# Print out a list of all the files which are not both newer than 
# ttt and owned by ``wnj''.
find / \! \( -newer ttt -user wnj \) -print

# Print out a list of all the files that are either owned by ``wnj'' 
# or that are newer than ttt.
find / \( -newer ttt -or -user wnj \) -print

# Print out a list of all the files whose inode change time is more 
# recent than the current time minus one minute.
find / -newerct '1 minute ago' -print

# Find files and directories that are at least seven levels deep in 
# the working directory /usr/src.
find /usr/src -name CVS -prune -o -depth +6 -print
{% endhighlight %}

<pre>-exec COMMAND \;</pre>

Carries out COMMAND on each file that `find` matches. The command sequence terminates with ";" (the ";" is escaped to make certain the shell passes it to `find` literally, without interpreting it as a special character).

If COMMAND contains {}, then `find` substitutes the full path name of the selected file for "{}".
{% highlight bash linenos %}
# Use the echo(1) command to print out a list of all the files.
# Do not confuse the option -exec with the exec shell builtin.
find / -type f -exec echo {} \;

# Delete all broken symbolic links in /usr/ports/packages.
find -L /usr/ports/packages -type l -exec rm -- {} +

{% endhighlight %}

###grep

The grep utility searches any given input files, selecting lines that match one or more patterns.  By default, a pattern matches an input line if the regular expression (RE) in the pattern matches the input line without its trailing newline.  An empty expression matches every line.  Each input line that matches at least one of the patterns is written to the standard output.

{% highlight bash linenos %}
# Select lines that contain "foo" in file "some_file".
grep foo some_file

# Select lines that contain "foo" in file "some_file" insensitively.
grep -i foo some_file

# Select lines that contain "foo" in file "some_file", 
# and precede each line by its relative line number.
grep -n foo some_file

# Only the names of files containing selected lines are listed.
# This argument is usually combined with -r or the find command.
grep -l foo some_dir

# Select lines that do not contain "foo" in file "some_file".
grep -v foo some_file
{% endhighlight %}

`grep` can also be combined with `find`:
{% highlight bash linenos %}
# Search for "foo" in all the .cpp files
find / "*.cpp" -exec grep foo {} \;
{% endhighlight %}

##time

###time
<pre>time [-lp] <u>utility</u></pre>
This command can be used to measure the clock time, user time and system time of process "utility". The output format depends on your shell being used.

{% highlight bash %}
$time grep -rl "test" .  > test.txt
{% endhighlight %}
Output:
{% highlight bash %}
real	0m0.011s
user	0m0.004s
sys	0m0.005s
{% endhighlight %}
