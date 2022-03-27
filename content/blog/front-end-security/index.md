---
title: Frontend Security
date: 2021-06-28
---

## XSS

The full name of XSS (Cross Site Scripting) is a cross-site scripting attack, which is the most common security problem of the front-end. XSS is a computer security vulnerability in web applications. It allows malicious web users to implant code into pages that are provided to other users. Attackers can inject illegal html tags or javascript codes to allow users to browse the web page. When, control the user's browser.

### DOM XSS

Use the flaws in the DOM itself to attack. The src will definitely fail to load, and then The malicious code injected in onerror will be executed to achieve the attack effect.

### Reflective XSS

Reflected XSS is also called non-persistent XSS, which is the most prone XSS vulnerability. The XSS code appears in the URL, and the attack is carried out by enticing the user to click on a malicious link that links to the target website.
The malicious link is as follows, where xxx is malicious code. After the parameter data passed to the server is received by the server, if the response page contains the variable data, malicious code will be injected into the page for attack.

### Storage Type XSS

Stored XSS is also known as persistent XSS. It is the most dangerous type of cross-site scripting. Compared with reflective XSS and DOM XSS, it has higher concealment, so it is more harmful and does not require users to manually trigger it.
When an attacker submits a piece of XSS code, it is received and stored by the server. When all viewers visit a certain page, they will be XSSed. The most typical example is the message board.

## XSS Solutions

### Filter

Filter the user's input, and remove the Style node, Script node, and Iframe node input by the user by escaping characters such as `<>` `''` `""`.

```js
function filterXss(str) {
  var s = ""
  if (str.length == 0) return ""
  s = str.replace(/&/g, "&amp;")
  s = s.replace(/</g, "&lt;")
  s = s.replace(/>/g, "&gt;")
  s = s.replace(/ /g, "&nbsp;")
  s = s.replace(/\'/g, "&#39;")
  s = s.replace(/\"/g, "&quot;")
  return s
}
```

### Encoding

Corresponding encoding is performed according to the context in which the output data is located. Data is placed in HTML elements and needs to be HTML encoded, and placed in URLs, needs to be URL encoded. In addition, there are JavaScript coding, CSS coding, HTML attribute coding, JSON coding and so on.

### Http Only

Set the HttpOnly attribute in the cookie so that the js script cannot read the cookie information. .

## CSRF

The full name of CSRF (Cross-Site Request Forgeries) cross-site request forgery. Refers to the attacker pretending to be a user to initiate a request (without the user's knowledge), and to accomplish something against the user's wishes.

Solutions:

- Use token

The server generates a token and stores it in the session, and at the same time sends the token to the client. When the client submits the form, the client takes the token. The server verifies whether the token is consistent with the session. If the token is consistent with the session, access is allowed, otherwise access is denied.

- Referer Verification

Referer refers to the source of the page request, which means that the server only responds if the request from this site is accepted; if it is not, it will be intercepted.

- Use verification code

For important requests, the user is required to enter a verification code, forcing the user to interact with the application to complete the final request.

## Clickjacking

Click hijacking is to make a dangerous website transparent, and then set a button above it. When you click this button, it will trigger certain events of the malicious website at the bottom.

Solutions:

- Set http response header X-Frame-Options

The X-Frame-Options HTTP response header is used to indicate to the browser whether a page can be displayed in `<frame>`, `<iframe>` or `<object>`. Websites can use this feature to ensure that the content of their own website is not embedded in others' websites.

- Use CSP (Content Security Policy) content security policy

## Insecure Third-Party Libraries

Nowadays, in application development, whether it is back-end server application or front-end application development, most of the time we use development frameworks and various class libraries for rapid development. However, some third-party dependencies or plug-ins have many security problems, as well as vulnerabilities of this kind, so use them with caution.

Solutions:

- Minimize using third-party dependencies and choose relatively mature dependency packages.

- Use automated tools to check whether these third-party codes have security issues, such as NSP (Node Security Platform), Snyk, and so on.

## Local Storage Data Breach

For the convenience of many developers, some personal information is directly stored locally or in a cookie without encryption. This is very insecure. Hackers can easily get the user's information.

Solutions:

- Don't store important data locally. Do not store sensitive and confidential information locally.

- Encryption: All the information in the cookie or localStorage should be encrypted. For encryption, you can define some encryption methods yourself or find some encrypted plug-ins on the Internet, or use base64 to encrypt multiple times and then decode multiple times.
