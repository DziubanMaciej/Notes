parentTopic:: [[Linux]]

# Search files by name
## Find
This will recursively list all files matching the criteria. Supports globbing.
```bash
find -name "NAME" ROOT_DIR
```
Useful options
- `-not -path PATH` - exclude path.
- `-not \( -path "PATH1" -o -path "PATH2" \)` - exclude multiple paths
- `-maxdepth DEPTH`, `mindepth DEPTH` - bounds for search depth. Search only in `ROOT_DIR` is `-maxdepth 1` (0 doesn't make sense).


excluding path
fuzzy

# Search files by content

## Grep
This will print file containing the pattern and the matched line.
```bash
grep -r PATTERN ROOT_DIR
```
Useful options (put before `PATTERN`):
- `--include "*.cpp"` - filter by extension. There can be multiple arguments like this.
- `--exclude-dir=.git` - filter out paths. There can be multiple arguments like this.
- `-i` - case insensitive.
- `-l` - only print unique files, without matched lines.
- `-n` - also number of the matched line.
- `-R` - recursive search following symlinks . Use instead of `-r`, which does not follow symlinks.
- `-F` - fixed string search instead of regex matching. Used to be `fgrep`.

## Find
Not recommended.
```bash
find . -name "*.cpp" -exec grep CmagProject {} \;
```

## Ripgrep
This will print matched lines along with their numbers sorted by files containing them. It automatically honors *.gitignore*.
```bash
rg PATTERN PATH
```
Useful options:
- `-uuu` - do not honor *gitignore*.
- `-t` - search by file type. Note this is not equivalent to extension. E.g. `-tcpp` will show both *.cpp* and *.h* files.
- `-g "*.cpp"` - filter by extension.
- `rg -p PATTERN PATH | less -R` - scrollable output.
- `--sort=path` - sort outputs making them deterministic.


# Rigrep-all

# Fzf

# Fd
