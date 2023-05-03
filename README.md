Download Link: https://assignmentchef.com/product/solved-cs332-lab1-file-management-system-calls
<br>
<strong>File Management System Calls  </strong>

<ul>

 <li>​A system call is a call to the operating system’s kernel requesting a service</li>

 <li>​Provides an interface between a user program (or a process) and the operating system</li>

 <li>​Can be viewed as special function calls</li>

 <li>​A typical system call can be grouped into one of the following categories:

  <ul>

   <li>​Process management</li>

  </ul></li>

</ul>

​<strong>– </strong>​File management

<ul>

 <li>​Directory management</li>

</ul>

<ul>

 <li>​Other system calls (Linux has over 300 different calls!), see ​<strong>man syscalls </strong></li>

 <li>​In this session, we will focus on system calls related to file management</li>

</ul>

access <strong>system call</strong>​ <strong>  </strong>

<ul>

 <li>​determines whether the calling process has access permission to a file</li>

 <li>​checks for read, write, execute permissions and file existence</li>

 <li>​takes two arguments

  <ul>

   <li>​argument 1: file path</li>

  </ul></li>

</ul>

​<strong>– </strong>​argument 2: ​R_OK, W_OK, and ​ ​X_OK, corresponding to read, write, and execute permission​

<ul>

 <li>​return value is 0, if the process has all the specified permissions</li>

 <li>​If the second argument is ​F_OK​, the call simply checks for file’s existence</li>

 <li>​If the file exists but the calling process does not have the specified permissions, access returns -1 and sets ​errno</li>

</ul>

<strong> </strong>

<strong>Example code snippet (incomplete) with system call (written in </strong><strong>C</strong>​ <strong>)</strong>​<strong>  </strong>

Snippet 1: check_file_permissions.c​

#include &lt;stdio.h&gt;

#include &lt;unistd.h&gt;  #include &lt;errno.h&gt;  int main (int argc, char* argv[])

{    char* filepath = argv[1];    int returnval;    // Check file existence    returnval = access (filepath, F_OK);    if (returnval == 0)      printf (“
 %s exists
”, filepath);    else    {      if (errno == ENOENT)        printf (“%s does not exist
”, filepath);      else if (errno == EACCES)        printf (“%s is not accessible
”, filepath);        return 0;

}




// Check read access …    // Check write access …    return 0;

}

<strong> </strong>

<strong>File Management System Calls  </strong>

<ul>

 <li>To​ create a new file, you use the <strong>creat</strong>​ system​ The named file is created as an empty file and opened for writing, and a positive integer, the open file identifier is returned. The file location is set to 0.</li>

 <li>In​ order to use a file, you first need to ask for it by name. This is called opening the file. The open​ system call creates an operating system object called an open file. The open file is logically connected to the file you named in the open system call. An open file has a file location (or file descriptor) associated with it and that is the offset in the file where the next read or write will start. The way in which the file is opened is specified by the flags argument: O_RDONLY​ to​ read; O_WRONLY​ to​ write; O_RDWR​ to​ both read and write; you can also do a bitwise OR with O_CREAT​ if​ you want the system to create the file if it doesn’t exist already (e.g., O_RDONLY​ |​ ​O_CREAT creates​ and open in read mode). The system call returns the file descriptor of the new file (or a negative number if there is an error, placing the code into errno​ ).​</li>

 <li>After​ you open a file, you can use the read​      or​           write​ ​system calls to read or write the open file. Each read or write system call increments a file location for a number of characters read or written. Thus, the file is read (or/and written) sequentially by default. It returns the number of bytes read or written, or -1 if it ran into an error.</li>

</ul>




<ul>

 <li>The ​ lseek ​    system call is used to achieve random access into the file since it changes the file​               location that will be used for the next read or write.</li>

 <li>You close the open file using the ​ close ​    system call, when you are done using it. A return code 0​  means the close succeeded.</li>

 <li>You delete the file from a directory using the ​ unlink ​  system call. A return code 0 means the unlink​       If an error occurs in trying to delete a file, a negative integer is returned.</li>

</ul>

Table 1: System calls for file management <strong>System Call Parameters</strong>​

<table width="561">

 <tbody>

  <tr>

   <td width="107"><strong>System Call </strong></td>

   <td width="123"><strong>Parameters </strong></td>

   <td width="65"><strong>Returns </strong></td>

   <td width="266"><strong>Comment </strong></td>

  </tr>

  <tr>

   <td width="107">open</td>

   <td width="123">name, flags</td>

   <td width="65">fd</td>

   <td width="266">Connect to open file</td>

  </tr>

  <tr>

   <td width="107">creat</td>

   <td width="123">name, mode</td>

   <td width="65">fd</td>

   <td width="266">Creates file and connect to open file</td>

  </tr>

  <tr>

   <td width="107">read</td>

   <td width="123">fd, buffer, count</td>

   <td width="65">count</td>

   <td width="266">Reads bytes from open file</td>

  </tr>

  <tr>

   <td width="107">write</td>

   <td width="123">fd, buffer, count</td>

   <td width="65">count</td>

   <td width="266">Write bytes from open file</td>

  </tr>

  <tr>

   <td width="107">lseek</td>

   <td width="123">fd, offset, whence</td>

   <td width="65">offset</td>

   <td width="266">Moves to position of next read or write</td>

  </tr>

  <tr>

   <td width="107">close</td>

   <td width="123">fd</td>

   <td width="65">code</td>

   <td width="266">Closes or Disconnect open file</td>

  </tr>

  <tr>

   <td width="107">unlink</td>

   <td width="123">name</td>

   <td width="65">code</td>

   <td width="266">Delete named file</td>

  </tr>

 </tbody>

