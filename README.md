<h1>Forensics Smartphone RAM Analysis App</h1>

<h3>Objective</h3>
The aim of this project is to find the user's smartphone unlock password of a provided memory image.
<br /> <br />
<h3>Skills Learned</h3>
● Reverse engineering of smartphones <br />
● Brute-force password <br />

<br />
<h3>Tools Used</h3>
● Volatility <br />
● Hashcat <br />
<br />

<h3>Steps</h3>
First we downloaded the provided zip called SDM_RAM. Once opened, in the SDM_RAM folder we found the file i9100-CM.bin and another zip i91000-CM_3.0.64, that contained the files module.dwarf and System.map. As this files will be useful for the analysis, we stored them in a separate folder.
<br />
<br />
For creating the profile, we followed these steps:
<br />
We created a zip with the module.dwarf and System.map files.
<br /> 
<img width="587" alt="smart1" src="https://github.com/user-attachments/assets/ef11d8e0-3cdd-4f52-b536-acc90f27558f">
<br />
<br />
*Ref 1: Zip file with module.dwarf and system.map*
<br /> <br />
Afterwards, we copied the file to the Volatility source tree to the path 
<br />
"/home/kali/Downloads/volatility-master/volatility/plugins/overlays/linux/"
<br />
<br />
To make sure that the profile was created correctly we looked for it in the profiles section of the Volatility help output.
<br />
<img width="503" alt="smart2" src="https://github.com/user-attachments/assets/0b599d57-2df9-442a-aa19-b8ca880e247f">
<br />
*Ref 2: Profile already created Linuxram_analysisARM*
<br /> <br />
To extract the system files we use the linux_recover_filesystem plugin
<br />
"python2.7 vol.py imageinfo -f /home/kali/Downloads/SDM_RAM/SDM_RAM/i9100-CM.bin --profile=Linuxram_analysisARM linux_recover_filesystem --dump-dir=/home/kali/Downloads/SDM_RAM/SDM_RAM/"
<br />
<br />
The output should be the system files.
In all the files contained in the dump, we search for something related to passwords and find this file:
<br />
<img width="585" alt="smart3" src="https://github.com/user-attachments/assets/fc74422c-cdf0-41ab-abe6-c3119efac56f">
<br />
*Ref 3: password.key file*
<br /> <br />
For displaying the text we use the command cat.
<br />
<img width="583" alt="smart4" src="https://github.com/user-attachments/assets/4428b94b-88ee-4ae0-8474-3f8b5b9b6268">
<br />
*Ref 4: cat command to display text*
<br /> <br />
This hexadecimal string (72 bytes) seems to be the concatenation of the sha1(password + salt) and md5 (password + salt).
<br />
● Password hashed in sha1 first. (A66A4A34A78AEC1A7058C8FA3BB3B0F1CC537DD0) <br />
● Password hashed in md5 (42F0F3F909F87D0706DCF139AB37F86E) then. <br />
The salt value can be found in the sqlite database locksettings.db, so we open this database.
<br />
<img width="581" alt="smart5" src="https://github.com/user-attachments/assets/20502f3e-ff7c-453d-bb75-2aa7c16a4d0e">
<br />
*Ref 5: sqlite database locksettings.db*
<br /> <br />
As indicated in the screenshot, the salt value is shown in row 5: 6140990771726895285.
<br />
<br />
Now that we have the salt, before brute forcing the password, we can know more info about it in the device_policies.xml file.
<br />
"cat /home/kali/Downloads/4.2Forensics/dump_filesystem/dump_salida/data/system/device_policies.xml"
<br />
<img width="586" alt="smart6" src="https://github.com/user-attachments/assets/b518755b-cd61-4e45-9f24-fa08a8adbdf6">
<br />
*Ref 6: device_policies.xml file*
<br /> <br />
Now we know now that the password has a length of 10 characters, with 5 uppercase and 2 lowercases, 7 letters, 1 number, 2 symbols and 3 “nonletters”. And according to the assignment, it follows this pattern: INS{xxxxx}. So we have to bruteforce 5 characters (2 uppercase, 2 lowercases and 1 nonletters.
<br />
To perform the bruteforce attack we choose the md5.
<br />
First we obtain the hexadecimal value of the salt as it is in decimal.
<br />
"printf "%x\n" -6140990771726895285"
<br />
<img width="402" alt="smart7" src="https://github.com/user-attachments/assets/eba56820-db0f-4b09-80ce-d6761e4662a4">
<br />
*Ref 7: Hex value of the salt*
<br /> <br />
Then we concatenate the hash and the hex value obtained, to use it later.
<br />
"echo "42F0F3F909F87D0706DCF139AB37F86E:aac6d16df244374b" > hash"
<br />
<img width="545" alt="smart8" src="https://github.com/user-attachments/assets/dcaa8eeb-6da9-4327-bbdb-246f45845119">
<br />
*Ref 8: Hash and hex value concatenated*
<br /> <br />
Then we run hashcat with the following attributes:
<br />
● m 10: hash type 10 (md5 pass+salt) <br />
● a 3: bruteforce attack <br />
● hash: md5 pass + salt <br />
● 1 ?l?u?d: can contain lowercase letters, uppercase and decimal numbers <br />
● INS{?1?1?1?1?1}: the password format <br />
"hashcat -m 10 -a 3 hash -1 ?l?u?d INS{?1?1?1?1?1}"
<br />
<br />
When trying to run the command in the Kali machine it didn’t work, as we got an error due to not enough allocatable device memory for this attack
<br />
<img width="586" alt="smart9" src="https://github.com/user-attachments/assets/955a1cc4-4844-4bbe-8d17-63bdc032182a">
<br />
*Ref 9: Not enough memory to run hashcat command*
<br /> <br />
So we tried it on a Windows machine.
<br />
So we tried it on a Windows machine.
<br />
"hashcat-6.2.6>hashcat.exe -m 10 -a 3 "42F0F3F909F87D0706DCF139AB37F86E:aac6d16df244374b" -1 ?l?u?d INS{?1?1?1?1?1}"
<br />
<img width="413" alt="smart10" src="https://github.com/user-attachments/assets/2b5e0393-c5a6-4436-b321-752e34893f82">
<br />
*Ref 10: Hashcat command on a Windows machine*
<br /> <br />
The password is then INS{t1MmY}.
