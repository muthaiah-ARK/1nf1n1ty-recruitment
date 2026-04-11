# Task 1 - Bandit Writeup

## Level 0 → 1
first i installed ssh in my laptop then i connected to the server
command i used: `ssh bandit0@bandit.labs.overthewire.org -p 2220`
password is bandit0 for first login
then i saw a file called readme so i just did `cat readme` and got the password
Password: 263JGJPfgU6LtdEvgfWU1XP5yac29mFx

## Level 1 → 2
the file name was `-` so i tried `cat -` first but it didnt work
it was just waiting and doing nothing, i didnt know why
then i learned that `-` means standard input in linux thats why it got stuck
so i used `cat ./-` which tells linux to look in current folder
that worked and i got the password
Password: MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

## Level 2 → 3
filename had spaces so i tried `cat spaces in this filename` first
that didnt work because linux thought each word was a different file
then i used tab key to autocomplete and it added backslashes before spaces
command: `cat spaces\ in\ this\ filename`
Password: 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

## Level 3 → 4
i did `ls` first but saw nothing in inhere folder
then i learned files starting with dot are hidden in linux
so i used `ls -la inhere/` to see all files including hidden ones
found a hidden file and used cat to read it
Password: 4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw

## Level 4 → 5
there were many files in inhere folder so i tried opening each one
most showed garbage characters because they were binary files
then i used `file ./-file*` command which tells what type each file is
found that -file07 was the only ASCII text file so i read that one
Password: HWasnPhtq9AVKe0dmk45nxy20cvUa6EG

## Level 5 → 6
there were so many folders i couldnt check each one manually
i used find command with the properties given - 1033 bytes and not executable
command: `find . -size 1033c -not -executable`
it directly showed me the file path and i read it
Password: morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj

## Level 6 → 7
file was hidden somewhere on whole server not just home folder
first i tried searching in home directory but found nothing
then i searched whole server using find with owner and group details
command: `find / -user bandit7 -group bandit6 -size 33c 2>/dev/null`
the 2>/dev/null part hides all permission error messages
found the file and read it
Password: dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc

## Level 7 → 8
data.txt was very huge file with millions of lines
i tried opening it with cat but it was too much to read
then i used grep to search for the word millionth directly
command: `grep millionth data.txt`
got the password next to that word
Password: 4CKMh1JI91bUIZZPXDqGanal4xvAg0JM

## Level 8 → 9
data.txt had many duplicate lines and password appeared only once
i tried reading file directly but too many lines
first i sorted the file so same lines come together
then used uniq -u which shows only lines that appear exactly once
command: `sort data.txt | uniq -u`
Password: FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey

## Level 9 → 10
file was binary so cat showed garbage characters
i tried reading it directly but nothing useful came
then i used strings command which extracts human readable text from binary
then filtered with grep for === because hint said password is near === signs
command: `strings data.txt | grep ===`
Password: dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr

## Level 10 → 11
file had weird characters, looked like random text
i tried cat but it was unreadable
then i checked and it was base64 encoded text
used base64 decode command: `base64 -d data.txt`
got normal readable text with password
Password: 7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4

## Level 11 → 12
file had readable text but password made no sense
i tried reading directly but it was scrambled letters
then i learned it was ROT13 which shifts every letter by 13 positions
used tr command to shift letters back: `cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'`
Password: FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn

## Level 12 → 13
this level took me longest time
file was a hexdump and also compressed many times on top of each other
first i reversed the hexdump using `xxd -r`
then i kept using `file` command to check what compression it was
then decompressed using gzip or bzip2 or tar depending on file type
had to repeat this many times until i finally got a text file
Password: MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS

## Level 13 → 14
this level had no password, instead had a private SSH key file
i tried looking for password file but there was none
then i saw sshkey.private file and used it to login to next level
command: `ssh -i sshkey.private bandit14@localhost`
Password: 8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo

## Level 14 → 15
had to send current password to a port on localhost
i tried browser first but that didnt work for raw ports
then used netcat which is a tool to connect to any port
command: `nc localhost 30000` then typed the password
it replied back with next password
Password: kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx

## Level 15 → 16
same idea as previous but normal netcat didnt work this time
connection was closing immediately without response
then i learned the port needed SSL encryption
used openssl command: `openssl s_client -connect localhost:30001`
sent password and got next one
Password: EReVavePLFHtFlFsjn3hyzMlvSuSAcRD

## Level 16 → 17
had to find correct port between 31000 and 32000
i tried connecting to random ports but most refused connection
then i used nmap to scan all ports in that range
command: `nmap localhost -p 31000-32000`
found open ports then tried openssl on each SSL port
one port returned an SSH private key instead of password
used that key to get next level password
Password: x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO

