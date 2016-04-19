---
layout: post
title: File Operations in C++
tags:
- cpp
comments: true
description: "some common file operations in c++"
keywords: "file, c++"
---

* toc
{:toc}
---

The following classes are needed for reading from or writing to files:

- `ofstream`: stream class to write on files
- `ifstream`: stream class to read from files
- `fstream`: stream class to both read from and write to files

## Open and Close

```c++
int main(){
    std::ofstream myfile;
    myfile.open("example.txt"); // check if the file has been opened successfully
    if(myfile.is_open()){
        // do something
    }

    myfile.close(); // close() also flushes the associated buffers. 
            // In case that an object is destroyed while still associated with an open file,
            // the destructor automatically calls it.
    return 0;
}
```

Signature of `ofstream::open()`:

```c++
void ofstream::open(const char* filename, ios_base::openmode mode = ios_base::out);
void ofstream::open(const string& filename, ios_base::openmode mode = ios_base::out); // c++11
```

`ifstream::open` and `fstream::open` are similar to this one except the default values to `mode`, which is listed below.

|---|---|
|mode| description|
|---|---|
|`ios::in`|Open for input operations.|
|`ios::out`|Open for output operations.|
|`ios::binary`|Open in binary mode.|
|`ios::ate`|Set the initial position at the end of the file. <br>If this flag is not set. The initial position is the beginning of the file.|
|`ios::app`|All output operations are performed at the end of file, appending the content to the current content of the file.|
|`ios::trunc`|If the file is opened for output operations and it already existed, its previous content is deleted and replaced by the new one.|

The default mode parameter for each stream:

|---|---|
|class|default mode parameter|
|---|---|
|ofstream|ios::out|
|ifstream|ios::in|
|fstream|ios::in\|ios::out|

## Read and Write

{% highlight cpp %}
if(myfile.is_open()){
    myfile << "Hello, world" << endl; // write to myfile
}
else{
    cerr << "unable to open myfile" << endl;
}

// read from myfile
string line;
if(myfile.is_open()){
    while(getline(myfile, line)){
        cout << line << endl;
    }
}
else{
    cerr << "unable to open myfile" << endl;
}
{% endhighlight %}

## Position a Stream

All I/O stream objects keep internally at least one internal position:

|----|-----|
|`ifstream`| keeps *get position* with the location of the element to be read in the next input operation|
|`ofstream`| keeps *put position* with the location where the next element has to be written|
|`fstream`| keeps both|

<br><br>
Get and set postion:

{% highlight cpp%}
// infile.txt: "hello, world"
// outfile.txt: empty file
ifstream infile("infile.txt");
ofstream outfile("outfile.txt");

// tellg(),tellp() to get position
// seekg(), seekp() to set position
streampos get_pos = infile.tellg();
streampos put_pos = outfile.tellp();

cout << get_pos << endl; // output: 0
cout << put_pos << endl; // output: 0

string line;
getline(infile, line);
outfile << line << endl; // outfile.txt: "hello, world"

get_pos = infile.tellg();
put_pos = outfile.tellp();
cout << get_pos << endl; // output: 13
cout << put_pos << endl; // output: 13

infile.seekg(ios::beg);
outfile.seekp(ios::beg);

getline(infile, line); // line: "hello, world"
outfile << "override the old content" << endl; // outfile.txt: "override the old content"
// note that the new content will override the old one
// from the current position, but the part of the old content that 
// exceeds the length of the new content is kept

infile.seekg(7, ios::beg); // offset from the beginning
outfile.seekp(-8, ios::end); // offset from the end

getline(infile, line); // line: "world"
outfile << "world" ; // outfile.txt: "override the old worldnt"
{% endhighlight %}

## Binary Files

For binary files, reading and writing data with the extraction and insertion operators (<< and >>) and functions like `getline` is not efficient, sinve we do not need to format any data and data is likely not formatted in lines. Instead, we should use `write` and `read`.

{% highlight cpp %}
streampos size;
char * memblock;

fstream file("example.bin", ios::in|ios::binary|ios::ate);
if(file.is_open()){
    size = file.tellg();
    memblock = new char[size];
    file.seekg(0, ios::beg);
    file.read(memblock, size);
    file.close();

    cout << "the entire file content is in memory" << endl;
    delete[] memblock;
}
else{
    cerr << "unable to open file" << endl;
}

{% endhighlight %}

## Buffers and Sybchronization

When we operate with file streams, these are associated to an internal buffer object of type `streambuf`. This buffer object may represent a memory block that acts as an intermediary between the stream ans the physical file. The operating system may also define other layers of buffering for reading and writing to files.

When the buffer is flushed, all the data contained in it is written to the physical medium (if it is an output stream). This process is called *synchronization* and takes place under any of the following circumstances:

- **When the file is closed**: before closing a file, all buffers that have not yet been flushed are synchronized and all pending data is written or read to the physical medium.
- **When the buffer is full**: buffers have a certain size. When the buffer is full it is automatically synchronized.
- **Explicitly, with manipulators**: When certain manipulators are used on streams, an explicit synchronization takes place. These manipulators are: `flush` and `endl`.
- **Explicitly, with member function sync()**: Calling the stream's member function `sunc()` causes an immediate synchronization. This function returns an `int` value equal to -1 if the stream has no associated buffer or in case of failure. Otherwise (if the stream buffer was successfully synchronized) it returns 0.


