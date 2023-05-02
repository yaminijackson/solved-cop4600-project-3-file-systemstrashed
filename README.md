Download Link: https://assignmentchef.com/product/solved-cop4600-project-3-file-systems__trashed
<br>
<h2>Overview</h2>

Your cover in the Lizard Legion was blown, and you’ve been revealed as a double agent and driven out! It was all very “James Bond”, if you do say so yourself, and what a daring underground helicopter escape it was… but you feel lucky to have escaped with your skin. (Literally… they would have used you to make a “human suit”!) Now that you’re back on the “outside”, you’ve been tasked with creating a scheme to allow remaining resistance fighters still within the Lizard Legion to clandestinely move information back to your organization without raising suspicion. As of late, members of the Lizard Legion have discovered the PC classic “DOOM”, and it has become all the rage to build new mods for it at headquarters, so your team has decided to use mods for this title as a vehicle for exfiltration. By burying encrypted bits within textures and other game data blocks, information can be hidden within innocuous “WAD” (Where’s All the Data) files.




In this project, you will implement a userspace filesystem daemon using the FUSE (Filesystem in UserSpacE) API to access data in WAD format, the standard used in a number of classic PC game titles (including DOOM and Hexen). In this critical early prototype, you have been tasked with implementing read-only access to files and directories within the WAD files as a proof-of-concept. As such, you will need to implement open, read, and release functionality for both files and directories within your FUSE-based program. We, as your comrades-inarms battling the Reptilian invasion, will provide sample WAD files to demonstrate the functionality of your implementation. (The resistance is counting on you!) The resistance uses university courses as cover for standard operations, so you’ll submit the project via Canvas.

<h2>Structure</h2>

The project is broken into three main parts:




<ul>

 <li>Develop a library to read a WAD file and create a directory and file structure from it.</li>

 <li>Implement a userspace daemon (via FUSE) to access the directory structure once mounted.</li>

 <li>Test your implementation by navigating the mounted directory and examining the names and file contents.</li>

</ul>




While exact implementation may vary, the daemon’s parameters must match those laid out in this document, and the directory structure, naming, and file contents must be properly presented via the filesystem.




<h3>File and Directory Requirements</h3>

Your daemon must implement, at a minimum, the following filesystem functions to provide read-only access:




<ul>

 <li>Retrieving file and directory attributes</li>

 <li>Opening, reading, and releasing files</li>

 <li>Opening, reading, and releasing directories</li>

</ul>




Directories should be only executable and readable for all users, while files should be only readable by all users.

<strong> </strong>

<h3>File Format</h3>

The WAD file format contains information in three sections: the <strong><em>header</em></strong>, which gives basic layout information, the <strong><em>descriptors</em></strong>, which describe elements in the file, and the <strong><em>lumps</em></strong>, which contain the data themselves. <strong>NOTE</strong>: all numbers are in <u>little-Endian</u> format and, where applicable, are designated in <u>bytes</u>!




<u>File Header</u>

The header contains the file <strong><em>magic</em></strong>, descriptor count, and location (offset) of the descriptors in the file:




The magic for a wad file is usually ASCII and always ends in the suffix <strong>“WAD”</strong> (e.g., <strong>“IWAD”</strong> or <strong>“PWAD”</strong>).




<u>Descriptors</u>

The file’s descriptors contain information about elements in the WAD file – its file offset, length, and name:




Some elements have a length that is zero. These “marker” elements will be interpreted by the daemon as directories and should be displayed accordingly in the filesystem (see below).




<u>Lumps</u>

Elements in the WAD format are stored as “lumps” described by the descriptors. These lumps will be represented in the filesystem by the daemon as individual files that can be opened, read, and closed.




<u>Marker Elements</u>

There are two primary types of marker elements in WAD files, each of which should be interpreted as directories by our daemon. The type includes <u>map markers</u> and <u>namespace markers</u>.




Map marker names are of the format <strong>“E#M#”</strong>, where <strong>#</strong> represents a single decimal digit (e.g., <strong>“E1M9”</strong>). They are followed by ten (10) map element descriptors. The elements for the next 10 descriptors <u>should be</u> <u>placed inside of a directory</u> with the map’s name.




