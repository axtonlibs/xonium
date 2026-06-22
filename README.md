# Xonium is a library for Web Automation and Scraping built for Axton Coding Language. 

## Examples

### Example One = Google Search Automation
 
import "xonium"

let drv = newsession("http://localhost:4444", {"browserName": "chrome"})

drv.get("https://www.google.com")
let searchbox = waitforvisible(drv, "name", "q")
searchbox.sendkeys("What is 6+6")
searchbox.sendkeys("\n")
sleep(2)
let results = drv.findelements("cssselector", "h3")
print("Found " + len(results) + " results")
for i in range(min(5, len(results))) {
    print(results[i].gettext())
}
drv.close()

### Example Two = Github Login Automation

import "xonium"

let drv = newsession("http://localhost:4444", {"browserName": "chrome"})

drv.get("https://github.com/login")
let username = waitforvisible(drv, "id", "login_field")
let password = drv.findelement("id", "password")
let submit = drv.findelement("name", "commit")

username.sendkeys("your_username")
password.sendkeys("your_password")
submit.click()
sleep(3)

if drv.getcurrenturl() == "https://github.com/" {
    print("Login successful!")
    let profile = drv.findelement("cssselector", ".avatar")
    if profile.isdisplayed() {
        print("Profile visible - authenticated")
    }
} else {
    print("Login failed")
    let error = drv.findelement("cssselector", ".flash-error")
    if error {
        print("Error: " + error.gettext())
    }
}
drv.close()

## Installation :
axton install xonium


# DOCUMENTATION

Xonium Usage Reference

Import

```
import "xonium"
// Loads xonium library for browser automation
```

Session

```
let drv = xonium.newsession()
// Creates new Chrome browser session on localhost:4444
// Returns driver object for controlling browser

let drv = xonium.newsession("http://hub:4444", {"browserName": "firefox"})
// Custom server and browser
// First arg: WebDriver server URL
// Second arg: browser capabilities

drv.close()
// Closes browser session and frees resources
```

Navigation

```
drv.get("https://google.com")
// Navigates to URL, waits for page load

let url = drv.getcurrenturl()
// Returns current page URL string

let title = drv.gettitle()
// Returns page title string from <title> tag

drv.back()
// Goes back one page in history

drv.forward()
// Goes forward one page in history

drv.refresh()
// Reloads current page

let html = drv.getsource()
// Returns full page HTML source string
```

Finding Elements

```
let elem = drv.findelement("id", "submit")
// Finds ONE element by locator
// Returns element object or none
// Locators: "id", "class name", "css selector", "xpath", "name", "tag name", "link text", "partial link text"

let elems = drv.findelements("class name", "button")
// Finds ALL matching elements
// Returns list of element objects
// Empty list if none found

// Using constants (avoids typos)
let elem = drv.findelement(xonium.id, "submit")
let elem = drv.findelement(xonium.cssselector, ".header .nav")
let elem = drv.findelement(xonium.xpath, "//button")
```

Element Actions

```
elem.click()
// C element, must be visible and enabled

elem.sendkeys("hello")
// Types text into input/textarea
// Use "\n" for Enter key

elem.clear()
// Clears input/textarea content

let text = elem.gettext()
// Returns visible text of element

let value = elem.getattribute("href")
// Returns attribute value or none

let prop = elem.getproperty("value")
// Returns DOM property value or none

let visible = elem.isdisplayed()
// Returns true if element is visible

let enabled = elem.isenabled()
// Returns true if element is enabled

let selected = elem.isselected()
// Returns true if radio/checkbox selected

let css = elem.getcssvalue("color")
// Returns CSS property value string
```

Waiting

```
let elem = xonium.waitfor(drv, "id", "content")
// Waits for element to exist in DOM, default 5 seconds
// Returns element when found
// Throws error if timeout

let elem = xonium.waitfor(drv, "id", "content", 10)
// Custom timeout of 10 seconds

let elem = xonium.waitforvisible(drv, "class name", "modal")
// Waits for element to be visible, default 5 seconds

let elem = xonium.waitforvisible(drv, "id", "popup", 8)
// Custom timeout of 8 seconds, must be visible
```

JavaScript

```
let result = drv.executescript("return document.title", [])
// Executes JavaScript in browser
// Returns JS return value

drv.executescript("document.body.style.background = 'red'", [])
// Modifies page, no return value

let elem = drv.findelement("id", "target")
drv.executescript("arguments[0].scrollIntoView()", [elem])
// Passes element as argument
// arguments[0] is first argument

let data = drv.executescript("""
    let items = document.querySelectorAll('.item');
    return items.length;
""", [])
// Multi-line script returns data
```

Error Handling

