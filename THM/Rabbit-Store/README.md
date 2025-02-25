<h1>Rabbit Store</h1>
<p><a target="_blank" href="https://tryhackme.com/room/rabbitstore">Challenge on TryHackMe</a></p>

<div style="display: flex; align-items: center;">
    <img src="imgs/logo.png" style="width: 25%">
    <p style="width: 75%; font-size: 17px">
        Demonstrate your web application testing skills and the basics of Linux to escalate your privileges.
    </p>
</div>

---

<br><br>

I started port discovery with a quick SYN scan using Nmap.  

![image](imgs/image-01.png)  

It appears that 4 ports are open. Now, I need to perform a version scan to identify the services running on the target and their versions.  

![image](imgs/image-02.png)  

The website provides a cloud storage service.  
After checking the web service, I noticed that it redirects to a custom domain: `cloudsite.thm`.  

![image](imgs/image-03.png)  

So, I added it to my `/etc/hosts` file.  

After some navigation on the website, I found a button that redirects to a subdomain: `storage.cloudsite.thm`.  

![image](imgs/image-04.png)  

I added it to the `/etc/hosts` file.  

Then, I found a login/signup form. I created an account, but I couldn’t do anything with it.  

![image](imgs/image-05.png)  

I checked the authentication token and found a JSON Web Token (JWT). I attempted various attacks, such as modifying the algorithm, downgrading it, removing the signature, and brute-forcing it, but none of them worked.  

![image](imgs/image-06.png)  
![image](imgs/image-07.png)  

Next, I tried a different method: **Mass Assignment** and it worked!  
I was able to activate my account during the registration process.  

![image](imgs/image-08.png)  
![image](imgs/image-09.png)  

Now, I could use the file upload functionality.  

![image](imgs/image-10.png)  

Here’s what it looked like when I uploaded a file. The server renamed my file with a random name, removing the extension. I could access its content at `/api/uploads/<new-file-name>`.  

![image](imgs/image-11.png)  

Then, I launched fuzzing on the `/api/` endpoint.  

![image](imgs/image-12.png)  

`common.txt` found 4 endpoints.  

![image](imgs/image-13.png)  

I tried accessing `docs` with and without my JWT access token, but both attempts resulted in *"Access denied."*  

![image](imgs/image-14.png)  

However, the `uploads` endpoint behaved differently. Without a token, it responded with *"Token not provided."*  
With a token, it returned a list of uploaded files.  

![image](imgs/image-15.png)  
![image](imgs/image-16.png)  

The website also offers a feature that allows file uploads via URLs.  
I set up a simple HTTP server and tested it. Everything worked as expected.  

![image](imgs/image-17.png)  

I intercepted the request with Burp Suite and modified it to point to the server’s `localhost`, hoping to retrieve the contents of the `docs` endpoint.  

![image](imgs/image-18.png)  
![image](imgs/image-19.png)  

Then, I requested the newly uploaded file by its name.  

**Booyah!!!**  
I could now access the documentation content.  

---

There was another API endpoint under development: `fetch_messages_from_chatbot`, which accepted `POST` requests.  

![image](imgs/image-20.png)  

First, I sent an empty JSON request and received an error message.  

![image](imgs/image-21.png)  

Then, I sent another request with a `username` parameter. I noticed that usernames were reflected in the response, which suggested a potential injection vulnerability.  

![image](imgs/image-22.png)  

After testing different injections, I found that the endpoint was vulnerable to **Server-Side Template Injection (SSTI)**.  

The template engine appeared to be Python’s `Jinja`.  

![image](imgs/image-23.png)  

I injected a reverse shell payload, which successfully connected back to my attacking machine.  

![image](imgs/image-24.png)  

I stabilized my shell.  

![image](imgs/image-25.png)  

And I got the user's flag.  

![image](imgs/image-26.png)  

---

During the enumeration stage, I found that the server was running a service called **Erlang Port Mapper**.  
To interact with it, I needed an **access cookie**.  

I found two files:  
1. generate_erlang_cookie.sh (owned by **root**), which generates the cookie content.  
2. The actual cookie file (owned by **rabbitmq**), which was readable by all users.  

![image](imgs/image-27.png)  

I used this cookie to interact with the Erlang node.  

![image](imgs/image-28.png)  
![image](imgs/image-29.png)  

Finally, I got the root hash.  

![image](imgs/image-30.png)  

---

For the final step (I forgot to take a screenshot):  
The hash included a salt (8 characters) and was encoded in Base64.  
To retrieve the actual root password, I had to reverse the encoding process:  

1. Decode it from Base64.  
2. Convert it to hexadecimal.  
3. Remove the first 8 characters (salt).
4. The result was the root password.  

At first, I attempted to crack it with **John the Ripper**, but it didn’t work. Then, I realized it wasn’t a hash.

it was the **actual password**.  

![image](imgs/image-32.png)  

---

*Thanks for making it this far!*  
**@r3dBust3r**  
