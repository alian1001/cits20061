# Lab 3: Access Control

## 3.1. Introduction
Access control has two fundamental components: authentication and authorization. Authentication is the process of verifying the identity of a user, process, or device. Authorization is the process of determining whether a user, process, or device is allowed to access a resource. In this lab, we will explore these concepts in more detail.

## 3.2. Authentication
Authentication is the process of verifying the identity of a user, process, or device. There are three primary methods of authentication: something you know, something you have, and something you are. These are often referred to as knowledge factors, possession factors, and inherence factors, respectively. In this lab, we'll try to implement some of these concepts, but due to physical limitations (i.e., only have access from your computer), some concepts like biometrics and tokens will not be covered. However, you'll have sufficient understanding to be able to implement them, if you need to.

### 3.2.1. Passwords
Passwords are the most common form of authentication. They are a knowledge factor, meaning that they are something you know. In this section, we will explore the use of passwords for authentication.

We'll start with the most simplest implementation, then add features to make it more secure. Download the template code for this section:

```
wget https://github.com/uwacyber/cits2006/raw/2024/cits2006-labs/files/password.py
```

Run this code to check that it is working correctly (i.e., with the right username and password, you can authenticate yourself).

<figure><img src="./img/password_base.png" alt=""><figcaption></figcaption></figure>

However, this is not secure at all. The password is stored in plaintext, and anyone who has access to the file can see the password. We will now add some security features to make it more secure.

### 3.2.2. Salting and Hashing
To make the password more secure, we will use a technique called salting and hashing. You should remember about hashing from Lab 1. Salting is the process of adding a random value to the password before hashing it. This makes it more difficult for an attacker to use a precomputed table of hashes (a rainbow table) to crack the password. Hashing is the process of converting the password into a fixed-length string of characters. This makes it difficult for an attacker to determine the original password from the hash.

We will use the `bcrypt` library to hash the password. Install this library if you haven't done already. This library automatically generates a random salt and hashes the password. But `bcrypt` uses byte string, so ensure to add `b` in front of your string (e.g., your password variable).

```python
database = {"user1": "123456", "user2": "654321"}
new_database = {}
for user, password in database.items():
    new_database[user] = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
```

The above code will convert the original database dictionary of plaintext to a new dictionary of hashed passwords. You can then use this new dictionary to authenticate users now. Checking the password using the `bcrypt` library is simple:

```python
bcrypt.checkpw(password, hashed_password)
```

#### TASK 1
Edit the rest of the code so that the authentication works correctly as intended. 


Note that, when checking the password you don't need to provide the salt to the checker, because the salt is stored in the hashed password. The `bcrypt` library will automatically extract the salt from the hashed password and use it to check the password. However, if you are implementing your own library or using other libraries, you may need to store the salt separately and provide it to the checker.




### 3.2.3. Time-based One-time Passwords (TOTP)
Time-based One-time Passwords (TOTP) are a form of two-factor authentication. They are a possession factor, meaning that they are something you have. TOTP works by generating a one-time password based on a shared secret and the current time. The shared secret is usually a QR code that is scanned into an authenticator app, such as Google Authenticator. The authenticator app then generates a one-time password based on the shared secret and the current time. The server can then verify the one-time password by generating the same one-time password based on the shared secret and the current time and comparing it to the one-time password provided by the user.

You can start with the basic template provided below:

```
wget https://github.com/uwacyber/cits2006/raw/2024/cits2006-labs/files/totp_mfa.py
```

The code is not quite complete, you will have to complete the `verify_totp` function to make it work properly (currently it will authenticate any code!). But first, let's look at the rest of the functions provided to you.

The `get_hotp_toke` function is the main one, where the hashing algorithm hmac is used to generate the token. First, it will convert the password (which is a string) to a byte string. It will use the current time (denoted as `intervals_no`) to create `msg`, and using the sha1 hashing algorithm, the hmac digest is created. The token is then generated by using the first four bytes from the hmac digest modulo 1000000 to generate a 6-digit token. Run the code with the same password a few times, you will notice that the token will not be the same!

#### TASK 2
Complete the `verify_totp` function to make it work properly. You will need to use the `get_hotp_token` function to generate the token, and then compare it with the token provided by the user. You will also need to use the `time` library to get the current time, also remembering that the token is supposed to expire after 10 seconds by default. The passed in `token` is the user-input, which means you should generate the possible tokens using the same password in the last 10 seconds and compare it with the user-input token.


