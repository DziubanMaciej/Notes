parentTopic:: [[Computer]]

# Arrays
Arrays are a bashism! Classical `/bin/sh` doesn't support them. There are two types of arrays: indexed and associative.

### Differences
| Criterion | Normal arrays | Associative arrays |
|---| -------------| ----|
| Indexing| With ints | With strings |
| Creating | `declare -a arr`</br>`a=()`</br>`a=(abc 10 dd)` | `declare -A arr` |

### Commonalities
```
 ${arr[*]}         # All of the items in the array
 ${!arr[*]}        # All of the indexes in the array
 ${#arr[*]}        # Number of items in the array
 ${#arr[0]}        # Length of item zero
 ```

### Common operations
Iterating over key-value pairs
```bash
for key in ${!arr[*]}; do 
    value="${arr[$key]}" 
    echo "$key: $value"
done
```

Iterating over values
```bash
for value in ${b[*]}; do
    echo "$value"
done
```





# Bashisms
- Arrays
- Backticks, e.g. ``a=`echo wow` ``. Use `a="$(echo a)""` instead.
- `$EUID`, `$UID`. Use `id -u`, `id -ru` instead.
- Parameter `echo -n`. Use `printf` instead.
- The `function` keyword. It is not needed, you can just write `myFunc() { echo "Hey"}`.
- Double brackets `[[ $a = 0 ]]`. Most of the stuff is posix-compliant, but not all, e.g. regex matching (use `grep` instead).
- The `local` specifier. There is no alternative.
- Process substitution.





# Looping over command
### Put command as for loop arg
- (+) Works in the same shell
- (-) Requires `IFS` to iterate over lines instead of words
- (+) Works in Dash
    ```bash
    IFS="
    "
    for line in $(cat /home/maciej/a); do
        echo "Line:   $line"
    done
    ```

### While loop
- (-) Works in subshell (cannot set variables inside and expect them to be accessible outside the loop)
- (+) Doesn't need `IFS`
- (+) Works in Dash
    ```bash
    cat /home/maciej/a | while read -r line ; do
        echo "Line:   $line"
    done
    ```

### For loop with here-string
- (+) Works in same shell
- (+) Doesn't require `IFS`
- (-) Doesn't work in Dash (non-posix)
    ```bash
    while read -r line; do
        echo "Line:    $line"
    done <<< "$(cat /home/maciej/a)"
    ```





# Looping over file
```bash
while read -r line; do
    echo "Line:    $line"
done < "/home/maciej/a"
```





# Looping n times
### Manually maintained loop
- (+) No extra processes
- (+) Works for variable `n`
- (+) Works in dash
```bash
i=0
count=10
while [ "$i" -lt "$count" ]; do
    echo $i
    i=$((i+1))
done
```

### With brace expansion
- (+) No extra processes
- (-) Only for constant `n`
- (-) Doesn't work in Dash (non posix)
```bash
for i in {1..10}; do echo $i; done
```

### With seq
- (-) Calls an extra process
- (+) Works for variable `n`
- (+) Works in dash
```bash
count=10
for i in $(seq $count); do  echo $i; done
```





# Assigning multiline string to variable

### Heredoc with `-d`
- (+) No extra processes
- (+) Can be indented with tabs (because of `<<-` instead of `<<`)
- (+) No need to escape quotes
- (-) Doesn't work in dash (`-d` switch is not posix)
```bash
read -r -d '' VAR <<- EOM
This is line 1.
This is line "2".
This is line 3.
EOM
```

### Heredoc
- (-) Calls an extra `cat` process
- (+) Can be indented with tabs (because of `<<-` instead of `<<`)
- (+) No need to escape quotes
- (+) Works in dash
```bash
VAR=$(cat <<-EOM
This is line 1.
This is line "2".
This is line 3.
EOM
)
```

### Printf
- (-) Calls an extra `printf` process
- (-) Escaping quotes (or using single quotes) is required
- (-) Cannot be nicely formatted
- (+) Works in dash
```bash
VAR=$(printf "This is line 1.\nThis is line \"2\".\nThis is line 3.\n")
```

### Simple assignment
- (+) No extra processes
- (-) Escaping quotes (or using single quotes) is required
- (-) Can be formatted in multiple lines, but no indents. Closing quote must be on the last line
- (+) Works in dash
```bash
VAR="\
This is line 1.
This is line \"2\".
This is line 3."
```





# Substring removal
It is possible to match a glob and remove it from the string. You can remove either from beginning (with an `#` character) or from end (with a `%` character). By default the asterisks in glob will match the shortest possible substring, but doubling the `#` or `%` makes it match the longest substring. This is posix compliant and works in Dash.
```bash
var="aabbcc aabbcc"
echo "${var#*b}"   # Output: "bcc aabbcc". '*' was matched to "aa" - the shortest possible string that comes before "b"
echo "${var##*b}"  # Output: "cc". '*' was matched to "aabbcc aab" - the longest possible string that comes before "b"
echo "${var%b*}"   # Output: "aabbcc aab". '*' was matched to "cc" - the shortest possible string that comes after "b"
echo "${var%%b*}"  # Output: "aa". '*' was matched to 'bcc aabbcc' - the longest possible string that comes after "b"
```

This can be used to split two word lines
```bash
line="abc def"
first="${line% *}"
second="${line#* }"
```

References: [tldp](https://tldp.org/LDP/abs/html/string-manipulation.html) [Luke Smith](https://www.youtube.com/watch?v=QXineadwG4E&list=PL-p5XmQHB_JREOtBfLdKSswBGYyXwXMUy&index=21)





# Substring replacement
Let's you avoid using sed for some simple cases. Doesn't work in Dash.
```bash
var="aaa bbb aaa bbb aaa"
echo "${var/aaa/AAA}"  # Output: "AAA bbb aaa bbb aaa". Only the first match was replaced
echo "${var//aaa/AAA}" # Output: "AAA bbb AAA bbb AAA". All matches were replaced
echo "${var/#aaa/AAA}" # Output: "AAA bbb aaa bbb aaa". Substite only at the front
echo "${var/%aaa/AAA}" # Output: "aaa bbb aaa bbb AAA". Substite only at the back
```





# Adding leading zeros to numbered files
A simple command to add additional leading zero character to names of all numbered files in a directory. For example when whe have files like `01.png`, `02.png`, `03.png`, etc. and at some point we have add hundredth, we'll have to rename to `001.png`, `002.png`, `003.png`.
```
find . -maxdepth 1 -mindepth 1 -type f | sed "s/^\.\///g" | grep -E "^[0-9]+\.[A-z]+$" | xargs -I{} mv "{}" "0{}"
```

Explanation:
1. `find . -maxdepth 1 -mindepth 1 -type f` returns all files in current directory 
2. `sed "s/^\.\///g"` removes leading `./`, e.g. `./032.png` -> `032.png`
3. `grep -E "^[0-9]+\.[A-z]+$"` filters out files which are not numbered files
4. `xargs -I{} mv "{}" "0{}"` renames the files





# Validating files after resizing images
```
DIR=MyPhotos bash -c 'find  $DIR -type f | cut -d. -f2 | sort | uniq -c; du -sh $DIR'
```
