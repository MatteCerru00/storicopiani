# storicopiani

<!DOCTYPE html>
<html lang="en-US"><head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="/assets/css/style.css?v=e5634ee13f080061920e82af9aa2110b3f99dce5">

<!-- Begin Jekyll SEO tag v2.7.1 -->
<title>Calcolatori Elettronici - Interprocess Communication (IPC) | Prof. Alessandro Armando</title>
<meta name="generator" content="Jekyll v3.9.0">
<meta property="og:title" content="Calcolatori Elettronici - Interprocess Communication (IPC)">
<meta property="og:locale" content="en_US">
<link rel="canonical" href="/ipc.html">
<meta property="og:url" content="/ipc.html">
<meta property="og:site_name" content="Prof. Alessandro Armando">
<meta name="twitter:card" content="summary">
<meta property="twitter:title" content="Calcolatori Elettronici - Interprocess Communication (IPC)">
<script type="application/ld+json">
{"headline":"Calcolatori Elettronici - Interprocess Communication (IPC)","url":"/ipc.html","@type":"WebPage","@context":"https://schema.org"}</script>
<!-- End Jekyll SEO tag -->

  </head>

  <body>

    <header>
      <div class="container">
        <a id="a-title" href="/">
          <h1>Prof. Alessandro Armando</h1>
        </a>
        <h2></h2>

        <section id="downloads">
          
          <a href="https://github.com/alessandroarmando/alessandroarmando.github.io" class="btn btn-github"><span class="icon"></span>View on GitHub</a>
        </section>
      </div>
    </header>

    <div class="container">
      <section id="main_content">
        <h1 id="calcolatori-elettronici---interprocess-communication-ipc">Calcolatori Elettronici - Interprocess Communication (IPC)</h1>

<h1 id="1-introduction">1. Introduction</h1>

<p>Interprocess communication (IPC) is the transfer of data among processes.</p>

<p>We considere five types of interprocess communication:</p>
<ul>
  <li><em>Shared memory</em> permits processes to communicate by simply reading and
writing to a specified memory location.</li>
  <li><em>Mapped memory</em> is similar to shared memory, except that it is associated with a
file in the filesystem.</li>
  <li><em>Pipes</em> permit sequential communication from one process to a related process.</li>
  <li><em>FIFOs</em> are similar to pipes, except that unrelated processes can communicate
because the pipe is given a name in the filesystem.</li>
  <li><em>Sockets</em> support communication between unrelated processes even on different
computers.</li>
</ul>

<p>These types of IPC differ by the following criteria:</p>
<ul>
  <li>Whether they restrict communication to related processes (processes with a
common ancestor), to unrelated processes sharing the same filesystem, or to any
computer connected to a network</li>
  <li>Whether a communicating process is limited to only write data or only
read data</li>
  <li>The number of processes permitted to communicate</li>
  <li>Whether the communicating processes are synchronized by the IPC—for
example, a reading process halts until data is available to read</li>
</ul>

<h2 id="11-shared-memory">1.1. Shared Memory</h2>

<p>Shared memory allows two or more processes to access the same memory as if they all
called malloc and were returned pointers to the same actual memory.</p>

<p>Key facts:</p>
<ul>
  <li>When one process changes the memory, all the other processes see the
modification.</li>
  <li>Shared memory is the fastest form of interprocess communication because all
processes share the same piece of memory.</li>
  <li>Access to this shared memory is as fast as accessing a process’s nonshared memory, and it does not require a system call or entry
to the kernel.</li>
  <li>It also avoids copying data unnecessarily.</li>
  <li>Kernel does not synchronize accesses to shared memory: you must provide your own synchronization (via, e.g., semaphores).</li>
</ul>

<h3 id="111-the-memory-model">1.1.1. The Memory Model</h3>

<p>To use a shared memory segment, one process must allocate the segment. Then each
process desiring to access the segment must attach the segment. After finishing its use
of the segment, each process detaches the segment. At some point, one process must
deallocate the segment.</p>

