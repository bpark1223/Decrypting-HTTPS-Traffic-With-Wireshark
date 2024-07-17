# Decrypting-HTTPS-Traffic-With-Wireshark
<p>In this project, I will analyze malicious traffic. I will download a pcap file that contains https data, decrypt it, and identify the type of malware that was used to infect one of the systems on my network. The exercise file I will use is part of the github repository provided by "Hackersploit" on youtube.  </p>
<h2>Utilities Used</h2>
</p>- Ubuntu </p>
</p>- UTM VM setup </p>
</p>- Wireshark </p>
</p>- VirusTotal </p>
<h2>Step-by-Step Walkthrough</h2>
</p> I download the pcap file from Hackersploit's github repository and extract it. </p>
<img width="1235" alt="Screenshot 2024-07-16 at 6 30 27 PM" src="https://github.com/user-attachments/assets/6ce14c6f-edd1-4812-ad29-ea2a7fac981b">
</p> Once the extraction is done, I am provided with the Wireshark tutorial keys. I open the pcap file using Wireshark (I don't need to run Wireshark with sudo privileges to analyze pcap files)</p> 
<img width="802" alt="Screenshot 2024-07-16 at 6 34 06 PM" src="https://github.com/user-attachments/assets/9d9e404c-98b4-4904-ab43-9ff4b3208baf">
</p> I head over to preferences to add the same columns that I added to the root user profile (source/destination ports). </p>
<img width="1291" alt="Screenshot 2024-07-16 at 6 40 33 PM" src="https://github.com/user-attachments/assets/acb88f40-39d6-463a-b211-1c76942b38ca">
</p> The objective now is to decrypt a particular system on this network, identify the infection, and figure out what happened. I use the filter to display successful tls handshakes. In the context of TLS, a handshake message with type == 1 specifically refers to the ClientHello message. The ClientHello message is the first step in the TLS handshake process.
</p>
<img width="1440" alt="Screenshot 2024-07-16 at 6 45 28 PM" src="https://github.com/user-attachments/assets/34b05274-3979-4482-99d9-53fe3ba373d9">
</p> Opening the packets, I observe that they are all encrypted, given that HTTPS is secure.
<img width="1230" alt="Screenshot 2024-07-16 at 6 49 20 PM" src="https://github.com/user-attachments/assets/673feb82-802c-42a2-8875-e17378de3300">
</p> In this exercise, I already have the ssl keys to decrypt https/tls traffic. In order to decrypt, I navigate as follows edit -- preferences -- protocols -- TLS -- (Pre)-Master-Secret log filename -- Wireshark-tutorial-KeysLogFile.txt -- open -ok
<img width="1440" alt="Screenshot 2024-07-16 at 6 54 28 PM" src="https://github.com/user-attachments/assets/8936d198-14d7-4520-8119-35a825e1581a">
</p> If I use the same filter from earlier to display tls handshakes and follow the tls stream, I can now observe the post requests </p>
<img width="1440" alt="Screenshot 2024-07-16 at 6 58 18 PM" src="https://github.com/user-attachments/assets/b04c0a1f-3a3e-483a-9fbf-3898cd21dec3">
</p> Now, the goal is to see what system was infected and what malware caused the infection. To initiate this, I will use a filter to display http requests or tls handshakes, excluding ssdp. </p>
<img width="1440" alt="Screenshot 2024-07-16 at 7 35 52 PM" src="https://github.com/user-attachments/assets/f02cce08-9649-484b-b007-17a091e7bb0e">
Through this filter we can see both get requests and post requests, as well as tls handshakes. One packet that raises concern is a get request for a dll. In this particular case, the system was infected with the drydex malware which is a type of malware that affects financial institutions and the infection method is through office documents such as spreadsheets with custom macros. It downloads utilities that are then used to download the final piece of the malware, and in this case, we have a dll. 
<img width="1440" alt="Screenshot 2024-07-16 at 7 43 14 PM" src="https://github.com/user-attachments/assets/960c8549-1224-4f99-b0cf-9b788070e33d">
</p> If I follow the http stream of the suspicious packet, I can actually see what happened. For example, I can see the source ip that initated the get request, the destination server, and application data containing the contents of the actual dll ("This program cannot be run in DOS mode" text is present in all .dll files).
<img width="1440" alt="Screenshot 2024-07-16 at 7 50 21 PM" src="https://github.com/user-attachments/assets/abc19166-3437-45fe-859c-cd85add0e7fa">
</p> The next step would be to save the contents of this dll and analyze it through a utility. I save the file as an object. </p>
<img width="1440" alt="Screenshot 2024-07-16 at 8 01 53 PM" src="https://github.com/user-attachments/assets/8a6d5c51-e960-4d4a-9598-da3902a7f5e7">
<img width="1440" alt="Screenshot 2024-07-16 at 8 02 19 PM" src="https://github.com/user-attachments/assets/69163107-1a79-48d3-ae06-e3fd119ad02c">
</p> Then, I navigate to virustotal.com to identify what type of malware this is. I drop the file in and after a few seconds, the tool gives specific details such as the original dll, the time at which the malware was created, etc. "execution parents" refer to the processes or files that are responsible for launching or executing a particular file or program. </p>
<img width="1440" alt="Screenshot 2024-07-16 at 8 06 08 PM" src="https://github.com/user-attachments/assets/a54bf6a1-166f-422e-a98d-01b32b01616f">
<img width="1440" alt="Screenshot 2024-07-16 at 8 33 02 PM" src="https://github.com/user-attachments/assets/33aa5296-6b9d-4889-a573-90aa324ce830">
</p> From the provided information, we can conclude that a user on the network downloaded the document (presumably through an email), opened it up with Excel, and executed the macro which initiated the code to download a dll, ultimately infecting the system. </p>
</p> Navigating back to Wireshark, there is a suspicious post request with an unusual source port that was made to a file called /docs.php HTTP/1.1. Through this, I can conclude that after the infection, there is a connection made to the command and control server being operated by the attacker. If I follow the tls stream, I can confirm that this is the c2 server (as shown where it says mitm proxy) 
<img width="1440" alt="Screenshot 2024-07-16 at 8 46 21 PM" src="https://github.com/user-attachments/assets/6fa354b4-1045-46b9-9354-fa1f1dc5bbee">
<img width="1440" alt="Screenshot 2024-07-16 at 8 48 16 PM" src="https://github.com/user-attachments/assets/7f029978-59f6-434b-92a0-0b94394d2e2e">
</p> In this project, I was able to https traffic and identify the type of malware that caused the infection as well as what system was infected. 
