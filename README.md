# CyCTF The Priate Bank Web Challenge (400 Point - Hard)
the challenge gave us some of its source code so we can review it, after some code review we had a simple bank web application that allow us to transfer balance between accounts and if i have more than 1,000,000$, i will be VIP then access another feature called "complain" that allow us to upload files.

first we need to get 1 million in balance to be vip to access complain page.

after code review i found everything ok, good sqli filters, good session managment (IS IT?).

i tried a lot of things but with no luck :(, after looking at reset password code i found the reset code random int between 0-99999999 and yeah i have a bad idea by opening first account (id = 1) by bruteforcing this pin code xd, (i tried this really xdd) 100 million request not too much if i had a lot of VPSs, in a min i got 10x VPSs with 16GB Ram from digitalocean and made an ssh script to access every VPS via ssh and run script to brute forcing the pin code, every VPS has a range to bruteforce, so VPS 1 will do from 0 to 5 million and VPS 2 from 5 million to 10 million... so on.

after 2 hours i got the account password :D (not too bad solution xd), but wait a minute the account has only 17k -_- so my walkthrough is bad and i had no time :"(
so i reviewed the the whole code again and finally found the bug, every php file verify the session by 2 session variables: `username` and `authenticated`
i can control `username` in the session by reseting the password for that user
```php
$_SESSION['username'] = (int)$_REQUEST['username'];
```
but i can't control `authenticated` variable and the only way to set it is by logging in, and all php files check it but i found transfer money file only check `username`
```php
if (!isset($_SESSION['username'])) {
    die("Unauthorized!");
}
```

![](https://github.com/D4rkTT/CyCTF-The-Priate-Bank-Web-Challenge/blob/main/noice.gif)

Now i can transfer money from any account by reseting the password for this account and use the same session in transfering the money to my account :D
i wrote a script to do that for me you can find it in this repo.
after few seconds i got 1 million and now i can upload my reverse shell but sadly theres a ext. filter to upload JPG or PNG only :"(
so after reviewing the code i found that the code move the uploaded file to `uploads` directory then check for the ext. if not JPG or PNG delete it
```php
move_uploaded_file($_FILES["upload"]["tmp_name"], $target_file);
if (checkFileType($target_file)) {
    echo "Thank you for contacting us, we will get back to you shortly";
} else {
    unlink($target_file);
    echo "Sorry, there was an error uploading your file.";
}
```
Now i can exploit the time between moving the file to uploads dir. and ext. checking to execute my php loader that will create another php reverse shell.

but sadly the uploaded file name changed to be sha1 hashed random number using this function:
```php
...
$target_file = $target_dir . sha1(super_random()) . "." . $ext;
...
function super_random()
{
    $rand = rand(0, 100);
    for ($i = 0; $i < 100; $i++) {
        $rand = $rand * rand(0, 100);
    }
    return $rand;
}
```
But this is not random as its name xd, the `$rand` variable can be 0 so the output of the function will be 0 so the hash will be `b6589fc6ab0dc82cf12099d1c2d40ab994e8410c`

so we will start a continuous loop to upload our loader and another loop on another thread trying to execute our php loader.
and now we got a php reverse shell :D,
now we can get the flag by `cat ../../flag.txt`
