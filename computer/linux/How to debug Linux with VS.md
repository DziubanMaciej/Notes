parentTopic:: [[Linux]]

# How to debug Linux programs with Visual Studio

Nomenclature:
- Host - Windows machine with Visual Studio and WSL installed
- Target - Linux machine, on which the application will run
  
Steps:
1. Download sources to host machine
2. In Visual Studio click File->Open->CMake and select root CMakeLists.txt file of the project
3. Go to Manage configurations->Edit Json. This should open CMakeSettings.json file. Here is a basic configuration, which will compile the project with WSL in Debug mode. Project can be further configured via "cmakeCommandArgs" field. (NOTE: sometimes manage configurations and all the CMake functionalities may be gone. In that case delete .vs folder and retry).
    ```json
    {
      "configurations": [
        {
          "configurationType": "Debug",
          "name": "Wsl${configurationType}",
          "generator": "Unix Makefiles",
          "buildRoot": "${projectDir}\\..\\build\\${name}",
          "installRoot": "${projectDir}\\..\\install\\${name}",
          "cmakeExecutable": "/usr/bin/cmake",
          "cmakeCommandArgs": "",
          "buildCommandArgs": "-j 144",
          "ctestCommandArgs": "",
          "inheritEnvironments": [ "linux_x64" ],
          "wslPath": "${defaultWSLPath}",
          "addressSanitizerRuntimeFlags": "detect_leaks=0"
        }
      ]
    }
    ```
4. Save the file. This should trigger the process of running CMake and configuring Visual Studio (it may take a while).
5. In Solution Explorer click on "Switch between solution and available views" icon and select "CMake targets view" to open the project. You should see all the targets (libraries, executables, etc.) from your project.
6. Compile required binaries.
7. Run&debug.
    1. Native Linux machine (by starting from VS)
        - TODO
    2. Native Linux machine (by attaching)
        - Copy compiled binaries to the target machine
        - Call "sudo sysctl -w kernel.yama.ptrace_scope=0" on the target machine to allow attaching debugger to running processes
        - Execute the application without sudo (possibly stall it at the beginning)
        - On the host machine press Ctrl+Alt+P in Visual Studio. Select SSH and fill the IP and password/key to the target machine
        - Select the process for debug and attach
    3. Local WSL (by starting from VS)
        - TODO make the snipped more project-agnostic
        - Snippet for NEO ULTs
            ```json
            {
              "version": "0.2.1",
              "defaults": {},
              "configurations": [

                {
                  "type": "cppdbg",
                  "name": "igdrcl_tests (bin\\igdrcl_tests)",
                  "project": "CMakeLists.txt",
                  "projectTarget": "igdrcl_tests (bin\\igdrcl_tests)",
                  "cwd": "/mnt/w/mdziuban/neo3/buildWSL/WSL-GCC-Debug/bin",
                  "program": "/mnt/w/mdziuban/neo3/buildWSL/WSL-GCC-Debug/bin/igdrcl_tests",
                  "MIMode": "gdb",
                  "externalConsole": true,
                  "environment": [
                    {
                      "name": "LD_LIBRARY_PATH",
                      "value": "/mnt/w/mdziuban/neo3/buildWSL/WSL-GCC-Debug/bin"
                    }
                  ],
                  "args": [
                    "--gtest_filter=*",
                    "--gtest_catch_exceptions=0",
                    "--enable_default_listener",
                    "--disable_pagefaulting_tests",
                    "--product",
                    "skl",
                    "--disable_alarm"
                  ],
                  "pipeTransport": {
                    "pipeProgram": "${debugInfo.windowsSubsystemPath}",
                    "pipeArgs": [
                      "${debuggerCommand}",
                      "--tty=${debugInfo.tty}"
                    ],
                    "debuggerPath": "/usr/bin/gdb"
                  },
                  "visualizerFile": "${debugInfo.linuxNatvisPath}",
                  "showDisplayString": true,
                  "setupCommands": [
                    {
                      "text": "Documentation: Learn how to configure remote debugging",
                      "description": "See here for more info http://aka.ms/vslinuxdebug",
                      "ignoreFailures": true
                    },
                    {
                      "text": "-enable-pretty-printing",
                      "ignoreFailures": true
                    }
                  ]
                }
              ]
            }
            ```
