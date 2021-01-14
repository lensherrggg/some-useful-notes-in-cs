# Data Wrangling

## sed

`sed` is a stream editor that you could give short command to it for modifying the file instead of manipulate the contents directly. 

There are tons of commands but one of the most common ones is `s`: substitution. For example we could write

```bash
ssh myserver journalctl | grep sshd | grep "Disconnected from" | sed 's/.*Disconnected from //'
```

The `s` command is written on the form: `s/REGEX/SUBSTITUTION/`, where `REGEX` is the regular expression that you want to search for and `SUBSTITUTION` is the text you want to substitute matching text. 

## Regular Expression

Regular Expression is a powerful way to match text. In regular expressions you have a number of special characters. You could not only match a character but also match some type of character or a particular set of sections. 

`.` means any single character. 

`*` means zero or more characters and `+` means one or more. 

`[]` let you match one of many different characters. For example `sed 's/[ab]//'` replaces any characters that is either a or b with nothing. 

```bash
>> echo 'aba' | sed 's/[ab]//'
ba
>> echo 'bba' | sed 's/[ab]//'
ba
```

In the above example we find that it only replaced once. In default mode it only match the pattern once. If we append a `g` at the end it will keep matching. 

```bash
>> echo 'bbac' | sed 's/[ab]//g'
c
```

You could also do things like add modifiers to it. For example

```bash
>> echo 'abcaba' | sed -E 's/(ab)*//g'
ca
```

The above example matches the string `ab`. If I have a stand alone a or b it will not be replaced but if I have ab it will be removed. The `-E` flag enables `sed` to support more mordern syntax. 

```bash
>> echo 'abcababc' | sed -E 's/(ab|bc)*//g'
cc
>> echo 'abcabbc' | sed -E 's/(ab|bc)*//g'
c
```

The `|` tells `sed` to remove patterns that matches ab or bc. 

Let's see another example. Suppose we want to extract the usernames from logs like the following

```bash
>> cat ssh.log
Jan 05 12:57:15 thesquareplanet.com sshd[10579]: Disconnected from user jon 84.211.213.165 port 37650
Jan 08 02:34:21 thesquareplanet.com sshd[15262]: Disconnected from invalid user admin 58.120.227.29 port 52978 [preauth]
Jan 10 23:09:08 thesquareplanet.com sshd[20215]: Disconnected from user jon 24.61.9.143 port 51098
Jan 10 23:09:16 thesquareplanet.com sshd[20093]: Disconnected from user jon 24:61.9.143 port 51086
Jan 13 21:58:01 thesquareplanet.com sshd[25433]: Disconnected from authenticating user root 65.60.109.219 port 41337 [preauth]
```

What should we do to match these lines?

```bash
>> cat ssh.log | sed -E 's/^.*Disconnected from (invalid |authenticating )?user .* [0-9.]+ port [0-9]+( \[preauth\])$//'
```

This example will transform every line into null string. 

Some of the lines have `invalid ` but some do not so we put parenthesis at the two sides of invalid and put a `?` which indicates that there is zero or one `invalid ` space. `[0-9.]` is used for ip address which contains only numbers and dots and `+` means there are many of those characters. The `^` and `$` are used for anchoring the expressions. The `^` matches the beginning of the line and the `$` matches the end. 

To extract the username

```bash
>> cat ssh.log | sed -E 's/^.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])$/\2/'
```

The parenthesises indicates a capture group. In the above example we have three capture groups: `(invalid |authenticating )`, `(.*)` and `( \[preauth\])`. The `\2` means we want the second capture group. 

## Some other useful commands

`wc` counts the number of words. If you add a `-l` flag it will count the nubmer of lines. 

`sort` sorts the input lines. 

`uniq` look at a sorted list of lines and only output those that are unique. The `-c` flag counts the number of duplicates for any lines. 

`awk` is a column based stream processor. It is more focused on column data. It will parse the input by white space separated columns. For instance, `awk '{print $2}'` prints the second column. 

`paste` is a command that takes a bunch of lines and puts them together in a single line, which is indicated by `-s`, and with delimiter comma, which is indicated by `-d.`. 

`gnuplot` is a good visulization tool. 

`xargs` takes a list of inputs and takes them into arguments. 