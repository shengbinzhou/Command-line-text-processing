# <a name="miscellaneous"></a>Miscellaneous

**Table of Contents**

* [cut](#cut)
    * [select specific fields](#select-specific-fields)
    * [suppressing lines without delimiter](#suppressing-lines-without-delimiter)
    * [specifying delimiters](#specifying-delimiters)
    * [complement](#complement)
    * [select specific characters](#select-specific-characters)
    * [Further reading for cut](#further-reading-for-cut)
* [tr](#tr)
    * [translation](#translation)
    * [escape sequences and character classes](#escape-sequences-and-character-classes)
    * [deletion](#deletion)
    * [squeeze](#squeeze)
    * [Further reading for tr](#further-reading-for-tr)
* [basename](#basename)
* [dirname](#dirname)

<br>

## <a name="cut"></a>cut

```bash
$ cut --version | head -n1
cut (GNU coreutils) 8.25

$ man cut
CUT(1)                           User Commands                          CUT(1)

NAME
       cut - remove sections from each line of files

SYNOPSIS
       cut OPTION... [FILE]...

DESCRIPTION
       Print selected parts of lines from each FILE to standard output.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="select-specific-fields"></a>select specific fields

* Default delimiter is **tab** character
* `-f` option allows to print specific field(s) from each input line

```bash
$ printf 'foo\tbar\t123\tbaz\n'
foo     bar     123     baz

$ # single field
$ printf 'foo\tbar\t123\tbaz\n' | cut -f2
bar

$ # multiple fields can be specified by using ,
$ printf 'foo\tbar\t123\tbaz\n' | cut -f2,4
bar     baz

$ # output is always ascending order of field numbers
$ printf 'foo\tbar\t123\tbaz\n' | cut -f3,1
foo     123

$ # range can be specified using -
$ printf 'foo\tbar\t123\tbaz\n' | cut -f1-3
foo     bar     123
$ # if ending number is omitted, select till last field
$ printf 'foo\tbar\t123\tbaz\n' | cut -f3-
123     baz
```

<br>

#### <a name="suppressing-lines-without-delimiter"></a>suppressing lines without delimiter

```bash
$ cat marks.txt
jan 2017
foobar  12      45      23
feb 2017
foobar  18      38      19

$ # by default lines without delimiter will be printed
$ cut -f2- marks.txt
jan 2017
12      45      23
feb 2017
18      38      19

$ # use -s option to suppress such lines
$ cut -s -f2- marks.txt
12      45      23
18      38      19
```

<br>

#### <a name="specifying-delimiters"></a>specifying delimiters

* use `-d` option to specify input delimiter other than default **tab** character
* only single character can be used, for multi-character/regex based delimiter use `awk` or `perl`

```bash
$ echo 'foo:bar:123:baz' | cut -d: -f3
123

$ # by default output delimiter is same as input
$ echo 'foo:bar:123:baz' | cut -d: -f1,4
foo:baz

$ # quote the delimiter character if it clashes with shell special characters
$ echo 'one;two;three;four' | cut -d; -f3
cut: option requires an argument -- 'd'
Try 'cut --help' for more information.
-f3: command not found
$ echo 'one;two;three;four' | cut -d';' -f3
three
```

* use `--output-delimiter` option to specify different output delimiter
* since this option accepts a string, more than one character can be specified
* See also [using $ prefixed string](https://unix.stackexchange.com/questions/48106/what-does-it-mean-to-have-a-dollarsign-prefixed-string-in-a-script)

```bash
$ printf 'foo\tbar\t123\tbaz\n' | cut --output-delimiter=: -f1-3
foo:bar:123

$ echo 'one;two;three;four' | cut -d';' --output-delimiter=' ' -f1,3-
one three four

$ # tested on bash, might differ with other shells
$ echo 'one;two;three;four' | cut -d';' --output-delimiter=$'\t' -f1,3-
one     three   four

$ echo 'one;two;three;four' | cut -d';' --output-delimiter=' - ' -f1,3-
one - three - four
```

<br>

#### <a name="complement"></a>complement

```bash
$ echo 'one;two;three;four' | cut -d';' -f1,3-
one;three;four

$ # to print other than specified fields
$ echo 'one;two;three;four' | cut -d';' --complement -f2
one;three;four
```

<br>

#### <a name="select-specific-characters"></a>select specific characters

* similar to `-f` for field selection, use `-c` for character selection
* See manual for what defines a character and differences between `-b` and `-c`

```bash
$ echo 'foo:bar:123:baz' | cut -c4
:

$ printf 'foo\tbar\t123\tbaz\n' | cut -c1,4,7
f       r

$ echo 'foo:bar:123:baz' | cut -c8-
:123:baz

$ echo 'foo:bar:123:baz' | cut --complement -c8-
foo:bar

$ echo 'foo:bar:123:baz' | cut -c1,6,7 --output-delimiter=' '
f a r

$ cut -c1-3 marks.txt
jan
foo
feb
foo
```

<br>

#### <a name="further-reading-for-cut"></a>Further reading for cut

* `man cut` and `info cut` for more options and detailed documentation
* [cut Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/cut?sort=votes&pageSize=15)

<br>

## <a name="tr"></a>tr

```bash
$ tr --version | head -n1
tr (GNU coreutils) 8.25

$ man tr
TR(1)                            User Commands                           TR(1)

NAME
       tr - translate or delete characters

SYNOPSIS
       tr [OPTION]... SET1 [SET2]

DESCRIPTION
       Translate, squeeze, and/or delete characters from standard input, writ‐
       ing to standard output.
...
```

<br>

#### <a name="translation"></a>translation

* one-to-one mapping of characters, all occurrences are translated
* as good practice, enclose the arguments in single quotes to avoid issues due to shell interpretation

```bash
$ echo 'foo bar cat baz' | tr 'abc' '123'
foo 21r 31t 21z

$ # use - to represent a range in ascending order
$ echo 'foo bar cat baz' | tr 'a-f' '1-6'
6oo 21r 31t 21z

$ # changing case
$ echo 'foo bar cat baz' | tr 'a-z' 'A-Z'
FOO BAR CAT BAZ
$ echo 'Hello World' | tr 'a-zA-Z' 'A-Za-z'
hELLO wORLD

$ echo 'foo;bar;baz' | tr ; :
tr: missing operand
Try 'tr --help' for more information.
$ echo 'foo;bar;baz' | tr ';' ':'
foo:bar:baz
```

* rot13 example

```bash
$ echo 'foo bar cat baz' | tr 'a-z' 'n-za-m'
sbb one png onm
$ echo 'sbb one png onm' | tr 'a-z' 'n-za-m'
foo bar cat baz

$ echo 'Hello World' | tr 'a-zA-Z' 'n-za-mN-ZA-M'
Uryyb Jbeyq
$ echo 'Uryyb Jbeyq' | tr 'a-zA-Z' 'n-za-mN-ZA-M'
Hello World
```

* use shell input redirection for file input

```bash
$ cat marks.txt
jan 2017
foobar  12      45      23
feb 2017
foobar  18      38      19

$ tr 'a-z' 'A-Z' < marks.txt
JAN 2017
FOOBAR  12      45      23
FEB 2017
FOOBAR  18      38      19
```

* if arguments are of different lengths

```bash
$ # when second argument is longer, the extra characters are ignored
$ echo 'foo bar cat baz' | tr 'abc' '1-9'
foo 21r 31t 21z

$ # when first argument is longer
$ # the last character of second argument gets re-used
$ echo 'foo bar cat baz' | tr 'a-z' '123'
333 213 313 213

$ # use -t option to truncate first argument to same length as second
$ echo 'foo bar cat baz' | tr -t 'a-z' '123'
foo 21r 31t 21z
```

<br>

#### <a name="escape-sequences-and-character-classes"></a>escape sequences and character classes

* Certain characters like newline, tab, etc can be represented using escape sequences or octal representation
* Certain commonly useful groups of characters like alphabets, digits, punctuations etc have character class as shortcuts
* See [gnu tr manual](http://www.gnu.org/software/coreutils/manual/html_node/Character-sets.html#Character-sets) for all escape sequences and character classes

```bash
$ printf 'foo\tbar\t123\tbaz\n' | tr '\t' ':'
foo:bar:123:baz

$ echo 'foo:bar:123:baz' | tr ':' '\n'
foo
bar
123
baz
$ # makes it easier to transform
$ echo 'foo:bar:123:baz' | tr ':' '\n' | pr -2ats'-'
foo-bar
123-baz

$ echo 'foo bar cat baz' | tr '[:lower:]' '[:upper:]'
FOO BAR CAT BAZ
```

* since `-` is used for character ranges, place it at the end to represent it literally
    * cannot be used at start of argument as it would get treated as option
    * or use `--` to indicate end of option processing
* similarly, to represent `\` literally, use `\\`

```bash
$ echo '/foo-bar/baz/report' | tr '-a-z' '_A-Z'
tr: invalid option -- 'a'
Try 'tr --help' for more information.

$ echo '/foo-bar/baz/report' | tr 'a-z-' 'A-Z_'
/FOO_BAR/BAZ/REPORT

$ echo '/foo-bar/baz/report' | tr -- '-a-z' '_A-Z'
/FOO_BAR/BAZ/REPORT

$ echo '/foo-bar/baz/report' | tr '/-' '\\_'
\foo_bar\baz\report
```

<br>

#### <a name="deletion"></a>deletion

* use `-d` option to specify characters to be deleted
* add complement option `-c` if it is easier to define which characters are to be retained

```bash
$ echo '2017-03-21' | tr -d '-'
20170321

$ echo 'Hi123 there. How a32re you' | tr -d '1-9'
Hi there. How are you

$ # delete all punctuation characters
$ echo '"Foo1!", "Bar.", ":Baz:"' | tr -d '[:punct:]'
Foo1 Bar Baz

$ # deleting carriage return character
$ cat -v greeting.txt 
Hi there^M
How are you^M
$ tr -d '\r' < greeting.txt | cat -v
Hi there
How are you

$ # retain only alphabets, comma and newline characters
$ echo '"Foo1!", "Bar.", ":Baz:"' | tr -cd '[:alpha:],\n'
Foo,Bar,Baz
```

<br>

#### <a name="squeeze"></a>squeeze

* to change consecutive repeated characters to single copy of that character

```bash
$ # only lower case alphabets
$ echo 'FFoo seed 11233' | tr -s 'a-z'
FFo sed 11233

$ # alphabets and digits
$ echo 'FFoo seed 11233' | tr -s '[:alnum:]'
Fo sed 123

$ # squeeze other than alphabets
$ echo 'FFoo seed 11233' | tr -sc '[:alpha:]'
FFoo seed 123

$ # only characters present in second argument is used for squeeze
$ echo 'FFoo seed 11233' | tr -s 'A-Z' 'a-z'
fo sed 11233

$ # multiple consecutive horizontal spaces to single space
$ printf 'foo\t\tbar \t123     baz\n'
foo             bar     123     baz
$ printf 'foo\t\tbar \t123     baz\n' | tr -s '[:blank:]' ' '
foo bar 123 baz
```

<br>

#### <a name="further-reading-for-tr"></a>Further reading for tr

* `man tr` and `info tr` for more options and detailed documentation
* [tr Q&A on unix stackexchange](http://unix.stackexchange.com/questions/tagged/tr?sort=votes&pageSize=15)

<br>

## <a name="basename"></a>basename

```bash
$ basename --version | head -n1
basename (GNU coreutils) 8.25

$ man basename
BASENAME(1)                      User Commands                     BASENAME(1)

NAME
       basename - strip directory and suffix from filenames

SYNOPSIS
       basename NAME [SUFFIX]
       basename OPTION... NAME...

DESCRIPTION
       Print  NAME  with  any leading directory components removed.  If speci‐
       fied, also remove a trailing SUFFIX.
...
```

**Examples**

```bash
$ pwd
/home/learnbyexample

$ basename "$PWD"
learnbyexample

$ basename '/home/learnbyexample/proj_adder/power.log'
power.log

$ #use suffix option -s to strip file extension from filename
$ basename -s '.log' '/home/learnbyexample/proj_adder/power.log'
power
```

<br>

## <a name="dirname"></a>dirname

```bash
$ dirname --version | head -n1
dirname (GNU coreutils) 8.25

$ man dirname
DIRNAME(1)                       User Commands                      DIRNAME(1)

NAME
       dirname - strip last component from file name

SYNOPSIS
       dirname [OPTION] NAME...

DESCRIPTION
       Output each NAME with its last non-slash component and trailing slashes
       removed; if NAME contains no  /'s,  output  '.'  (meaning  the  current
       directory).
...
```

**Examples**

```bash
$ pwd
/home/learnbyexample

$ dirname "$PWD"
/home

$ dirname '/home/learnbyexample/proj_adder/power.log'
/home/learnbyexample/proj_adder
```