Namespace markers <u>come in pairs</u>. A namespace’s <em>beginning</em> is marked with a descriptor whose name has the suffix <strong>“_START”</strong> (e.g., <strong>“F1_START”</strong>), and its ending is marked with a descriptor whose name has the suffix <strong>“_END”</strong> (e.g., <strong>“F1_END”</strong>). Any descriptors for elements falling between the beginning and ending markers for a namespace should be placed within a directory with the namespace’s name (e.g., <strong>“F1”</strong>).




For example, the following descriptors, in order, in the descriptor list, should result in this organization:

<table width="551">

 <tbody>

  <tr>

   <td width="78"><strong>Offset </strong></td>

   <td width="72"><strong>Length </strong></td>

   <td width="90"><strong>Name </strong></td>

   <td rowspan="17" width="88">à</td>

   <td rowspan="2" width="224"><strong>Directory Structure </strong></td>

  </tr>

  <tr>

   <td width="78">0</td>

   <td width="72">0</td>

   <td width="90">F_START</td>

  </tr>

  <tr>

   <td width="78">0</td>

   <td width="72">0</td>

   <td width="90">F1_START</td>

   <td rowspan="15" width="224">FF1<span style="text-decoration: line-through;">  </span> E1M1<span style="text-decoration: line-through;">  </span> THINGS<span style="text-decoration: line-through;">  </span> LINEDEFS<span style="text-decoration: line-through;">  </span> SIDEDEFS<span style="text-decoration: line-through;">  </span> VERTEXES<span style="text-decoration: line-through;">  </span> SEGS<span style="text-decoration: line-through;">  </span> SSECTORS<span style="text-decoration: line-through;">  </span> NODES<span style="text-decoration: line-through;">  </span> SECTORS<span style="text-decoration: line-through;">  </span> REJECTBLOCKMAP         LOLWUT</td>

  </tr>

  <tr>

   <td width="78">67500</td>

   <td width="72">0</td>

   <td width="90">E1M1</td>

  </tr>

  <tr>

   <td width="78">67500</td>

   <td width="72">1380</td>

   <td width="90">THINGS</td>

  </tr>

  <tr>

   <td width="78">68880</td>

   <td width="72">6650</td>

   <td width="90">LINEDEFS</td>

  </tr>

  <tr>

   <td width="78">75532</td>

   <td width="72">19440</td>

   <td width="90">SIDEDEFS</td>

  </tr>

  <tr>

   <td width="78">94972</td>

   <td width="72">1868</td>

   <td width="90">VERTEXES</td>

  </tr>

  <tr>

   <td width="78">96840</td>

   <td width="72">8784</td>

   <td width="90">SEGS</td>

  </tr>

  <tr>

   <td width="78">105624</td>

   <td width="72">948</td>

   <td width="90">SSECTORS</td>

  </tr>

  <tr>

   <td width="78">106572</td>

   <td width="72">6608</td>

   <td width="90">NODES</td>

  </tr>

  <tr>

   <td width="78">113180</td>

   <td width="72">2210</td>

   <td width="90">SECTORS</td>

  </tr>

  <tr>

   <td width="78">115392</td>

   <td width="72">904</td>

   <td width="90">REJECT</td>

  </tr>

  <tr>

   <td width="78">116296</td>

   <td width="72">6922</td>

   <td width="90">BLOCKMAP</td>

  </tr>

  <tr>

   <td width="78">42</td>

   <td width="72">9001</td>

   <td width="90">LOLWUT</td>

  </tr>

  <tr>

   <td width="78">0</td>

   <td width="72">0</td>

   <td width="90">F1_END</td>

  </tr>

  <tr>

   <td width="78">0</td>

   <td width="72">0</td>

   <td width="90">F_END</td>

  </tr>

 </tbody>

</table>




<h3>Library</h3>

Your library will contain a class to represent WAD data as described in this section.




<u>Wad Class</u>

The Wad class is used to represent WAD data and should have the following functions. The root of all paths in the WAD data should be <strong>“/”</strong>, and each directory should be separated by <strong>‘/’</strong> (e.g., <strong>“/F/F1/LOLWUT”</strong>).




