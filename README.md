Download Link: https://assignmentchef.com/product/solved-smashing-lab
<br>
This assignment investigates an old-fashioned way of breaking into systems executing x86 machine code, along with a couple of machine-level defenses against this attack. It’s chosen not to give you a toolkit to break in to other sites – the method is well-known and ought to be commonly defended against nowadays – but instead, to give a general idea of how an attacker can break into a system by exploiting behavior that is undefined at the C level but defined at the machine level, and what we can do about it at the machine level.

<h2>Useful pointers</h2>

<ul>

 <li>Elias Levy a.k.a. Aleph One, <a href="http://insecure.org/stf/smashstack.html">Smashing the stack for fun and profit</a>, <em>Phrack</em> <strong>7</strong>, 49 (1996-11-08), file 14</li>

 <li>Jake Edge, <a href="https://lwn.net/Articles/584225/">“Strong” stack protection for GCC</a>, LWN.net (2014-02-05).</li>

 <li>Konstantin Serebryany, Derek Bruening, Alexander Potapenko, and Dmitry Vyukov, <a href="https://code.google.com/p/address-sanitizer/">AddressSanitizer: a fast memory error detector</a> (2012)</li>

</ul>

<h2>Keep a log</h2>

Keep a log in the file <samp>smashinglab.txt</samp> of what you do in the lab so that you can reproduce the results later. This should not merely be a transcript of what you typed: it should be more like a true lab notebook, in which you briefly note down what you did and what happened. The log should help us verify that you’ve done the steps listed below, so that we can in principle reproduce them. For example, if one of the steps says “get a backtrace”, make sure the backtrace goes into the log.

<h2>That’s a nice little program you got there. Shame if anything were to happen to it</h2>

Consider the following patch to <a href="http://opensource.dyc.edu/sthttpd">sthttpd</a>. This patch applies to <a href="sthttpd-2.27.0.tar.gz">sthttpd 2.27.0</a>, and deliberately introduces a bug into it. The idea is to suppose the the original programmer wrote this version by mistake.