## Level 17 → 18
there were two files passwords.old and passwords.new
i tried reading both files but they had many lines
then i used diff command which shows differences between two files
command: `diff passwords.old passwords.new`
it showed exactly which line changed and that was the password
Password: cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8

## Level 18 → 19
when i tried to SSH in it immediately logged me out
i tried multiple times but same thing happened every time
then i learned .bashrc file was modified to run logout on login
i bypassed it by passing the command directly in SSH without opening shell
command: `ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"`
it ran the command before bashrc could logout
Password: 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO

## Level 19 → 20
there was a weird binary file in home directory
i tried running it normally but it said wrong usage
then i checked it and it was a setuid binary meaning it runs as bandit20
used it to read the password file that only bandit20 can access
command: `./bandit20-do cat /etc/bandit_pass/bandit20`
Password: EeoULMCra2q0dSkYj561DX7s1CpBuOBt

## Level 20 → 21
there was a program called suconnect that connects to a port
i tried running it alone but it needed a listener sending the password
so i opened two terminals, one for listener and one for suconnect
in first terminal: `echo "password" | nc -l -p 1234`
in second terminal: `./suconnect 1234`
suconnect connected, got the password, sent back next password
Password: tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q

## Level 21 → 22
i checked /etc/cron.d/ folder and found scripts running automatically
read the script and it was writing password to a file in /tmp folder
the filename was based on the username
i found the exact file path from the script and read it
command: `cat /tmp/the_filename_from_script`
Password: 0Zf11ioIjMVN551jX3CmStKLYqjk54Ga

## Level 22 → 23
cron script was more complex this time
it was creating filename using md5hash of a string
i read the script carefully and ran the same md5 command manually
command: `echo I am user bandit23 | md5sum | cut -d ' ' -f 1`
that gave me the filename in /tmp and i read it
Password: gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8

## Level 23 → 24
cron was executing any script placed in /var/spool/bandit24 folder
i wrote a small shell script that copies bandit24 password to /tmp
then gave it execute permission and placed it in that folder
waited for cron to run it then read the password from /tmp
Password: iCi86ttT4KSNe1armKiwbQNmB3YJP3q4

## Level 24 → 25
had to brute force a 4 digit PIN combined with password
i tried manually first but there were 10000 combinations
so i wrote a script to generate all combinations and send to port 30002
script tried all 0000 to 9999 and one of them worked
Password: s0773xxkk0MXfdqOfPRVr9L3jJBUOgCZ

## Level 25 → 26
SSH key was given but login kept closing immediately
i tried normal SSH but shell was not bash
then i found bandit26 was using more command as shell
i made my terminal window very small so more would pause
then pressed v to open vim inside more
then in vim used `:e /etc/bandit_pass/bandit26` to read password
Password: upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB

## Level 26 → 27
i was already inside vim from previous level
used vim command `:e /etc/bandit_pass/bandit27` to read next password
Password: Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN

## Level 27 → 28
had to clone a git repo from the server
i tried normal git clone first but needed special SSH port
used correct command: `git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo`
cloned it and found password in README file
Password: 4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7

## Level 28 → 29
cloned the repo but README showed password as hidden with stars
i tried reading different files but nothing useful
then i checked git history using `git log`
found a previous commit had the real password before it was hidden
used `git show commitid` to see old version
Password: qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL

## Level 29 → 30
cloned repo but README said no passwords in production
i tried reading all files but nothing
then i checked other branches using `git branch -a`
found a dev branch and switched to it using `git checkout dev`
README in dev branch had the password
Password: fb5S2xb7bRyFmAvQYQGEqsbhVyJqhnDy

## Level 30 → 31
cloned repo but README was empty and no useful files
i checked git log and git branch but nothing helpful
then i checked git tags using `git tag`
found a tag called secret
used `git show secret` and got the password
Password: 3O9RfhqyAlVBEZpVb6LYStshZoqoSx5K

## Level 31 → 32
had to push a specific file to the git repo
i created the file with required content
but .gitignore was blocking txt files from being added
i used `git add -f filename.txt` to force add it
then committed and pushed, server replied with the password
Password: tQdtbs5D5i2vJwkO8mEyYEyTL8izoeJ0

## Level 32 → 33
shell was converting everything i typed to uppercase
i tried all normal commands but they all failed because of uppercase
then i learned $0 is a special variable that refers to current shell
typed `$0` and it opened a normal bash shell
then used `cat /etc/bandit_pass/bandit33` to get the password
Password: tQdtbs5D5i2vJwkO8mEyYEyTL8izoeJ0