<p>Under Linux, each process’s virtual memory is split into pages. Each
process maintains a mapping from its memory addresses to these virtual memory pages,
which contain the actual data. Even though each process has its own addresses, multiple
processes’ mappings can point to the same page, permitting sharing of memory.</p>

<p>Allocating a new shared memory segment causes virtual memory pages to be created.
Because all processes desire to access the same shared segment, only one process
should allocate a new shared segment.</p>

<p>Allocating an existing segment does not create
new pages, but it does return an identifier for the existing pages.</p>

<p>To permit a process to use the shared memory segment, a process attaches it, which adds entries mapping
from its virtual memory to the segment’s shared pages.</p>

<p>When finished with the segment, these mapping entries are removed.</p>

<p>When no more processes want to access these shared memory segments, exactly one process must deallocate the virtual
memory pages.</p>

<p>All shared memory segments are allocated as integral multiples of the system’s page
size, which is the number of bytes in a page of memory. On Linux systems, the page
size is 4KB, but you should obtain this value by calling the <code class="language-plaintext highlighter-rouge">getpagesize</code> function.</p>

<h3 id="112-allocation">1.1.2. Allocation</h3>

<p>A process allocates a shared memory segment using <code class="language-plaintext highlighter-rouge">shmget</code> (“SHared Memory
GET”):</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int shmget(key_t key, size_t size, int shmflg);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">key</code> is an integer key that specifies which segment to create.
Unrelated processes can access the same shared segment by specifying the same key
value. If other processes choose the same fixed key, then a conflict arises. Using the special constant IPC_PRIVATE as the key value guarantees that a brand new memory segment is created.</li>
  <li><code class="language-plaintext highlighter-rouge">size</code> specifies the number of bytes in the segment. Because segments are allocated using pages, the number of actually allocated bytes is rounded up
to an integral multiple of the page size.</li>
  <li><code class="language-plaintext highlighter-rouge">shmflg</code> specifies options (usually expressed as bitwise OR of predefined flag values).</li>
</ul>

<p>For example, the following invocation of <code class="language-plaintext highlighter-rouge">shmget</code> creates a new shared memory segment (or
access to an existing one, if <code class="language-plaintext highlighter-rouge">shm_key</code> is already used) that’s readable and writeable to
the owner but not other users.</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int segment_id = shmget(shm_key, getpagesize(), IPC_CREAT|S_IRUSR|S_IWUSER);
</code></pre></div></div>

<p>If the call succeeds, <code class="language-plaintext highlighter-rouge">shmget</code> returns a segment identifier. If the shared memory segment
already exists, the access permissions are verified and a check is made to ensure that
the segment is not marked for destruction.</p>

<h3 id="113-attachment-and-detachment">1.1.3 Attachment and Detachment</h3>

<p>To make the shared memory segment available, a process must use <code class="language-plaintext highlighter-rouge">shmat</code>, “SHared
Memory ATtach”:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>void *shmat(int shmid, const void *shmaddr, int shmflg);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">shmid</code> is the shared memory segment identifier returned by
<code class="language-plaintext highlighter-rouge">shmget</code>.</li>
  <li><code class="language-plaintext highlighter-rouge">shmaddr</code> is a pointer that specifies where in your process’s address
space you want to map the shared memory; if you specify NULL, Linux will choose
an available address.</li>
  <li><code class="language-plaintext highlighter-rouge">shmflg</code> specifies options</li>
</ul>

<p>If the call succeeds, it returns the address of the attached shared segment.</p>

<p>When you’re finished with a shared memory segment, the segment should be
detached using <code class="language-plaintext highlighter-rouge">shmdt</code> (“SHared Memory DeTach”):</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int shmdt(const void *shmaddr);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">shmaddr</code> is the address returned by <code class="language-plaintext highlighter-rouge">shmat</code>.</li>
</ul>

<p>If the segment has been detached and this was the last process using it, it is
removed.</p>

