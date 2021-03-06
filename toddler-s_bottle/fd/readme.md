#Objective:
http://pwnable.kr/play.php

fd - 1pt

Mommy! what is a file descriptor in Linux?

* try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link: https://www.youtube.com/watch?v=blAxTfcW9VU

ssh fd@pwnable.kr -p2222 (pw:guest)

#Solution

Starting off, it gives up an address and credentials to ssh in. To do that run the follwoing command in a terminal

```
ssh fd@pwnable.kr -p2222
```

When it asks you for a password, use guest (it gives it to you with (pw:guest))

Now when we establish the ssh conenction, we can see what files there are with the following command

```
ls
```

Now there are three files "flag", "fd", and "fd.c". We cannot read the flag file which probably holds the key, we have an executable "fd", and we have the c code for the exeuctable "fd.c"

Let's try to run the executable and see what happens. To do so run the following command. The program requires a number to be passed to it as an argument, so that is why there is a 2.

```
./fd 2
```

It tells us to just "learn about Linux file IO", so let's take a look at the source code.

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

Analyzing the code, there are two lines that stick out

```
int fd = atoi( argv[1] ) - 0x1234;
len = read(fd, buf, 32);
```

The first line imports the argument, converts it to a decimal and then subtracts 4460 (0x1234) from it
The second line uses the read command. Thing is, if fd is 0 in this case that will mean std_input which just asks for input. So we can set buf equal to "LETMEWIN" 
So to get the if then statement later on, we will need to set the argument to "4460" so it will set fd equal to 0, then we just need to type in "LETMEWIN" and it should pop out the flag. We can truncate the process and just type in the following command.

```
echo "LETMEWIN" | ./fd 4660
```

and thus we get the flag
