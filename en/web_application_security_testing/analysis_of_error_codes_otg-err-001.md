# Analysis of Error Codes (OTG-ERR-001)


### Summary

Often, during a penetration test on web applications, we come up against many error codes generated from applications or web servers.
It's possible to cause these errors to be displayed by using a particular requests, either specially crafted with tools or created manually.
These codes are very useful to penetration testers during their activities, because they reveal a lot of information about databases, bugs, and other technological components directly linked with web applications.

This section analyses the more common codes (error messages) and bring into focus their relevance during a vulnerability assessment.
The most important aspect for this activity is to focus one's attention on these errors, seeing them as a collection of information that will aid in the next steps of our analysis. A good collection can facilitate assessment efficiency by decreasing the overall time taken to perform the penetration test.

Attackers sometimes use search engines to locate errors that disclose information. Searches can be performed to find any erroneous sites as random victims, or it is possible to search for errors in a specific site using the search engine filtering tools as described in [Conduct Search Engine Discovery and Reconnaissance for Information Leakage (OTG-INFO-001)](https://www.owasp.org/index.php/Conduct_search_engine_discovery/reconnaissance_for_information_leakage_%28OTG-INFO-001%29)


####Web Server Errors

A common error that we can see during testing is the HTTP 404 Not Found.
Often this error code provides useful details about the underlying web server and associated components. For example:

```
Not Found
The requested URL /page.html was not found on this server.
Apache/2.2.3 (Unix) mod_ssl/2.2.3 OpenSSL/0.9.7g  DAV/2 PHP/5.1.2 Server at localhost Port 80
```

This error message can be generated by requesting a non-existent URL.
After the common message that shows a page not found, there is information about web server version, OS, modules and other products used.
This information can be very important from an OS and application type and version identification point of view.

Other HTTP response codes such as 400 Bad Request, 405 Method Not Allowed, 501 Method Not Implemented, 408 Request Time-out and 505 HTTP Version Not Supported can be forced by an attacker. When receiving specially crafted requests, web servers may provide one of these error codes depending on their HTTP implementation.

Testing for disclosed information in the Web Server error codes is related testing for information disclosed in the HTTP headers as described in the section [Fingerprint Web Server (OTG-INFO-002)](https://www.owasp.org/index.php/Fingerprint_Web_Server_%28OTG-INFO-002%29).

####Application Server Errors

Application errors are returned by the application itself, rather than the web server. These could be error messages from framework code (ASP, JSP etc.) or they could be specific errors returned by the application code.
Detailed application errors typically provide information of server paths, installed libraries and application versions.

####Database Errors

Database errors are those returned by the Database System when there is a problem with the query or the connection. Each Database system, such as MySQL, Oracle or MSSQL, has their own set of errors. Those errors can provide sensible information such as Database server IPs, tables, columns and login details.

In addition, there are many SQL Injection exploitation techniques that utilize detailed error messages from the database driver, for in depth information on this issue see [Testing for SQL Injection (OTG-INPVAL-005)](https://www.owasp.org/index.php/Testing_for_SQL_Injection_%28OTG-INPVAL-005%29) for more information.

Web server errors aren't the only useful output returned requiring security analysis. Consider the next example error message:

```
Microsoft OLE DB Provider for ODBC Drivers (0x80004005)
[DBNETLIB][ConnectionOpen(Connect())] - SQL server does not exist or access denied
```

What happened? We will explain step-by-step below.

In this example, the 80004005 is a generic IIS error code which indicates that it could not establish a connection to its associated database. In many cases, the error message will detail the type of the database. This will often indicate the underlying operating system by association. With this information, the penetration tester can plan an appropriate strategy for the security test.

By manipulating the variables that are passed to the database connect string, we can invoke more detailed errors.
```
Microsoft OLE DB Provider for ODBC Drivers error '80004005'
[Microsoft][ODBC Access 97 ODBC driver Driver]General error Unable to open registry key 'DriverId'
```

In this example, we can see a generic error in the same situation which reveals the type and version of the associated database system and a dependence on Windows operating system registry key values.

Now we will look at a practical example with a security test against a web application that loses its link to its database server and does not handle the exception in a controlled manner. This could be caused by a database name resolution issue, processing of unexpected variable values, or other network problems.

Consider the scenario where we have a database administration web portal, which can be used as a front end GUI to issue database queries, create tables, and modify database fields. During the POST of the logon credentials, the following error message is presented to the penetration tester. The message indicates the presence of a MySQL database server:

```
Microsoft OLE DB Provider for ODBC Drivers (0x80004005)
[MySQL][ODBC 3.51 Driver]Unknown MySQL server host
```

If we see in the HTML code of the logon page the presence of a **hidden field** with a database IP, we can try to change this value in the URL with the address of database server under the penetration tester's control in an attempt to fool the application into thinking that the logon was successful.

Another example: knowing the database server that services a web application, we can take advantage of this information to carry out a SQL Injection for that kind of database or a persistent XSS test.


### How to Test
Below are some examples of testing for detailed error messages returned to the user. Each of the below examples has specific information about the operating system, application version, etc.

**Test: 404 Not Found**
```
telnet <host target> 80
GET /<wrong page> HTTP/1.1
host: <host target>
<CRLF><CRLF>
```
**Result:**
```
HTTP/1.1 404 Not Found
Date: Sat, 04 Nov 2006 15:26:48 GMT
Server: Apache/2.2.3 (Unix) mod_ssl/2.2.3 OpenSSL/0.9.7g
Content-Length: 310
Connection: close
Content-Type: text/html; charset=iso-8859-1
...
<title>404 Not Found</title>
...
<address>Apache/2.2.3 (Unix) mod_ssl/2.2.3 OpenSSL/0.9.7g at <host target> Port 80</address>
...
```


**Test:**
```
Network problems leading to the application being unable to access the database server
```
**Result:**
```
Microsoft OLE DB Provider for ODBC Drivers (0x80004005) '
[MySQL][ODBC 3.51 Driver]Unknown MySQL server host
```


**Test:**
```
Authentication failure due to missing credentials
```
**Result:**

Firewall version used for authentication:<br>
```
Error 407
FW-1 at <firewall>: Unauthorized to access the document.
•  Authorization is needed for FW-1.
•  The authentication required by FW-1 is: unknown.
•  Reason for failure of last attempt: no user
```


**Test: 400 Bad Request**
```
telnet <host target> 80
GET / HTTP/1.1
<CRLF><CRLF>
```
**Result:**
```
HTTP/1.1 400 Bad Request
Date: Fri, 06 Dec 2013 23:57:53 GMT
Server: Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch
Vary: Accept-Encoding
Content-Length: 301
Connection: close
Content-Type: text/html; charset=iso-8859-1
...
<title>400 Bad Request</title>
...
<address>Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch at 127.0.1.1 Port 80</address>
...
```


**Test: 405 Method Not Allowed**
```
telnet <host target> 80
PUT /index.html HTTP/1.1
Host: <host target>
<CRLF><CRLF>
```
**Result:**
```
HTTP/1.1 405 Method Not Allowed
Date: Fri, 07 Dec 2013 00:48:57 GMT
Server: Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch
Allow: GET, HEAD, POST, OPTIONS
Vary: Accept-Encoding
Content-Length: 315
Connection: close
Content-Type: text/html; charset=iso-8859-1
...
<title>405 Method Not Allowed</title>
...
<address>Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch at <host target> Port 80</address>
...
```


**Test: 408 Request Time-out**
```
telnet <host target> 80
GET / HTTP/1.1
-	Wait X seconds – (Depending on the target server, 21 seconds for Apache by default)
```
**Result:**
```
HTTP/1.1 408 Request Time-out
Date: Fri, 07 Dec 2013 00:58:33 GMT
Server: Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch
Vary: Accept-Encoding
Content-Length: 298
Connection: close
Content-Type: text/html; charset=iso-8859-1
...
<title>408 Request Time-out</title>
...
<address>Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch at <host target> Port 80</address>
...
```



**Test: 501 Method Not Implemented**
```
telnet <host target> 80
RENAME /index.html HTTP/1.1
Host: <host target>
<CRLF><CRLF>
```
**Result:**
```
HTTP/1.1 501 Method Not Implemented
Date: Fri, 08 Dec 2013 09:59:32 GMT
Server: Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch
Allow: GET, HEAD, POST, OPTIONS
Vary: Accept-Encoding
Content-Length: 299
Connection: close
Content-Type: text/html; charset=iso-8859-1
...
<title>501 Method Not Implemented</title>
...
<address>Apache/2.2.22 (Ubuntu) PHP/5.3.10-1ubuntu3.9 with Suhosin-Patch at <host target> Port 80</address>
...
```

**Test:**
```
Enumeration of directories by using access denied error messages:<br>

http://<host>/<dir>
```
**Result:**
```
Directory Listing Denied
This Virtual Directory does not allow contents to be listed.
```
```
Forbidden
You don't have permission to access /<dir> on this server.
```


### Tools
* [1] ErrorMint - http://sourceforge.net/projects/errormint/ <br>
* [2] ZAP Proxy - https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project <br>

### References

* [1] [[RFC2616](http://www.ietf.org/rfc/rfc2616.txt?number=2616)] Hypertext Transfer Protocol -- HTTP/1.1
* [2] [[ErrorDocument](http://httpd.apache.org/docs/2.2/mod/core.html#errordocument)] Apache ErrorDocument Directive
* [3] [[AllowOverride](http://httpd.apache.org/docs/2.2/mod/core.html#allowoverride)] Apache AllowOverride Directive
* [4] [[ServerTokens](http://httpd.apache.org/docs/2.2/mod/core.html#servertokens)] Apache ServerTokens Directive
* [5] [[ServerSignature](http://httpd.apache.org/docs/2.2/mod/core.html#serversignature)] Apache ServerSignature Directive

###Remediation

####Error Handling in IIS and ASP .net

ASP .net is a common framework from Microsoft used for developing web applications. IIS is one of the commonly used web servers. Errors occur in all applications, developers try to trap most errors but it is almost impossible to cover each and every exception (it is however possible to configure the web server to suppress detailed error messages from being returned to the user).

IIS uses a set of custom error pages generally found in c:\winnt\help\iishelp\common to display errors like '404 page not found' to the user. These default pages can be changed and custom errors can be configured for IIS server. When IIS receives a request for an aspx page, the request is passed on to the dot net framework.

There are various ways by which errors can be handled in dot net framework. Errors are handled at three places in ASP .net:

1. Inside Web.config customErrors section
2. Inside global.asax Application_Error Sub
3. At the the aspx or associated codebehind page in the Page_Error sub


**Handling errors using web.config**
```
<customErrors defaultRedirect="myerrorpagedefault.aspx" mode="On|Off|RemoteOnly">
   <error statusCode="404" redirect="myerrorpagefor404.aspx"/>
   <error statusCode="500" redirect="myerrorpagefor500.aspx"/>
</customErrors>
```

mode="On" will turn on custom errors. mode=RemoteOnly will show custom errors to the remote web application users. A user accessing the server locally will be presented with the complete stack trace and custom errors will not be shown to him.

All the errors, except those explicitly specified, will cause a redirection to the resource specified by defaultRedirect, i.e., myerrorpagedefault.aspx.
A status code 404 will be handled by myerrorpagefor404.aspx.


**Handling errors in Global.asax**

When an error occurs, the Application_Error sub is called. A developer can write code for error handling/page redirection in this sub.

```
Private Sub Application_Error (ByVal sender As Object, ByVal e As System.EventArgs)
     Handles MyBase.Error
End Sub
```


**Handling errors in Page_Error sub**

This is similar to application error.

```
Private Sub Page_Error (ByVal sender As Object, ByVal e As System.EventArgs)
     Handles MyBase.Error
End Sub
```


**Error hierarchy in ASP .net**

Page_Error sub will be processed first, followed by global.asax Application_Error sub, and, finally, customErrors section in web.config file.

Information Gathering on web applications with server-side technology is quite difficult, but the information discovered can be useful for the correct execution of an attempted exploit (for example, SQL injection or Cross Site Scripting (XSS) attacks) and can reduce false positives.


**How to test for ASP.net and IIS Error Handling**

Fire up your browser and type a random page name

```
http:\\www.mywebserver.com\anyrandomname.asp
```

If the server returns

```
The page cannot be found

Internet Information Services
```

it means that IIS custom errors are not configured. Please note the .asp extension.

Also test for .net custom errors. Type a random page name with aspx extension in your browser

```
http:\\www.mywebserver.com\anyrandomname.aspx
```

If the server returns

```
Server Error in '/' Application.
--------------------------------------------------------------------------------

The resource cannot be found.
Description: HTTP 404. The resource you are looking for (or one of its dependencies) could have been removed, had its name changed, or is temporarily unavailable. Please review the following URL and make sure that it is spelled correctly.
```

custom errors for .net are not configured.

####Error Handling in Apache

Apache is a common HTTP server for serving HTML and PHP web pages. By default, Apache shows the server version, products installed and OS system in the HTTP error responses.

Responses to the errors can be configured and customized globally, per site or per directory in the apache2.conf using the ErrorDocument directive [2]

```
ErrorDocument 404 “Customized Not Found error message”
ErrorDocument 403 /myerrorpagefor403.html
ErrorDocument 501 http://www.externaldomain.com/errorpagefor501.html
```

Site administrators are able to manage their own errors using .htaccess file if the global directive AllowOverride is configured properly in apache2.conf [3]

The information shown by Apache in the HTTP errors can also be configured using the directives ServerTokens [4] and ServerSignature [5] at apache2.conf configuration file. “ServerSignature Off” (On by default) removes the server information from the error responses, while ServerTokens [ProductOnly|Major|Minor|Minimal|OS|Full] (Full by default) defines what information has to be shown in the error pages.


####Error Handling in Tomcat

Tomcat is a HTTP server to host JSP and Java Servlet applications. By default, Tomcat shows the server version in the HTTP error responses.

Customization of the error responses can be configured in the configuration file web.xml.

```
<error-page>
	<error-code>404</error-code>
	<location>/myerrorpagefor404.html</location>
</error-page>
```