<h3 id="example-simple">Example: Simple</h3>
<p>The following <a href="/code/shm.c">program</a> makes use of shared memory.</p>

<h3 id="example-client-server">Example: Client-Server</h3>
<p>The implementation of the following <a href="/code/shmclient.c">client</a> and <a href="/code/shmserver.c">server</a> makes also use of shared memory.</p>

<h2 id="12-mapped-memory">1.2. Mapped Memory</h2>

<p><em>Mapped memory</em> permits different processes to communicate via a shared file.</p>

<p>Although you can think of mapped memory as using a shared memory segment
with a name, there are technical differences.</p>

<p>Mapped memory can be used for interprocess communication or as an easy way to access
the contents of a file.</p>

<p>Mapped memory forms an association between a file and a process’s memory.</p>

<p>Linux splits the file into page-sized chunks and then copies them into virtual memory
pages so that they can be made available in a process’s address space.</p>

<p>Thus, the process can read the file’s contents with ordinary memory access.
It can also modify the file’s contents by writing to memory.</p>

<p>This permits fast access to files.</p>

<p>You can think of mapped memory as allocating a buffer to hold a file’s entire contents, and then reading the file into the buffer and (if the buffer is modified) writing
the buffer back out to the file afterward.</p>

<p>Linux handles the file reading and writing operations for you.</p>

<h3 id="mapping-an-ordinary-file">Mapping an Ordinary File</h3>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">addr</code> is the address at which you would like Linux to map the file into your process’s address space; the value NULL allows Linux
to choose an available start address.</li>
  <li><code class="language-plaintext highlighter-rouge">length</code> is the length of the map in bytes.</li>
  <li><code class="language-plaintext highlighter-rouge">prot</code> specifies the protection on the mapped address range (bitwise “or” of <code class="language-plaintext highlighter-rouge">PROT_READ</code>, <code class="language-plaintext highlighter-rouge">PROT_WRITE</code>, and <code class="language-plaintext highlighter-rouge">PROT_EXEC</code>).</li>
  <li><code class="language-plaintext highlighter-rouge">flags</code> is a flag value that specifies additional options.</li>
  <li><code class="language-plaintext highlighter-rouge">fd</code> is a file descriptor opened to the file to be mapped.</li>
  <li><code class="language-plaintext highlighter-rouge">offset</code> is the offset from the beginning of the file from which to start the map.</li>
</ul>

<p>You can map all or part of the file into memory by choosing the starting offset and length appropriately.</p>

<p>When you’re finished with a memory mapping, release it by using <code class="language-plaintext highlighter-rouge">munmap</code>:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int munmap(void *addr, size_t length);
</code></pre></div></div>

<h3 id="example">Example:</h3>
<p>The first program <a href="/code/02_mmap-write.c">02_mmap-write.c</a> generates a random number and writes it to a memory-mapped file.</p>

<p>The second program <a href="/code/02_mmap-read.c">02_mmap-read.c</a> reads the number, prints it, and replaces it in the memory-mapped file with double the value.</p>

<p>Both take a command-line argument of the file to map.</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&gt; ./02_mmap-write.exe integer-file
&gt; cat integer-file
15
&gt; ./02_mmap-read.exe integer-file
value:15
&gt; cat integer-file
30
</code></pre></div></div>

<h2 id="13-pipes">1.3. Pipes</h2>

<p>A pipe is a communication device that permits unidirectional communication:</p>
<ul>
  <li>Data written to the “write end” of the pipe is read back from the “read end.”</li>
  <li>Pipes are serial devices: the data is always read from the pipe in the same order it was written.</li>
  <li>Typically, a pipe is used to communicate between two threads in a single process or
between parent and child processes.</li>
</ul>