```
try {
    let elem = drv.findelement("id", "missing")
    elem.click()
} catch e {
    print("Error: " + e)
}
// Catches all errors, prevents crash

try {
    let elem = xonium.waitfor(drv, "id", "slow", 2)
    elem.sendkeys("text")
} catch e {
    print("Element not found or timeout")
    // Fallback code here
}
// Handles wait timeouts and missing elements
```

Cookies

```
let cookies = drv.executescript("return document.cookie", [])
// Returns all cookies as string: "name1=value1; name2=value2"

drv.executescript("document.cookie = 'session=abc123; path=/'", [])
// Sets a cookie, must include path

drv.executescript("document.cookie = 'session=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/'", [])
// Deletes cookie by setting expires to past
```

Scrolling

```
drv.executescript("window.scrollTo(0, 0)", [])
// Scroll to top of page

drv.executescript("window.scrollTo(0, document.body.scrollHeight)", [])
// Scroll to bottom of page

let elem = drv.findelement("id", "target")
drv.executescript("arguments[0].scrollIntoView(true)", [elem])
// Scroll element into view, true = top alignment, false = bottom
```

Screenshots

```
let img = drv.executescript("""
    let canvas = document.createElement('canvas');
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    let ctx = canvas.getContext('2d');
    ctx.fillStyle = 'white';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    return canvas.toDataURL();
""", [])
// Returns screenshot as base64 data URL
// Starts with "data:image/png;base64,"
```

Forms

```
let username = drv.findelement("id", "username")
username.sendkeys("alice")
// Type into username field

let password = drv.findelement("id", "password")
password.sendkeys("secret123")
// Type into password field

let checkbox = drv.findelement("id", "agree")
if not checkbox.isselected() {
    checkbox.click()
}
// Check checkbox if unchecked

let submit = drv.findelement("id", "login")
submit.click()
// Submit the form
```

Lists

```
let items = drv.findelements("class name", "product")
// Get all product elements

for item in items {
    let name = item.findelement("class name", "name")
    let price = item.findelement("class name", "price")
    print(name.gettext() + ": " + price.gettext())
}
// Loop through items and extract data

let first = items[0]
// First element by index

let count = len(items)
// Number of elements found
```

Multiple Sessions

```
let drv1 = xonium.newsession()
let drv2 = xonium.newsession()
// Two independent browser sessions

drv1.get("https://google.com")
drv2.get("https://github.com")
// Each navigates separately

drv1.close()
drv2.close()
// Close each session independently
```

Async

```
let task = async fn() {
    let drv = xonium.newsession()
    drv.get("https://example.com")
    let elem = xonium.waitfor(drv, "id", "content")
    let text = elem.gettext()
    drv.close()
    return text
}

let result = await task()
// Runs async, non-blocking
// Returns when complete
```

Constants

```
import "xonium"

// Locator strategy constants
xonium.id              // "id"
xonium.classname       // "class name"
xonium.cssselector     // "css selector"
xonium.xpath           // "xpath"
xonium.name            // "name"
xonium.tagname         // "tag name"
xonium.linktext        // "link text"
xonium.partiallinktext // "partial link text"

// Usage prevents typos
let elem = drv.findelement(xonium.id, "submit")
// Same as drv.findelement("id", "submit")
```

Common Patterns

```
// Page Object Pattern
let loginpage = fn(drv) {
    return {
        "login": fn(username, password) {
            let user = xonium.waitfor(drv, "id", "username")
            user.sendkeys(username)
            let pass = drv.findelement("id", "password")
            pass.sendkeys(password)
            let submit = drv.findelement("id", "login")
            submit.click()
            return xonium.waitfor(drv, "id", "dashboard")
        }
    }
}

// Usage
let page = loginpage(drv)
let dashboard = page.login("alice", "123")
```

```
// Retry Pattern
let retryfind = fn(drv, by, value, attempts) {
    for i in range(attempts) {
        try {
            return drv.findelement(by, value)
        } catch e {
            sleep(1)
        }
    }
    return none
}

// Usage
let elem = retryfind(drv, "id", "slow-element", 3)
```

```
let testcases = [
    {"user": "alice", "pass": "123"},
    {"user": "bob", "pass": "456"}
]

for test in testcases {
    let drv = xonium.newsession()
    try {
        drv.get("https://example.com/login")
        drv.findelement("id", "username").sendkeys(test.user)
        drv.findelement("id", "password").sendkeys(test.pass)
        drv.findelement("id", "login").click()
        let welcome = xonium.waitfor(drv, "id", "welcome")
        print("Success: " + test.user)
    } catch e {
        print("Failed: " + test.user + " - " + e)
    } finally {
        drv.close()
    }
}
```


