parentTopic:: [[Computer]]

# Processes and executable files querying

## Processes

| Problem                                       | Linux                                      | Windows                                                                                                                                                                |
| --------------------------------------------- | ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What dynamic libraries are loaded by process? | `lsof -p <PID>`                            | Resource Monitor -> CPU -> Tick the process -> Associated modules</br>Process explorer -> Ctrl+D to open lower pane with DLLs</br>Attach VS debugger -> Modules window |
| What files are opened by process?             | `lsof -p <PID>`</br>`ls /proc/<PID>/fd -l` | Process explorer -> Ctrl+D to open lower pane with Handles                                                                                                             |

## Executable file

| Problem                                                | Linux                 | Windows                                                                                                                                                                                                                                                              |
| ------------------------------------------------------ | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What symbols does a library export?                    | `readelf -CWs <FILE>` | Nirsoft dllexp (GUI)</br>[Dependencies]([https://github.com/lucasg/Dependencies](https://github.com/lucasg/Dependencies)) (GUI)</br>[strings](https://learn.microsoft.com/en-us/sysinternals/downloads/strings) (CLI)</br>Don't use DependencyWalker, it hangs often |
| What libraries does an executable load (statically)?   | `ldd <FILE>`          | [Dependencies]([https://github.com/lucasg/Dependencies](https://github.com/lucasg/Dependencies))                                                                                                                                                                     |
| What dynamic libraries does an executable file import? | `objdump -T <FILE>`   | [Dependencies]([https://github.com/lucasg/Dependencies](https://github.com/lucasg/Dependencies))                                                                                                                                                                     |