<p>In a shell, the symbol <code class="language-plaintext highlighter-rouge">|</code> creates a pipe. For example, this shell command causes the shell to produce two child processes, one for <code class="language-plaintext highlighter-rouge">ls</code> and one for <code class="language-plaintext highlighter-rouge">less</code>:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&gt; ls | less
</code></pre></div></div>
<p>The shell also creates a pipe connecting the standard output of the <code class="language-plaintext highlighter-rouge">ls</code> subprocess with
the standard input of the <code class="language-plaintext highlighter-rouge">less</code> process.</p>

<p>The filenames listed by <code class="language-plaintext highlighter-rouge">ls</code> are sent to less in exactly the same order as if they were sent directly to the terminal.</p>

<p>A pipe’s data capacity is limited:</p>
<ul>
  <li>If the writer process writes faster than the reader
process consumes the data, and if the pipe cannot store more data, the writer process
blocks until more capacity becomes available.</li>
  <li>If the reader tries to read but no data is
available, it blocks until data becomes available.</li>
</ul>

<p>Thus, the pipe automatically synchronizes the two processes.</p>

<h3 id="131-creating-pipes">1.3.1. Creating Pipes</h3>

<p>To create a pipe, invoke the <code class="language-plaintext highlighter-rouge">pipe</code> command. Supply an integer array of size 2.
The call to <code class="language-plaintext highlighter-rouge">pipe</code> stores the reading file descriptor in array position 0 and the writing file
descriptor in position 1.</p>

<p>For example, consider this code:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int pipe_fds[2];
int read_fd, int write_fd;

pipe(pipe_fds);
read_fd = pipe_fds[0];
write_fd = pipe_fds[1];
</code></pre></div></div>

<p>Data written to the file descriptor <code class="language-plaintext highlighter-rouge">read_fd</code> can be read back from <code class="language-plaintext highlighter-rouge">write_fd</code>.</p>

<h3 id="132-communication-between-parent-and-child-processes">1.3.2. Communication Between Parent and Child Processes</h3>

<p>A call to pipe creates file descriptors, which are valid only within that process and its
children.</p>

<p>A process’s file descriptors cannot be passed to unrelated processes; however,
when the process calls <code class="language-plaintext highlighter-rouge">fork</code>, file descriptors are copied to the new child process.</p>

<p>Thus, pipes can connect only related processes.</p>

<h3 id="example-1">Example</h3>
<p>In the program <a href="/code/04_pipe.c">04_pipe.c</a>, a fork spawns a child process.</p>

<p>The child inherits the pipe file descriptors.</p>

<p>The parent writes a string to the pipe, and the child reads it out.</p>

<p>The sample program converts these file descriptors into <code class="language-plaintext highlighter-rouge">FILE*</code> streams using <code class="language-plaintext highlighter-rouge">fdopen</code>.
This allows for the use of higher-level standard C library I/O functions such as <code class="language-plaintext highlighter-rouge">printf</code> and <code class="language-plaintext highlighter-rouge">fgets</code>.</p>

<h2 id="14-fifos-named-pipes">1.4. FIFOs (Named Pipes)</h2>

<p>A first-in, first-out (FIFO) file is a pipe that has a name in the filesystem.</p>

<p>Any process can open or close the FIFO.</p>

<p>The processes on either end of the pipe need not be related to each other.</p>

<p>FIFOs are also called named pipes.</p>

<p>You can make a FIFO using the <code class="language-plaintext highlighter-rouge">mkfifo</code> command.</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&gt; mkfifo /tmp/fifo
&gt; ls -la /tmp/fifo
prw-rw-r-- 1 armando armando 0 mag 10 10:28 /tmp/fifo
</code></pre></div></div>

<p>In one window, read from the FIFO by invoking the following:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&gt; cat &lt; /tmp/fifo
</code></pre></div></div>

<p>In a second window, write to the FIFO by invoking this:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&gt; cat &gt; /tmp/fifo
</code></pre></div></div>

<p>Then type in some lines of text.</p>

<p>Each time you press Enter, the line of text is sent through the FIFO and appears in the first window.</p>

