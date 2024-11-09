# Solution

*Interviewee's note: This document is not thoroughly edited (spelling, grammar and lots and lots
of informal tone and, hopefully, minimal technical inaccuracy). Were this for a much broader 
audience, e.g. students viewing an online course, I would most certainly give this the white glove
treatment.*

There is a JavaScript-based solution for transforming XML-formatted data to HTML markup,
using XSLT (eXtensible Stylesheet Language Transformation). In fact, it is well-documented on 
[w3schools.com](https://www.w3schools.com/xml/xsl_client.asp). (By the way, I literally
was not aware that modern browsers had this capability *built in*. Or, I just plumb forgot 
because I have not done it in so long<sup>1</sup>.)

In any case, all that is required for the XSLT stylesheet to take effect is to simply reference
it from the XML document we want rendered differently.

## JavaScript (for the browser).

Adding the following to the **list.xml** file near the top of the document:

```
<?xml-stylesheet type="text/xsl" href="form.xsl"?>
```

and then serving up these files from the current working directory with a web server<sup>2</sup>, e.g.

```
npx serve 
```

and we can observe (I'm using Chrome.)

![blank page -- wha happen](./javascript/1.png)

...nothing!

### Bug fixing

The built-in browser console says there are issues with the stylesheet, itself. (And the 
interviewee stops and goes to work.)

To be thorough, I had to convince myself the xml document was also well-formed. I am 
not fond of eyeballing XML markup for syntax (and, also I am lazy) so I relied on any 
existing tools that could provide that service for me: [Codebeautify.org](https://codebeautify.org/xml-parser-online)
has an XML parser that will also validate a document. Niiiiice.

![Looks OK](./javascript/2.png)

Now that we are certain that we have a valid XML document, we can proceed with fixing up the XSL stylesheet. 
For reference, here is the provided bitmap image of the desired output:

![screenshot of correct output](./tableOutput.png)

#### XSLT fixes

1. Error at line 14. So clearly this part of the solution is left intentionally incomplete,
as a prompt for the interviewee to get to work. The surrounding HTML markup e.g.

```
	<xsl:form-each select="catalog/cd">
	<tr>
	  <td><xsl: /></td>
	  <td><xsl: /></td>
	</tr>
	</xsl:form-each>
```

implies that we want to extract just the title and artist portion of the **cd** element. Or,
said differently, the first two children of that element. There is probably more than one way
to specify this as an XSLT pattern match, but I'm going with an xpath<sup>3</sup> pattern that
calls out each element by name. Line 12 handily provides a working example of a pattern-match that, 
hopefully, I can customize for the purpose of rendering the **title** information.

Additional explanation: After some troubleshooting of the pattern-match rule to achieve the 
foregoing, I am reminded that the rules are *relative* to the "parent" rule that got the parser to
this point of the XML document.

```
title
```

is where we need to pick up from rule **catalog/cd**.

Further, the w3cschools article says I need the XSL **value-of** rule to extract the **title** so giving 
this a shot and refreshing the page, the console no longer reports an error at line 14. Hey, that worked out!

Moving on to Line 15, the error appears to require the same solution, so:

```
artist
```

in combination with **value-of** and this error also goes away when refreshing the page. 
Almost there.

Line 25 has a complaint about some extra content at the end of the document?? Taking a closer
look -- *oh*. Instructions for the interviewee. The sample output does not specify that this 
"extra" content be visible in the rendering, so I am simply removing it altogether (from lines
23+).

Refreshing the browser once more, all of the console errors are gone but the **h2** title does
not read precisely as specified, so that's a small addendum to the markup.

Oh wait. My rows of data are not rendering. Looks like **form-each** is a typo and should 
actually be **for-each**. HOPEFULLY, that's all that's left, e.g. corrected on Line 12:

```
	<xsl:for-each select="catalog/cd">
```

and doing similar for line 17 (for the closing tag). Refresh the browser aaaaand:

![Rendering matches the expected output.](./javascript/3.png)

Oila!<sup>4</sup>

### More

The solution obviously has to be executed using the web browser's built-in capability. I like this 
feature because it is very accessible and makes for fast prototyping. And, I think, for 
modest implementations this is likely all that is required. However, some additional thoughts
come to mind and make me pause:

* Will the browser cache the transformation result, or does it really re-execute the stylesheet
upon every request?
* If a transformation were to be applied to a sufficiently large and/or complex document, might there 
be an observable *delay* while the browser renders the result? If such an observation were made, I
would consider an equivalent server-side solution.

Some additional thoughts on the XSL file itself:

* Externalize the CSS, which would be particularly helpful for a sufficiently complex set of rules, to
help de-clutter the HTML document.
* In a more extreme, hypothetical problem where the XML data itself were not a static file, I would 
consider a server-side solution that further separates concerns: dynamic generation of XML (e.g. as a 
result of a database query or, perhaps, as a result of an external EDI process), separate from XSLT transformation. 
Even the XSLT rule could, conceivably, be dynamically generated based on conditions found in the resultant XML.
* XML files make me nervous and I confess to the inclination to run each document through a validator. Human error prevails, in spite of our best effort oft-times.
* I did discover a third party option for transformation of XML: [Saxonica](https://www.saxonica.com/html/download/download_page.html) 
supports browser, and several programming language bindings. Are there yet more offerings, from other vendors? Absolutely.

## Java

The Java solution implements the same XML specifications that the JavaScript version does.

### Getting Started with Java
You need your environment set up first. 
1. Get a distribution. There are many choices. I like [SdkMan](https://sdkman.io/), to manage my runtime environment. \
Pick any distribution that supports no higher than Java 12<sup>7</sup>.
2. Get the [XercesJ distribution](https://www.apache.org/dyn/closer.cgi/xerces/j/)<sup>5</sup>. The runtime is archived in a variety of formats. \
Pick the download that you can work with and, if necessary, verify the downloaded files' signature. When you are satisfied you have \
a safe copy of the files, you should end up with a folder called **xerces-2_12_2-xml-schema-1.1**, containing a *bewildering* assortment of files. What we really care about are the following:

	a. xml-apis.jar
	b. xercesIpml.jar
	c. xalan.jar (missing; see Step 3)
	d. serializer.jar

	(Note: A plethora of choices exist for obtaining the necessary runtimes to transform XML with Java<sup>6</sup>. \
	I probably should have attempted an implementation with [Gradle](https://gradle.org/), one of three modern \
	choices for managing a Java project. This choice also determines where from project runtimes are downloaded, \
	due to a lack of centralized repositories for the Java ecosystem. But I was just following my nose on Google \
	and I am short on time.)

3. Get [Xalan](https://archive.apache.org/dist/xml/xalan-j/binaries/). We care only about:

	a. xalan.jar
	b. xerces.jar

4. Set your classpath, so that Java knows from where to find its runtime executables, which you can choose to set as an environment variable, or, on the command line when you run Java. I hate excessive typing, so I'm going with the former. For *nix:

```
export CLASSPATH=$HOME/my/path/to/xml-apis.jar:$CLASSPATH
```

which you repeat for each JAR cited above.

### Testing the solution
The preceding was *a lot* just so that we could execute the following on the command line:

```
java org.apache.xalan.xslt.Process -IN list.xml -XSL form.xsl -OUT output.html
```

If your runtime environment is set up correctly, **output.html** should show the transformed data as HTML markup. \
You will notice that it uses the same XML and XSL solution. The only thing we did differently here is change the \
runtime environment from the web browser to the built-in XSLT command line program embedded in the xalan.jar. Whew!

[Video Playback of Java demonstration.](./java/java.webm)

## C++

Same for C++.

-----------------------------------

<sup>1</sup> [This Stack Overflow article](http://stackoverflow.com/questions/3466854/ddg#3466912) 
says the feature became available from all major web browsers in 2010.

<sup>2</sup> I use [NodeJS](https://nodejs.org/) and many supporting tools e.g. **npx** and **[pnpm](https://pnpm.io/)** for package management.

<sup>3</sup> Boy, I hope I am using this terminology correctly. It's late; I'm in a rush.

<sup>4</sup> Compliments on the selection of 90s music!

<sup>5</sup> This open source library was retired a long time ago, and was intended to be a reference implementation. It is no longer actively maintained.

<sup>6</sup> I am many, many years behind the times. The last time I was actively wrangling XML with Java, I was using the [XercesJ distribution](https://www.apache.org/dyn/closer.cgi/xerces/j/)<sup>5</sup>.

<sup>7</sup> This XML library needs Java 9 through Java 12 to function. It is incompatible with Java 13.