<pre><samp>--- sthttpd-2.27.0/src/thttpd.c	2014-10-02 15:02:36.000000000 -0700+++ sthttpd-2.27.0-delta/src/thttpd.c	2015-04-30 19:15:24.820042000 -0700@@ -999,7 +999,7 @@ static void read_config( char* filename )     {     FILE* fp;-    char line[10000];+    char line[100];     char* cp;     char* cp2;     char* name;@@ -1012,7 +1012,7 @@ read_config( char* filename ) 	exit( 1 ); 	}-    while ( fgets( line, sizeof(line), fp ) != (char*) 0 )+    while ( fgets( line, 1000, fp ) != (char*) 0 ) 	{ 	/* Trim comments. */ 	if ( ( cp = strchr( line, '#' ) ) != (char*) 0 )</samp></pre>

<h2>Steps to illustrate and defend against the bug</h2>

<ol>

 <li>Make sure that <samp>/usr/local/cs/bin</samp> is at the start of your PATH; the command <samp>which gcc</samp> should output “<samp>/usr/local/cs/bin/gcc</samp>“.</li>

 <li>Build 32-bit sthttpd with this patch applied. Configure it with the shell command:<pre><samp>./configure    CFLAGS='-m32'    LDFLAGS="-Xlinker --rpath=/usr/local/cs/gcc-$(gcc -dumpversion)/lib"</samp></pre>Compile it three times, with the following sets of compiler options:

  <dl>

   <dt>

    (SP) for strong stack protection:

   </dt>

   <dd>

    <samp>-m32 -g3 -O2 -fno-inline -fstack-protector-strong</samp>

   </dd>

   <dt>

    (AS) for address sanitization:

   </dt>

   <dd>

    <samp>-m32 -g3 -O2 -fno-inline -fsanitize=address</samp>

   </dd>

   <dt>

    (NO) for neither:

   </dt>

   <dd>

    <samp>-m32 -g3 -O2 -fno-inline</samp>

   </dd>

  </dl>Call the resulting executables <samp>src/thttpd-sp</samp>, <samp>src/thttpd-as</samp>, and <samp>src/thttpd-no</samp>. You can do this with, for example, the command <samp>make clean</samp> followed by <samp>make CFLAGS=’-m32 -g3 -O2 -fno-inline -fstack-protector-strong’</samp> and then by <samp>mv src/thttpd src/thttpd-sp</samp> and similarly for the other two variants.</li>

 <li>Run each of the modified sthttpd daemons on port (12330 + 3 * (X % 293) + Y) on one of the SEASnet GNU/Linux servers, where X is your 9-digit student ID and Y is either 1, 2, or 3 depending on which variant of the daemon you’re running (1=SP, 2=AS, 3=NO). For example, if your student ID is 123-456-789, (12330 + 3 * (123456789 % 293) + 1) equals 12532, so you can run the command <samp>src/thttpd-sp -p 12532 -D</samp>; the <samp>-p</samp> option specifies the port number and the <samp>-D</samp> is for debugging. You may find the <a href="http://www.acme.com/software/thttpd/thttpd_man.html">thttpd man page</a> useful.</li>

 <li>Verify that your web servers work in the normal case. You can use the <samp>curl</samp> command to do this, e.g., the shell command <samp>curl http://localhost:12532/foo.txt</samp> where <samp>foo.txt</samp> is a text file in the working directory of your HTTPD server. The shell command <samp>man curl</samp> will give you more information about <samp>curl</samp>.</li>

 <li>Make variant SP crash by invoking it in a suitable way. Run it under GDB, and get a backtrace immediately after the crash. Identify which machine instruction caused the crash, and why.</li>

 <li>Make variant AS crash by invoking it in a similar way. Similarly, get a backtrace for it, and identify the machine instruction that crashed it and wy.</li>

 <li>Likewise for variant NO.</li>

 <li>Generate the assembly language code for <samp>thttpd.c</samp> three times, one for each variant, by using <samp>gcc -S</samp> rather than <samp>gcc -c -g3</samp> when compiling the file. (Use the same <samp>-O</samp> and <samp>-f</samp> flags as before.) Call the resulting files <samp>src/thttpd-sp.s</samp> and <samp>src/thttpd-as.s</samp> and <samp>src/thttpd-no.s</samp>. Compare the three assembly-language files’ implementations of the <samp>handle_read</samp> function. Describe the techniques used by <samp>-fstack-protector-strong</samp> and <samp>-fsanitize=address</samp> to prevent buffer-overrun exploits in <samp>handle_read</samp>.</li>

 <li>Build an exploit for the bug in variant NO that relies on the attacker tricking the victim into invoking <samp>thttpd</samp> with a particular value for the <samp>-C</samp> option. (Admittedly this is not much of an exploit, but we don’t want you to have to put more easily exploitable HTTP servers on our network.) Your exploit should cause the victim web server to remove the file <samp>target.txt</samp> in the working directory of the web server. Or, if such an exploit is impossible, explain why not, and investigate simple ways to alter the compiler flags (or, in the worst case, to insert plausible bugs into the source code) to make the exploit possible.</li>

</ol>

<h2>Submit</h2>

Submit the files <samp>smashinglab.txt</samp>, <samp>thttpd-sp.s</samp>, <samp>thttpd-as.s</samp>, and <samp>thttpd-no.s</samp> as described above, along with any other files needed to explain your exploit.

All text files should be ASCII files, with no carriage returns, and with no more than 200 columns per line. For example, the shell command

<pre><samp>expand smashinglab.txt thttpd-sp.s thttpd-as.s thttpd-no.s |  awk '/r/ || 200 &lt; length'</samp></pre>

should output nothing.

<hr>

<address> </address>