<em>public static </em><em>Wad* </em><strong>loadWad</strong>(const string &amp;path)

Object allocator; <u>dynamically</u> creates a <strong>Wad</strong> object and loads the WAD file data from <strong>path</strong> into memory. <u>Caller</u> must deallocate the memory using the <strong>delete</strong> keyword.




<em>public </em><em>string </em><strong>getMagic</strong>()

Returns the <strong><em>magic</em></strong> for this WAD data.




<em>public </em><em>bool </em><strong>isContent</strong>(const string &amp;path)

Returns <strong>true</strong> if <strong>path</strong> represents content (data), and <strong>false</strong> otherwise.




<em>public </em><em>bool </em><strong>isDirectory</strong>(const string &amp;path)

Returns <strong>true</strong> if <strong>path</strong> represents a directory, and <strong>false</strong> otherwise.




<em>public </em><em>int </em><strong>getSize</strong>(const string &amp;path)

If <strong>path</strong> represents content, returns the number of bytes in its data; otherwise, returns <strong>-1</strong>.




<em>public </em><em>int </em><strong>getContents</strong>(const string &amp;path, char *buffer, int length, int offset = 0)

If <strong>path</strong> represents content, copies as many bytes as are available, up to <strong>length</strong>, of content’s data into the preexisting <strong>buffer</strong>. If <strong>offset</strong> is provided, data should be copied starting from that byte in the content. Returns number of bytes copied into <strong>buffer</strong>, or <strong>-1</strong> if <strong>path</strong> does not represent content (e.g., if it represents a directory).




<em>public </em><em>int </em><strong>getDirectory</strong>(const string &amp;path, vector&lt;string&gt; *directory)

If <strong>path</strong> represents a directory, places entries for immediately contained elements in <strong>directory</strong>. The elements should be placed in the directory in the same order as they are found in the WAD file. Returns the number of elements in the directory, or <strong>-1</strong> if <strong>path</strong> does not represent a directory (e.g., if it represents content).

<strong> </strong>

<h3>Daemon Command &amp; Parameters</h3>

Your daemon should have name <strong>wadfs</strong> and should accept at a minimum two parameters – the target WAD file and mount directory. For example, this command should mount <strong>TINY.WAD</strong> in <strong>/home/reptilian/mountdir</strong>…




<strong>$</strong> <strong>./wadfs TINY.WAD /home/reptilian/mountdir </strong><strong>$</strong> <strong> </strong>

…and this should result from executing the <strong>ls</strong> command to show part of its contents:

<strong> </strong>

<strong>$</strong> <strong>ls /home/reptilian/mountdir/F/F1 -al </strong><strong>total 0 dr-xr-xr-x. 2 root root    0 Jan  1  1970 </strong><strong>.</strong> <strong>dr-xr-xr-x. 2 root root    0 Jan  1  1970 </strong><strong>..</strong> <strong>dr-xr-xr-x. 2 root root    0 Jan  1  1970 </strong><strong>E1M1</strong> <strong>-r–r–r–. 2 root root 9001 Jan  1  1970 LOLWUT</strong>




Your daemon should <u>run in the background</u>. <strong>Do not hard-code</strong> the debug flag (<strong>-d</strong>)!

<h2>Building with FUSE</h2>

FUSE is a userspace filesystem API that is supported directly by the Linux kernel. It allows userspace programs to provide information to the kernel about filesystems the kernel cannot interpret on its own.




<h3>Installation &amp; Setup</h3>

To use the FUSE library, you will need to install it within Reptilian and change the FUSE permissions:




<strong>$</strong> <strong>sudo apt install libfuse-dev </strong>

<strong>$</strong> <strong>sudo chmod 666 /dev/fuse </strong>




<strong>NOTE</strong>: if you reboot the virtual machine, you will need to re-add the FUSE permissions, as they will be reset!




<h3>Build Directives</h3>

In order to build programs using the FUSE library system, you will need to specify the file offset bits as 64 and identify the FUSE version. We recommend specifying FUSE version 26 (though this is optional):




<strong>$</strong> <strong>g++ -D_FILE_OFFSET_BITS=64 -DFUSE_USE_VERSION=26 myproggy.cpp -o myproggy -lfuse</strong>