# Pipe Toys

* fatcat: buffered cat to multiple outputs
* tailpipe: binary tail, follow sequence of numbered files
* spliff: split input into numbered fragment files

## fatcat

```
  Usage: fatcat [options] <file>...

  Options:
    -i, --input  <file>  Input file
    -b, --buffer <size>  Buffer size
    -h, --help           See this text
```

Fat Cat (fatcat) is a combination of cat, buffer and tee. It accepts
input from a single source, buffers it in memory and distributes it to
one or more destinations.

At its simplest it behaves like cat:

```
  $ ls | fatcat | sort  # is just like...
  $ ls | cat | sort     # ...this
```

Unlike cat (and like tee) any named parameters are files to output to:

```
  $ cat somefile | fatcat other1 other2
```

behaves like

```
  $ cat somefile | tee other1 other2 > /dev/null
```

Note that when outputting to a file or files output to stdout is
disabled.

An input can be named like this:

```
  $ fatcat -i somefile other1 other2
```

Sometimes it's useful to have a version of cat that can buffer more data
to even out disparities between the rates at which data is being
produced and consumed. By default a 1MB buffer is allocated. This can be
overridden using the -b option:

```
  $ fatcat -b 100MB
```

# spliff

```
  Usage: spliff [options] <file>...

  Options:
    -s, --size <size>    Chunk size
    -i, --input  <file>  Input file
    -b, --buffer <size>  Buffer size
    -v, --verbose        Verbose output
    -h, --help           See this text
```

Split the input (which may be a named file or standard input) into fixed
size, sequentially named chunks.

By default each chunk will contain 1MiB. The chunk size can be set using
the -s option.

If multiple files are named spliff will hard link together all the files
for each particular chunk to produce sequences of chunks that can be
consumed by independent tailpipe instances:

```
  $ some-command | spliff s1/000000000 s2/000000000 &
  $ tailpipe -D -i s1/000000000 | some-other-command &
  $ tailpipe -D -i s2/000000000 | yet-another-command &
```

# tailpipe

```
  Usage: tailpipe [options] <file>...

  Options:
    -i, --increment           Follow numbered files
    -t, --timeout <seconds>   How long to wait for file growth
        --wait [=<seconds>]   Wait for first file
    -D, --delete              Delete each file after reading it
        --fd=<fd>             Output to fd instead of stdout
    -h, --help                See this text
    -v, --verbose             Verbose output
```

tailpipe does for binary files what tail -f does for text files. Actually
tail -f works with binaries too - but tailpipe is dedicated to the task
and provides a few features that tail does not.

Invoked with a filename like this

```
  $ tailpipe somefile
```

tailpipe will output the entire current contents of somefile and then
wait for somefile to be appended to. Each time more data is written to
somefile tailpipe will output it.

You can give tailpipe multiple files

```
  $ tailpipe somefile anotherfile
```

As before all of somefile will be read and output. What happens then
depends on whether anotherfile exists. If it doesn't exist tailpipe will
continue to watch somefile and output any additional data. As soon as
anotherfile is created (or if it already exists) tailpipe will switch to
anotherfile.

To make tailpipe give up waiting altogether after a certain period of
time use the timeout option:

```
  $ tailpipe -t 10 somefile
```

As before tailpipe will output the contents of somefile. It will then
wait for up to 10 seconds for somefile to be extended. Each time
somefile grows the timer resets. The net effect is that tailpipe will
give up waiting when somefile hasn't grown for 10 seconds.

Normally tailpipe will complain if the first named file doesn't initially
exist. You can ask tailpipe to wait for the first file to be created
using the wait option:

```
  $ tailpipe --wait=10 somefile
```

That will wait for up to 10 seconds for somefile to be created. If you
use --wait on its own without specifying a timeout tailpipe will
wait for ever.

Often it's useful to be able to work with sequentially named files:

```
  $ ls
  dat00000.bin dat00001.bin dat00002.bin
```

The -i option tells tailpipe to attempt to increment a numeric filename
to generate a sequence of such names:

```
  $ tailpipe -i dat00000.bin
```

Given the above three files (dat000000.bin, dat00001.bin & dat00002.bin)
tailpipe will read each of them in turn and then wait for dat00003.bin to
be created.

It is possible to name multiple files with the -i option:

```
  $ tailpipe -i dat00000.bin extra00000.bin
```

In that case tailpipe will read as many dat#####.bin files as it can find
but switch to reading the extra#####.bin sequence as soon as
extra00000.bin is created.

Numbering can start at any decimal value. Numeric fields must be fixed
width; they're incremented odometer-style. The rightmost contiguous set
of digits are the only part of the name tailpipe will incrememnt and when
it runs out of digits the sequence of files is considered complete and
tailpipe will move on to the next command line argument - if any:

```
  $ tailpipe -i file00.txt
```

would read file00.text up to file99.txt but no further.

One interesting use for tailpipe, in conjunction with spliff, is to
implement a pipe that is backed by the filesystem. This can be useful
when you need a pipe with effectively unlimited buffer size. To do this
have spliff split a stream into a directory:

```
  $ some-process | spliff -s 10K /tmp/work/00000000
```

And in another terminal have tailpipe read destructively from the same
directory:

```
  $ tailpipe -D -i /tmp/work/00000000 | some-other-process
```

Given enough disk space the consuming process can be stopped and started
and can run more slowly than the producing process without blocking it.
The consumer doesn't even have to run at the same time: the same pattern
works when the consumer isn't started until after the producer has
terminated.

Andy Armstrong, andy@hexten.net