{% hint style="danger" %}
More tasks coming soon...
{% endhint %}

<!-- {% hint style="danger" %}
READ: Any knowledge and techniques presented here are for your learning purposes only. It is **ABSOLUTELY ILLEGAL** to apply the learned knowledge to others without proper consent/permission, and even then, you must check and comply with any regulatory restrictions and laws.
{% endhint %}

## 3.0. Introduction

Many malware use obfuscation techniques to try to hide the information about how they function. In this lab, we will try to uncover their mechanisms using reverse engineering techniques.

You only need to use the Kali VM for this lab.

{% hint style="info" %}
Later, we will use a tool named Ghidra, but it is a bit large. So install it now in a separate terminal:

sudo apt-get update -y

sudo apt-get install ghidra -y
{% endhint %}

## 3.1. Reverse Engineering using GDB

GDB, the GNU Project debugger, allows you to see what is going on \`inside' another program while it executes -- or what another program was doing at the moment it crashed. So think of it like a debugger, not for the program you are writing, but for one that has been compiled already. By understanding how they function, it is also possible to reverse the damage caused (e.g., decrypting files that the ransomware encrypted).

{% hint style="danger" %}
We will be using a malware code, so you should only conduct this lab within a VM!
{% endhint %}

One of the most interesting stories about reverse engineering is the story about the ransomware WannaCry. WannaCry propagated across the internet using the EternalBlue exploit, which was developed by the NSA and leaked by an anonymous hacker group called the Shadow Brokers. It was devastating computers across the world, until Marcus Hitchins reverse engineered the ransomware. Marcus found an unregistered domain within the malware and decided to register the domain. Consequently, he inadvertently found the kill switch for the ransomware, stopping one of the largest cyber-attacks known to this day.

In this section, we will be reverse engineering a newly discovered ransomware called `free_bitcoin`, specifically designed to target the Kali VM AMD64 chip users.

{% hint style="warning" %}
This section of the lab does not work on Apple Silicon computers or any other ARM-based architectures, because the instruction sets are vastly different between the AMD and ARM architectures when you compile codes.

Alternate ways to do this section is to work with others in the lab (suggested), or you can also start an Ubuntu VM in the cloud and follow the instructions there, which will work also (but can cost you if you don't have free credit).
{% endhint %}

```
wget https://github.com/uwacyber/cits3006/raw/2023S2/cits3006-labs/files/free_bitcoin
```

It was reported that a victim tried to get free bitcoin by running the program, but instead encrypted everything in the working directory. We will try and reverse engineer the malware to retrieve the encryption key used to encrypt the victim’s files.

For this task, we will use a light-weight tools namely GDB to reverse engineer the ransomware, but we will first use other simple tools (`strings` and `objdump`) to discover more about the ransomware at hand.

### 3.1.1. Using strings command

We will begin our analysis of the ransomware by running the `strings` command on the binary. Below is a picture of the output of strings being piped into grep to highlight some key library functions and hardcoded strings that were found inside the ransomware.

```
strings free_bitcoin | grep "EVP\|1234567890abcdef"
```

![](<../.gitbook/assets/image (5) (3).png>)

The above screenshot shows that the malware uses the OpenSSL Crypto library. The ransomware is also using the AES 128-bit encryption with the CBC mode, which means that the key used to encrypt the files is 128 bits (16 bytes) long.

The other interesting detail is the string “1234567890abcdef” inside the program, which could be the key since it is 16 bytes long, or the key could be generated using this string in some way, or it is just there to throw off our investigation. We will just take a note of it for now.

Next, we will collect more info using `objdump`.

### 3.1.2. Using objdump

This is where we start looking at the assembly code of the ransomware. Run:

```
objdump -d free_bitcoin
```

Ignoring the included functions from libraries, we find that the malware has the functions `main`, `encrypt_file`, `decrypt_file` and `gen_key`. Let us take a closer look at the `gen_key` function since this is most likely where the key is created to be used for encryption. Below is the assembly code of this function.

![](<../.gitbook/assets/image (18).png>)

Of interest is that the function calls `srand` (at line 7, address `40128b`), which is the C function for setting the seed for the random number generator. To try and figure out what is the value of the seed, we will compile our own test program and compare the assembly code. We have provided you with the test code `srand_test.c`.

```
wget https://github.com/uwacyber/cits3006/raw/2023S2/cits3006-labs/files/srand_test.c
gcc -o srand_test srand_test.c
```

The test code uses the value of 16 (0x10 in hexadecimal) to set the seed, so we will look for where this value is in the assembly code.

![](<../.gitbook/assets/image (16).png>)

When you run `objdump` on the compiled file, you should see the main function as:

![](<../.gitbook/assets/image (1) (1) (2).png>)

We can see that our seed value of `0x10` is pushed onto the stack directly before the program calls `srand`. Comparing this procedure to the assembly code from above, we can see that just before the `srand` call at the machine instruction address of `40129b` in `gen_key` the hexadecimal value of `0x4d2` is pushed to the stack. This means that in `gen_key`, the seed is set to `1234` (i.e., 0x4d2 in decimal format).

Now we are ready to debug our ransomware.

### 3.1.3. Using GDB-peda

`GDB`, as described above, is a debugging tool. However, its interface is quite difficult to use without spending time learning more about it. To make your life (slightly) less miserable, we will install also the `peda`, a Python Exploit Development Assistant for `GDB` (which makes the presentation and usage a bit more novice-friendly).

First, install GDB:

```
sudo apt-get update -y
sudo apt-get install gdb -y
```

Next, install `peda` (line by line):

```
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
```

Also install OpenSSL library:

```
sudo apt-get install libssl-dev
```

Below we list some useful commands for inside the `gdb-peda` shell to help you reverse engineer the ransomware.

* `gdb-peda$ info func` : Prints out all the functions inside of the program.
* `gdb-peda$ disas <function name>` : Print the assembly code and machine instruction number of a function.
* `gdb-peda$ b *<machine instruction address>` : Pauses the program's execution at the machine instruction address and prints the program's state.
* `gdb-peda$ x/2x $esp` **:** Prints the first 2\*4=8 bytes from the start of the stack ($esp)
* `gdb-peda$ r` : Starts the program's execution from the very start.
* `gdb-peda$ c` **:** Continue the program's execution to the next breakpoint or until completion.
* `gdb-peda$ si` : Execute the next machine instruction and then print the state of the program.

For a list of more commands to use gdb, take a look at [https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf).

Since the ransomware is poorly designed and only encrypts the files in the working directory, we will create a test folder to execute the malware from. Ideally, if you are doing real malware analysis you would want to completely isolate it inside a separate VM before executing it. However, for our purposes running it from inside an isolated directory should be sufficient since it only encrypts files inside the working directory.

You can use the commands below to prepare your test folder and start `gdb-peda`.

```
mkdir test
cp free_bitcoin test/
cd test/
chmod 500 free_bitcoin
gdb free_bitcoin
```

We will begin our analysis by getting the machine instruction for when the function `rand` is called and set a breakpoint at that instruction so we can analyse the state of the program. We will also set another breakpoint directly after `gen_key` returns to the function `encrypt_file`, so that we can pause the program's execution before any files are encrypted. Below are the commands with snippets to help you set up the breakpoints before starting the program.

![](<../.gitbook/assets/image (4) (2).png>)

![](<../.gitbook/assets/image (7) (2).png>)

![](<../.gitbook/assets/image (9) (4).png>)

We will start running the program to see the state of the registers and stack at each time the `rand` function is called. Run the program by entering `r`. Then you can continue running the program by entering `c`.

![](<../.gitbook/assets/image (13) (2).png>)

The screenshot above shows the state of the program after reaching the `rand` function a second time (continuing the execution of the program once). This snapshot of the program’s state tells us two important things about how the key is generated.

* Firstly, the key is generated inside a loop since when the program continued after reaching the first breakpoint it paused at the same breakpoint a second time, instead of reaching the breakpoint in `encrypt_file`.
* The second observation is that the character `e` is stored inside the `EDX` register, as shown as `RDX`, (i.e., line 4 in the registers section). This can mean that `e` is the result of some operations following the first `rand` call, and is possibly (and most likely) the first character of the encryption key.

To investigate this further, we will now set a breakpoint after the rand call at the machine instruction at the address of `0x4012aa` and step through the program’s execution by machine instruction (`c`, then using the `si` command) until we find something interesting in the registers or the stack. At every step (after each `si` command), try to inspect the registers, code and stack to see if you can find any useful information. Once you reach the code `movzx`, you will see the below state.

![](<../.gitbook/assets/image (1) (3).png>)

At this stage, you can see that the address `0x402008` is being moved to `EDX` (it is noted as RDX in the registers), which contains a familiar string we found before. As soon as you step in (`si`), you will notice that letter '4' is now loaded onto `EDX`. This is shown below.

![](<../.gitbook/assets/image (5) (1) (1).png>)

So definitely, the string "`1234567890abcdef`" is used to generate the key string!

Based on our findings, we can conclude that:

1. The ransomware sets the random seed to be `1234`.
2. The `rand` generator is used to select a char from a string "`1234567890abcdef`".
3. Step 2 is repeated until the key size is 16 bytes (i.e., looped 16 times).
4. Using the generated key from step 3, aes-128-cbc is used to encrypt files.

You can now either (1) continue debugging the ransomware to find the key (keep running until you generate the first 16 bytes of the key), or (2) write a code that mimics the key generation steps described above (i.e., set the seed to `1234` and choose char from "`1234567890abcdef`". The first output is "`e`", followed by "4" and so on). Either way, you should converge to the same key.

## 3.2. Another tool: Ghidra

Ghidra is a tool for reverse engineering, which has been used for many years by special services. Now it is available to everyone.

By now, you should have completed installing Ghidra. Since the required JDK is already installed on Kali, your ghidra should be good to go (if using other OS VM, install necessary requirements yourselves).

Once you run ghidra (just type `ghidra` from the terminal), you will first be greeted with the agreement notice - press "agree". Then, you see the Ghidra Help - you can read this at your own time to get more familiar with Ghidra, but otherwise you can close it for now. Finally, you will see the main ghidra window and the tip window (close this also). Now we are ready to get started!

### 3.2.1. Opening a project in Ghidra

Download the files we will be using for this section.

{% tabs %}
{% tab title="Intel (AMD64)" %}
```
wget https://github.com/uwacyber/cits3006/raw/2023S2/cits3006-labs/files/crackme-linux.zip
```
{% endtab %}

{% tab title="Apple Silicon (ARM64)" %}
```
wget https://github.com/uwacyber/cits3006/raw/2023S2/cits3006-labs/files/crackme-arm.zip
```
{% endtab %}

{% tab title="Source (if none of them works)" %}
```
wget https://github.com/uwacyber/cits3006/raw/2023S2/cits3006-labs/files/crackme-source.zip
```

Once downloaded, compile codes using the makefile provided.
{% endtab %}
{% endtabs %}

{% hint style="info" %}
You can somewhat follow most of the steps on Apple Silicon, but because the instructions are different between AMD64 and ARM64, the displayed output differs. So, it is easiest to follow the Ghidra sample using the AMD64 example, but you can check the binary using the ARM64 example to run on your VM.
{% endhint %}

On Ghidra, create a new project (doesn't matter shared or not). You can name it `crackme0`.

Next, import `crackme0x00` from the unzipped folder to Ghidra, you can either drag and drop, or import file from the menu. You can leave the other settings unchanged, and finish importing the file.

![](<../.gitbook/assets/image (3) (1) (4).png>)

Open the analyser by double-clicking the binary. You will be prompted with the analyser, which you simply press "yes" (the pre-selected analysers are sufficient here). Then it will get you here:

![](<../.gitbook/assets/image (4) (1) (3).png>)

On the CodeBrowser console, you see a few windows:

* **Program Trees**: This window displays the code sections of the binary.
* **Symbol Tree**: This window displays the import, export, functions, labels, classes and namespaces of the binary.
* **Data Type Manager**: This window displays all specific types, including built-in ones, specific for the binary file and other types included in Ghidra.
* **Listing**: This window displays the reverse-engineered code.
* **Decompiler**: This window displays the high-level code generated by Ghidra from the assembly code shown in the Listing window. To see, scroll down in the Listing window, and select some functions to see their code representations.

Now we will inspect our binary file. The behaviour we observed was that it prompts for the password, checks the password, and then responds based on the user input provided.

![](<../.gitbook/assets/image (1) (2).png>)

### 3.2.2. `crackme0x00` walkthrough using Ghidra

Let's start by inspecting the program strings: WIndow -> Defined Strings.

![](<../.gitbook/assets/image (7) (3).png>)

![](<../.gitbook/assets/image (2) (2).png>)

Well, it seems the password was stored in cleartext in the binary as shown above. Nevertheless, we will still have a look at whether this password indeed is the one that works with the binary. Double-click the `Password` entry in the `Defined Strings` window, which will take you to the section where the string is stored.

![](<../.gitbook/assets/image (9) (1).png>)

You will see that it is referencing something in the main function (the green text on the RHS). So let's follow by double-clicking the address, which takes you here:

![](<../.gitbook/assets/image (3) (1) (5).png>)

You will see that there is a `scanf` call after the reference to the `Password`, and then followed by the `strcmp`. This looks pretty much like where the password was prompted when the binary was run, and how the password is checked! Having a look at the decompiled code makes this suspicion a reality:

![](<../.gitbook/assets/image (6) (1).png>)

The entered password is saved to the `local_lc` variable. The string value 250382 has been stored in the `local_3c` variable (see the assembly code). The result from `strcmp` is then checked, with zero being the same string. Hence, the string `250382` is our password!

![](<../.gitbook/assets/image (12) (3).png>)

### 3.2.3. Solve `crackme0x01` and `crackme0x02` using Ghidra

Try the next two binaries `crackme0x01` and `crackme0x02` yourself and see if you can crack the password!

### 3.2.4. `crackme0x03` walkthrough using Ghidra

We start off similar to the previous questions, but obviously, this won't have the password saved the same as before. When we inspect the strings, we can still see the word "Password" as the prompt, so it is a good place to start. Inspecting the code where the password in entered first:

![](<../.gitbook/assets/image (29).png>)

The main function can be inspected from here, and indeed the way the password check is done is different. Instead of checking the password in the main, it calls another function `test`, with two variables passed in.

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7fBivtRyeRgCSUaXucCZ%2Fuploads%2FhJxypjT3OcenDqdfxvmr%2Fimage.png?alt=media\&token=32248987-87b3-46f6-8c65-02fc5ac4b7ca)

But at this point, you probably guessed that the second arg 0x52b24 is probably the password we are looking for. If you try that as is, it will fail because of course the representation is in hex. You have to convert it to decimal first, and this is already done for you - right-click on the variable and it will show you other commonly used conversion values. The decimal value 338724 seems like a good candidate, so try that as a password.

![](<../.gitbook/assets/image (13).png>)

Indeed, that was the password!

![](<../.gitbook/assets/image (4) (4).png>)

whetherAnyway, let's inspect the function test to see whether this is indeed the place where the password is checked or not. From the decompiler window, double-click the function name `test`.

![](<../.gitbook/assets/image (3) (1) (2).png>)

de the test function, it is showing some shift functions, which aren't conventional c functions so it must be doing something, possibly shifting. So let us try shifting the letters.

![](<../.gitbook/assets/image (19) (1).png>)

Function called `shift` is being used, this isn't any built-in function so is a custom, and is probably doing some shifting. Double-click the shift function to see what it does.

![](<../.gitbook/assets/image (20) (1).png>)

If you read the function carefully, the operation is quite simple. To make the readability better, let's rename some variables (you can press "`L`", or right-click to see the option):

* `local_80` -> `i`
* `local_7c` -> `output`
* `sVar1` -> `str_len`

Then we have:

![](<../.gitbook/assets/image (22).png>)

So basically the loop goes over each char from the input arg `param_1`, and shift it by -0x3 (remember, we are working in hex). We can shift from terminal using Python:

![](<../.gitbook/assets/image (21).png>)

Indeed, those were the messages displayed when guessing the password!

There are more `crackme` puzzles provided in the zip, so have a go at them at your own speed :)

## 3.3. Conclusion

We learned additional tools to help us reverse engineer binary files and inspect their functions to gather important information about their operations. This is especially useful for dissecting binaries such as malware, where you can also be able to reverse the damage caused. For example, the WannaCry ransomware was shut down by reverse engineering the malware binary and finding out its terminating condition.

Next up, privilege escalation.

Credit: some materials were adopted from the IOLI workshop with minor edits/updates. -->
