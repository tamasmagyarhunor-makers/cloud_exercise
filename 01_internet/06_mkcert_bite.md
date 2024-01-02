# mkcert

Learn to explain how Internet connections are secured.

_Note: While the principles and explanation are the same, the exact steps in the
video differ slightly from the commands below. Use the commands below instead of
the ones in the video, as the new commands avoid `openssl` version issues._

[Video Alternative](https://www.youtube.com/watch?v=Ivym1ZaBxfI&t=766s)

## The question: How are Internet connections secured?

Here's an interesting problem — how do you send me a secret message?

You and I might never meet in person, so if you want to send me a message it'll
probably be over the Internet. Remember `traceroute` and how packets go across a
network of many different routers? Do you trust all of those computers?

Perhaps you do. But imagine if it was so secret that you needed to be absolutely
certain that no one in the middle could ever find out. How would you do it?

Here's one way: you and I could agree on a secret password. We could use that
password to encrypt our data. OK that sounds good. Let's try it out. Here's my
data:

```
-----BEGIN AGE ENCRYPTED FILE-----
YWdlLWVuY3J5cHRpb24ub3JnL3YxCi0+IHNjcnlwdCA3bVd3ekJQSm1SdDZLYlRO
QTVyVHV3IDE4Cmlhdm5FYnR6UTFkR1lJdFRZSmVFUmVFMGc3SmNaTjkxdnU2cTNQ
WjR6V2cKLS0tICtWQXJoeGx4d2x0U1hxMWR2dWFkWW9hc3Q2ckJZb3E0MUhSK3hR
ZmhHbXMKpppH/I93wmhK4PgY2EO5obh0mt4/yucR1rM1H+i/4nH7nY70vo/KmhdO
ttYKPm+HfAflt9Q65aKDK8FRqaXgJjPr9/vxFzl9i977Av3TyRF+uOUhaCi9xuIl
R5QGPt0MdC1As1T8AHzWxK7F7cwcG0y1uH6GIWkdLBMjP5mtxeyfpKC3MFQpgb7/
MxYUVy2/PIU9qR2t5PcSCl3wYL/qBSfdSR/FPF5yVBVksljH9w4Z8mEl4zejZOp7
nF12MRd5J7Fprq09RUH0rShyXwKvmLvvMm84wV9SLprXxQiTexHa7gLdoUBTVxMQ
cXf1pvJStC9Fjws50lYdmB74kg5IRaAbasD2w3LJQFa9dUrpRWNDWtHOqqdKBtUX
KF6v3jiJL8LSAvlDaNLSJxDZDkdYRSm9d7l2d6jUqp/qOKfS4kta+NRYCW43
-----END AGE ENCRYPTED FILE-----
```

Save it to a file `secret.txt.enc` and you can decrypt it using this command:

```shell
# First install `age`, an encryption tool
; brew install age
; age -d secret.txt.enc
```

You just need the password... oh wait, no, how am I going to send you the
password? The only way I can do that is by using the Internet. I suppose I could
try to hint at it? It's the name of the music artist I'm listening to at the
moment, lowercase...

But no, this clearly won't work. Not if we need to set up an Internet where
billions of people need to send secret information between each other daily. 

This problem is called the key distribution problem. To communicate securely
over distance you need a secret key, and sending that secret key in an insecure
way doesn't help because someone in the middle could steal the key and use it to
decrypt all your messages.

This problem went unsolved until 1976. It was solved by something called
Public-Key Cryptography. A team of mathematicians worked out how you could
generate a pair of keys, one public and one private. The public key can be used
to encrypt messages, but only the private key can be used to decrypt them.

Here's how we could use this to allow you to send me a secret message:

1. I'll generate a public and private key.
2. I'll publish my public key right here.
3. You use it to encrypt your secret message
4. Send your encrypted message to me on Slack (I'm Kay Lack)
5. I'll decrypt it using my private key and respond.

The public key allows you to encrypt messages, but only the _private_ key can
decrypt them. So no one in the middle will be able to decrypt it.

Here's my public key:

```
age136adjknxx86sv6xjkcnnfsxs4hqp64w4g5glrf8sspr57att8dhszzwqwa
```

You can use it to encrypt data with this command

```shell
; age -a -r age136adjknxx86sv6xjkcnnfsxs4hqp64w4g5glrf8sspr57att8dhszzwqwa
# Type in a message and then hit ctrl+d to finish
```

You'll see a big chunk of letters and numbers. You can send these to the person
who has the corresponding private key and only they will be able to decrypt it —
even if they got hold of your message and the public key you used up there to
encrypt it.

I've set up the Makers Slack bot `Gnaw` to demonstrate this. It has the private
key for the above message. DM the output to [Gnaw on
Slack](https://makersstudents.slack.com/archives/D9RJ9TSR4) and it will read
your message back to you.

<!-- OMITTED -->

<details>
  <summary>:unamused: A bot? Seems a bit impersonal...</summary>

  ---

  If you like, we can exchange some encrypted messages together. You can send me
  an encrypted message only I can read, and then I'll send you an encrypted
  message only you can read.

  To do this, you'll need your own private & public key pair. Generate one using
  this command:

  ```shell
  ; age-keygen -o ~/Desktop/my_secret_key.txt
  ```

  This will generate a private key in `my_secret_key.txt` and a public key,
  which it will print to the terminal and also save a copy of in the file.

  Keep `my_secret_key.txt` safe! The above command puts on your Desktop so
  you'll know where it is. You can put it elsewhere if you like, just don't lose
  it.

  Now encrypt a message to me using *my* public key:

  ```shell
  # Note this public key is different from the one above
  ; age -a -r age1vpc8mry0q5ncp56ve7p9judgs92lejyyulaht622elnqdvhjmqmqsrwyth
  # Type your message here...
  # End it by telling me your own public key. But don't send me your secret key!
  # You'll know it is secret because it starts with `AGE-SECRET-KEY`.
  # When you're done hit ctrl+d to finish
  ```

  Then send the encrypted message to me via email at `kay@kaylack.net`. I'll
  send you one back using your public key. You can decrypt it using your private
  key via this command:

  ```shell
  ; age -d -i ~/Desktop/my_secret_key.txt
  # Paste in the encrypted message
  ```

  ---
</details>

Let's see how this technique is used to secure web connections.

## The tool: `mkcert`

We're going to set up a local web server and create a secure connection with it.

First, create a new Flask app. Here's a quickstart:

```shell
; mkdir secure_flask
; cd secure_flask
; pipenv install --python 3.11 flask
; pipenv shell
; touch app.py
```

```python
# File: app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello, world!'

if __name__ == '__main__':
    app.run(port=4321)
```

```shell
; python app.py
# Server starts
; open http://localhost:4321/
```

Note that when you visit the page, your browser won't show the green padlock to
show the connection is secure. If you click into the icon in the address bar you
will see something like this:

![A browser notice to say that the website connection is not
secure via the "secure site certificate" button in Safari](../resources/insecure-notice.png)

You can also open `termshark` and look at the contents of the packets:

```shell
; termshark -i lo0 -f "port 4321"
# Now reload http://localhost:4321/
```

Here's my packet capture:

![Termshark screenshot showing unencrypted
packets](../resources/termshark-insecure-http.png)

Note that you can see the contents "Hello, world!". If this weren't localhost,
and was instead a website on the Internet, any of the routers in the middle
could see that contents!

Now we're going to use a tool called
[`mkcert`](https://github.com/FiloSottile/mkcert) to create a certificate for
our server. This will allow us to create a secure connection.

```shell
; brew install mkcert
; mkcert -install
# This will install a root certificate on your computer so that you can trust
# certificates created by mkcert.
# You can ignore "ERROR: no Firefox security database found" if you see it.
; mkcert localhost
# This will create a certificate pair valid for localhost and store them in
# `localhost.pem` and `localhost-key.pem`. You can take a look at them if you
# like.
```

Now let's configure our Flask app to use the certificate:

```python
# Replace the two lines at the end of `app.py` with these:
if __name__ == '__main__':
    app.run(port=4321, ssl_context=('localhost.pem', 'localhost-key.pem'))
```

Now restart the server and open the page again:

```shell
; python app.py
; open https://localhost:4321/
```

<details>
  <summary>:confused: I get 'This site can't be reached!'</summary>

  ---

  Make sure you've got `https` at the start of the url _with the `s`_. It won't
  work if you haven't.

  Contact your coach if you're still having trouble.

  ---

</details>

Now when you click into the icon in the address bar you will see something like
this:

![A browser notice to say that the website connection is
secure via the "secure site certificate" button in Safari](../resources/secure-notice.png)

And if you try to snoop on the connection with `termshark` you'll see that the
contents are encrypted. You won't be able to see the plain text contents of the
packets.

## Investigations

_These exercises are marked with :hot_pepper: emojis to denote how challenging
they are. A single chilli :hot_pepper: is the most straightforward, and five
:hot_pepper::hot_pepper::hot_pepper::hot_pepper::hot_pepper: would be
challenging even for a professional engineer. Pick whichever you prefer._

This is a set of questions you can investigate to learn more. Pick the ones that
interest you.

* :hot_pepper: When is it appropriate to share your private key?
* :hot_pepper::hot_pepper: What is meant by trusting or not trusting a
  certificate?
* :hot_pepper::hot_pepper: What is Let's Encrypt? What was its purpose and what
  impact has it had on the web?
* :hot_pepper::hot_pepper: What other protocols use public-key cryptography?
* :hot_pepper::hot_pepper::hot_pepper: So we can secure HTTP, but what about
  DNS? Can anyone see your DNS lookups and thus the domains you're intending to
  visit?
* :hot_pepper::hot_pepper::hot_pepper::hot_pepper: Let's say I gained control of
  a router between you and your bank's website. When you connect via HTTPS and
  ask for the server's public key, in principle I might intercept this and send
  my own public key to you instead. Then when you encrypt your bank details, I
  can decrypt them with my private key. What technologies and systems prevent
  this? In what situation might they fail?

<!-- OMITTED -->


[Next Challenge](07_docker_bite.md)

<!-- BEGIN GENERATED SECTION DO NOT EDIT -->

---

**How was this resource?**  
[😫](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😫) [😕](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😕) [😐](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😐) [🙂](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=🙂) [😀](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😀)  
Click an emoji to tell us.

<!-- END GENERATED SECTION DO NOT EDIT -->
