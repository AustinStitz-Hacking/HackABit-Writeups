# HackABit Writeups

This year, I competed in the inaugural [HackABit](https://www.hackabit.com/) competition!

I've greatly enjoyed participating, completing three rounds of fun challenges! The first round was a quiz, the second a CTF competition, and the third a combination of CTF challenges including web, binex, and forensics, as well as attack/defense!

I was personally able to complete all of the challenges in round 3 except for BadAI (which had an outage most of the competition) and the King of the Hill attack/defense challenges (which also was shut down by a competitor and not brought back up). 

The challenges are posted at [https://github.com/Shift-Cyber/hab-challenges-public](https://github.com/Shift-Cyber/hab-challenges-public).

# dvGPT

## Leaky

**Points:** 75

**Author:** Nate Singer (helix)

**Description:** People leave dumb comments about vulnerabilities all over client-side code... most modern web applications don't display all the possible code at once though, for a variety of reasons.

**Links:** [https://wtashuxsyj.final.hackabit.com/](https://wtashuxsyj.final.hackabit.com/)

### Writeup

Based on the challenge description describing "dumb comments about vulnerabilities all over client-side code," it is pretty clear that the flag can be found in client-side code.

So let's start up DevTools!

If we look in the elements tab, we can find an interesting ID as a part of the chat panel:

![Leaky Image 1](leaky1.png)

If we then look for the chat panel in the sources tab through `dvgpt/src/components/ChatPane.vue`, we find even more:

![Leaky Image 2](leaky2.png)

From this, we can get the flag, `flag{the_DOM_is_like_crazy_virtual_maaaan}`


## Extractor

**Points:** 100

**Author:** Nate Singer (helix)

**Description:** We gave dvGPT a flag and for some reason trust it to keep it safe. These AIs are super duper secure my dude.

**Links:** [https://wtashuxsyj.final.hackabit.com/](https://wtashuxsyj.final.hackabit.com/)

### Writeup

Now, looking at this, it seems we have to do some LLM prompt engineering.

Let's start simple:

![Extractor Image 1](extractor1.png)

Now again...

![Extractor Image 2](extractor2.png)

So it seems we have to be Robert to get the flag...

![Extractor Image 3](extractor3.png)

And there we have it, the flag is `flag{hey_you're_not_robert}`!

## BadML

**Points:** 125

**Author:** Nate Singer (helix)

**Description:** Where does all this data come from? Have you found all the app endpoints?

*The flag is located at `./flag.txt`*

**Links:** [https://wtashuxsyj.final.hackabit.com/](https://wtashuxsyj.final.hackabit.com/)

### Writeup

Since this challenge discusses endpoints, let's try the DevTools network tab to see what requests are made...

![BadML Image 1](badml1.png)

Most of these endpoints seem to be simple JS and CSS files, which you won't be able to get much out of, them being client-side code. However, `/footer` provides something interesting, with a parameter of `message` in the search query.

If we open that endpoint in a new tab, assuming the `message` parameter points to a file, we can set `?message=flag.txt` to get access to the flag.

![BadML Image 2](badml2.png)

And there we have the flag, `flag{LFI_LetsgoFindIt}`!

## BadAI

**Points:** 150

**Author:** Nate Singer (helix)

**Description:** So we have this guy named Bill that works here--he handles the support tickets for dvGPT. If you have any problems let us know and Bill will check it out. Bill does nothing but stare at the ticket feed, so you can expect him to check your request within a couple seconds.

Bill can only view pages from dvGPT though, so don't bother sending him anything else.

*The flag is stored in a cookie in Bill's browser.*

> -- If you think you're close feel free to DM me (Nate) and I will help you depending on what you're asking and how close you are. There are some browser nuances you might not be aware of and if you are 99% of the way there we'll probably give it to you.

**Links:** [https://wtashuxsyj.final.hackabit.com/](https://wtashuxsyj.final.hackabit.com/)

### Writeup

Do later

# Corruption

## Santa

**Points:** 75

**Author:** Nate Singer (helix)

**Description:** You all asked for it so here it is, an intro to binary exploitation!

Let's get started nice and simple, baby steps to reverse engineering (RE).

*All challenges in this section use the same binary. The target is x86 and ASLR is on but it shouldn't be relevant to any of your exploits.*

**Files:** [corruption](files/corruption)

### Writeup

This first challenge is relatively simple. Let's launch `ghidra` to look a bit more at the code for the binary file, `corruption`!

Analyzing with the defaults, we can look at the symbol tree to find the data stored in the binary file.

Nested under `Labels/f`, we find a label interestingly named `flag`! Let's look at that!

![Santa Image 1](santa1.png)

And there's our flag, `flag{baby_steps_gift_just_for_you}`!

## Coredump

**Points:** 100

**Author:** Nate Singer (helix)

**Description:** Now that we have at least inspected the binary, lets go a bit deeper. You can't just overflow the buffer with a bunch of A's--reverse engineer the software and figure out your payload format. Smash the stack to get the flag, no control necessary yet. Once you have a working exploit, fire it against the remote target to get the real flag.

*All challenges in this section use the same binary. The target is x86 and ASLR is on but it shouldn't be relevant to any of your exploits.*

**Files:** [corruption](files/corruption)

**Links:** juieqtrsdp.final.hackabit.com:54321

### Writeup

Now, we have to actually develop a payload for the binary file as it runs. We also have a remote location to connect to, `juieqtrsdp.final.hackabit.com:54321`.

Looking in ghidra at the `main` function, we can find something interesting: `strncmp` is used to compare the first bits of our input to `"UNLOCK"`.

![Coredump Image 1](coredump1.png)

We can tell this since `fgets` reads the first 500 characters of `stdin` into `local_23e`, which is compared to `local_18`, set to `"UNLOCK"`, using the first `__n` characters, which is defined as the length of `local_18`, essentially meaning that the input must begin with `"UNLOCK"` for iVar1 to be 0, since `strncmp` returns 0 when the first bytes of the strings are equal.

Now, if we connect to our remote location with netcat, we can try this payload:

![Coredump Image 2](coredump2.png)

It works, and now we have our flag, `flag{look_like_ur_a_real_RE}`!

## bitsANDbytes

**Points:** 125

**Author:** Nate Singer (helix)

**Description:** Now that you have the ability to smash the stack, it's time to get control of the instruction pointer. Use your reverse engineering to figure out proper addresses, we've given you the code required to pull the flag.

*All challenges in this section use the same binary. The target is x86 and ASLR is on but it shouldn't be relevant to any of your exploits.*

**Files:** [corruption](files/corruption)

**Links:** juieqtrsdp.final.hackabit.com:54321

### Writeup

As we start this challenge, we need to do a bit more in order to get a different flag. Looking at ghidra, we can see that the function `flag_function` includes an environment variable very similar to the title of this challenge:

![bitsANDbytes Image 1](bitsANDbytes1.png)

This is where we start.

Looking back at `main`, we see that the address of `flag_function` is printed on program execution, and the code exits with an interestingly-named function, `vulnerable_function`.

![bitsANDbytes Image 2](bitsANDbytes2.png)

Looking at `vulnerable_function`, it seems that `local_3e` might be vulnerable to buffer overflow, as the payload extends beyond 54 characters.

![bitsANDbytes Image 3](bitsANDbytes3.png)

Let's open GDB and locally try a 54-character payload beginning with `UNLOCK`.

![bitsANDbytes Image 4](bitsANDbytes4.png)

It seems to exit normally, but let's keep trying adding additional characters until we get a different exit address:

![bitsANDbytes Image 5](bitsANDbytes5.png)

Now, let's keep trying until we can manipulate this address:

![bitsANDbytes Image 6](bitsANDbytes6.png)

Now, we have 30 at the end of our exit address, which is the hexadecimal code for `0`.

Let's replace this last 0 with 4321:

![bitsANDbytes Image 7](bitsANDbytes7.png)

`0x31323334` is hexadecimal for `1234`, meaning if we replace `4321` with the correct bytes in reverse, we can directly control this exit location.

Let's convert our payload to hexadecimal so we can manipulate this address accordingly.

![bitsANDbytes Image 8](bitsANDbytes8.png)

The payload ends with `0a` to print a new line at the end. However, if we edit the previous 8 characters, we can control the exiting address and set it to `flag_function`.

If we connect remotely through netcat, we find the address `0x804920d`. Therefore, the final bytes of our payload must be `0d920408`, followed by the `0a` for the new line.

![bitsANDbytes Image 9](bitsANDbytes9.png)

Now, if we use `xxd` to convert back to bytes and pipe this into our netcat connection, we can get the flag!

![bitsANDbytes Image 10](bitsANDbytes10.png)

And our flag is `flag{big_ret2flag_energy}`!

## Controller

**Points:** 150

**Author:** Nate Singer (helix)

**Description:** Nice, last step, you have a leak but not for the right function, how can we tackle this problem?

*All challenges in this section use the same binary. The target is x86 and ASLR is on but it shouldn't be relevant to any of your exploits.*

**Files:** [corruption](files/corruption)

**Links:** juieqtrsdp.final.hackabit.com:54321

### Writeup

This challenge works very similarly to bitsANDbytes, but we need to use the function `controller_flag` instead. 

Using GDB, we can find the addresses locally of each function:

![Controller Image 1](controller1.png)

![Controller Image 1](controller1.png)

Our addresses are `0x08049221` and `0x08049264`.

If we use JavaScript to quickly apply the same offset to the `flag_function` address of our local connection, we get `0x08049250`.

![Controller Image 3](controller3.png)

So, if we use this new address for our payload, we can get this flag!

![Controller Image 4](controller4.png)

And our flag is `flag{almost_totally_full_control}`!


# Triage

## Sluth

**Points:** 75

**Author:** Nate Singer (helix)

**Description:** Everything in life is iterative...

**NON-STANDARD FLAG FORMAT**

**Files:** [morse.txt](files/morse.txt)

### Writeup

As we download the file, `morse.txt`, it is clear the name refers to Morse Code, and based on the challenge description, it is clear that this is iterative, meaning this is a Morse Code code in Morse Code!

Let's look at the file through our browser (mousepad crashes due to the file size) so we can see the format clearly:

![Sluth Image 1](sluth1.png)

Normally, Morse Code uses a space to separate characters. However, this seems to use `/`, which is usually used to separate words.

An alternative for separating words is two spaces, so we can also suggest that two `/`s can mean the same thing with this format. 

We can also use the online Morse Code translator for this, `https://morsecode.world/international/translator.html`.

In our JavaScript console, we can define a function to help process this information:
```js
const go = txt => copy(txt.split("  ").join("#").split(" ").join("").split("#").join(" "));
```

Since the translator converts `/` to spaces, we want to get rid of single-spaces while preserving double-spaces as single-spaces, which this function does. Since it automatically copies the data, each iteration, we can just run this function on the output and paste the result in the input box.

Ultimately, we get this:

![Sluth Image 2](sluth2.png)

The translator fails on `..--.-`, but with a quick Google search, we find that this is the underscore character, meaning our flag (non-standard format) is `FIVE_TIMES_0.O`.

## Inspector

**Points:** 100

**Author:** Nate Singer (helix)

**Description:** It's just a simple stream challenge, how hard can it be?

Both challenges for this section use the same pcapng file.

**Files:** [triage.pcapng](files/triage.pcapng)

### Writeup

In this challenge, we are given a PCAP file to analyze, which we can open with Wireshark!

Once in Wireshark, since we can assume from the challenge description that the flag is in plaintext, we can just search for a partial string, `g{` from the flag format, or `677b` in hexadecimal.

Doing this, we eventually find an HTTP request with an interesting ending to the request:

![Inspector Image 1](inspector1.png)

Although the flag is broken up some, we have our flag, `flag{tcp_streams_reveal_more}`!

## Coverup

**Points:** 125

**Author:** Nate Singer (helix)

**Description:** There is a challenge hidden in coverup.jpg, extract the flag and profit.

**Files:** [challenge.jpg](files/challenge.jpg)

### Writeup

In this challenge, since we are given an image and told there is information within it, we know it uses steganography. 

Two of the most common steganography extraction tools are `binwalk` and `steghide`. Here, `binwalk` returns nothing of use, but `steghide` gives us something interesting.

It asks for a passphrase as usual, but leaving it blank still gives us the flag!

![Coverup Image](coverup.png)

And our flag is `flag{the_truth_is_burried_deep}`!

## Extraction

**Points:** 150

**Author:** Nate Singer (helix)

**Description:** Check out the pcap file, something weird is going on in here...

Both challenges for this section use the same pcapng file.

**Files:** [triage.pcapng](files/triage.pcapng)

### Writeup

Going back to our PCAP file, we know there is something else hidden in there.

Quite a few interesting things can be found, including a hostname, MEOWMEOW that seems to be doing interesting things on the network, and information on a bunch of Macs and iPads, but none of these are the flag.

However, looking at DNS requests, we find some interesting URLs...

![Extraction Image 1](extraction1.png)

The subdomain appears to be binary, and the first, `01100110`, translates to `f`, the beginning of the traditional `flag{}` format.

All of these use the hackabit domain, so we can search for that to collect the required subdomains.

Collecting all of these subdomains as binary and putting them into a binary translator such as `https://www.rapidtables.com/convert/number/binary-to-ascii.html`, we can get our flag!

![Extraction Image 2](extraction2.png)

And our flag is `flag{what_firewall?_what_IDS?}`!


# Range

## Connection

**Points:** 75

**Author:** Nate Singer (helix)

**Description:** This section is a series of challenges in a semi-isolated cyber range. Your goal is to compromise the boxes and get the flags. Your first challenge is more of a sanity-check/confirmation. We wanted to use private keys for this but logistics of distributing them was challenge so its just password login for now. Check your email, at exactly 5pm PST Friday you received a credential and IP address for this jumpbox. You can also use the connection info listed below.

You will use this jumpbox to attack other machines in the network. We've installed nmap, metasploit and netcat for your convience. If you want other tooling installed later please reach out to staff and will consider those requests as you ask. Remember that you can use techniques like proxychains over SSH to emulate much of this functionality.

**Links:** range.final.hackabit.com

### Writeup

**Note:** Unfortunately, the Range is down as I am writing this, after completing the competition, so I cannot provide screenshots.

This first challenge of the Range mostly just tests your ability to log into the jumpbox and find a flag on the filesystem.

We recieved an email with the proper login information (including a direct IP address of `range.final.hackabit.com`), which you then just had to connect to with SSH, using `ssh user@ip` and entering the given password when prompted.

I also ran `bash` to provide a more stable shell, including the current working directory in the prompt.

After logging in, you could find the flag either by traversing the filesytem manually or automating this.

One way is with the following command:

```bash
cat /*/flag.txt
```

If this didn't work, it would be possible to move up another level to `cat /*/*/flag.txt`.

However, this does work, and we get the flag of `flag{welcome_to_the_range}`!

## RightFace

**Points:** 100

**Author:** Nate Singer (helix)

**Description:** If you did the more advanced challenges during the qualifier this should already be familiar. Your goal here is to compromise the `10.128.0.5` machine and get access as the `breakme` user.

Remember that there may be non-vulnerable services on the machine. Recon is the #1 priority. Keep this shell open once you have it, you'll need it for `Left Face`.

**Links:** range.final.hackabit.com

### Writeup

**Note:** Unfortunately, the Range is down as I am writing this, after completing the competition, so I cannot provide screenshots directly of the machine.

Since local IP addresses are used, we must use the jumpbox for all parts of this challenge.

First, we can enumerate by running the following command:

```bash
nmap -sC -sV -v 10.128.0.5`
```

With the result of this, we find a `vsFTPd 2.3.4` service running on FTP port 21. When we try to connect to this port, we are able to download one file, `escalator`, but unable to download others, including `flag.txt`. However, since we can find this file here, despite not opening it, we know this is where we need to start.

Looking at `msfconsole`, we can search for this process and find an interesting exploit:

![RightFace Image 1](rightface1.png)

To use this exploit, we can use `use 0`, `show options` to show the available options, `set RHOSTS 10.128.0.5` to set the remote host, and `exploit` to run the exploit!

Once in, we can just read `flag.txt` and we get the flag, `flag{remember_this_one?_pays_to_be_a_winner}`!

## LeftFace

**Points:** 125

**Author:** Nate Singer (helix)

**Description:** With access to `10.128.0.5` your goal is to escelate priveleges to the `breakme-harder` user. Ultimately your goal is simply to read out the flag in `/home/breakme-harder/`.

**Links:** range.final.hackabit.com

### Writeup

From within our session in RightFace, we find an interesting file, `escalator.c`, compiled as `escalator`. 

`escalator.c` reads as follows:

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    FILE *file;
    char ch;

    // Check if a filename argument is provided
    if (argc < 2) { return 1; }

    // Open the file in read mode
    file = fopen(argv[1], "r");
  
    // Check if the file was opened successfully
    if (file == NULL) {
        printf("An error occured.\n");
        return 1;
    }

    // Read and print each character from the file
    while ((ch = fgetc(file)) != EOF) {
        putchar(ch);
    }

    // Close the file
    fclose(file);

    return 0;
}
```

We can see that this file allows us to read other files, so we can run `./escalator /home/breakme-harder/flag.txt` to get the flag!

And our flag is `flag{evaluate_this_my_dude}`!

## AboutFace

**Points:** 150

**Author:** Nate Singer (helix)

**Description:** Different box this time, your target is now `10.128.0.4`--straight to root. Remember that there may be non-vulnerable services on the machine. Recon is the #1 focus.

Once you have access to the box stay at the top of the hill and listen for flags on localhost:5000/tcp. You will get alive notices once every 60 seconds while you are connected so you can be sure that you'll receive flags at the specific release times. To see the release times check out the other section.

**Caviets** You are root. This means you can do whatever you want, for the most part. You are welcome to lock people out but only in specific ways. If you accidentially take down any of the existing services or remove Nate's ability to SSH in for monitoring the machine will be reverted to the last known-good snapshot and you will probably lose access. This also goes for changing the flag files. If we determine that the flags has been altered the machine will also be reverted. Lastly, if someone has concurrent access for four flags in a row we may block them from future access to give others the opportunity to attack the machine--we'll decide this in real time during the weekend depending on how things go.

**Links:** range.final.hackabit.com

### Writeup

From within the jumpbox, we can run the following command:

```bash
nmap -sC -sV -v 10.128.0.4`
```

This shows us a few ports, including ProFTPd on port 21 and MiniServ 1.890 Webmin on port 10000.

ProFTPd has a few vulnerabilities that can be found through Metasploit and Exploit-DB, but Webmin provides the specific version name which can allow us to find an exploit easier.

Performing a quick Google search, we can find [this POC](https://github.com/foxsin34/WebMin-1.890-Exploit-unauthorized-RCE/blob/master/webmin-1.890_exploit.py) of CVE-2019–15107, which provides unauthorized RCE on this specific version of Webmin.

Running this from the jumpbox, I was able to obtain remote code execution on the machine. My first attempt to spawn a shell was to add my SSH key to `authorized_keys` to then SSH into the machine for a more stable shell, but, as I later learned was due to @Eth007's use of `chattr +i`, I was unable to edit the `authorized_keys` file. I then tried a Python reverse shell, but Python was not present on the system, so I finally settled with Perl.

From this, I was able to spawn a shell to retrieve the key in `/root/flag.txt`, `flag{bestow_the_crown}`.

Then it was time for King of the Hill!

I didn't have much time before the machine was shut down, but I was able to spawn a listener through netcat.

I had a listener for a decent amount of the time I was on the machine, but did run into a few issues, including my listener's process being killed and netcat being deleted repeatedly. I was able to solve the netcat deletion by setting up a Python server on the jumpbox to send over a copy of `nc` to then run on the machine. However, before I could do anything further, the server was shut down by another competitor.

I had plans to patch the vulnerable service, Webmin, and set up a backdoor within the service, as well as potentially fix the SSH issues to add my key, but was unable to do so due to the shutdown.