<p>Close the FIFO by sending a EOF (press Ctrl+D) in the second window.</p>

<p>Remove the FIFO with this line:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&gt; rm /tmp/fifo
</code></pre></div></div>

<h3 id="141-creating-a-fifo">1.4.1. Creating a FIFO</h3>

<p>Create a FIFO programmatically using the <code class="language-plaintext highlighter-rouge">mkfifo</code> function:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int mkfifo(const char *pathname, mode_t mode);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">pathname</code> is the path at which to create the FIFO;</li>
  <li>mode specifies the pipe’s owner, group, and world permissions. (Since a pipe must have a reader and a writer, the permissions must include both read and write permissions.)</li>
</ul>

<h3 id="142-accessing-a-fifo">1.4.2. Accessing a FIFO</h3>

<p>Access a FIFO just like an ordinary file.</p>

<p>To communicate through a FIFO, one program must open it for writing, and another program must open it for reading.</p>

<p>For example, to write a buffer of data to a FIFO using low-level I/O routines, you
could use this code:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int fd = open (fifo_path, O_WRONLY);
write (fd, data, data_length);
close (fd);
</code></pre></div></div>

<p>To read a string from the FIFO using C library I/O functions, you could use
this code:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>FILE* fifo = fopen (fifo_path, “r”);
fscanf (fifo, “%s”, buffer);
fclose (fifo);
</code></pre></div></div>
<p>A FIFO can have multiple readers or multiple writers. Bytes from each writer are
written atomically up to a maximum size of <code class="language-plaintext highlighter-rouge">PIPE_BUF</code> (4KB on Linux). Similar rules apply to simultaneous reads.</p>

<h2 id="15-sockets">1.5. Sockets</h2>

<p>A socket is a bidirectional communication device that can be used to communicate with
another process on the same machine or with a process running on other machines.</p>

<p>Internet programs such as Telnet, rlogin, FTP, talk, and the World Wide Web use sockets.</p>

<p>For example, you can obtain the WWW page from a Web server using the
Telnet program because they both use sockets for network communications. 4
To open a connection to a WWW server at <code class="language-plaintext highlighter-rouge">www.codesourcery.com</code>, use</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&gt; telnet www.codesourcery.com 80
Trying 107.23.79.96...
Connected to www-redirect.mentor-esd.com.
Escape character is '^]'.
GET
&lt;!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN"&gt;
&lt;html&gt;&lt;head&gt;
&lt;title&gt;301 Moved Permanently&lt;/title&gt;
&lt;/head&gt;&lt;body&gt;
&lt;h1&gt;Moved Permanently&lt;/h1&gt;
&lt;p&gt;The document has moved &lt;a href="http://www.mentor.com/"&gt;here&lt;/a&gt;.&lt;/p&gt;
&lt;/body&gt;&lt;/html&gt;
Connection closed by foreign host.
</code></pre></div></div>

<h3 id="151-system-calls">1.5.1. System Calls</h3>

<p>These are the system calls involving sockets:</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">socket</code>: Creates a socket</li>
  <li><code class="language-plaintext highlighter-rouge">close</code>: Destroys a socket</li>
  <li><code class="language-plaintext highlighter-rouge">connect</code>: Creates a connection between two sockets</li>
  <li><code class="language-plaintext highlighter-rouge">bind</code>: Labels a server socket with an address</li>
  <li><code class="language-plaintext highlighter-rouge">listen</code>: Configures a socket to accept conditions</li>
  <li><code class="language-plaintext highlighter-rouge">accept</code>: Accepts a connection and creates a new socket for the connection</li>
</ul>

<p>Sockets are represented by file descriptors.</p>