</table>

<strong> </strong>

<strong>An example to open an existing file: open_file.c </strong>

#include &lt;stdio.h&gt;

#include &lt;sys/types.h&gt;

#include &lt;sys/stat.h&gt;

#include &lt;fcntl.h&gt;

#include &lt;string.h&gt;  #include &lt;errno.h&gt;  int main(int argc, char *argv[])

{     int fd;     if(argc != 2)

{        printf(“
 Usage : 
”);        return 1;

}    errno = 0;     fd = open(argv[1], O_RDONLY);     if(fd == -1)

{        printf(“
 open() failed with error [%s]
”,strerror(errno));       return 1;

}    else     {         printf(“
 Open() Successful
”);

/* open() succeeded, now one can do read operations on the        file opened since we opened it in read-only mode. Also once done         with processing, the file needs to be closed.*/

}    return 0;

}

The code above gives a basic usage of the open() function. Please note that this code does not show any read or close operations once the open() is successful. As mentioned in the comment in code, when done with the opened file, a close() should be called on the file descriptor opened. Now, let’s run the code and see the output: First we try with a file (say, sample.txt) present in the same directory from where the executable was run. (Note, for files that are not in the same directory, an absolute path needs to be specified).




<strong>Compile above C Code: </strong>

$ gcc open_file.c -o open

<ul>

 <li>This creates a binary executable file with the name <strong>open</strong>​ as we used -o option.​</li>

 <li>The -o option is used to specify the output(executable) file name. If we don’t use this option executable with name a.out would be generated.</li>

</ul>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong>Run the executable: </strong>$ ./open sample.txt

Open() Successful

<strong>How the system calls are processed? </strong>

<strong>Error codes  </strong>

<ul>

 <li>System calls occasionally run into a problem​</li>

 <li>Unix system calls handle this by returning some bad value (usually -1) and setting a global integer​ variable errno ​        to some value that indicates the problem​</li>

 <li>Subroutine ​ perror() ​ (​ it’s not a system call – it’s just a regular library function) that prints a mean- ingful error message to the standard error file So here’s how a typical segment might look.  <strong>fd = open(“filename”, O_RDONLY);  if(fd &lt; 0) /* ah, there’s an error */  </strong></li>

</ul>

<strong>{    printf(“sorry, I couldn’t open filename
”);    perror(“open”); /* This will explain why */    return;  </strong>

<strong>} </strong>

<strong> </strong>

<strong>TASK 1  </strong>

<strong> </strong>

<strong>DUE: Feb 18, 2020, 11:59 PM – 30 Points  </strong>

<strong>Feb 25, 2021, 11:59 PM – 20 Points  </strong>

<ol>

 <li>(a) Extend code snippet 1 to check for read and write access permissions of a given file</li>

</ol>

(b) Write a C program where open ​              ​system call creates a new file (say, destination.txt​             )​ and then  opens it. (Hint: use the bitwise OR flag)

<ol>

 <li>UNIX cat ​ command has three functions with regard to text files: displaying them, combining copies​   of them and creating new ones.</li>

</ol>

Write a C program to implement a command called <strong>printcontent</strong>​ that​ takes a (text) file name as argument and displays its contents. Report an appropriate message if the file does not exist or can’t be opened (i.e. the file doesn’t have read permission). You are to use open()​ ,​ read()​ ​, write()​ and​ close() system calls.​

<ol start="2">

 <li>The cp ​ command copies the source file specified by the SourceFile parameter to the destination file​             specified by the DestinationFile parameter.</li>

</ol>

Write a C program that mimics the cp​ command​ using open()​ system​ call to open source.txt​ file​ in read-only mode and copy the contents of it to destination.txt ​ using ​ read() ​ and ​ write() ​ system calls.​

<ol start="3">

 <li>Repeat part 2 (by writing a new C program) as per the following procedure:

  <ul>

   <li>Read the next 50 characters from txt​ , and among characters read, replace each char-​                      acter ’5’ with character ‘A’’ and all characters are then written in destination.txt​</li>

   <li>Write characters “XYZ” into file txt​</li>

  </ul></li>

</ol>

(​ c) Repeat the previous steps until the end of file source.txt​           . The last read step may not have 50​                        characters.




<strong>General Instructions: </strong>

Use <strong>perror</strong>​ ​ subroutine to display meaningful error messages in case of system call failures.​

Properly close the files using <strong>close</strong>​ system​ call after you finish read/write. Learn to use <strong>man</strong>​ pages​ to know more about file management system calls (e.g, man read​ ).​


