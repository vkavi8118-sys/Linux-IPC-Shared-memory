# Linux-IPC-Shared-memory
Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that illustrates two processes communicating using shared memory.

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/wait.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written;
    char some_text[TEXT_SZ];
};

int main() {
    int shmid;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;

    // Create shared memory (use IPC_PRIVATE to avoid key issues)
    shmid = shmget(IPC_PRIVATE, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    if (shmid == -1) {
        perror("shmget failed");
        exit(EXIT_FAILURE);
    }

    printf("Shared memory id = %d\n", shmid);

    // Attach
    shared_memory = shmat(shmid, NULL, 0);
    if (shared_memory == (void *)-1) {
        perror("shmat failed");
        exit(EXIT_FAILURE);
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written = 0;

    pid_t pid = fork();

    if (pid < 0) {
        perror("fork failed");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {   // Child (Consumer)
        while (1) {
            while (shared_stuff->written == 0)
                sleep(1);

            printf("Consumer received: %s", shared_stuff->some_text);

            if (strncmp(shared_stuff->some_text, "end", 3) == 0)
                break;

            shared_stuff->written = 0;
        }

        shmdt(shared_memory);
        exit(EXIT_SUCCESS);
    } else {   // Parent (Producer)
        char buffer[TEXT_SZ];

        while (1) {
            printf("Enter Some Text: ");
            fgets(buffer, TEXT_SZ, stdin);

            strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
            shared_stuff->written = 1;

            printf("Producer sent: %s", shared_stuff->some_text);

            if (strncmp(buffer, "end", 3) == 0)
                break;

            while (shared_stuff->written == 1)
                sleep(1);
        }

        wait(NULL);

        shmdt(shared_memory);
        shmctl(shmid, IPC_RMID, NULL);

        exit(EXIT_SUCCESS);
    }
}



## OUTPUT
<img width="1155" height="816" alt="os6(1)" src="https://github.com/user-attachments/assets/4b6c93b9-87da-446d-a860-7d2a7da46e9f" />
<img width="1051" height="867" alt="os6(2)" src="https://github.com/user-attachments/assets/64917e3c-0a2c-4b03-8761-9033d9cbef10" />


# RESULT:
The program is executed successfully.
