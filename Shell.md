# Shell

Note of 6.NULL

## Define a variable

```bash
foo=bar
echo $foo
>> bar
```

Remeber don't add spaces
```bash
foo = bar
>> command not found: foo
```

## Define a string

```bash
echo "Hello"
>> Hello
echo 'World'
>> World
# difference between single and double quotes
echo "Value is $foo"
>> Value is bar
echo 'Value is $foo'
>> Value is $foo
```

## Define a function

Run `vim mcd.sh`

```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

`$1` is the first argument. 
In `mcd` function we first make a directory and then `cd` into that folder. 

```bash
>> source mcd.sh
>> mcd test
```

Then we'll find that we've created a test directory and we are inside that directory. 

In a function, `$0` is the name of the script, the `$2` to `$9` are the second to ninth arguments. `$?` get the error code from previous command. `$_` will give us the last argument of the previous command. 

```bash
>> rmdir test
>> mkdir test
>> cd $_
```

We deleted the test folder and re-made it. 

```bash
>> echo "Hello"
Hello
>> echo $?
0
>> grep foobar mcd.sh
>> echo $?
1 # there is no foobar string in mcd.sh
true
>> echo $?
0
false
>> echo $?
1
```

`!!` is interesting. Sometimes we would to run a command but we could find that we don't have the permission to execute it. Then we could type `sudo !!` and click `Tab`. The `!!` will be replaced by the last failed command.  

```bash
>> mkdir /mnt/new
mkdir: /mnt/new: Permission denied
# First type in "sudo !!" and click on tab and the following command will show up
>> sudo mkdir /mnt/new
Password: # get sudo permission
```

Conditions

```bash
>> false || echo "Oops fail"
Oops fail
>> true || echo "Will not be printed"

>> Ttrue && echo "Things went well"
hings went well
>> false && echo"This will not print"

>> false; echo "This will always print"
This will always print
```

Use return value as a variable

```bash
>> echo "We are in $(pwd)"
We are in $Your directory$
>> foo=$(pwd)
>> echo $foo
$Your directory$
```

Take a look at the following example
```bash
echo "Starting program at $(date)" # Data will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do # $@ means all the arguments
    grep foobar "$file" > /dev/null 2>/dev/null # 2> is for redirecting STDERR
    # When pattern is not found, grep has exit status
    # We redirect STDOUT and STDERR to a null register

    if [[ "$?" -ne 0 ]]; then
        echo "File $file does not have any foo bar, adding one"
        echo "# foobar" >> "$file" # > will replace all of the content, >> will append to the end of the file
    fi
done
```

## Pattern Matching

```bash
>> ls
example.sh image.png mcd.sh project1 project2 script.py test
>> ls *.sh
example.sh mcd.sh
>> mkdir project42
>>ls project? # expand to a single character
project1:
src

project2:
src
>> convert image.png image.jpg 
# type in convert image.{png,jpg} and click Tab then it will automatically expand to the above line. convert command is used for image convertion. 
# mkdir -p proj{1,2}/src{1,2}
>> mkdir -p proj1/src1 proj1/src2 proj2/src1 proj2/src2
>> mkdir foo bar
# touch {foo,bar}/{a..d}
>> touch foo/a foo/b foo/c foo/d bar/a bar/b bar/c bar/d
```

## Default path

Script.py

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
# foobar
```

Run the command
```bash
>> python script.py a b c
c
b
a
>> ./script.py a b c # first line of the script gives the path of interpreter
c b a
```

You could also use `#!/usr/bin/env python`

## Debug

```bash
>> shellcheck mcd.sh
```

Errors might happen will show up

## Help information

```bash
man mv
```

The help info of command `mv` will show up. 

Sometimes the help info from `man` could be hard to understand. There is another useful tool called `tldr` that might give you information that is easier to understand because it will show you some examples. 

## Find files

find command

```bash
>> find . -name src -type d # find file whose name is src and type is directory
./project1/src
./project2/src
>> find . -path '**/test/*.py' -type f # ** matches all files and directories
./project1/src/test/test1.py
./project1/src/test/test2.py
./project2/src/test/test1.py
./project2/src/test/test2.py
./project2/src/test/test3.py
>> find . -mtime -l # files modified within last day. mtime means modified time
>> find . -name "*.tmp" -exc rm {} \; # find files with suffix .tmp and delete them. The result will be put into {}
>> find . -name "*.tmp"
>> fd ".*py" # a replacement of find which is more powerful
```

The locate command searches for a database `/var/lib/locatedb` which contains all local file info instead of searching in a specific directory. The db is automatically created by Linux and will be updated everyday. 

```bash
locate shell
```

The grep command will help you find something withi the content. 

```bash
grep -R foobar . # recursively search "foobar" from files in current directory
```

## History command

Up arrow may not be very efficient. 

```bash
>> history
Some of your history commands
>> history 1
History from beginning
>> history 1 | grep convert
History command that contains convert
>> # ctrl + R, type in the keywords
History command that contains keyword # ctrl + R again and it will go through the older command contains keyword
# type Enter to execute or type right arrow to print it to your terminal. You can choose to execute or modify it. 
```

## List structure of directory

Run `ls` recursively

```bash
ls -R
```

For a nicer format

```bash
tree
```