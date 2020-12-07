# System Programming 2020: BabyThread
## 0. The Goal
1. Understand and simulate the concept of context switch.
2. Understand and learn some system calls about signals.
3. Understand and learn to use setjmp(), longjmp().
4. Get points to pass this course :)

## 1. Problem Description
In this assignment, you need to simulate a user-thread library by using longjmp(), setjmp(), etc. For simplicity, we use a function to repersent a thread. In other words, you,ll have to "context switch" between functions. The "context switch" may occur in three different scenario: signal caught, timeslice reached, program finished a loop. we'll introduce the way to choose between three different scenarios later.

You are expected to complete the following tasks:
1. Complete the user-thread library "babythread.h".
2. Implement three functions: BlackholeNumber, BinarySearch, FibonacciSequence.
## 2. BabyThread.h
Please complete the half-done file BabyThread.h in this repository. 

## 3. Three Functions
The function of Baby 
The Function
```cpp=
#include  <stdio.h>
#include  <stdlib.h>
#include  <setjmp.h>
#include  <signal.h>

extern int timeslice;
extern int switchmode;

typedef struct TCB_NODE *TCB_ptr;
typedef struct TCB_NODE
{
	jmp_buf  Environment;
	int      Thread_id;
	TCB_ptr  Next;
	TCB_ptr  Prev;
	int i,N;
	int x,y,z;

}TCB;

jmp_buf MAIN;
jmp_buf SCHEDULER;

TCB_ptr Head;
TCB_ptr Current;
TCB_ptr Work;

struct sigaction act;
sigset_t base_mask, waiting_mask;
void sighandler(int signo)
{
   sigprocmask(SIG_SETMASK, &base_mask, NULL);
   if (signo == SIGTSTP)
   {
		printf("TSTP\n");
		int remainder = alarm(0);
		printf("remainder %d\n", remainder);
		if (remainder == 0)
			alarm(1);
		else
			alarm(remainder);
   }
   if (signo == SIGALRM)
   {
		printf("ALRM\n");
		printf("timeslice%d\n", timeslice);
		alarm(timeslice);
   }

   longjmp(SCHEDULER,1);

   
}







#define ThreadCreate(function,thread_id,init,maxiter)  \
{                                                      \
	if(setjmp(MAIN) == 0)                              \
		(function)(thread_id,init,maxiter);            \
}

#define ThreadInit(thread_id,init,maxiter)     \
{                                              \
	Work = (TCB_ptr) malloc(sizeof(TCB));      \
	Work->Thread_id = thread_id;               \
	if (Head == NULL)                          \
		Head = Work;                           \
	else                                       \
	{                                          \
		Current->Next = Work;                  \
		Work->Prev = Current;                  \
	}                                          \
	Work->Next = Head;                         \
	Head->Prev = Work;                         \
	Current = Work;                            \
	Current->N = maxiter;                      \
	Current->x = init;                         \
	if (setjmp(Work->Environment) == 0)        \
		longjmp(MAIN,1);                       \
}

#define ThreadJoin()                            \
{                                               \
	if (setjmp(Current->Environment) == 0)      \
		longjmp(SCHEDULER, 2);                  \
}

#define ThreadYield()                                                                          \
{                                                                                              \
	if(switchmode==0)                                                                          \
	{                                                                                          \
		if (setjmp(Current->Environment) == 0)                                                 \
			longjmp(SCHEDULER, 1);                                                             \
	}                                                                                          \
	else                                                                                       \
	{                                                                                          \
		sigpending(&waiting_mask);                                                             \
		if (sigismember(&waiting_mask, SIGTSTP) || sigismember(&waiting_mask, SIGALRM) )       \
		{                                                                                      \
			if (setjmp(Current->Environment) == 0)                                             \
				sigprocmask(SIG_UNBLOCK, &base_mask, NULL);                                    \
		}                                                                                      \
	}                                                                                          \
}



void Scheduler()
{
	
	int state = setjmp(SCHEDULER);
	printf("state %d\n", state);
	if(state == 0)
		longjmp(MAIN, 1);
	else if(state == 2)
	{
		printf("stop\n");
		if(Current == Current->Next)
		{
			printf("end\n");
			longjmp(MAIN, 2);
		}
		Current->Prev->Next = Current->Next;
		Current->Next->Prev = Current->Prev;

	}
	Current = Current->Next;
	printf("to_func%d\n", Current->Thread_id);
	longjmp(Current->Environment, 1);
}

```
## . Grading
TA will complie your with
1. (2 pt)Your thread library supports context switch by iteration count.
2. (2 pt)Your thread library supports context switch by time slice.
3. (2 pt)Your thread library supports context switch by signal caught.
4. (2 pt)You write functions (BlackholeNumber, BinarySearch, FibonacciSequence)on your own.