<p>The <code class="language-plaintext highlighter-rouge">socket</code> and <code class="language-plaintext highlighter-rouge">close</code> functions create and destroy sockets, respectively:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int socket(int domain, int type, int protocol);
int close(int fd);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">domain</code> specifies a communication domain (e.g. <code class="language-plaintext highlighter-rouge">AF_UNIX</code> and <code class="language-plaintext highlighter-rouge">AF_LOCAL</code> for local communication and <code class="language-plaintext highlighter-rouge">AF_INET</code> for Internet Protocols), i.e. the protocol family which will be used for communication.</li>
  <li><code class="language-plaintext highlighter-rouge">type</code> specifies the communication semantics (e.g. <code class="language-plaintext highlighter-rouge">SOCK_STREAM</code> for sequenced, reliable, two-way, connection-based byte streams and <code class="language-plaintext highlighter-rouge">SOCK_DGRAM</code> fr datagrams, i.e connectionless, unreliable messages of a fixed maximum length).</li>
  <li><code class="language-plaintext highlighter-rouge">protocol</code> specifies a particular protocol to be used with the socket.  Normally only a single protocol exists to support a particular socket type within a given protocol family, in which case protocol can be specified as 0.</li>
</ul>

<p>To create a connection between two sockets, the client calls <code class="language-plaintext highlighter-rouge">connect</code>:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">sockfd</code> is the the file description of the socket</li>
  <li><code class="language-plaintext highlighter-rouge">addr</code> is the address of the server</li>
  <li><code class="language-plaintext highlighter-rouge">addrlen</code> specifies the size of addr</li>
</ul>

<p>The client is the process initiating the connection and the server is the process waiting to accept connections.</p>

<p>The client calls <code class="language-plaintext highlighter-rouge">connect</code> to initiate a connection from a local socket to the server socket whose address is <code class="language-plaintext highlighter-rouge">addr</code>.</p>

<p>A server’s life cycle consists of</p>
<ul>
  <li>the creation of a connection-style socket,</li>
  <li>binding an address to its socket,</li>
  <li>placing a call to listen that enables connections to the socket,</li>
  <li>placing calls to accept incoming connections, and then</li>
  <li>closing the socket.</li>
</ul>

<p>An address must be bound to the server’s socket using <code class="language-plaintext highlighter-rouge">bind</code>:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code> int bind(int sockfd, const struct sockaddr *addr,
          socklen_t addrlen);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">sockfd</code> is the socket file descriptor,</li>
  <li><code class="language-plaintext highlighter-rouge">addr</code> is a pointer to a socket address structure</li>
  <li><code class="language-plaintext highlighter-rouge">addrlen</code> is the length of the address structure, in bytes.</li>
</ul>

<p>When an address is bound to a connection-style socket, it must invoke <code class="language-plaintext highlighter-rouge">listen</code> to indicate that it is a server:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int listen(int sockfd, int backlog);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">sockfd</code> is the socket file descriptor,</li>
  <li><code class="language-plaintext highlighter-rouge">backlog</code> specifies how many pending connections are queued. If the queue is full, additional connections will be rejected.  (This does not limit the total number of connections that a server can handle; it limits just the number of clients attempting to connect that have not yet
been accepted.)</li>
</ul>

<p>A server accepts a connection request from a client by invoking <code class="language-plaintext highlighter-rouge">accept</code>:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
</code></pre></div></div>
<p>where</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">sockfd</code> is the socket file descriptor,</li>
  <li><code class="language-plaintext highlighter-rouge">addr</code> is a pointer to a socket address structure</li>
  <li><code class="language-plaintext highlighter-rouge">addrlen</code> is the length of the address structure, in bytes.</li>
</ul>

<p>The server can use the client address to determine whether it really wants to communicate with the client.</p>

<p>The call to <code class="language-plaintext highlighter-rouge">accept</code> creates a new socket for communicating with the client and returns the corresponding file descriptor.</p>

<p>The original server socket continues to accept new client connections.</p>

<h3 id="example-2">Example</h3>
<p>Server <a href="/code/07_socket_server.c">07_socket_server.c</a> and client <a href="/code/07_socket_client.c">07_socket_client.c</a> using sockets.</p>

      </section>
    </div>

    
  

</body>
</html>
