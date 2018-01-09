# <a name="ruby-one-liners"></a>Ruby one liners

**Table of Contents**

* [Executing Ruby code](#executing-ruby-code)
* [Simple search and replace](#simple-search-and-replace)
    * [inplace editing](#inplace-editing)
* [Line filtering](#line-filtering)
    * [Regular expressions based filtering](#regular-expressions-based-filtering)
    * [Fixed string matching](#fixed-string-matching)
    * [Line number based filtering](#line-number-based-filtering)
* [Field processing](#field-processing)
    * [Field comparison](#field-comparison)
    * [Specifying different input field separator](#specifying-different-input-field-separator)
    * [Specifying different output field separator](#specifying-different-output-field-separator)

<br>

```
$ ruby --version
ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]

$ man ruby
RUBY(1)                Ruby Programmers Reference Guide                RUBY(1)

NAME
     ruby — Interpreted object-oriented scripting language

SYNOPSIS
     ruby [--copyright] [--version] [-SUacdlnpswvy] [-0[octal]] [-C directory]
          [-E external[:internal]] [-F[pattern]] [-I directory] [-K[c]]
          [-T[level]] [-W[level]] [-e command] [-i[extension]] [-r library]
          [-x[directory]] [--{enable|disable}-FEATURE] [--dump=target]
          [--verbose] [--] [program_file] [argument ...]

DESCRIPTION
     Ruby is an interpreted scripting language for quick and easy object-ori‐
     ented programming.  It has many features to process text files and to do
     system management tasks (like in Perl).  It is simple, straight-forward,
     and extensible.

     If you want a language for easy object-oriented programming, or you don't
     like the Perl ugliness, or you do like the concept of LISP, but don't
     like too many parentheses, Ruby might be your language of choice.
...
```

**Prerequisites and notes**

* familiarity with programming concepts like variables, printing, control structures, arrays, etc
* familiarity with regular expression
* this tutorial is primarily focussed on short programs that are easily usable from command line, similar to using `grep`, `sed`, `awk`, `perl` etc
* unless otherwise specified, consider input as ASCII encoded text only
* this is an attempt to translate [Perl chapter](./perl_the_swiss_knife.md) to `ruby`, I don't have prior experience of using `ruby`

<br>

## <a name="executing-ruby-code"></a>Executing Ruby code

* One way is to put code in a file and use `ruby` command with filename as argument
    * another is to use [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) at beginning of script, make the file executable and directly run it
* For short programs, one can use `-e` commandline option to provide code from command line itself
    * this entire chapter is about using `ruby` this way from commandline

```bash
$ cat code.rb
print "Hello Ruby\n"
$ ruby code.rb
Hello Ruby

$ # same as: perl -e 'print "Hello Perl\n"'
$ ruby -e 'print "Hello Ruby\n"'
Hello Ruby

$ # multiple commands can be issued separated by ;
$ # puts adds newline character if input doesn't end with a newline
$ # similar to: perl -E '$x=25; $y=12; say $x**$y'
$ ruby -e 'x=25; y=12; puts x**y'
59604644775390625
```

**Further Reading**

* `ruby -h` for summary of options
    * [explainshell](https://explainshell.com/explain?cmd=ruby+-F+-l+-anpe+-i+-0) - to quickly get information without having to traverse through the docs
* [ruby-lang documentation](https://www.ruby-lang.org/en/documentation/) - manuals, tutorials and references

<br>

## <a name="simple-search-and-replace"></a>Simple search and replace

* More detailed examples with regular expressions will be covered in later sections
* Just like other text processing commands, `ruby` will automatically loop over input line by line when `-n` or `-p` option is used
    * like `sed`, the `-n` option won't print the record
    * `-p` will print the record, including any changes made
    * default record separator is newline character
    * `$_` will contain the input record content, including the record separator (like `perl` and unlike `sed/awk`)
* and similar to other commands, `ruby` will work with both stdin and file input
    * See other chapters for examples of [seq](./miscellaneous.md#seq), [paste](./restructure_text.md#paste), etc

```bash
$ # change only first ',' to ' : '
$ # same as: perl -pe 's/,/ : /'
$ seq 10 | paste -sd, | ruby -pe 'sub(/,/, " : ")'
1 : 2,3,4,5,6,7,8,9,10

$ # change all ',' to ' : '
$ # same as: perl -pe 's/,/ : /g'
$ seq 10 | paste -sd, | ruby -pe 'gsub(/,/, " : ")'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10

$ # sub(/,/, " : ") is shortcut for $_.sub!(/,/, " : ")
$ # gsub(/,/, " : ") is shortcut for $_.gsub!(/,/, " : ")
$ # sub! and gsub! do inplace changing
$ # sub and gsub returns the result, similar to perl's s///r modifier
$ # () is optional, sub /,/, " : " can be used instead of sub(/,/, " : ")
```

<br>

#### <a name="inplace-editing"></a>inplace editing

```bash
$ cat greeting.txt
Hi there
Have a nice day

$ # original file gets preserved in 'greeting.txt.bkp'
$ # same as: perl -i.bkp -pe 's/Hi/Hello/' greeting.txt
$ ruby -i.bkp -pe 'sub(/Hi/, "Hello")' greeting.txt
$ cat greeting.txt
Hello there
Have a nice day

$ # use empty argument to -i with caution, changes made cannot be undone
$ ruby -i -pe 'sub(/nice day/, "safe journey")' greeting.txt
$ cat greeting.txt
Hello there
Have a safe journey
```

* Multiple input files are treated individually and changes are written back to respective files

```bash
$ cat f1
I ate 3 apples
$ cat f2
I bought two bananas and 3 mangoes

$ # same as: perl -i.bkp -pe 's/3/three/' f1 f2
$ ruby -i.bkp -pe 'sub(/3/, "three")' f1 f2
$ cat f1
I ate three apples
$ cat f2
I bought two bananas and three mangoes
```

**Further Reading**

* [ruby-doc Pre-defined variables](https://ruby-doc.org/core-2.5.0/doc/globals_rdoc.html#label-Pre-defined+variables) for explanation on `$_` and other such special variables
* [ruby-doc gsub](https://ruby-doc.org/core-2.5.0/String.html#method-i-gsub) for `gsub` syntax details

<br>

## <a name="line-filtering"></a>Line filtering

<br>

#### <a name="regular-expressions-based-filtering"></a>Regular expressions based filtering

* one way is to use `variable =~ /REGEXP/FLAGS` to check for a match
    * `variable !~ /REGEXP/FLAGS` for negated match
    * by default acts on `$_` if variable is not specified
    * see [ruby-doc Regexp](https://ruby-doc.org/core-2.5.0/Regexp.html) for regular expression details
* as we need to print only selective lines, use `-n` option
    * by default, contents of `$_` will be printed if no argument is passed to `print`

```bash
$ cat poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # same as: perl -ne 'print if /^[RS]/' poem.txt
$ # /^[RS]/ is shortcut for $_ =~ /^[RS]/
$ ruby -ne 'print if /^[RS]/' poem.txt
Roses are red,
Sugar is sweet,

$ # same as: perl -ne 'print if /and/i' poem.txt
$ ruby -ne 'print if /and/i' poem.txt
And so are you.

$ # same as: perl -ne 'print if !/are/' poem.txt
$ # !/are/ is shortcut for $_ !~ /are/
$ ruby -ne 'print if !/are/' poem.txt
Sugar is sweet,

$ # same as: perl -ne 'print if /are/ && !/so/' poem.txt
$ ruby -ne 'print if /are/ && !/so/' poem.txt
Roses are red,
Violets are blue,
```

* using different delimiter
* quoting from [ruby-doc Percent Strings](https://ruby-doc.org/core-2.5.0/doc/syntax/literals_rdoc.html#label-Percent+Strings)

> If you are using “(”, “[”, “{”, “<” you must close it with “)”, “]”, “}”, “>” respectively. You may use most other non-alphanumeric characters for percent string delimiters such as “%”, “|”, “^”, etc.

```bash
$ cat paths.txt
/foo/a/report.log
/foo/y/power.log
/foo/abc/errors.log

$ # same as: perl -ne 'print if /\/foo\/a\//' paths.txt
$ ruby -ne 'print if /\/foo\/a\//' paths.txt
/foo/a/report.log

$ # same as: perl -ne 'print if m#/foo/a/#' paths.txt
$ ruby -ne 'print if %r#/foo/a/#' paths.txt
/foo/a/report.log

$ # same as: perl -ne 'print if !m#/foo/a/#' paths.txt
$ ruby -ne 'print if !%r#/foo/a/#' paths.txt
/foo/y/power.log
/foo/abc/errors.log
```

<br>

#### <a name="fixed-string-matching"></a>Fixed string matching

* To match strings literally, use `index`
* See [ruby-doc index](https://ruby-doc.org/core-2.5.0/String.html#method-i-index) for details

```bash
$ # index returns matching position(starts at 0) and nil if not found
$ # same as: perl -ne 'print if index($_, "a[5]") != -1'
$ echo 'int a[5]' | ruby -ne 'print if /a[5]/'
$ echo 'int a[5]' | ruby -ne 'print if $_.index("a[5]")'
int a[5]

$ # however, string within double quotes gets interpolated, for ex
$ ruby -e 'a=5; puts "value of a: #{a}"'
value of a: 5

$ # so, for commandline usage, better to pass string as environment variable
$ # they are accessible via the ENV hash variable
$ # same as: perl -le 'print $ENV{SHELL}'
$ ruby -e 'puts ENV["SHELL"]'
/bin/bash

$ echo 'int #{a}' | ruby -ne 'print if $_.index("#{a}")'
-e:1:in `<main>': undefined local variable or method `a' for main:Object (NameError)
$ echo 'int #{a}' | s='#{a}' ruby -ne 'print if $_.index(ENV["s"])'
int #{a}
```

* `index` allows to use regex as well

```bash
$ # passing string
$ ruby -ne 'print if $_.index("a+b")' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # passing regex
$ ruby -ne 'print if $_.index(/a+b/)' eqns.txt
$ ruby -ne 'print if $_.index(/a\+b/)' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b
```

* return value is useful to match at specific position
* for ex: at start/end of line

```bash
$ cat eqns.txt
a=b,a-b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # start of line
$ # same as: s='a+b' perl -ne 'print if index($_, $ENV{s})==0' eqns.txt
$ s='a+b' ruby -ne 'print if $_.index(ENV["s"])==0' eqns.txt
a+b,pi=3.14,5e12

$ # optional 2nd argument allows to specify offset to start searching
$ # similar to: s='a+b' perl -ne 'print if index($_, $ENV{s})>0' eqns.txt
$ s='a+b' ruby -ne 'print if $_.index(ENV["s"], 1)' eqns.txt
i*(t+9-g)/8,4-a+b

$ # end of line
$ # same as: s='a+b' perl -ne '$pos = length() - length($ENV{s}) - 1;
$ #                  print if index($_, $ENV{s}) == $pos' eqns.txt
$ s='a+b' ruby -ne 'pos = $_.length - ENV["s"].length - 1;
                    print if $_.index(ENV["s"]) == pos' eqns.txt
i*(t+9-g)/8,4-a+b
```

<br>

#### <a name="line-number-based-filtering"></a>Line number based filtering

* special variable `$.` contains total records read so far, similar to `NR` in `awk`
    * as far as I've checked the docs, there's no equivalent of awk's `FNR`
* See also [ruby-doc eof](https://ruby-doc.org/core-2.5.0/IO.html#method-i-eof)

```bash
$ # print 2nd line
$ # same as: perl -ne 'print if $.==2' poem.txt
$ ruby -ne 'print if $.==2' poem.txt
Violets are blue,

$ # print 2nd and 4th line
$ # same as: perl -ne 'print if $.==2 || $.==4' poem.txt
$ # can also use: ruby -ne 'print if [2, 4].include?($.)' poem.txt
$ ruby -ne 'print if $.==2 || $.==4' poem.txt
Violets are blue,
And so are you.

$ # print last line
$ # same as: perl -ne 'print if eof' poem.txt
$ # $< is like filehandle for input files/stdin given from commandline
$ ruby -ne 'print if $<.eof' poem.txt
And so are you.
```

* for large input, use `exit` to avoid unnecessary record processing
* See [ruby-doc Control Expressions](https://ruby-doc.org/core-2.5.0/doc/syntax/control_expressions_rdoc.html) for syntax details

```bash
$ # same as: perl -ne 'if($.==234){print; exit}'
$ seq 14323 14563435 | ruby -ne 'if $.==234 then print; exit end'
14556

$ # mimicking head command
$ # same as: head -n3 or sed '3q' or perl -pe 'exit if $.>3'
$ seq 14 25 | ruby -pe 'exit if $.>3'
14
15
16

$ # same as: sed '3Q' or perl -pe 'exit if $.==3'
$ seq 14 25 | ruby -pe 'exit if $.==3'
14
15
```

* selecting range of lines
* See [ruby-doc Range](https://ruby-doc.org/core-2.5.0/Range.html) for syntax details

```bash
$ # in this context, the range is compared against $.
$ # same as: perl -ne 'print if 3..5'
$ seq 14 25 | ruby -ne 'print if 3..5'
16
17
18

$ # selecting from particular line number to end of input
$ # same as: perl -ne 'print if $.>=10'
$ seq 14 25 | ruby -ne 'print if $.>=10'
23
24
25
```

<br>

## <a name="field-processing"></a>Field processing

* `-a` option will auto-split each input record based on one or more continuous white-space
    * similar to default behavior in `awk` and same as `perl -a`
* Special variable array `$F` will contain all the elements, indexing starts from 0
    * negative indexing is also supported, `-1` gives last element, `-2` gives last-but-one and so on

```bash
$ cat fruits.txt
fruit   qty
apple   42
banana  31
fig     90
guava   6

$ # print only first field, indexing starts from 0
$ # same as: perl -lane 'print $F[0]' fruits.txt
$ ruby -ane 'puts $F[0]' fruits.txt
fruit
apple
banana
fig
guava

$ # print only second field
$ # same as: perl -lane 'print $F[1]' fruits.txt
$ ruby -ane 'puts $F[1]' fruits.txt
qty
42
31
90
6
```

* by default, leading and trailing whitespaces won't be considered when splitting the input record
    * same as `awk`'s default behavior and `perl -a`

```bash
$ printf ' a    ate b\tc   \n'
 a    ate b     c
$ printf ' a    ate b\tc   \n' | ruby -ane 'puts $F[0]'
a
$ printf ' a    ate b\tc   \n' | ruby -ane 'puts $F[-1]'
c

$ # number of elements
$ printf ' a    ate b\tc   \n' | ruby -ane 'puts $F.length'
4
```

<br>

#### <a name="field-comparison"></a>Field comparison

* operators `=`, `!=`, `<`, etc will work for both string/numeric comparison
* unlike `perl`, numeric comparison for text requires converting to appropriate numeric format
    * See [ruby-doc string methods](https://ruby-doc.org/core-2.5.0/String.html#method-i-to_c) for details

```bash
$ # if first field exactly matches the string 'apple'
$ # same as: perl -lane 'print $F[1] if $F[0] eq "apple"' fruits.txt
$ ruby -ane 'puts $F[1] if $F[0] == "apple"' fruits.txt
42

$ # print first field if second field > 35 (excluding header)
$ # same as: perl -lane 'print $F[0] if $F[1]>35 && $.>1' fruits.txt
$ ruby -ane 'puts $F[0] if $F[1].to_i > 35 && $.>1' fruits.txt
apple
fig

$ # print header and lines with qty < 35
$ # same as: perl -ane 'print if $F[1]<35 || $.==1' fruits.txt
$ ruby -ane 'print if $F[1].to_i < 35 || $.==1' fruits.txt
fruit   qty
banana  31
guava   6

$ # if first field does NOT contain 'a'
$ # same as: perl -ane 'print if $F[0] !~ /a/' fruits.txt
$ ruby -ane 'print if $F[0] !~ /a/' fruits.txt
fruit   qty
fig     90
```

<br>

#### <a name="specifying-different-input-field-separator"></a>Specifying different input field separator

* by using `-F` command line option

```bash
$ # second field where input field separator is :
$ # same as: perl -F: -lane 'print $F[1]'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts $F[1]'
123

$ # last field, same as: perl -F: -lane 'print $F[-1]'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts $F[-1]'
789
$ # second last field, perl -F: -lane 'print $F[-2]'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts $F[-2]'
bar

$ # second and last field, same as: perl -F: -lane 'print "$F[1] $F[-1]"'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts "#{$F[1]} #{$F[-1]}"'
123 789

$ # use quotes to avoid clashes with shell special characters
$ echo 'one;two;three;four' | ruby -F';' -ane 'puts $F[2]'
three
```

* last element of `$F` array will contain the record separator as well
    * note that default `-a` option without `-F` won't have this issue as whitespaces at start/end are stripped
* it doesn't make visual difference when `puts` is used as it adds newline only if not already present
* if the record separator is not desired, use `-l` option to remove the record separator from input

```bash
$ echo 'foo 123' | ruby -ane 'puts "#{$F[-1]}xyz"'
123xyz

$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts "#{$F[-1]}a"'
789
a
$ echo 'foo:123:bar:789' | ruby -F: -lane 'puts "#{$F[-1]}a"'
789a
```

* Regular expressions based input field separator

```bash
$ # same as: perl -F'\d+' -lane 'print $F[1]'
$ echo 'Sample123string54with908numbers' | ruby -F'\d+' -ane 'puts $F[1]'
string

$ # first field will be empty as there is nothing before '{'
$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[0]'

$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[1]'
foo
$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[2]'
bar
$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[-1]'
baz
```

* to process individual characters, simply use indexing on input string
* See [ruby-doc Encoding](https://ruby-doc.org/core-2.5.0/Encoding.html) for details on handling different string encodings

```bash
$ # same as: perl -F -lane 'print $F[0]'
$ echo 'apple' | ruby -ne 'puts $_[0]'
a

$ # if needed, chomp the record separator using -l
$ # same as: perl -F -lane 'print $F[-1]'
$ echo 'apple' | ruby -lne 'puts $_[-1]'
e

$ ruby -e 'puts Encoding.default_external'
UTF-8
$ printf 'hi👍 how are you?' | ruby -ne 'puts $_[2]'
👍
$ # use -E option to explicitly specify external/internal encodings
$ printf 'hi👍 how are you?' | ruby -E UTF-8:UTF-8 -ne 'puts $_[2]'
👍
```

<br>

#### <a name="specifying-different-output-field-separator"></a>Specifying different output field separator

* use `$,` to change separator between `print` arguments
    * could be remembered easily by noting that `,` is used to separate `print` arguments
    * note that `$,` doesn't affect `puts` which always uses newline as separator
* the `-l` option is useful here in more than one way
    * it removes input record separator
    * and appends the record separator to `print` output

```bash
$ # by default, the various arguments are concatenated
$ echo 'foo:123:bar:789' | ruby -F: -lane 'print $F[1], $F[-1]'
123789

$ # change $, if different separator is needed
$ # same as: perl -F: -lane '$,=" "; print $F[1], $F[-1]'
$ echo 'foo:123:bar:789' | ruby -F: -lane '$,=" "; print $F[1], $F[-1]'
123 789
$ echo 'foo:123:bar:789' | ruby -F: -lane '$,="-"; print $F[1], $F[-1]'
123-789

$ # array's join method also uses $,
$ # same as: perl -F: -lane '$,=" - "; print @F'
$ echo 'foo:123:bar:789' | ruby -F: -lane '$,=" - "; print $F.join'
foo - 123 - bar - 789
$ # or pass the separator as argument to join method
$ echo 'foo:123:bar:789' | ruby -F: -lane 'print $F.join(" - ")'
foo - 123 - bar - 789
```

* use `BEGIN` if same separator is to be used for all lines
    * statements inside `BEGIN` are executed before processing any input text

```bash
$ # same as: perl -lane 'BEGIN{$,=","} print @F' fruits.txt
$ ruby -lane 'BEGIN{$,=","}; print $F.join' fruits.txt
fruit,qty
apple,42
banana,31
fig,90
guava,6
```


<br>

<br>

<br>

*More to come*
