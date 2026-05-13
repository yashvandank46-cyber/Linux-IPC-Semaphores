# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.


```
#include <stdio.h>      
#include <stdlib.h>     
#include <unistd.h>     
#include <sys/types.h>  
#include <sys/ipc.h>    
#include <sys/sem.h>    
#include <sys/wait.h>   
#include <time.h>      

#define NUM_LOOPS 10   // Number of producer-consumer cycles

// Define union semun
union semun {
    int val;
    struct semid_ds *buf;
    unsigned short int *array;
    struct seminfo *__buf;
};

// Function for wait (P operation)
void wait_semaphore(int sem_set_id) {
    struct sembuf sem_op;

    sem_op.sem_num = 0;
    sem_op.sem_op = -1;   // Decrement semaphore
    sem_op.sem_flg = 0;

    semop(sem_set_id, &sem_op, 1);
}

// Function for signal (V operation)
void signal_semaphore(int sem_set_id) {
    struct sembuf sem_op;

    sem_op.sem_num = 0;
    sem_op.sem_op = 1;    // Increment semaphore
    sem_op.sem_flg = 0;

    semop(sem_set_id, &sem_op, 1);
}

int main() {

    int sem_set_id;
    union semun sem_val;
    int child_pid;

    // Create semaphore set
    sem_set_id = semget(IPC_PRIVATE, 1, 0600);

    if (sem_set_id == -1) {
        perror("semget");
        exit(1);
    }

    printf("Semaphore set created, ID = %d\n", sem_set_id);

    // Initialize semaphore value to 0
    sem_val.val = 0;

    if (semctl(sem_set_id, 0, SETVAL, sem_val) == -1) {
        perror("semctl");
        exit(1);
    }

    // Create child process
    child_pid = fork();

    if (child_pid < 0) {
        perror("fork");
        exit(1);
    }

    // Child Process --> Consumer
    if (child_pid == 0) {

        for (int i = 0; i < NUM_LOOPS; i++) {

            wait_semaphore(sem_set_id);

            printf("Consumer consumed item %d\n", i);

            fflush(stdout);
        }

        exit(0);
    }

    // Parent Process --> Producer
    else {

        for (int i = 0; i < NUM_LOOPS; i++) {

            printf("Producer produced item %d\n", i);

            fflush(stdout);

            signal_semaphore(sem_set_id);

            usleep(500000);   // Delay for synchronization
        }

        // Wait for child process
        wait(NULL);

        // Remove semaphore
        //semctl(sem_set_id, 0, IPC_RMID, sem_val);

        printf("Semaphore removed successfully\n");
    }

    return 0;
}
```

## OUTPUT
$ ./sem.o 

<img width="488" height="599" alt="image" src="https://github.com/user-attachments/assets/f8207699-9b77-4518-b14a-7ac2c640ef74" />

$ ipcs

<img width="544" height="160" alt="image" src="https://github.com/user-attachments/assets/904d1409-b8ad-4009-bf66-4e8ec00d73b4" />


# RESULT:
The program is executed successfully.
