---
title: Sending Information to Servers
chapter: 8
prev: chrome
next: scripts
...

So far, our browser has seen the web as read only---but when you post
on Facebook, fill out a survey, or search Google, you're sending
information *to* servers as well as receiving information from them.
In this chapter, we'll start to transform our browser into a platform
for web applications by building out support for HTML forms, the
simplest way for a browser to send information to a server.

How forms work
==============

HTML forms have a couple of moving pieces.

First, in HTML, there is a `form` element, which contains `input`
elements,[^or-others] which in turn can be edited by the user. So a
form might look like this:

[^or-others]: There are other elements similar to `input`, such as
    `select` and `textarea`. They work similarly enough; they just
    represent different kinds of user controls, like dropdowns and
    multi-line inputs.

``` {.html}
<form action="/submit" method="post">
    <p>Name: <input name=name value=1></p>
    <p>Comment: <input name=comment value=2></p>
    <p><button>Submit!</button></p>
</form>
```

This form contains two text entry boxes called `name` and `comment`.
When the user goes to this page, they can click on those boxes to edit
their values. Then, when they click the button at the end of the form,
the browser collects all of the name/value pairs and bundles them into
an HTTP `POST` request (as indicated by the `method` attribute), sent
to the URL given by the `form` element's `action` attribute, with the
usual rules of relative URLs---so in this case, `/submit`. The `POST`
request looks like this:

``` {.example}
POST /submit HTTP/1.0
Host: example.org
Content-Length: 16

name=1&comment=2
```

In other words, it's lot like the regular `GET` requests we've already
seen, except that it has a body---you've already seen HTTP responses
with bodies, but requests can have them too. Note the `Content-Length`
header; it's mandatory for `POST` requests. The server responds to
this request with a web page, just like normal, and the browser then
does everything it normally does.

Implementing forms requires extending many parts of the browser, from
implementing HTTP `POST` through new layout objects that draw `input`
elements to handling buttons clicks. That makes it a great starting
point for transforming our toy browser into an application platform,
our goal for these next few chapters. Let's get started implementing
it all!

::: {.further}
HTML forms were first standardized in [HTML+][htmlplus], which also
proposed tables, mathematical equations, and text that wraps around
images. Amazingly, all three of these technologies survive, but in
totally different standards: tables in [RFC 1942][rfc1942], equations
in [MathML][mathml], and floating images in [CSS 1.0][css1].
:::

[htmlplus]: https://www.w3.org/MarkUp/htmlplus_paper/htmlplus.html
[rfc1942]: https://datatracker.ietf.org/doc/html/rfc1942
[mathml]: https://www.w3.org/Math/
[css1]: https://www.w3.org/TR/REC-CSS1/#floating-elements

Rendering widgets
=================

First, let's draw the input areas that the user will type
into.[^styled-widgets] Input areas are inline content, laid out in
lines next to text. So to support inputs we'll need a new kind of
layout object, which I'll call `InputLayout`. We can copy `TextLayout`
and use it as a template, though we'll need to make some quick edits.

[^styled-widgets]: Most applications use OS libraries to draw input
areas, so that those input areas look like other applications on that
OS. But browsers need a lot of control over application styling, so
they often draw their own input areas.

First, there's no `word` argument to `InputLayout`s:

``` {.python}
class InputLayout:
    def __init__(self, node, parent, previous):
        self.node = node
        self.children = []
        self.parent = parent
        self.previous = previous
```

Second, `input` elements usually have a fixed width:

``` {.python}
INPUT_WIDTH_PX = 200

class InputLayout:
    def layout(self):
        # ...
        self.width = INPUT_WIDTH_PX
        # ...
```

The `input` and `button` elements need to be visually distinct so the
user can find them easily. Our browser's styling capabilities are
limited, so let's use background color to do that:

``` {.css}
input {
    font-size: 16px; font-weight: normal; font-style: normal;
    background-color: lightblue;
}
button {
    font-size: 16px; font-weight: normal; font-style: normal;
    background-color: orange;
}
```

When the browser paints an `InputLayout` it needs to draw the
background:

``` {.python}
class InputLayout:
    def paint(self, display_list):
        bgcolor = self.node.style.get("background-color",
                                      "transparent")
        if bgcolor != "transparent":
            x2, y2 = self.x + self.width, self.y + self.height
            rect = DrawRect(self.x, self.y, x2, y2, bgcolor)
            display_list.append(rect)
```

It also needs to draw the text inside:

``` {.python}
class InputLayout:
    def paint(self, display_list):
        # ...
        if self.node.tag == "input":
            text = self.node.attributes.get("value", "")
        elif self.node.tag == "button":
            text = self.node.children[0].text

        color = self.node.style["color"]
        display_list.append(
            DrawText(self.x, self.y, text, self.font, color))
```

By this point in the book, you've seen many layout objects, so I'm
glossing over these changes. The point is that new layout objects are
one standard way to extend the browser.

We now need to create some `InputLayout`s, which we can do in
`BlockLayout`:

``` {.python}
class BlockLayout:
    def recurse(self, node):
        if isinstance(node, Text):
            self.text(node)
        else:
            if node.tag == "br":
                self.new_line()
            elif node.tag == "input" or node.tag == "button":
                self.input(node)
            else:
                for child in node.children:
                    self.recurse(child)
```

Note that I don't recurse into `button` elements, because the `button`
element draws its own contents.[^but-exercise] Since `input` elements
are self-closing, they never have children.

[^but-exercise]: Though you'll need to do this differently for one of
    the exercises below.

Finally, this new `input` method is similar to the `text` method,
creating a new layout object and adding it to the current line:

``` {.python}
class BlockLayout:
    def input(self, node):
        w = INPUT_WIDTH_PX
        if self.cursor_x + w > self.width:
            self.new_line()
        line = self.children[-1]
        input = InputLayout(node, line, self.previous_word)
        line.children.append(input)
        self.previous_word = input
        font = self.get_font(node)
        self.cursor_x += w + font.measure(" ")
```

