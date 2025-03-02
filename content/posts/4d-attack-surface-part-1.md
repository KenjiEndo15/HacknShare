---
title: "4D Attack Surface — Part 1"
date: 2024-03-02
lastmod: 2024-03-02
author: KenjiEndo
categories: [cybersecurity]
draft: false
---

[4D](https://us.4d.com/) is a company that provides a platform for developing applications.
It can help with tasks like building websites, running a web server, managing an SQL server, and much more.

The technology is relatively unknown in the world of web frameworks. Together with [Enzo Cadoni](https://enzo-cadoni.fr/), we decided to explore it and examine the 4D attack surface for penetration testing.

This series is divided into two parts. 
The first part introduces the technology, explains its core functionalities and explores template injection vulnerabilities. The second part delves into advanced template injection techniques and examines a range of related security issues: [4D Attack Surface — Part 2](https://enzo-cadoni.fr/posts/4dattacksurface_part2/).

> [!NOTE]
> 4D documentation already warns against potential misuse we'll describe in this blog post.
> Our work aims to provide pentesters with everything they need to perform their security assessment.

---

## 4D Usages in the World
A quick search on Shodan reveals that 4D is used by [6.363 web servers](https://www.shodan.io/search?query=%22Server%3A+4D%22) — according to the following filter `"Server: 4D"`. 

Despite this significant number, we have yet to come across detailed blog posts addressing its security. This is why we believe this preliminary analysis could benefit the community by expanding our understanding of the technology.

During penetration testing, it could help improve the testing of websites. Additionally, for 4D, it emphasizes the need for greater caution regarding its functionalities, which could potentially be exploited by malicious actors.

---

## Creating a Simple Page
The first step is to create a basic web page to get familiar with 4D. We'll first need to download, install and create a new project as described below.

### Download 4D (Trial)
Before downloading the trial version of 4D, you need to have an account. You can use a temporary one from [bugmenot.com](https://bugmenot.com/view/account.4d.com) on the [authentication page](https://account.4d.com/en/login.shtml).

After logging in, [download](https://us.4d.com/4d-free-trial) the trial version either on Windows or Mac.

> [!NOTE]
> For this blog post, we are using 4D 20 R7.

### Installing the Trial Version
Install 4D using the installer you downloaded in the previous step.

Once the installation is complete, you'll see two icons: one for [4D](https://doc.4d.com/4Dv20R7/4D/20-R7/Welcome.300-7270699.en.html) and another for [4D Server](https://doc.4d.com/4Dv20R7/4D/20-R7/4D-Server-Architecture.300-7425558.en.html).
The former will be used to create a lightweight web application. The latter can be used, for example, to manage websites or databases with remote access.

### Creating a Project
When you open 4D, you'll see a welcome page. To start your free trial, click on "New to 4D" and then "Click here to start a free trial." After that, you can create a new project and choose any name you'd like.

### Template Pages
Before we start writing some code, let's understand what are [Template pages](https://developer.4d.com/docs/20/WebServer/templates).

Template pages allow us to combine static HTML code with 4D references to create web pages more efficiently. 4D references are linked to [transformation tags](https://developer.4d.com/docs/20/Tags/tags):

> 4D provides a set of transformation tags which allow you to insert references to 4D variables or expressions, or to perform different types of processing within a source text, referred to as a "template".

There are many ways to use templates and tags to build web pages, but we’ll show a simple example in the next section.

> [!IMPORTANT]
> Two important points should be taken into consideration:
> 1. As the [documentation](https://developer.4d.com/docs/20/WebServer/templates#tag-parsing) says, the parsing of the HTML source code is not carried out by the 4D web server when HTML pages are called using simple URLs that are suffixed with `.html` or `.htm`. In order to "force" the parsing of HTML pages in this case, you must add the suffix `.shtm` or `.shtml`.
> 2. Methods have various settings that are disabled by default for [security reasons](http://cbeinfo.net/4ddocs/v13/4D/13.4/Connection-Security.300-1218003.en.html). One of these settings must be enabled to process transformation tags: "Available through 4D HTML tags and URLs (4DACTION...)".

### Creating a Form
This section explains how to create a simple web page with a form. The form will ask the user to input their name, and the web application will display it back to them.

#### Web Page
Before we start writing code in a 4D method, we need to create a file that will contain our web page. Its name will be `name.shtml`:

```HTML
<!DOCTYPE html>
<html lang="en" data-theme="light">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="description" content="4D template injection example" />
    <meta name="author" content="Enzo Cadoni & KenjiEndo15" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.blue.min.css" />
    <title>4D Template Injection</title>
  </head>
  <body>
    <main class="container">
      <header>
        <h1>4D Template Injection</h1>
      </header>
      <section>
        <div>
          <form action="/4DACTION/injection" method="POST">
            <input type="text" id="inputName" name="inputName" placeholder="Please enter your name..." required />
            <button type="submit">Submit</button>
          </form>
        </div>
      </section>
      <section>
        <div>
          <p>Your name is: <!--#4DHTML outputName --></p>
        </div>
      </section>
    </main>
  </body>
</html>
```

Let’s break down the two important lines:
- **Line 19**: The form uses the [action](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#attributes_for_form_submission) attribute to specify the URL (`/4DACTION/injection`) that will process the form submission. The `/4DACTION` path links any HTML object to the `/injection` method we'll later create. There are [many different paths](https://doc.4d.com/4Dv18/4D/18.4/URLs-and-Form-Actions.300-5232844.en.html), and some of them will be detailed in the second part of the blog post.

- **Line 27**: The paragraph contains an important element: `<!--#4DHTML name -->`. This is a transformation tag called [4DHTML](https://developer.4d.com/docs/19/Tags/tags#4dhtml). It allows us to access a 4D variable or expression that returns a value and insert it directly into the HTML as an expression.

#### 4D Method
We can start by creating a new method to hold our 4D code. For testing purposes, let's name the method “injection”.

> [!NOTE]
> The functionalities of methods are clearly explained in the [documentation](https://developer.4d.com/docs/Concepts/methods).

Before proceeding, it's essential to address the following questions briefly.

##### 1 - How to use 4D variables?
According to the following [documentation](https://developer.4d.com/docs/Concepts/variables), variables can be declared using the `var` keyword. 
It is important to note that when variables are declared,
> [...] they are initialized to the [default value corresponding to their type](https://developer.4d.com/docs/Concepts/data-types#default-values), which they will keep during the session as long as they have not been assigned. Alternatively, when declaring variables, you can initialize their value along with their data type all within one line.

> [!NOTE]
> Using the `var` keyword is not necessary, but encouraged.

A variable that holds text data can be declared and assigned as follows:
```plaintext
var $paramName : Text
$paramName:="inputName"
```

Alternatively, both can be done simultaneously as follows:
```plaintext
var $paramName : Text:="inputName"
```

##### 2 - How to retrieve and parse parameters sent through HTTP?
As mentioned [earlier](#web-page) with our `name.shtml` file, we are sending data through HTTP. 
Specifically, a string is sent within a parameter named `inputName`. 
To capture this parameter, 4D offers the use of [`ARRAY TEXT`](https://doc.4d.com/4Dv18/4D/18.4/ARRAY-TEXT.301-5232710.en.html) and [`WEB GET VARIABLES`](https://doc.4d.com/4Dv20/4D/20.5/WEB-GET-VARIABLES.301-7388211.en.html) commands.  

We can use `ARRAY TEXT` to initialize two arrays with explicit names (their size will be `0`, meaning they’ll adapt their size as necessary).
Finally, the `WEB GET VARIABLES` command will fill these two arrays with the retrieved HTTP parameter:  


```plaintext
ARRAY TEXT($paramsNames; 0)
ARRAY TEXT($paramsValues; 0)
WEB GET VARIABLES($paramsNames; $paramsValues)
```

##### 3 - How to assign a value for our output?
Again, as we’ve [seen](#web-page) with our `name.shtml` file, we print the user’s name through the `outputName` variable. 
Initially, the variable is undefined. We have to define it for the HTML page to display it.
Since we are going to just reflect the user’s name through its own input, we can assign the value of `inputName` to the `outputName` variable.
 
To do so, we first need to retrieve the `inputName`’s value contained in the `$paramsValues` array. 
To access the value, we need to indicate the variable name and the array `$paramsNames` in the `Find in array` command. 
Once we find the index where it lies, we can initialize the `outputName` variable with `inputName`’s value:


```
$indexUserName:=Find in array($paramsNames; $paramName)
outputName:=$paramsValues{$indexUserName}
```

##### 4 - How to send an HTTP response?
Finally, we can send back an HTTP response with the `WEB SEND HTTP REDIRECT` command.
The HTML page will successfully print the `outputName`’s value as it was initialized in the 4D method before sending back an HTTP response.

> [!NOTE]
> I'm actually clueless as to how it works internally.
> We didn't find detailed explanations besides this [blog post](http://cbeinfo.net/4ddocs/v13/4D/13.4/Binding-4D-objects-with-HTML-objects.300-1218005.en.html):
> > [...] after a Web form is submitted back, the value of an HTML object can be returned into a 4D variable. To do so, within the HTML source of the form, you create an HTML object whose name is the same as the name of the 4D process variable you want to bind.

We must specify a web page for the redirection:

```plaintext
var $webPage : Text:="/name.shtml"
WEB SEND HTTP REDIRECT($webPage)
```

### Testing our Web Application
Here’s our final code for our `injection` method:

> [!IMPORTANT]
> Don't forget to enable the processing of transformation tags by accessing the method's properties and selecting "[Available through 4D HTML tags and URLs (4DACTION…)](http://cbeinfo.net/4ddocs/v13/4D/13.4/Connection-Security.300-1218003.en.html)".

```plaintext
var $paramName : Text:="inputName"
var $webPage : Text:="/name.shtml"

// Retrieve the "inputName" parameter from the web request
ARRAY TEXT($paramsNames; 0)
ARRAY TEXT($paramsValues; 0)
WEB GET VARIABLES($paramsNames; $paramsValues)

// Change the value of the "outputName" variable by a user defined string in the "inputName" variable
var $indexUserName:=Find in array($paramsNames; $paramName)
outputName:=$paramsValues{$indexUserName}

// Redirect to a specific web page ("/name.shtml")
WEB SEND HTTP REDIRECT($webPage)
```

To start our web server, we can select “Run” at the top of the 4D window, then choose “Start Web server” and “Test Web server”. We can access our index page through `http://127.0.0.1/`.  

When we browse the `http://127.0.0.1/name.shtml` page, we can see that initially, the variable `outputName` is undefined (`<!--#4DHTML outputName--> : Undefined`):


{{< image src="/images/4d-attack-surface-part-1/4d-page1.png" caption="" >}}

When we enter an arbitrary name, it’s successfully reflected on the page thanks to our transformation tag:

{{< image src="/images/4d-attack-surface-part-1/4d-page2.png" caption="" >}}

Our simple web form is functioning correctly.

---

## Template Exploits
Template injection is a well-known web vulnerability class, which can be [client-side](https://portswigger.net/kb/issues/00200308_client-side-template-injection) or [server-side](https://portswigger.net/web-security/server-side-template-injection). 
The vulnerability arises from uninitialized data being sent directly to the template engine. 
This data is then evaluated and executed. In the worst-case scenario, the web server can be compromised, as we’ll demonstrate later on.

### Probing for Template Injection
A technique to identify template injection is to send data into user-controlled inputs that are evaluated and executed. 
The data can be arbitrary, ranging from simple mathematical operations to executing shell commands.


Typically, the payload value used for template engines like Jinja2 is `${7*7}` (a simple multiplication). 
The result would be `49`. So in Jinja2, if you enter the payload inside a text form and the returned value is `49`, it means you've successfully demonstrated a potential template injection.  

In 4D, the principle is the same. However, instead of sending `${7*7}`, we will send the following payload:

```plaintext
<!--#4DEVAL 7*7-->
```

So, what is this? The payload is a [4DEVAL](https://developer.4d.com/docs/19/Tags/tags/#4deval) transformation tag which evaluates any 4D expression. 
We can demonstrate it with our dummy web application:

{{< image src="/images/4d-attack-surface-part-1/4d-page3.png" caption="" >}}

We can indeed see that the displayed name is `49`.

### Delivering Malicious Payloads
Now that we’ve found a way to probe for template injection, we can attempt to send malicious payloads to the web server.

#### Payloads Examples
As we've already seen, 4D has many different commands at its disposal. 
One of them is [`QUIT 4D`](https://doc.4d.com/4Dv18/4D/18.4/QUIT-4D.301-5233472.en.html):  
> The `QUIT 4D` command exits the current 4D application and returns to the Desktop.

This can lead to an interesting Denial of Service (DoS) during a template injection. 
By using 4D Server to host our web application locally, we can observe just how impactful this command can be.

The payload is as simple as this:

```plaintext
<!--#4DEVAL QUIT 4D-->
```

This can cause the 4D Server to quit entirely, closing access for all users.

On a different note, we can also execute client-side code, such as JavaScript.
Using the well-known probing payload `alert()`, it will pop up a window in the browser:

```plaintext
<script>alert("XSS - Template Injection")</script>
```

This will be inserted into the HTML code as follows:

```
      <section>
        <div>
          <p>Your name is: <script>alert("XSS - Template Injection")</script></p>
        </div>
      </section>
```

{{< image src="/images/4d-attack-surface-part-1/4d-page5.png" caption="" >}}

If the value is persistent and displayed to all users of the application, we'd get a [stored XSS](https://portswigger.net/web-security/cross-site-scripting/stored) that could cause some damage.

#### Remote Code Execution
The "most powerful" technique for leveraging 4D template injection is the use of the [SystemWorker](https://developer.4d.com/docs/API/SystemWorkerClass) class.

To put it simply:
> System workers allow the 4D code to call any external process (a shell command, PHP, etc.) on the same machine.

To launch the `ipconfig` utility on Windows OS, we can use the following command (example based on the official documentation):
```plaintext
var $myWinWorker : 4D.SystemWorker
var $ipConfig : Text
$myWinWorker:= 4D.SystemWorker.new("ipconfig")
$ipConfig:=$myWinWorker.wait(1).response
```

An instantiation is created by sending a command line to the constructor via [4D.SystemWorker.new](https://developer.4d.com/docs/API/SystemWorkerClass#4dsystemworkernew).
This starts an external process that launches the `ipconfig` command.

> [!IMPORTANT]
> A [.wait()](https://developer.4d.com/docs/API/SystemWorkerClass#wait) value can be set to wait for the completion of the `SystemWorker` execution or for the specified [.timeout()](https://developer.4d.com/docs/API/SystemWorkerClass#terminate).
> 
> If not set correctly, this can cause issues, potentially preventing you from receiving a result from your command. This can happen if you don't specify `.wait()` or set a value that is too low for the command's duration.

We can now create a concise one-liner using the `4DEVAL` transformation tag.
The one-liner will be placed inside a vulnerable, user-controlled input, which could trigger the command execution on the web server:
```plaintext
<!--#4DEVAL 4D.SystemWorker.new("ipconfig").wait().response-->
```

> [!IMPORTANT]
> For added stealth, the [.hideWindow](https://developer.4d.com/docs/API/SystemWorkerClass#hidewindow)
> option can be used to conceal the DOS console window or the window of the launched executable on Windows OS.

To take advantage of our finding, we simply need to start a reverse shell to connect back to the web server!

*Since this isn't the focus of this blog post, no additional examples will be provided.*

### Transformation Tags & Best Practices
The company behind 4D is already aware of the exploitability of transformation tags.
The [documentation](https://developer.4d.com/docs/WebServer/templates) states:

{{< image src="/images/4d-attack-surface-part-1/4d-page6.png" caption="" >}}

**TL;DR**: The `4DTEXT` tag should be used to escape special HTML characters to avoid interpreting malicious code.

---

## End Notes
This blog post discussed an exploit already addressed in the 4D documentation.
However, it may serve as an introduction for pentesters, providing them with the information needed to test 4D instances effectively.

The second part of the blog post will explore advanced template injection techniques and other potential issues that may not be sufficiently warned about in the 4D documentation: [4D Attack Surface — Part 2](https://enzo-cadoni.fr/posts/4dattacksurface_part2/).