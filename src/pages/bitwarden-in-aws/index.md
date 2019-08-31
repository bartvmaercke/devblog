---
title: run bitwarden in aws
date: "2019-09-01"
---

Running a self hosted bitwarden in the cloud sounded pretty nifty to me. And Since there is serverless implementation that runs in the aws free teer, it sounded like an easy job with a lot of new things to learn for me.

<!-- end -->

I already had Visual Studio Code and Git on my computer. So I just needed to installed node.js.  And for the fun of it, I also installed aws toolkit for visual studio code.  

You can find node.js on <https://nodejs.org/en/download/>  
The bitwarden implementation can be found here: <https://github.com/vvondra/bitwarden-serverless>

You will need an acces key to connect to AWS. To find your Access Key and Secret Access Key:  
>Log in to your AWS Management Console.  
Click on your user name at the top right of the page.  
Click on the Security Credentials link from the drop-down menu.  
Find the Access Credentials section, and copy the latest Access Key ID.  
Click on the Show link in the same row, and copy the Secret Access Key.  

First you need to install serverlessA and serverless-webpack. Afterwards I check if it's running by checking the version. There's probably a shorter version for this, but this worked for me.

```bash
npm install -g serverless
npm install -g serverless-webpack
npm install
serverless --version
```

then I tried to enter my credentials with `sls config credentials --provider aws --key I...F --secret t...z`  
but got an error message about Select-String not beeing formatted correctly  
There are two solutions for this, remove the alias (temporary) 

```powershell
Remove-Item alias:sls
```

or just use the full command "serverless"

```bash
serverless config credentials --provider aws --key I...F --secret t...z
```

download <https://github.com/vvondra/bitwarden-serverless>  
go to dir  

```bash
serverless deploy --region us-east-1 --stage prod
```

you get an url in the response like this:  
https://yblablaf.execute-api.us-east-1.amazonaws.com/prod/

* intall bitwarden plugin  
* click gear icon  
* enter the api url you just got  
* create user in bitwarden plugin  
* log in with newly created login  

mfa was not so easy to setup. Tried running the bash script in wsl - Ubuntu 18.04, but it failed too.
connect with aws toolkit for visual studio code

```
Invoke function bitwarden-serverless-prod-two_factor_setup
{
  "email": "myemail@domain.com"
}
```

got  Payload:
"data:image/png;base64,i..T="
used <https://onlinepngtools.com/convert-base64-to-png> to convert base64 to png
scanned png with google authenticator

```
Invoke function bitwarden-serverless-prod-two_factor_complete
{
  "email": "myemail@domain.com",
  "code": "123456"
}
```

Testing it in firefow and on the android app: it all seems to work.