But actually, there are a couple more complications due to the way we decided to
resolve the block-mixed-with-inline-siblings problem (see
[Chapter 5](layout.md#layout-modes)). One is that if there are no children for
a node, we assume it's a block element. But `<input>` elements don't have
children, yet must have inline layout or else they won't draw correctly.
Likewise, a `<button>` does have children, but they are treated
specially.^[This situation is specific to these elements in our browser, but
only because they are the only elements with special painting
behavior within an inline context. These are also two examples of
[atomic inlines](https://www.w3.org/TR/CSS2/visuren.html#inline-boxes).]

We can fix that with this change to `layout_mode`:

``` {.python}
def layout_mode(node):
    if isinstance(node, Text):
        return "inline"
    elif node.children:
        for child in node.children:
            if isinstance(child, Text): continue
            if child.tag in BLOCK_ELEMENTS:
                return "block"
        return "inline"
    elif node.tag == "input":
        return "inline"
    else:
        return "block"
```

The second problem is that, again due to having block siblings, sometimes an
`InputLayout` will end up wrapped in a `BlockLayout` that refers to to the
`<input>` or `<button>` node. But both `BlockLayout` and `InputLayout` have a
`paint` method, which means we're painting the node twice. We can fix that
with some simple logic to skip painting them via `BlockLayout`
in this case:[^atomic-inline-input]

``` {.python}
class BlockLayout:
    # ...
    def paint(self, display_list):
        # ...
        is_atomic = not isinstance(self.node, Text) and \
            (self.node.tag == "input" or self.node.tag == "button")

        if not is_atomic:
            if bgcolor != "transparent":
                x2, y2 = self.x + self.width, self.y + self.height
                rect = DrawRect(self.x, self.y, x2, y2, bgcolor)
                display_list.append(rect)

```

[^atomic-inline-input]: See also the footnote earlier about how atomic inlines
are often special in these kinds of ways. It's worth noting that there are
various other ways that our browser does not fully implement all the
complexities of inline painting---one example is that it does not correctly
paint nested inlines with different background colors.

With these changes the browser should now draw `input` and `button`
elements as blue and orange rectangles.

::: {.further}
The reason buttons surround their contents but input areas don't is
that a button can contain images, styled text, or other content. In a
real browser, that relies on the [`inline-block`][inline-block]
display mode: a way of putting a block element into a line of text.
There's also an older `<input type=button>` syntax more similar to
text inputs.
:::

[inline-block]: https://developer.mozilla.org/en-US/docs/Web/CSS/display

Interacting with widgets
========================

We've got `input` elements rendering, but you can't edit their
contents yet. But of course that's the whole point! So let's make
`input` elements work like the address bar does---clicking on one will
clear it and let you type into it.

Clearing is easy, another case inside `Tab`'s `click` method:

``` {.python}
class Tab:
    def click(self, x, y):
        while elt:
            # ...
            elif elt.tag == "input":
                elt.attributes["value"] = ""
            # ...
```

However, if you try this, you'll notice that clicking does not
actually clear the `input` element. That's because the code above
updates the HTML tree---but we need to update the layout tree and then
the display list of the change to appear on the screen.

Right now, the layout tree and display list are computed in `load`,
but we don't want to reload the whole page; we just want to redo the
styling, layout, paint and draw phases. Together these are called
*rendering*. So let's extract these phases into a
 new `Tab` method, `render`:

``` {.python}
class Tab:
    def load(self, url, body=None):
        # ...
        self.render()

    def render(self):
        style(self.nodes, sorted(self.rules, key=cascade_priority))
        self.document = DocumentLayout(self.nodes)
        self.document.layout()
        self.display_list = []
        self.document.paint(self.display_list)
```

For this code to work, you'll also need to change `nodes` and `rules`
from local variables in the `load` method to new fields on a `Tab`.
Note that styling moved from `load` to `render`, but downloading the
style sheets didn't---we don't re-download the style
sheets[^update-styles] every time you type!

[^update-styles]: Actually, some changes to the web page could delete
    existing `link` nodes or create new ones. Real browsers respond to
    this correctly, either removing the rules corresponding to deleted
    `link` nodes or downloading new style sheets when new `link` nodes
    are created. This is tricky to get right, and typing into an input
    area definitely can't make such changes, so let's skip this in our
    browser.
    
Now when we click an `input` element and clear its contents, we can
call `render` to redraw the page with the `input` cleared:

``` {.python}
class Tab:
    def click(self, x, y):
        while elt:
            elif elt.tag == "input":
                elt.attributes["value"] = ""
                return self.render()
```


So that's clicking in an `input` area. But typing is harder. Think
back to how we [implemented the address bar](chrome.md): we added a
`focus` field that remembered what we clicked on so we could later
send it our key presses. We need something like that `focus` field for
input areas, but it's going to be more complex because the input areas
live inside a `Tab`, not inside the `Browser`.

Naturally, we will need a `focus` field on each `Tab`, to remember
which text entry (if any) we've recently clicked on:

``` {.python}
class Tab:
    def __init__(self):
        # ...
        self.focus = None
```

Now when we click on an input element, we need to set `focus`:

``` {.python}
class Tab:
    def click(self, x, y):
        while elt:
            elif elt.tag == "input":
                self.focus = elt
                # ...
```

But remember that keyboard input isn't handled by the `Tab`---it's
handled by the `Browser`. So how does the `Browser` even know when
keyboard events should be sent to the `Tab`? The `Browser` has to
remember that in its own `focus` field!

In other words, when you click on the web page, the `Browser` updates
its `focus` field to remember that the user is interacting with the
page, not the browser interface:

``` {.python}
class Browser:
    def handle_click(self, e):
        if e.y < CHROME_PX:
            self.focus = None
            # ...
        else:
            self.focus = "content"
            # ...
        self.draw()
```

The `if` branch that corresponds to clicks in the browser interface
unsets `focus` by default, but some existing code in that branch will
set `focus` to `"address bar"` if the user actually clicked in the
address bar.

When a key press happens, the `Browser` sends it either to the address
bar or calls the active tab's `keypress` method:

``` {.python}
class Browser:
    def handle_key(self, e):
        # ...
        elif self.focus == "content":
            self.tabs[self.active_tab].keypress(e.char)
            self.draw()
```

That `keypress` method then uses the tab's `focus` field to put the
character in the right text entry:

``` {.python}
class Tab:
    def keypress(self, char):
        if self.focus:
            self.focus.attributes["value"] += char
            self.render()
```

Note that here we call `render` instead of `draw`, because we've
modified the web page and thus need to regenerate the display list
instead of just redrawing it to the screen.

Hierarchical focus handling is an important pattern for combining
graphical widgets; in a real browser, where web pages can be embedded
into one another with `iframe`s,[^iframes] the focus tree can be
arbitrarily deep.

[^iframes]: The `iframe` element allows you to embed one web page into
    another as a little window.

So now we have user input working with `input` elements. Before we
move on, there is one last tweak that we need to make: drawing the
text cursor in the `Tab`'s `draw` method. We'll first need to figure
out where the text entry is located, onscreen, by finding its layout
object:

``` {.python}
class Tab:
    def draw(self, canvas):
        # ...
        if self.focus:
            obj = [obj for obj in tree_to_list(self.document, [])
                   if obj.node == self.focus and \
                        isinstance(obj, InputLayout)][0]
```

Then using that layout object we can find the coordinates where the
cursor starts:

``` {.python indent=8}
if self.focus:
    # ...
    text = self.focus.attributes.get("value", "")
    x = obj.x + obj.font.measure(text)
    y = obj.y - self.scroll + CHROME_PX
```

And finally draw the cursor itself:

``` {.python indent=8}
if self.focus:
    # ...
    canvas.create_line(x, y, x, y + obj.height)
```

Now you can click on a text entry, type into it, and modify its value.
The next step is submitting the now-filled-out form.

::: {.further}
The code that draws the text cursor here is kind of clunky---you could
imagine each layout object knowing if it's focused and then being
responsible for drawing the cursor. That's the more traditional
approach in GUI frameworks, but Chrome for example keeps track of a global
[focused element][focused-element] to make sure the cursor can be
[globally styled][frame-caret].
:::

[focused-element]: https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/renderer/core/dom/document.h;l=881;drc=80def040657db16e79f59e7e3b27857014c0f58d
[frame-caret]: https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/renderer/core/editing/frame_caret.h?q=framecaret&ss=chromium


Submitting forms
================

You submit a form by clicking on a `button`. So let's add another
condition to the big `while` loop in `click`:

``` {.python}
class Tab:
    def click(self, x, y):
        while elt:
            # ...
            elif elt.tag == "button":
                # ...
            # ...
```

Once we've found the button, we need to find the form that it's in, by
walking up the HTML tree:


``` {.python indent=12}
elif elt.tag == "button":
    while elt:
        if elt.tag == "form" and "action" in elt.attributes:
            return self.submit_form(elt)
        elt = elt.parent
```

The `submit_form` method is then in charge of finding all of the input
elements, encoding them in the right way, and sending the `POST`
request. First, we look through all the descendents of the `form` to
find `input` elements:

``` {.python}
class Tab:
    def submit_form(self, elt):
        inputs = [node for node in tree_to_list(elt, [])
                  if isinstance(node, Element)
                  and node.tag == "input"
                  and "name" in node.attributes]
```

For each of those `input` elements, we need to extract the `name`
attribute and the `value` attribute, and _form-encode_ both of them.
Form encoding is how the name/value pairs are formatted in the HTTP
`POST` request. Basically: name, then equal sign, then value; and
name-value pairs are separated by ampersands:

``` {.python}
class Tab:
    def submit_form(self, elt):
        # ...
        body = ""
        for input in inputs:
            name = input.attributes["name"]
            value = input.attributes.get("value", "")
            body += "&" + name + "=" + value
        body = body [1:]
```

Now, any time you see something like this, you've got to ask: what if
the name or the value has an equal sign or an ampersand in it? So in
fact, "percent encoding" replaces all special characters with a
percent sign followed by those characters' hex codes. For example, a
space becomes `%20` and a period becomes `%2e`. Python provides a
percent-encoding function as `quote` in the `urllib.parse`
module:[^why-use-library]

``` {.python indent=8}
for input in inputs:
    # ...
    name = urllib.parse.quote(name)
    value = urllib.parse.quote(value)
    # ...
```

[^why-use-library]: You can write your own `percent_encode` function
using Python's `ord` and `hex` functions if you'd like. I'm using the
standard function for expediency. [Earlier in the book](http.md),
using these library functions would have obscured key concepts, but by
this point percent encoding is necessary but not conceptually
interesting.

Now that `submit_form` has built a request body, it needs to make a
`POST` request. I'm going to defer that responsibility to the `load`
function, which handles making requests:

``` {.python}
def submit_form(self, elt):
    # ...
    url = resolve_url(elt.attributes["action"], self.url)
    self.load(url, body)
```

The new argument `load` is then passed through to `request`:

``` {.python indent=4}
def load(self, url, body=None):
    # ...
    headers, body = request(url, body)
    # ...
```

In `request`, this new argument is used to decide between a `GET` and
a `POST` request:

``` {.python}
def request(url, payload=None):
    # ...
    method = "POST" if payload else "GET"
    # ...
    body = "{} {} HTTP/1.0\r\n".format(method, path)
    # ...
```

If there it's a `POST` request, the `Content-Length` header is mandatory:

``` {.python}
def request(url, payload=None):
    # ...
    if payload:
        length = len(payload.encode("utf8"))
        body += "Content-Length: {}\r\n".format(length)
    # ...
```

Note that the `Content-Length` is the length of the payload in bytes,
which might not be equal to its length in letters.[^unicode] Finally,
after the headers, we send the payload itself:

``` {.python}
def request(url, payload=None):
    # ...
    body += "\r\n" + (payload if payload else "")
    s.send(body.encode("utf8"))
    # ...
```

[^unicode]: Because characters from many languages are encoded as
    multiple bytes.

So that's how the `POST` request gets sent. Then the server responds
with an HTML page and the browser will render it in the totally normal
way. That's basically it for forms!

::: {.further}
While most form submissions use the form encoding described here,
forms with file uploads (using `<input type=file>`) use a [different
encoding][multi-part] that includes metadata for each key-value pair
(like the file name or file type). There's also an obscure
[`text/plain` encoding][plain-enc] option, which uses no escaping and
which even the standard warns against using.
:::

[multi-part]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST
[plain-enc]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#text/plain-encoding-algorithm

How web apps work
=================

So... How do web applications (a.k.a. web apps) use forms? When you
use an application from your browser---whether you are registering to
vote, looking at pictures of your baby cousin, or checking your
email---there are typically[^exceptions] two programs involved: client
code that runs in the browser, and server code that runs on the
server. When you click on things or take actions in the application,
that runs client code, which sends then data to the server via HTTP
requests.

[^exceptions]: Here's I'm talking in general terms. There are some
    browser applications without a server, and others where the client
    code is exceptionally simple and almost all the code is on the
    server.

For example, imagine a simple message board application. The server
stores the state of the message board---who has posted what---and has
logic for updating that state. But all the actual interaction with the
page---drawing the posts, letting the user enter new ones---happens in
the browser. Both components are necessary.

The browser and the server interact over HTTP. The browser first makes
a GET request to the server to load the current message board. The
user interacts with the browser to type a new post, and submits it to
the server (say, via a form). That causes the browser to make a POST
request to the server, which instructs the server to update the
message board state. The server then needs the browser to update what
the user sees; with forms, the server sends a new HTML page in its
response to the POST request.

Forms are a simple, minimal introduction to this cycle of request and
response and make a good introduction to how browser applications
work. They're also implemented in every browser and have been around
for decades. These days many web applications use the form elements,
but replace synchronous POST requests with asynchronous ones driven by
Javascript,[^ajax] which makes applications snappier by hiding the time
to make the HTTP request. In return for that snappiness, that
JavaScript code must now handle errors, validate inputs, and indicate
loading time. In any case, both synchronous and asynchronous uses of
forms are based on the same principles of client and server code.

[^ajax]: In the early 2000s, the adoption of asynchronous HTTP
    requests sparked the wave of innovative new web applications
    called [Web 2.0][web20].
    
[web20]: https://en.wikipedia.org/wiki/Web_2.0

::: {.further}
There are request types besides GET and POST, like [PUT][put-req]
(create if nonexistant) and [DELETE][del-req], or the more obscure
CONNECT and TRACE. In 2010 the [PATCH method][patch-req] was
standardized in [RFC 5789][rfc5789]. New methods were intended as a
standard extension mechanism for HTTP, and some protocols were built
this way (like [WebDav][webdav]'s PROPFIND, MOVE, and LOCK methods),
but this did not become an enduring way to extend the web, and HTTP
2.0 and 3.0 did not add any new methods.
:::

[put-req]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT
[del-req]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE
[patch-req]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH
[webdav]: https://en.wikipedia.org/wiki/WebDAV
[rfc5789]: https://datatracker.ietf.org/doc/html/rfc5789

Receiving POST requests
=======================

To better understand the request/response cycle, let's write a simple
web server. It'll implement an online guest book,^[They were very hip
in the 90s---comment threads from before there was anything to comment
on.] kind of like an open, anonymous comment thread. Now, this is a
book on web *browser* engineering, so I won't discuss web server
implementation that thoroughly. But I want you to see how the server
side of an application works.

A web server is a separate program from the web browser, so let's
start a new file. The server will need to:

-   Open a socket and listen for connections
-   Parse HTTP requests it receives
-   Respond to those requests with an HTML web page

Let's start by opening a socket. Like for the browser, we need to
create an internet streaming socket using TCP:

``` {.python file=server}
import socket
s = socket.socket(
    family=socket.AF_INET,
    type=socket.SOCK_STREAM,
    proto=socket.IPPROTO_TCP,
)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

The `setsockopt` call is optional. Normally, when a program has a
socket open and it crashes, your OS prevents that port from being
reused[^why-wait] for a short period. That's annoying when developing
a server; calling `setsockopt` with the `SO_REUSEADDR` option allows
the OS to immediately reuse the port.

[^why-wait]: When your process crashes, the computer on the end of the
    connection won't be informed immediately; if some other process
    opens the same port, it could receive data meant for the old,
    now-dead process.

Now, with this socket, instead of calling `connect` (to connect to
some other server), we'll call `bind`, which waits for other computers
to connect:

``` {.python file=server}
s.bind(('', 8000))
s.listen()
```

Let's look at the `bind` call first. Its first argument says who
should be allowed to make connections *to* the server; the empty
string means that anyone can connect. The second argument is the port
others must use to talk to our server; I've chosen `8000`. I can't use
80, because ports below 1024 require administrator privileges, but you
can pick something other than 8000 if, for whatever reason, port 8000
is taken on your machine.

Finally, after the `bind` call, the `listen` call tells the OS that
we're ready to accept connections.

To actually accept those connections, we enter a loop that runs once
per connection. At the top of the loop we call `s.accept` to wait for
a new connection:

``` {.python file=server}
while True:
    conx, addr = s.accept()
    handle_connection(conx)
```

That connection object is, confusingly, also a socket: it is the
socket corresponding to that one connection. We know what to do with
those: we read the contents and parse the HTTP message. But it's a
little trickier in the server than in the browser, because the server
can't just read from the socket until the connection closes---the
browser is waiting for the server and won't close the connection.

So we've got to read from the socket line-by-line. First, we read the
request line:

``` {.python file=server}
def handle_connection(conx):
    req = conx.makefile("b")
    reqline = req.readline().decode('utf8')
    method, url, version = reqline.split(" ", 2)
    assert method in ["GET", "POST"]
```

Then we read the headers until we get to a blank line, accumulating the
headers in a dictionary:

``` {.python file=server}
def handle_connection(conx):
    # ...
    headers = {}
    while True:
        line = req.readline().decode('utf8')
        if line == '\r\n': break
        header, value = line.split(":", 1)
        headers[header.lower()] = value.strip()
```

Finally we read the body, but only when the `Content-Length` header
tells us how much of it to read (that's why that header is mandatory on
`POST` requests):

``` {.python file=server}
def handle_connection(conx):
    # ...
    if 'content-length' in headers:
        length = int(headers['content-length'])
        body = req.read(length).decode('utf8')
    else:
        body = None
```

Now the server needs to generate a web page in response. We'll get to
that later; for now, just abstract that away behind a `do_request`
call:

``` {.python file=server}
def handle_connection(conx):
    # ...
    status, body = do_request(method, url, headers, body)
```

The server then sends this page back to the browser:

``` {.python file=server}
def handle_connection(conx):
    # ...
    response = "HTTP/1.0 {}\r\n".format(status)
    response += "Content-Length: {}\r\n".format(
        len(body.encode("utf8")))
    response += "\r\n" + body
    conx.send(response.encode('utf8'))
    conx.close()
```

This is all pretty bare-bones: our server doesn't check that the
browser is using HTTP 1.0 to talk to it, it doesn't send back any
headers at all except `Content-Length`, it doesn't support TLS, and so
on. Again: this is a web *browser* book---it'll do.

::: {.further}
Ilya Grigorik's [*High Performance Browser Networking*][hpbn] is an
excellent deep dive into networking and how to optimize for it in a
web application. There are things the client can do (make fewer
requests, avoid polling, reuse connections) and things the server can
do (compression, protocol support, sharing domains).
:::

[hpbn]: https://hpbn.co

Generating web pages
====================

So far all of this server code is "boilerplate"---any web application
will have similar code. What makes our server a guest book, on the
other hand, depends on what happens inside `do_request`. It needs to
store the guest book state, generate HTML pages, and respond to `POST`
requests.

Let's store guest book entries in a Python list. Usually web
applications use *persistent* state, like a database, so that the
server can be restarted without losing state, but our guest book need
not be that resilient.

``` {.python file=server}
ENTRIES = [ 'Pavel was here' ]
```

Next, `do_request` has to output HTML that shows those entries:

``` {.python file=server expected=False}
def do_request(method, url, headers, body):
    out = "<!doctype html>"
    for entry in ENTRIES:
        out += "<p>" + entry + "</p>"
    return "200 OK", out
```

This is definitely "minimal" HTML, so it's a good thing our browser
will insert implicit tags and has some default styles! You can test it
out by running this minimal web server and, while it's running, direct
your browser to `http://localhost:8000/`, where `localhost` is what
your computer calls itself and `8000` is the port we chose earlier.
You should see one guest book entry.

It's probably better to use a real web browser, instead of this book's
toy browser, to debug this web server. That way you don't have to
worry about browser bugs while you work on server bugs. But this
server does support both real and toy browsers.

We'll use forms to let visitors write in the guest book:

``` {.python file=server}
def do_request(method, url, headers, body):
    # ...
    out += "<form action=add method=post>"
    out +=   "<p><input name=guest></p>"
    out +=   "<p><button>Sign the book!</button></p>"
    out += "</form>"
    # ...
```

When this form is submitted, the browser will send a `POST` request to
`http://localhost:8000/add`. So the server needs to react to these
submissions. That means `do_request` will field two kinds of requests:
regular browsing and form submissions. Let's separate the two kinds of
requests into different functions.

First rename the current `do_request` to `show_comments`:

``` {.python file=server}
def show_comments():
    # ...
    return out
```

This then frees up the `do_request` function to figure out which
function to call for which request:

``` {.python file=server}
def do_request(method, url, headers, body):
    if method == "GET" and url == "/":
        return "200 OK", show_comments()
    elif method == "POST" and url == "/add":
        params = form_decode(body)
        return "200 OK", add_entry(params)
    else:
        return "404 Not Found", not_found(url, method)
```

When a `POST` request to `/add` comes in, the first step is to decode
the request body:

``` {.python file=server}
def form_decode(body):
    params = {}
    for field in body.split("&"):
        name, value = field.split("=", 1)
        name = urllib.parse.unquote_plus(name)
        value = urllib.parse.unquote_plus(value)
        params[name] = value
    return params
```

Note that I use `unquote_plus` instead of `unquote`, because browsers
may also use a plus sign to encode a space. The `add_entry` function
then looks up the `guest` parameter and adds its content as a new
guest book entry:

``` {.python file=server}
def add_entry(params):
    if 'guest' in params:
        ENTRIES.append(params['guest'])
    return show_comments()
```

I've also added a "404" response. Fitting the austere stylings of our
guest book, here's the 404 page:

``` {.python file=server}
def not_found(url, method):
    out = "<!doctype html>"
    out += "<h1>{} {} not found!</h1>".format(method, url)
    return out
```

Try it! You should be able to restart the server, open it in your
browser, and update the guest book a few times. You should also be
able to use the guest book from a real web browser.

::: {.further}
Typically connection handling and request routing is handled by a web
framework; this book, for example uses [bottle.py][bottle-py].
Frameworks parse requests into convenient data structures, route
requests to the right handler, and can also provide tools like HTML
templates, session handling, database access, input validation, and
API generation.
:::

[bottle-py]: https://bottlepy.org/docs/dev/

Summary
=======

With this chapter we're starting to transform our browser into an
application platform. We've added:

- Layout objects for input areas and buttons.
- Code to click on buttons and type into input areas.
- Hierarchical focus handling.
- Code to submit forms and send them to a server.

Plus, our browser now has a little web server friend. That's going to
be handy as we add more interactive features to the browser.

::: {.signup}
:::

Outline
=======

The complete set of functions, classes, and methods in our browser 
should now look something like this:

::: {.cmd .python .outline html=True}
    python3 infra/outlines.py --html src/lab8.py
:::

There's also a server now, but it's much simpler:

::: {.cmd .python .outline html=True}
    python3 infra/outlines.py --html src/server8.py
:::

If you run it, it should look something like this:

::: {.widget height=691}
    lab8-browser.html
:::

Exercises
=========

*Enter key*: In most browsers, if you hit the "Enter" or "Return" key
while inside a text entry, that submits the form that the text entry
was in. Add this feature to your browser.

*GET forms*: Forms can be submitted via GET requests as well as POST
requests. In GET requests, the form-encoded data is pasted onto the
end of the URL, separated from the path by a question mark, like
`/search?q=hi`; GET form submissions have no body. Implement GET form
submissions.

*Blurring*: Right now, if you click inside a text entry, and then
inside the address bar, two cursors will appear on the screen. To fix
this, add a `blur` method to each `Tab` which unfocuses anything that
is focused, and call it before changing focus.

*Tab*: In most browsers, the `<Tab>` key (on your keyboard) moves
focus from one input field to the next. Implement this behavior in
your browser. The "tab order" of input elements should be the same as
the order of `<input>` elements on the page.[^tabindex]

[^tabindex]: The [`tabindex`][tabindex] property lets a web page
    change this tab order, but its behavior is pretty weird.

[tabindex]: https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex

*Check boxes*: In HTML, `input` elements have a `type` attribute. When
set to `checkbox`, the `input` element looks like a checkbox; it's
checked if the `checked` attribute is set, and unchecked
otherwise.[^checked-attr] When the form is submitted, a checkbox's
`name=value` pair is included only if the checkbox is checked. (If the
checkbox has no `value` attribute, the default is the string `on`.)

[^checked-attr]: Technically, the `checked` attribute [only affects
    the state of the checkbox when the page loads][mdn-checked];
    checking and unchecking a checkbox does not affect this attribute
    but instead manipulates internal state.
    
[mdn-checked]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#attr-checked

*Resubmit requests*: One reason to separate GET and POST requests is
that GET requests are supposed to be *idempotent* (read-only,
basically) while POST requests are assumed to change the web server
state. That means that going "back" to a GET request (making the
request again) is safe, while going "back" to a POST request is a bad
idea. Change the browser history to record what method was used to
access each URL, and the POST body if one was used. When you go back
to a POST-ed URL, ask the user if they want to resubmit the form.
Don't go back if they say no; if they say yes, submit a POST request
with the same body as before.

*Message board*: Right now our web server is a simple guest book.
Extend it into a simple message board by adding support for topics.
Each topic should have its own URL and its own list of messages. So,
for example, `/cooking` should be a page of posts (about cooking) and
comments submitted through the form on that page should only show up
when you go to `/cooking`, not when you go to `/cars`. Make the home
page, from `/`, list the available topics with a link to each topic's
page. Make it possible for users to add new topics.

*Persistence*: Back the server's list of guest book entries with a
file, so that when the server is restarted it doesn't lose data.

*Rich buttons*: Make it possible for a button to contain arbitrary
elements as children, and render them correctly. The children should
be contained inside button instead of spilling out---this can make a
button really tall. Think about edge cases, like a button that
contains another button, an input area, or a link, and test real
browsers to see what they do.
