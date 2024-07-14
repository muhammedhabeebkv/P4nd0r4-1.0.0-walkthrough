# P4nd0r4-1.0.0-walkthrough

## Task 1 Web App Testing

To gather information about ports perform nmap scan.

```
nmap -sC -sV -Pn MACHINE_IP
```

The scan has discovered two ports which provides SSH and HTTP. And first task is about finding hidden paths in the web server.
> What is the name of the hidden path on the web server?
In order to do that you should use one of the directory enumerator programs. In this case I'm using Gobuster with Dirbuster wordlist.
```
gobuster dir -u http://10.10.147.136 -w /usr/share/wordlists/dirbuster/directory-list-comman.txt
```

![1_5sShP9kCtFhSYICo1xOq6Q](https://github.com/user-attachments/assets/dc3dd101-e0fd-4723-a3e5-5a5168a52c14)

![1_NK0OXAisfpMtYHNjiMo59g](https://github.com/user-attachments/assets/2aab6009-0fe0-4eb7-98c0-8739642b8b89)

I discovered a chat web at `/********` where users can share images. One user shared an image which I suspected to be hiding information using steganography techniques. Then I download the image shared by user from chat web page.

![1_shL0DAvCOp3eUxE9u1RRQg](https://github.com/user-attachments/assets/701cff0c-faf1-4dd2-bcc4-ccf1477e83aa)

Using the steganography tool stegcracker. I attempted to extract hidden information from the image.

![1_Wbfhfzkr-qQ67Am-oV9xLQ](https://github.com/user-attachments/assets/321ff90e-e042-4610-a7c8-69bbb04561be)

The extracted file contained text encoded with morse code. To decode this, I used the <a href="https://gchq.github.io/CyberChef/">CyberChef</a> online tools.

The decoded text revealed the following path `/********` and navigating to `/*********`, I found three files.

![1_76ZlNygQxACPtR1bHmOLeg](https://github.com/user-attachments/assets/5fed7f73-86e6-4b3a-b215-f34437025a5d)

The `0********-pub.asc` file contained a PGP public key. to decrypt messages encrypted with this key, both the public and the corresponding private key are required, along with the passphrase found in the passphrase file. Since only the public and passphrase were available, further steps involving decryption were limited.

![1_xZzcGYnBHaUbSipTdGf1Qg](https://github.com/user-attachments/assets/d3261743-325f-4b99-a61f-51c966aacd51)

<a href="https://pgptool.org/">PGP Tools</a>

The passphrase in the passphrase file appeared to be encoded in multiple formats.

![1_Fd3_0uo5bl9N77aUiwXn3w](https://github.com/user-attachments/assets/db3cfab1-9b43-4cce-b3ac-59abeb9b8fc9)

This passphrase is encoded using an similar to hex, but with some difference like underscore. I suspected it involved another encoding. Using online tools to identify the what type of encoding.

![1_lnwcJfnodFEFFm5V7lNnOA](https://github.com/user-attachments/assets/458e4b36-b189-47be-be7f-4ceb0054de0a)

I found that it uses ROT47. After decoding it, I found that the resulting code was encoded like hex. I decoded that, and then the result was encoded in base64. Finally, I got the real text.

![1_ahhB-npsN1FkXKp4UEV3Yw](https://github.com/user-attachments/assets/d7a05746-5a2a-4684-9a53-e7cd3ee5b23e)

I analysed the `pandora_toroubleshoot.pcapng` file using Wireshark. The network capture revealed a website URL `p********g` along with an email and password combination.
To access the website, I added the IP address and hostname to my system's hosts file:

![1_160TgcVRRK6Uiw8pLIrzZQ](https://github.com/user-attachments/assets/184f4546-94ed-4ff7-8a97-0f158e55c600)

![1__qhIgXfnFA_Qs5v4-BP5lQ](https://github.com/user-attachments/assets/5e3e91e6-ba26-4df6-a73e-c9783589e51b)

```
sudo nano /etc/hosts
```

![1_ZKIG4R85PEbdw3BJSHecyA](https://github.com/user-attachments/assets/02317802-6069-41e3-98cf-933b321d242b)

![1_gkMteM0e-M3bijMhAPQmZg](https://github.com/user-attachments/assets/02456ff4-3d0c-4d24-a67c-f0ee67c92e02)

Using the email and password obtained from the network capture file, I logged into the website `p*********g`. A new option, "Assets," appeared on the navigation menu. Clicking on "Assets" presented an option to download a file named archive_data.zip.

![1_tJV-o6xZPDEe9aqA4ZVmjA](https://github.com/user-attachments/assets/5d561ca8-719f-452a-9b48-ce9b20360c17)

I downloaded and extracted `archive_data.zip` which contained a `.git` directory and `info.txt`

```
unzip archive_data.zip
```
![1_U97-olEGat67mPieclsivg](https://github.com/user-attachments/assets/3ae6b920-8e82-4549-b83c-d35653e4d1c6)

Then I read the `info.txt`, which shows the hint for the file. Then I use to git commands to analyse the repo.

```
git log - all
```
![1_CQglui3OovbpLIPzOdIgdg](https://github.com/user-attachments/assets/1f6600cc-0681-435e-8925-011683c3c49d)

This showed one commit. Then I try this command.

```
git status
```

![1_sUOZs3Bfsnq8d-iEsAI3OA](https://github.com/user-attachments/assets/7d58dca8-8546-4a86-b124-d62e0cc6af33)

This revealed that 10 files were deleted. I restored these files using:

```
git restore assets index.html upload utility
```

These files were part of a website backup. Upload directory contained the same file `archive_data.zip` file. utility directory contained two files `0********.asc` and `0********-sec.asc.signed.txt` , but both were empty except for a message: "Do you think you can pull off some digital wizardry and get this file another way?"

The message suggested that the files might be accessible via the website. I used Burp Suite to intercept the download request for ``archive_data.zip`` and modified the URL to access the hidden files:
![1_QzB1VQfeEPNIEhx7JvjvCg](https://github.com/user-attachments/assets/77e9914f-d6cf-49ae-8528-b2b606398040)

Then I modified the request to ?file=archive_data.zip to `?file=../u********/0********-sec.asc` and forwarded the request. It worked

![1_mBSV454jMGdAfPPnzAhH_Q](https://github.com/user-attachments/assets/786e99eb-c69a-4486-aaf0-8fbf278ef9f0)

![1_Va_EYcxQ6HBKfDycBeLebg](https://github.com/user-attachments/assets/aabd36f9-8705-4b53-ac60-8fb8e38740bd)

With the `0********-sec.asc` and `0********-sec.asc.signed.txt` files in hand, I proceeded to decode the encryption message:

![1_ZUOfegod-2B_ZqihrWdr-g](https://github.com/user-attachments/assets/a6ab9e5f-ad75-48e1-a3d5-39365ab4dab0)

Using the obtained SSH credentials, I logged into the machine:

![1_PwuB3s86pZv81ps-42nJGw](https://github.com/user-attachments/assets/353e04d2-0c00-498a-b39d-d8c63163e756)

Once logged in, I began analysing the system for privilege escalation opportunities. I examined the `/etc/crontab` file and found the following cron job running as root:
![1_wi3394_5ml_XPTx-wvLY-Q](https://github.com/user-attachments/assets/1711e5cd-1114-4d91-8be5-7e67837c1397)

This cron job downloads and executes a script from `http://p4nd0r4.org/backup/setup.sh` every minute. I decided to exploit this by redirecting the domain to my local machine and serving a malicious script.

![1_vkVHRoPI6l7owFuD5INmzA](https://github.com/user-attachments/assets/8f7be2bd-02d6-4186-a17d-c0f3f0347013)

I edited the `/etc/hosts` file on the target system to redirect p4nd0r4.org to my attack machine's IP address:
![1_dKI76oNCnqZFB3TJU-pFbQ](https://github.com/user-attachments/assets/01fde217-2acc-4f22-8559-81071abd1a14)

On my attack machine, I set up a directory structure to serve the malicious script:

![1_Hcj2DkAL8Tqv9PISxhfkOg](https://github.com/user-attachments/assets/9f1b123d-8078-4d96-9178-13a790ce5986)

Create `setup.sh` with the following content:
![1_uKnmWRmQt5yuSH4oY5_MGA](https://github.com/user-attachments/assets/685a29b7-00a9-4db0-bfd7-bf27787b3ba4)


I hosted the directory using Python's built-in HTTP server and I set up a netcat listener on my attack machine to catch the reverse shell:

![1_bbDCwfdkt8IGvjuRxYU1Xw](https://github.com/user-attachments/assets/9f3b5c7f-5cd3-4b9f-a572-fe382cc07a61)

> Wait for the Cron Job to Execute

The cron job on the target machine executed the malicious script, establishing a reverse shell connection back to my attack machine, granting me root access.

![1_WuRdz6f5MxHM0s_RwYZAaQ](https://github.com/user-attachments/assets/692554bb-218e-4ea9-8494-a966d279ae0c)

# Thanks For Reading.