## . Submission


## 1. Problem Description
In class, we've learn the importance of Due to the coronavirus, people are now strongly recommended to wear a mask in public. To make the process of ordering masks go smoothly, you are expected to implement a simplified mask pre-order system: `csieMask`.

The `csieMask` system is composed of read and write servers, both can access a file `preorderRecord` that records infomation of consumer's order. When a server gets a request from clients, it will response according to the content of the file. A read server can tell the client how many masks can be ordered. A write server can modify the file to record the orders.

You are expected to complete the following tasks:
1. Implement the servers. The provided sample source code `server.c` can be compiled as simple read/write servers so you don't have to code them from stretch. Details will be described in the later part.
2. Modify the code so the servers will not be blocked by a single request, but can deal with many requests at the same time. You might want to use `select()` to implement the multiplexing system.
3. Guarantee the correctness of file content when it is being modified. You might want to use file lock to protect the file.

## 2. Running the Sample Servers

The provided sample code can be compiled as simple servers. 

**Compile**
You should **write your own makefile** to compile the code.
You can use the provided command to produce execution files first.
```bash=
$ gcc server.c -o write_server
$ gcc server.c -D READ_SERVER -o read_server
```

**Run**
The server will be listening to {port}
```bash=
$ ./write_server {port}
$ ./read_server {port}
```

The sample server handled the part of socket programming, so you are able to connect to the sample server once you run it. Your client and server can communicate to each other.

Feel free to modify any part of the code as you need, or implement your own server.


## 3. Testing your Servers at Client Side

```shell=
telnet {hostname} {port}
```
If your server is running on linux1.csie.ntu.edu.tw, and listening to port 3333, your command on the client side will be `telnet linux1.csie.ntu.edu.tw 3333`

If you are running sample servers, try to type something on the client side. You will see some message from server.

## 4. About the Preorder Record File 

The servers should be able to access the file `preorderRecord`.

The file includes 20 consumer structures, each with a consumer id (range from 902001 to 902020), and the number of masks each consumer can order.
A consumer can order 10 adult masks and 10 children masks in total. 

```cpp=
typedef struct {
    int id; //902001-902020
    int adultMask; //set to 10 by default
    int childrenMask; //set to 10 by default
} preorderRecord
```



## 5. Sample input and output

**Read Server**

A client can check how many masks they can order. 
Once it connect to a read server, the terminal will show the following:

```shell
Please enter your id (to check how many masks you can order):
```

If you type a id (for example, `902001`) on the client side, the server shall reply:


```shell
You can order 10 adult mask(s) and 10 children mask(s).
```
and close the connection from client.


But, if someone else is ordering using the same id, the server shall reply:
```shell
Locked.
```
and close the connection from client.


**Write Server**

A consumer can make preorders. It will first show how many masks a consumer can order just like a read server, then ask for masktype and number of mask to preorder.

Once it connect to a write server, the terminal will show the following:
```shell
Please enter your id (to check how many masks you can order):
```
If you type an id (for example, `902001`) on the client side, the server shall reply the numbers, following by a prompt on the nextline.

```shell
You can order 10 adult mask(s) and 10 children mask(s).
Please enter the mask type (adult or children) and number of mask you would like to order:
```
If you type a command (for example, `adult 2`), the server shall modify the `preorderRecord` file and reply:
```
Pre-order for 902001 successed, 2 adult mask(s) ordered.
```
and close the connection from client.

But, if someone else is ordering using the same id, the server shall reply:
```shell
Locked.
```
and close the connection from client.



## 6. Grading
1. Finish the Makefile. (1 points)
    * The servers can be produced successfully by `make` command.
2. A read server handles requests correctly. (1 point)
    * There would be only one request at a time for the subtask.
3. A write server handles requests correctly. (1 point)
    * There would be only one request at a time for the subtask.
4. Two requests issued to a read server. (1 point)
5. Two requests issued to a write server. (1.5 points)
6. Requests issued to write servers (1 point)
7. Requests issued to read servers and write servers (1.5 points)

## 7. Submission
Your assignment should be submitted to github before deadline. The submisstion should include at least two files:
1. Makefile
2. server.c 
You can submit other .c, .h files, as long as they can be compiled to two executable files named read_server and write_server by Makefile.

You are encouraged to submit a readme.txt, where you can briefly state how do you finish your program and something valuable you want to explain. 

## 8. Reminder
1. Plagiarism is STRICTLY prohibited.
3. Your credits will be deducted 10% for each day delay. A late submission is better than absence.
4. If you have any question, feel free to contact us via email ntucsiesp@gmail.com or come during TA hours.
5. Please start your work as soon as possible, do NOT leave it until the last day!
