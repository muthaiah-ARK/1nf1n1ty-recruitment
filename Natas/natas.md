# Task 2 - Natas Writeup

## Level 0 → 1
i just opened the page and saw nothing special
then i right clicked and clicked view page source
i saw some html code and found a comment with the password in it
i didnt know what html comment was before this level
Password: TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI

## Level 1 → 2
i tried right clicking but nothing happened, right click was blocked
i was confused for some time thinking how to view source now
then i remembered keyboard shortcut Ctrl+U opens source directly
opened it and found password in comment same as before
so blocking right click is useless if keyboard shortcuts work
Password: 3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH

## Level 2 → 3
page just said there is nothing here
i checked source and saw an image tag with files/pixel.png
i thought if there is a files folder maybe i can open it directly
so i typed /files/ at end of url in browser
it showed a directory listing and i found users.txt there
opened it and got the password
Password: QryZXc2e0zahULdHrtHxzyYkj59kUxLQ

## Level 3 → 4
source had a comment saying not even google will find it
i was thinking what does google have to do with this
then i learned google uses robots.txt to know which folders to skip
so i opened /robots.txt and it showed a folder called /s3cr3t/
i opened that folder and found users.txt with password inside
Password: 3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH

## Level 4 → 5
page said access denied and i need to come from natas5 url
i had no idea what that meant at first
then i learned about Referer header which tells server where you came from
browser sends this automatically but we can fake it using curl
i used curl with -H "Referer: http://natas5.natas.labs.overthewire.org/"
server checked referer and thought i came from natas5 so gave access
Password: QryZXc2e0zahULdHrtHxzyYkj59kUxLQ

## Level 5 → 6
page said i am not logged in
i opened f12 and went to application tab then cookies
saw a cookie named loggedin with value 0
i just changed 0 to 1 and submitted the form
server trusted the cookie and gave me access
learned that client side cookies should never be trusted
Password: 0n35PkggAPm2zbEpOU802c0x0Msn1ToK

## Level 6 → 7
there was a box asking to enter secret
i clicked view sourcecode and read the php code
code was including another file called includes/secret.inc
i just opened that file url directly in browser
it showed the secret value so i entered it in the box and got password
Password: 0RoJwHdSKWFTYR5WuiAewauSuNaBXned

## Level 7 → 8
page had home and about links
i noticed url was changing like ?page=home and ?page=about
source code had a hint saying password is in /etc/natas_webpass/natas8
i just replaced home with that file path in url
server opened that file and showed the password directly
this was my first time seeing file inclusion vulnerability
Password: xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q

## Level 8 → 9
there was input box for secret again
this time sourcecode showed encoded secret string
encoding was base64 then reversed then hex, three steps
i had to do all three in reverse order to get original
first i tried php but it wasnt installed so used python3
command i used:
python3 -c "import base64, binascii; print(base64.b64decode(binascii.unhexlify('3d3d516343746d4d6d6c315669563362')[::-1]).decode())"
got oubWYf2kBq and entered it and got the password
Password: SdqIqBsFcz3yotlNYErZSZwblkm0lrvx

## Level 9 → 10
page had search box to find words in dictionary
i saw in sourcecode that my input was directly put inside grep command
no filtering at all on what i type
first i tried semicolon trick but it kept loading and not working
then i realised i can just add another file path to grep command
grep can search multiple files so i typed: a /etc/natas_webpass/natas10
grep searched both files and printed password from password file
Password: t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu

## Level 10 → 11
same search box but now some characters were blocked
semicolon, pipe and & were filtered this time
but they only blocked those specific characters
spaces and file paths were still working fine
so i used exact same trick as level 9 since grep trick didnt need those chars
payload: a /etc/natas_webpass/natas11
Password: UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk

## Level 11 → 12
cookie was xor encrypted this time
i checked sourcecode and understood how encryption worked
i used python to find the xor key by comparing original and encrypted data
then i changed showpassword value from no to yes in the data
encrypted it again with same key and sent using curl
took many tries because browser kept overwriting my cookie
finally used curl directly to send modified cookie
Password: yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB

## Level 12 → 13
page had file upload saying jpeg only
i checked sourcecode and found server uses a hidden field for filename
it doesnt use actual uploaded filename at all
so i opened f12 elements tab and found hidden input field
changed the value from random.jpg to shell.php
created a php file that reads password file
uploaded it and clicked the generated link
php ran on server and printed the password
Password: trbs5fFML1MqmDMhfkD9bMHC4vRiNmBk

## Level 13 → 14
same upload but now it checks if file is actually an image
i tried just uploading php file but got rejected saying not an image
then i tried adding fake jpeg header bytes but that also failed
finally i used imagemagick to create a real small jpeg image
then appended my php code to end of that real image file
image check passed because start of file was valid jpeg
changed hidden filename field to .php same as before
clicked link and password showed up
Password: z3UYcr4v4uBpeX8f7EZbMHlzK4eMgciT

## Level 14 → 15
login form with username and password
sourcecode showed my input directly used in sql query with no filtering
i tried normal login first obviously didnt work
then i tried putting quote in username and got sql error
that confirmed sql injection was possible
tried payload: " or 1=1# in username with empty password
query became always true and i got logged in
had to use curl because browser form was giving issues
Password: SdqIqBsFcz3yotlNYErZSZwblkm0lrvx

## Level 15 → 16
only shows user exists or not, password not shown anywhere
this is blind sql injection, much harder than normal
i cant see the password directly so had to guess character by character
wrote python script that checks each character one by one
used BINARY keyword to make it case sensitive otherwise missed uppercase
script ran through all letters and numbers for each position
took around 3 minutes to get full 32 character password
Password: hpkjkyvilqctew33qmuxl6edvfmw4sgo
