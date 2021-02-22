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
[![Contributors][contributors-shield]][contributors-url]
[![MIT License][license-shield]][license-url]
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
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

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

<!-- ROADMAP -->
## Roadmap

See the [open issues](https://github.com/othneildrew/Best-README-Template/issues) for a list of proposed features (and known issues).



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

Your Name - [@your_twitter](https://twitter.com/your_username) - email@example.com

Project Link: [https://github.com/your_username/repo_name](https://github.com/your_username/repo_name)



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
[contributors-shield]: https://img.shields.io/github/contributors/othneildrew/Best-README-Template.svg?style=for-the-badge
[contributors-url]: https://github.com/othneildrew/Best-README-Template/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/othneildrew/Best-README-Template.svg?style=for-the-badge
[forks-url]: https://github.com/othneildrew/Best-README-Template/network/members
[stars-shield]: https://img.shields.io/github/stars/othneildrew/Best-README-Template.svg?style=for-the-badge
[stars-url]: https://github.com/othneildrew/Best-README-Template/stargazers
[license-shield]: https://img.shields.io/github/license/othneildrew/Best-README-Template.svg?style=for-the-badge
[license-url]: https://github.com/othneildrew/Best-README-Template/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/sylvester-a-adelaja-954918124/
[product-screenshot]: images/2021-02-22 20_33_43-Adelaja_Akolade_Buffer_Overflow_Report.docx - Word.png
