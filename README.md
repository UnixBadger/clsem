DESCRIPTION:
    The clsem (command line semaphore) application provides command line access to the Unix semaphore
    facility. A semaphore is a number that never goes negative. It is usually initialized to the 
    maximum count of some resource: number of processes, megabytes of memory, number of network
    connections, etc. Resource users decrement the semaphore value. If value is 0, the user blocks
    until some other user, another process or thread, increments the semaphore.

INSTALLATION:
    Running:
        ./configure --prefix=PATH
        make
        make install
    will build the clsem application and install
        ${PREFIX}/bin/clsem
        ${PREFIX}/share/man/man1/clsem.1
    "./configure" is the same as "./configure --prefix=/usr/local"

USAGE:
    clsem has two main subcommands.
        clsem new ... 
            Creates a semaphore.
        clsem incr ... 
            Tries to increase or decrease a semaphore. If the semaphore is 0, the clsem call blocks
            until another process increments it. 
    See clsem(1).

    Typical usage looks like:
        clsem new SEM_ENV ./run_progs.sh
    This creates a semaphore and executes run_progs.sh. $SEM_ENV in the run_progs.sh execution
    environment contains the semaphore identifier.

    run_progs.sh could run 8 programs, 4 at a time, with a code fragment like this:

        clsem incr $SEM_ENV 4
        for prog in prog1 prog2 prog3 prog4 prog5 prog6 prog7 prog8
        do
            clsem incr $SEM_ENV -1
            ( $prog; clsem incr $SEM_ENV 1; ) & 
        done

    Assume prog1 prog2 prog3 prog4 prog5 prog6 prog7 prog8 take a long time.
    Semaphore starts with value 4.
    First pass through the loop decrements the semaphore to 3 and runs prog1 in a background shell.
    Second pass through the loop decrements the semaphore to 2 and runs prog2 in a background shell.
    Third pass through the loop decrements the semaphore to 1 and runs prog3 in a background shell.
    Fourth pass through the loop decrements the semaphore to 0 and runs prog4 in a background shell.
    Fifth pass blocks, assuming all 4 prog's are still running, because the semaphore cannot be
    decremented below 0. Suppose prog2 finishes first. When prog2 exits, its parent shell increments
    the semaphore before exiting. This unblocks the clsem call at the start of the loop, allowing
    prog5 to run. prog1, prog3, prog4, and prog5 will run in the background. The loop pass for prog6
    will block at the first clsem call until another prog exits, and the parent shell of the exiting
    prog increments the semaphore. The blocking and unblocking continues such that the number of
    background processes running at any one time is at most 4.

REFERENCES:
    https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap04.html#tag_04_17
    https://pubs.opengroup.org/onlinepubs/9699919799/functions/semget.html
    https://pubs.opengroup.org/onlinepubs/9699919799/functions/semctl.html
    https://pubs.opengroup.org/onlinepubs/9699919799/functions/semop.html
    Rochkind, Marc J. Advanced UNIX Programming, Second Edition. Pearson Eduction. 2004.


LICENSE:
    The software is distributed under the terms of the Academic Free License, in AFL-3.0.
    Please email feedback to Gordon Carrie dev1960@polarismail.net.

