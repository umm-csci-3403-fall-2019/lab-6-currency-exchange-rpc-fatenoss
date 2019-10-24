# Remote procedure call (RPC) for Currency Exchange <!-- omit in toc -->

This lab explores the idea of client-server organization.  In a client-server
configuration one central machine called the **server** acts as a central source
for some resource or service.  Other machines known as **clients** utilize the
resource or service provided by the server.  A good example would be a web-server
providing web-pages to multiple browsers on multiple computers.  

The term **client-server** refers to the configuration of the service provider and
service consumer.  It does not require multiple machines. It is possible for the same
computer to act as server and as client.

This lab illustrates the encapsulation of a
JSON-based web service as a simple remote procedure call (RPC), where we wrap
a call to a remote service (a service which provides currency exchange rate data)
in a way that allows users to access that service through what appear to be
local function calls.

- [RPC encapsulation](#rpc-encapsulation)
  - [Reading from a URL](#reading-from-a-url)
  - [Parsing JSON](#parsing-json)

------------------------------------------------------------------------

## RPC encapsulation

Financial programs often need access to financial rates such as stock prices and
exchange rates. There are numerous services out there that provide that
data, often through rich (and complex) APIs. There are also, however,
simple services that provide the data in response to
standard HTTP requests.

[Fixer.io](http://fixer.io/), for
example, provides a simple currency exchange rate service that allows
you to specify a date as components of a URL, and generates the exchange
rates for a number of major currency for that date. If you add a working access key (which requires free registration), for example, the URL
<http://data.fixer.io/api/2008-10-15?access_key=...>
generates an JSON document containing a variety of exchange rates for the date specified in the URL (15 Oct 2008). See [Fixer's
documentation](https://fixer.io/documentation)
for more info.

This is nice if we just want to look up a single date and
read through it by hand, but is somewhat awkward if we want to access
this data programmatically (i.e., as part of a piece of software we're
writing). The goal of this lab is to build a simple remote procedure call (RPC)
encapsulation of this service, essentially providing a wrapper that
isolates users (programmers in this case) from the details of accessing
and parsing the data. We'll provide two key methods:

```java
public float getExchangeRate(String currencyCode, int year, int month, int day);

public float getExchangeRate(String fromCurrency, String toCurrency, int year, int month, int day);
```

The first provides the exchange on the given date for the given currency
against the base currency (which for Fixer is the Euro). The second
takes a date and two currencies, and returns the exchange rate of the
first vs. the second. Currencies are specified using [ISO 4217 currency
codes](http://en.wikipedia.org/wiki/ISO_4217), and dates are the year
(as a four digit integer), the month as a two digit integer (01=Jan,
12=Dec), and the day of the month as a two digit integer.

We've provided
some simple JUnit tests and a stub in the project. The
first four tests all reference static JSON files which Nic has provided on
`facultypages.morris.umn.edu`; these are also included in the project in the
`JSON_files` directory. The fifth one (which is initially marked with
`@Ignore` so it won't actually run) refers to Fixer's web site. You should
wait until you get the first four to pass before you try the last one as
we don't want to be hammering on Fixer's web site while we're trying to
get our code to work. When you're ready to run that last test just
remove the `@Ignore` line, add a working access key, and it will run.

There are two major pieces
here that you may have never seen:

- You'll need to read the result of requesting a URL
- You'll need to parse an JSON document

### Reading from a URL

This is actually quite easy in Java. This little block of code:

```java
String urlString = "http://www.morris.umn.edu/";
URL url = new URL(urlString);
InputStream inputStream = url.openStream();
```

will generate an `InputStream` that will provide the (HTML) contents of
the Morris home page. You can then pass that `InputStream` to any other reading
tools like a `BufferedReader` or (or more importantly for this lab) an
JSON parser.

### Parsing JSON

There are a ton of Java JSON parsing tools out there, including several
included as part of Java's standard libraries. We're not going to provide
a full tutorial here, but there's lots of stuff out there on the
Internet. We recommend using [Google's GSON library](https://github.com/google/gson);
there's [a simple example on StackOverflow](https://stackoverflow.com/a/31743324/2557372) that (very briefly) includes
everything you need here.

The basic structure is:

- Construct a `Reader` on the `InputStream` you get from `URL` as described above.
- Use a GSON `JsonParser` to parse the contents of that reader using something like `new JsonParser().parse(reader).getAsJsonObject()`. This returns a `JsonObject`.
- Once you have a GSON `JsonObject` you can use methods like `get()` and `getAsJsonObject()` to walk through the JSON object extracting components as necessary.

You might want to write a method `getRate(JsonObject ratesInfo, String currency)` that encapsulates the walking through the JSON object so you don't end up repeating that logic in your solution.
