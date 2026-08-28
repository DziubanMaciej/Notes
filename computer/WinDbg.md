parentTopic:: [[Computer]]

# How to connect to WinDBG

- Prepare target ([link](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/setting-up-a-network-debugging-connection)). Run these commands as admin. Second command will output the key to paste on the host side.
    ```powershell
    bcdedit /debug on
    bcdedit /dbgsettings net hostip:10.22.73.24 port:50005  # This will output the key needed on the host side
    bcdedit /set "{dbgsettings}" busparams 10.0.0           # Busparams is BDF accessed in Device Manager->Properties->Details->Location Information
    bcdedit /dbgsettings                                    # This doesn't do anything, just prints the current settings
    ```
- Attach windbg on host (install [windbg](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)):
    - Command line (use key printed by the target machine)
        ```powershell
        C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\windbg.exe -k net:port=50005,key=lsiqq7e2lky.2z4n5whgphqih.2gthbbqesmvjz.3vac0b1rmymgz
        ```
    - GUI: windbg -> File -> Kernel debug
- Target:
    ```powershell
    shutdown /r -t 0
    ```
- Windbg should connect after restart
- Remember to disable kernel debugging after finishing work. It may dramatically slow down the ethernet connection.

# DML
DML - Debugger markup language. It's a language used by certain WinDbg commands to display interactive, clickable output. Clicking the output will trigger other commands. The commands to be run will be displayed at the bottom on hover:
![[windbg_dml.png]]


# Commands
Command types:
- Native commands
- Debugger meta commands (start with `.`)
- Extension commands (start with `!`)

Some typical debugger commands

| Command  | Description |
|----------|-------------|
| `k`      | Show stack frame. Some nice variants:</br>`kc` - clean trace, without addresses</br>`kf` - size of stack for each frame |
| `bl`     | Show breakpoints |
| `bp`     | Set breakpoint, e.g. `bp main` or `bp BinaryName!main` |
| `bp /w`  | Set conditional breakpoint, e.g. `bp /w "var == 4" main` |
| `bc`     | Clear breakpoint by its index, e.g. `bc 3` |

Some typical misc commands

| Command    | Example                                    | Meaning of mnemonic    | Description                               |
| ---------- | ------------------------------------------ | ---------------------- | ----------------------------------------- |
| `x`        | `x Notepad!WinMain`</br>`x Notepad!*Main*` | E**x**amine symbols    | Look for address of symbol                |
| `k`        |                                            |                        |                                           |
| `lm`       |                                            | **L**ist **m**odules   | Lists loaded modules                      |
| `lmvm`     |                                            |                        | Display information about a loaded binary |
| `.hh`      |                                            | **H**tml **Help**      | Shows help window                         |
| `!analyze` |                                            | Usually used with `-v` |                                           |


# Debug techniques

## Anti-anti-debugger

The simplest anti-cheat on Windows is the regular Win32 function IsDebuggerPresent().  If you put a breakpoint on that, if it gets hit, then step forward to the RET, then add a breakpoint at that point that clears rax, so it returns 0 all the time:

```
bu . "r rax=0 ; gc"
```

Disable the bp on IsDebuggerPresent and run.

## Buffer overrun

Use this, when you see error like this:
![[heap_corruption.png]]

The WinDbg package includes `gflags.exe`. You can tell the OS to do "heap tail checking". It arranges things so allocations have the end of their block at the end of a page of memory. It sets the next page as a guard page – when the buffer overrun occurs it writes to the guard page and raises an exception. You should get a callstack that points to the overrun. 

You can find `gflags.exe` in the same place as `windbg.exe`. Switch to the Image tab, add the name of the .exe in the space provided, and press TAB. If that .exe was not already being tracked, all the checkboxes will be cleared. The upper-left "heap tail checking" – check that box. Use Apply/OK, and whenever that exe runs, heap tail checking will be active. When debugging is done, repeat the Image tab procedure, and uncheck the “heap tail checking” box. When no options are set for that exe, it will be removed from the registry.

That same Image tab can set other things, including the ability to launch a debugger when the OS launches a specific .exe. This is helpful when an application does a bunch of startup stuff, and then launches another program to carry out the work, and that is the program you need to debug. The `gflags.exe `will make it so any time the *program-to-be-debugged* is launched, the debugger will be launched, and the debugger will launch the program you need to debug.

## Symbols

Symbol paths can be changed in the dialog box (shortcut: `ctrl+s`), but for some reason it seems to display a little bit different things than the commands below. Let's trust the commands:

`.sympath` - print symbols paths

`.sympath  C:\MyPath` - set symbols path

`.sympath+ C:\MyPath` - append to symbols paths

`.symfix  C:\MyLocalCacheDir` - set symbols path to Microsoft symbols

`.symfix+ C:\MyLocalCacheDir` - append Microsoft symbols to symbols paths

`.reload` - run it after changing symbols

`!sym noisy` - print verbose information during symbol reload in order to find out, why the symbols couldn't be loaded. Disable it with `!sym quiet`.
