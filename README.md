# FlareSolverr

[![Latest release](https://img.shields.io/github/v/release/FlareSolverr/FlareSolverr)](https://github.com/FlareSolverr/FlareSolverr/releases)
[![Docker Pulls](https://img.shields.io/docker/pulls/flaresolverr/flaresolverr)](https://hub.docker.com/r/flaresolverr/flaresolverr/)
[![GitHub issues](https://img.shields.io/github/issues/FlareSolverr/FlareSolverr)](https://github.com/FlareSolverr/FlareSolverr/issues)
[![GitHub pull requests](https://img.shields.io/github/issues-pr/FlareSolverr/FlareSolverr)](https://github.com/FlareSolverr/FlareSolverr/pulls)
[![Donate PayPal](https://img.shields.io/badge/Donate-PayPal-yellow.svg)](https://www.paypal.com/paypalme/diegoheras0xff)
[![Donate Bitcoin](https://img.shields.io/badge/Donate-Bitcoin-f7931a.svg)](https://www.blockchain.com/btc/address/13Hcv77AdnFWEUZ9qUpoPBttQsUT7q9TTh)
[![Donate Ethereum](https://img.shields.io/badge/Donate-Ethereum-8c8c8c.svg)](https://www.blockchain.com/eth/address/0x0D1549BbB00926BF3D92c1A8A58695e982f1BE2E)

FlareSolverr is a proxy server to bypass Cloudflare and DDoS-GUARD protection.

## How it works

FlareSolverr starts a proxy server, and it waits for user requests in an idle state using few resources.
When some request arrives, it uses [Selenium](https://www.selenium.dev) with the
[undetected-chromedriver](https://github.com/ultrafunkamsterdam/undetected-chromedriver)
to create a web browser (Chrome). It opens the URL with user parameters and waits until the Cloudflare challenge
is solved (or timeout). The HTML code and the cookies are sent back to the user, and those cookies can be used to
bypass Cloudflare using other HTTP clients.

**NOTE**: Web browsers consume a lot of memory. If you are running FlareSolverr on a machine with few RAM, do not make
many requests at once. With each request a new browser is launched.

It is also possible to use a permanent session. However, if you use sessions, you should make sure to close them as
soon as you are done using them.

## Installation

### Docker

It is recommended to install using a Docker container because the project depends on an external browser that is
already included within the image.

Docker images are available in:
* GitHub Registry => https://github.com/orgs/FlareSolverr/packages/container/package/flaresolverr
* DockerHub => https://hub.docker.com/r/flaresolverr/flaresolverr

Supported architectures are:

| Architecture | Tag          |
|--------------|--------------|
| x86          | linux/386    |
| x86-64       | linux/amd64  |
| ARM32        | linux/arm/v7 |
| ARM64        | linux/arm64  |

We provide a `docker-compose.yml` configuration file. Clone this repository and execute
`docker-compose up -d` _(Compose V1)_ or `docker compose up -d` _(Compose V2)_ to start
the container.

If you prefer the `docker cli` execute the following command.
```bash
docker run -d \
  --name=flaresolverr \
  -p 8191:8191 \
  -e LOG_LEVEL=info \
  --restart unless-stopped \
  ghcr.io/flaresolverr/flaresolverr:latest
```

If your host OS is Debian, make sure `libseccomp2` version is 2.5.x. You can check the version with `sudo apt-cache policy libseccomp2` 
and update the package with `sudo apt install libseccomp2=2.5.1-1~bpo10+1` or `sudo apt install libseccomp2=2.5.1-1+deb11u1`.
Remember to restart the Docker daemon and the container after the update.

### Precompiled binaries

> **Warning**
> Precompiled binaries are only available for x64 architecture. For other architectures see Docker images.

This is the recommended way for Windows users.
* Download the [FlareSolverr executable](https://github.com/FlareSolverr/FlareSolverr/releases) from the release's page. It is available for Windows x64 and Linux x64.
* Execute FlareSolverr binary. In the environment variables section you can find how to change the configuration.

### From source code

> **Warning**
> Installing from source code only works for x64 architecture. For other architectures see Docker images.

* Install [Python 3.11](https://www.python.org/downloads/).
* Install [Chrome](https://www.google.com/intl/en_us/chrome/) (all OS) or [Chromium](https://www.chromium.org/getting-involved/download-chromium/) (just Linux, it doesn't work in Windows) web browser.
* (Only in Linux) Install [Xvfb](https://en.wikipedia.org/wiki/Xvfb) package.
* (Only in macOS) Install [XQuartz](https://www.xquartz.org/) package.
* Clone this repository and open a shell in that path.
* Run `pip install -r requirements.txt` command to install FlareSolverr dependencies.
* Run `python src/flaresolverr.py` command to start FlareSolverr.

### From source code (FreeBSD/TrueNAS CORE)

* Run `pkg install chromium python39 py39-pip xorg-vfbserver` command to install the required dependencies.
* Clone this repository and open a shell in that path.
* Run `python3.9 -m pip install -r requirements.txt` command to install FlareSolverr dependencies.
* Run `python3.9 src/flaresolverr.py` command to start FlareSolverr.

### Systemd service

We provide an example Systemd unit file `flaresolverr.service` as reference. You have to modify the file to suit your needs: paths, user and environment variables.

## Usage

Example Bash request:
```bash
curl -L -X POST 'http://localhost:8191/v1' \
-H 'Content-Type: application/json' \
--data-raw '{
  "cmd": "request.get",
  "url": "http://www.google.com/",
  "maxTimeout": 60000
}'
```

Example Python request:
```py
import requests

url = "http://localhost:8191/v1"
headers = {"Content-Type": "application/json"}
data = {
    "cmd": "request.get",
    "url": "http://www.google.com/",
    "maxTimeout": 60000
}
response = requests.post(url, headers=headers, json=data)
print(response.text)
```

Example PowerShell request:
```ps1
$body = @{
    cmd = "request.get"
    url = "http://www.google.com/"
    maxTimeout = 60000
} | ConvertTo-Json

irm -UseBasicParsing 'http://localhost:8191/v1' -Headers @{"Content-Type"="application/json"} -Method Post -Body $body
```

### Commands

#### + `sessions.create`

This will launch a new browser instance which will retain cookies until you destroy it with `sessions.destroy`.
This comes in handy, so you don't have to keep solving challenges over and over and you won't need to keep sending
cookies for the browser to use.

This also speeds up the requests since it won't have to launch a new browser instance for every request.

| Parameter | Notes                                                                                                                                                                                                                                                                                                            |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| session   | Optional. The session ID that you want to be assigned to the instance. If isn't set a random UUID will be assigned.                                                                                                                                                                                              |
| proxy     | Optional, default disabled. Eg: `"proxy": {"url": "http://127.0.0.1:8888"}`. You must include the proxy schema in the URL: `http://`, `socks4://` or `socks5://`. Authorization (username/password) is supported. Eg: `"proxy": {"url": "http://127.0.0.1:8888", "username": "testuser", "password": "testpass"}` |

#### + `sessions.list`

Returns a list of all the active sessions. More for debugging if you are curious to see how many sessions are running.
You should always make sure to properly close each session when you are done using them as too many may slow your
computer down.

Example response:

```json
{
  "sessions": [
    "session_id_1",
    "session_id_2",
    "session_id_3..."
  ]
}
```

#### + `sessions.destroy`

This will properly shutdown a browser instance and remove all files associated with it to free up resources for a new
session. When you no longer need to use a session you should make sure to close it.

| Parameter | Notes                                         |
|-----------|-----------------------------------------------|
| session   | The session ID that you want to be destroyed. |

#### + `request.get`

| Parameter           | Notes                                                                                                                                                                                                                                                                                                                                        |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| url                 | Mandatory                                                                                                                                                                                                                                                                                                                                    |
| session             | Optional. Will send the request from and existing browser instance. If one is not sent it will create a temporary instance that will be destroyed immediately after the request is completed.                                                                                                                                                |
| session_ttl_minutes | Optional. FlareSolverr will automatically rotate expired sessions based on the TTL provided in minutes.                                                                                                                                                                                                                                      |
| maxTimeout          | Optional, default value 60000. Max timeout to solve the challenge in milliseconds.                                                                                                                                                                                                                                                           |
| cookies             | Optional. Will be used by the headless browser. Eg: `"cookies": [{"name": "cookie1", "value": "value1"}, {"name": "cookie2", "value": "value2"}]`.                                                                                                                                                                                           |
| returnOnlyCookies   | Optional, default false. Only returns the cookies. Response data, headers and other parts of the response are removed.                                                                                                                                                                                                                       |
| proxy               | Optional, default disabled. Eg: `"proxy": {"url": "http://127.0.0.1:8888"}`. You must include the proxy schema in the URL: `http://`, `socks4://` or `socks5://`. Authorization (username/password) is not supported. (When the `session` parameter is set, the proxy is ignored; a session specific proxy can be set in `sessions.create`.) |

> **Warning**
> If you want to use Cloudflare clearance cookie in your scripts, make sure you use the FlareSolverr User-Agent too. If they don't match you will see the challenge.

Example response from running the `curl` above:

```json
{
    "solution": {
        "url": "https://www.google.com/?gws_rd=ssl",
        "status": 200,
        "headers": {
            "status": "200",
            "date": "Thu, 16 Jul 2020 04:15:49 GMT",
            "expires": "-1",
            "cache-control": "private, max-age=0",
            "content-type": "text/html; charset=UTF-8",
            "strict-transport-security": "max-age=31536000",
            "p3p": "CP=\"This is not a P3P policy! See g.co/p3phelp for more info.\"",
            "content-encoding": "br",
            "server": "gws",
            "content-length": "61587",
            "x-xss-protection": "0",
            "x-frame-options": "SAMEORIGIN",
            "set-cookie": "1P_JAR=2020-07-16-04; expires=Sat..."
        },
        "response":"<!DOCTYPE html>...",
        "cookies": [
            {
                "name": "NID",
                "value": "204=QE3Ocq15XalczqjuDy52HeseG3zAZuJzID3R57...",
                "domain": ".google.com",
                "path": "/",
                "expires": 1610684149.307722,
                "size": 178,
                "httpOnly": true,
                "secure": true,
                "session": false,
                "sameSite": "None"
            },
            {
                "name": "1P_JAR",
                "value": "2020-07-16-04",
                "domain": ".google.com",
                "path": "/",
                "expires": 1597464949.307626,
                "size": 19,
                "httpOnly": false,
                "secure": true,
                "session": false,
                "sameSite": "None"
            }
        ],
        "userAgent": "Windows NT 10.0; Win64; x64) AppleWebKit/5..."
    },
    "status": "ok",
    "message": "",
    "startTimestamp": 1594872947467,
    "endTimestamp": 1594872949617,
    "version": "1.0.0"
}
```

### + `request.post`

This is the same as `request.get` but it takes one more param:

| Parameter | Notes                                                                    |
|-----------|--------------------------------------------------------------------------|
| postData  | Must be a string with `application/x-www-form-urlencoded`. Eg: `a=b&c=d` |

## Environment variables

| Name               | Default                | Notes                                                                                                                                                         |
|--------------------|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| LOG_LEVEL          | info                   | Verbosity of the logging. Use `LOG_LEVEL=debug` for more information.                                                                                         |
| LOG_HTML           | false                  | Only for debugging. If `true` all HTML that passes through the proxy will be logged to the console in `debug` level.                                          |
| CAPTCHA_SOLVER     | none                   | Captcha solving method. It is used when a captcha is encountered. See the Captcha Solvers section.                                                            |
| TZ                 | UTC                    | Timezone used in the logs and the web browser. Example: `TZ=Europe/London`.                                                                                   |
| LANG               | none                   | Language used in the web browser. Example: `LANG=en_GB`.                                                                                   |
| HEADLESS           | true                   | Only for debugging. To run the web browser in headless mode or visible.                                                                                       |
| BROWSER_TIMEOUT    | 40000                  | If you are experiencing errors/timeouts because your system is slow, you can try to increase this value. Remember to increase the `maxTimeout` parameter too. |
| TEST_URL           | https://www.google.com | FlareSolverr makes a request on start to make sure the web browser is working. You can change that URL if it is blocked in your country.                      |
| PORT               | 8191                   | Listening port. You don't need to change this if you are running on Docker.                                                                                   |
| HOST               | 0.0.0.0                | Listening interface. You don't need to change this if you are running on Docker.                                                                              |
| PROMETHEUS_ENABLED | false                  | Enable Prometheus exporter. See the Prometheus section below.                                                                                                 |
| PROMETHEUS_PORT    | 8192                   | Listening port for Prometheus exporter. See the Prometheus section below.                                                                                     |

Environment variables are set differently depending on the operating system. Some examples:
* Docker: Take a look at the Docker section in this document. Environment variables can be set in the `docker-compose.yml` file or in the Docker CLI command.
* Linux: Run `export LOG_LEVEL=debug` and then run `flaresolverr` in the same shell.
* Windows: Open `cmd.exe`, run `set LOG_LEVEL=debug` and then run `flaresolverr.exe` in the same shell.

## Prometheus exporter

The Prometheus exporter for FlareSolverr is disabled by default. It can be enabled with the environment variable `PROMETHEUS_ENABLED`. If you are using Docker make sure you expose the `PROMETHEUS_PORT`.

Example metrics:
```shell
# HELP flaresolverr_request_total Total requests with result
# TYPE flaresolverr_request_total counter
flaresolverr_request_total{domain="nowsecure.nl",result="solved"} 1.0
# HELP flaresolverr_request_created Total requests with result
# TYPE flaresolverr_request_created gauge
flaresolverr_request_created{domain="nowsecure.nl",result="solved"} 1.690141657157109e+09
# HELP flaresolverr_request_duration Request duration in seconds
# TYPE flaresolverr_request_duration histogram
flaresolverr_request_duration_bucket{domain="nowsecure.nl",le="0.0"} 0.0
flaresolverr_request_duration_bucket{domain="nowsecure.nl",le="10.0"} 1.0
flaresolverr_request_duration_bucket{domain="nowsecure.nl",le="25.0"} 1.0
flaresolverr_request_duration_bucket{domain="nowsecure.nl",le="50.0"} 1.0
flaresolverr_request_duration_bucket{domain="nowsecure.nl",le="+Inf"} 1.0
flaresolverr_request_duration_count{domain="nowsecure.nl"} 1.0
flaresolverr_request_duration_sum{domain="nowsecure.nl"} 5.858
# HELP flaresolverr_request_duration_created Request duration in seconds
# TYPE flaresolverr_request_duration_created gauge
flaresolverr_request_duration_created{domain="nowsecure.nl"} 1.6901416571570296e+09
```

## Captcha Solvers

> **Warning**
> At this time none of the captcha solvers work. You can check the status in the open issues. Any help is welcome.

Sometimes CloudFlare not only gives mathematical computations and browser tests, sometimes they also require the user to
solve a captcha.
If this is the case, FlareSolverr will return the error `Captcha detected but no automatic solver is configured.`

FlareSolverr can be customized to solve the CAPTCHA automatically by setting the environment variable `CAPTCHA_SOLVER`
to the file name of one of the adapters inside the [/captcha](src/captcha) directory.

## Related projects

* C# implementation => https://github.com/FlareSolverr/FlareSolverrSharp


## DrissionPage Documentation Deep Dive: A Thorough Explanation

This document provides a detailed explanation of the DrissionPage library, a powerful Python tool for web automation. We'll delve deeper into its functions, methods, and objects, going beyond a simple list and offering comprehensive usage examples to illustrate their capabilities.

**What is DrissionPage?**

DrissionPage is a Python library designed for web automation, offering two primary modes of operation:

1. **Browser Control (d mode):** This mode allows you to directly manipulate a Chromium-based browser (like Chrome or Edge), mimicking user interactions such as clicking, typing, and navigating.
2. **Packet Sending and Receiving (s mode):** This mode uses the `requests` library to directly send and receive HTTP requests, enabling data scraping and web interaction without a visible browser.

DrissionPage elegantly combines both modes, enabling seamless switching and even data sharing between them. This flexibility allows developers to choose the most efficient approach for different tasks.

**Installation**

You can easily install DrissionPage using `pip`:

```bash
pip install DrissionPage
```

**Upgrade**

To upgrade to the latest stable version:

```bash
pip install DrissionPage --upgrade
```

For a specific version:

```bash
pip install DrissionPage==4.0.0b17
```

**Importing Components**

DrissionPage offers various components, each serving a specific purpose in web automation. 

**Page Classes**

These classes form the core of DrissionPage, providing the tools to control browsers or handle HTTP requests.

* **`ChromiumPage`:** Use this for browser-specific automation tasks.

```python
from DrissionPage import ChromiumPage
```

* **`SessionPage`:** This class is ideal for packet-based web interaction and data scraping.

```python
from DrissionPage import SessionPage
```

* **`WebPage`:** The most comprehensive class, offering both browser control and packet handling capabilities.

```python
from DrissionPage import WebPage
```

**Configuration Tools**

These tools allow you to customize the behavior of DrissionPage.

* **`ChromiumOptions`:** Used to set browser startup parameters when launching a new instance, such as headless mode, window size, and user data directory.

```python
from DrissionPage import ChromiumOptions
```

* **`SessionOptions`:** Configures parameters for sending and receiving packets, including headers, proxies, and authentication.

```python
from DrissionPage import SessionOptions
```

* **`Settings`:** Defines global settings for DrissionPage, like element search timeout, handling of exceptions, and default behaviors.

```python
from DrissionPage.common import Settings
```

**Other Tools**

DrissionPage offers several utility tools to aid in web automation tasks.

* **`Keys`:** Represents keyboard keys for simulating keyboard input, including special keys and combinations.

```python
from DrissionPage.common import Keys
```

* **`Actions`:**  Enables complex sequences of actions, like mouse movements, clicks, and keyboard inputs, usually accessible through the page object.

```python
from DrissionPage.common import Actions
```

* **`By`:** Compatible with Selenium's `By` class for easier migration from Selenium-based projects.

```python
from DrissionPage.common import By
```

* Additional tools:
    * `wait_until`: Waits for a given condition to become True.
    * `make_session_ele`: Creates a `ChromiumElement` object from raw HTML text.
    * `configs_to_here`: Copies the default configuration file to the current directory.
    * `get_blob`: Retrieves a specific blob resource.
    * `tree`: Prints the structure of page objects or element objects.
    * `from_selenium`: Facilitates integration with Selenium code.
    * `from_playwright`: Enables interoperability with Playwright code.

**Exceptions**

DrissionPage defines specific exceptions to handle various error scenarios, residing in the `DrissionPage.errors` module.

```python
from DrissionPage.errors import ElementNotFoundError
```

**Derived Object Types**

These objects are generated by page objects and represent elements, tabs, and other browser components.

```python
from DrissionPage.items import SessionElement, ChromiumElement, ShadowRoot, NoneElement, ChromiumTab, MixTab, ChromiumFrame
```

**Preparation**

Before diving into browser control, ensure DrissionPage can access your browser. By default, it attempts to control Chrome.

1. **Test Browser Startup:**
    
    Run the following code:

    ```python
    from DrissionPage import ChromiumPage

    page = ChromiumPage()
    page.get('http://DrissionPage.cn')
    ```
    
    If the browser launches and loads the website, skip the next step.
    
2. **Set Browser Path:**

    If the test failed, manually set the browser path. DrissionPage will store this setting in its configuration file.

    * **Method 1:** Create a temporary Python file with this code, replace the path with your browser executable, and run it:
        
        ```python
        from DrissionPage import ChromiumOptions

        path = r'C:\Program Files\Google\Chrome\Application\chrome.exe' 
        ChromiumOptions().set_browser_path(path).save()
        ```
        
    * **Method 2:**  Use the command line (ensuring the Python environment matches your project):

        ```bash
        dp -p "C:\Program Files\Google\Chrome\Application\chrome.exe"
        ```

3. **Retry Browser Control:**

    Rerun the code from step 1 to confirm the browser launch.

**Controlling the Browser**

Let's automate logging into Gitee (https://gitee.com/login) using `ChromiumPage`.

1. **Page Analysis:**

    Inspect the Gitee login page using your browser's developer tools (usually by pressing F12). Identify the relevant elements for inputting the username and password, and the login button. 

2. **Code Example:**

    Replace the placeholders with your actual Gitee credentials.

    ```python
    from DrissionPage import ChromiumPage

    page = ChromiumPage()  # Create a ChromiumPage object, launching or taking over the browser
    page.get('https://gitee.com/login')  # Navigate to the login page

    username_input = page.ele('#user_login')  # Locate the username input box
    username_input.input('your_gitee_username')  # Input your username

    page.ele('#user_password').input('your_gitee_password')  # Locate and input your password
    page.ele('@value=登 录').click()  # Locate and click the login button
    ```

3. **Code Explanation:**

    * `from DrissionPage import ChromiumPage`: Imports the `ChromiumPage` class.
    * `page = ChromiumPage()`: Creates a `ChromiumPage` object, which either launches a new browser instance or takes over an existing one.
    * `page.get('https://gitee.com/login')`: Opens the Gitee login page in the browser. The `get()` method waits for the page to load completely before continuing.
    * `username_input = page.ele('#user_login')`:  The `ele()` method uses a locator to find the username input field. The '#' prefix denotes an ID selector. The `ele()` method also incorporates an implicit wait, pausing execution until the element loads or a timeout occurs (default 10 seconds).
    * `username_input.input('your_gitee_username')`: Types your username into the located input field.
    * `page.ele('#user_password').input('your_gitee_password')`: Finds the password input field and enters your password using chained operations.
    * `page.ele('@value=登 录').click()`:  Locates the login button using its 'value' attribute and clicks it. The '@' prefix signifies searching by an attribute name.

**Sending and Receiving Packets**

Let's scrape data from Gitee's explore page (https://gitee.com/explore/all) using `SessionPage`.

1. **Page Analysis:**

    Examine the Gitee explore page HTML to identify the elements containing the project names and links. Understand how the URL changes for different pages (e.g., page number parameters).

2. **Code Example:**

    ```python
    from DrissionPage import SessionPage

    page = SessionPage()  # Create a SessionPage object

    for i in range(1, 4):  # Scrape the first 3 pages
        page.get(f'https://gitee.com/explore/all?page={i}')  # Visit each page
        project_links = page.eles('.title.project-namespace-path')  # Find all project links

        for link in project_links:  # Iterate through each link
            print(link.text, link.link)  # Print the project name and URL
    ```

3. **Code Explanation:**

    * `from DrissionPage import SessionPage`: Imports the `SessionPage` class.
    * `page = SessionPage()`:  Creates a `SessionPage` object.
    * `for i in range(1, 4)`: Loops through pages 1 to 3.
    * `page.get(f'https://gitee.com/explore/all?page={i}')`: Visits the URL of each page.
    * `project_links = page.eles('.title.project-namespace-path')`: The `eles()` method locates multiple elements using a CSS selector. The '.' prefix indicates a class selector.
    * `for link in project_links`: Iterates through the found elements.
    * `print(link.text, link.link)`:  Prints the text (project name) and link (URL) of each element.

**Mode Switching**

`WebPage` allows you to switch between d mode (browser control) and s mode (packet handling) seamlessly. Here's a demonstration of searching on Gitee and then retrieving data using s mode.

1. **Page Analysis:**

    Inspect the Gitee explore page HTML to identify the search input field and the structure of the search results.

2. **Code Example:**

    ```python
    from DrissionPage import WebPage

    page = WebPage()  # Create a WebPage object
    page.get('https://gitee.com/explore/all')  # Visit Gitee explore page

    page('#q').input('DrissionPage')  # Find the search box and input the keyword
    page('t:button@tx():搜索').click()  # Find and click the search button
    page.wait.load_start()  # Wait for the page to start loading

    page.change_mode()  # Switch to s mode

    items = page('.ui.relaxed.divided.items.explore-repo__list').eles('.item')  # Locate all search result items
    for item in items:  # Iterate through the results
        print(item('t:h3').text)  # Print the project title
        print(item('.project-desc.mb-1').text)  # Print the project description
        print()
    ```

3. **Code Explanation:**

    * `from DrissionPage import WebPage`: Imports the `WebPage` class.
    * `page = WebPage()`: Creates a `WebPage` object, starting in d mode by default.
    * `page.get('https://gitee.com/explore')`: Opens the Gitee explore page.
    * `page('#q').input('DrissionPage')`: Locates the search box by ID ('#q') and enters 'DrissionPage'.
    * `page('t:button@tx():搜索').click()`: Finds and clicks the search button using a combination of simplified locator syntax. It searches for a button element ('t:button') containing the text '搜索' ('@tx():搜索').
    * `page.wait.load_start()`: Waits for the page to begin loading after the search.
    * `page.change_mode()`: Switches from d mode to s mode. This reloads the current URL using the s mode connection, preserving the search results.
    * `items = page('#hits-list').eles('.item')`: Locates all search result items using a combination of ID and class selectors in s mode.
    * `for item in items`: Loops through the result items.
    * `print(item('.title').text)`: Prints the project title within each result item.
    * `print(item('.desc').text)`: Prints the project description.

**Basic Concepts**

Understanding the core components and logic of DrissionPage is essential for effective web automation.

**Web Automation Approaches**

Traditionally, web automation relies on two primary methods:

1. **Direct Packet Sending:** Libraries like `requests` send HTTP requests to retrieve data. This approach is lightweight and efficient but can become complex for websites using sophisticated data structures or encryption.
2. **Browser Control:** Tools like Selenium automate browser interactions. This provides greater flexibility and ease of handling complex web pages but comes with performance overhead and resource consumption.

DrissionPage integrates both methods, providing a unified and efficient approach.

**DrissionPage Architecture**

DrissionPage employs the Page Object Model (POM), encapsulating page interactions into objects. It interacts with browsers through the Chromium protocol and with web servers using the `requests` library.

**Key Objects**

* **Page Objects:** These represent web pages or sections of web pages, providing methods for finding elements and interacting with them. DrissionPage offers three page objects: `ChromiumPage`, `SessionPage`, and `WebPage`.
* **Element Objects:** These represent individual elements within a web page, offering methods for actions like clicking, typing, and retrieving information. DrissionPage uses `ChromiumElement` for browser elements and `SessionElement` for static elements extracted from HTML.

**Workflow**

DrissionPage follows a consistent workflow:

1. Create a page object.
2. Use the page object to locate element objects.
3. Interact with the web page by manipulating these element objects (e.g., clicking buttons, filling forms, extracting data).

**Working Modes**

`WebPage` supports both d mode (browser control) and s mode (packet handling).

* **d Mode (Driver and Dynamic):**
    * Controls the browser directly.
    * Can read page information and perform actions like clicking, typing, and executing JavaScript.
    * More powerful but slower and resource-intensive.
* **s Mode (Session and Speed/Silence):**
    * Uses HTTP requests to interact with the web server.
    * Faster and lighter than d mode but with limited functionality.
    * Ideal for data scraping and simpler interactions.

**Switching Modes**

`WebPage` allows seamless mode switching using the `change_mode()` method, synchronizing login information between modes.

**Configuration Management**

DrissionPage uses configuration files (`configs.ini`) to manage settings for both `requests` and browser control. This centralizes and simplifies configuration management.

**Locators**

DrissionPage introduces a powerful locator syntax to locate elements, offering concise and intuitive methods compared to traditional approaches like XPath or CSS selectors.

**Example Comparison**

**Finding an element containing the text 'abc':**

* **DrissionPage:** `page('abc')`
* **Selenium:** `driver.find_element(By.XPATH, '//*[contains(text(), "abc")]')`

**Locating an element with class 'abc':**

* **DrissionPage:** `page('.abc')`
* **Selenium:** `driver.find_element(By.CLASS_NAME, 'abc')`

**Finding the next sibling element:**

* **DrissionPage:** `ele.next()`
* **Selenium:** `ele.find_element(By.XPATH, './/following-sibling::*')`

The DrissionPage locator syntax is significantly more concise, readable, and flexible.

**SessionPage in Depth**

`SessionPage` allows interaction with web pages using HTTP requests. This section provides a deeper understanding of its features.

**Creating a SessionPage Object**

You can create a `SessionPage` object in various ways, each offering different levels of control over the initial configuration.

* **Direct Creation:** This approach uses the default configuration from the `configs.ini` file or the built-in defaults if the file is absent.

```python
from DrissionPage import SessionPage

page = SessionPage()
```

* **Creation through Configuration Information (`SessionOptions`):** Use `SessionOptions` to customize settings like proxies, headers, and cookies.

```python
from DrissionPage import SessionPage, SessionOptions

so = SessionOptions().set_proxies(http='127.0.0.1:1080')  # Set proxy information
page = SessionPage(session_or_options=so)
```

* **Creation from a Specific INI File:**  Specify a custom INI file for configuration.

```python
from DrissionPage import SessionPage, SessionOptions

so = SessionOptions(ini_path=r'./custom_config.ini')
page = SessionPage(session_or_options=so)
```

* **Disabling INI File Usage:** Configure settings directly in the code without using an INI file.

```python
from DrissionPage import SessionPage, SessionOptions

so = SessionOptions(read_file=False)  # Disable INI file reading
so.set_retry(5)  # Set retry attempts
page = SessionPage(so)
```

* **Passing Session Control:** For multiple page objects sharing the same `Session` object, retrieve the `session` attribute from an existing page and pass it to the new page object.

```python
page1 = SessionPage()
session = page1.session
page2 = SessionPage(session_or_options=session)
```

**Visiting Web Pages**

`SessionPage` utilizes the `requests` library for network connections. While it supports all request methods, DrissionPage primarily focuses on `get()` and `post()`.

**`get()` Method**

* **Visiting Online Web Pages:**
    * Syntax mirrors the `requests.get()` method but adds retry functionality for failed connections.
    * Unlike `requests`, it doesn't return a `Response` object directly.

    ```python
    page.get('https://www.example.com')  # Basic web page access
    
    url = 'https://www.example.com/search'
    headers = {'referer': 'https://www.google.com'}
    cookies = {'session_id': 'xyz123'}
    proxies = {'http': '127.0.0.1:8080'}
    page.get(url, headers=headers, cookies=cookies, proxies=proxies)  # Access with custom parameters
    ```

* **Reading Local Files:** `get()` can also load HTML from local files for parsing.

```python
page.get(r'C:\my_html_file.html')
```

**`post()` Method**

* Sends POST requests.
* Similar usage to `get()`.

```python
data = {'username': 'user123', 'password': 'pass456'}
page.post('https://www.example.com/login', data=data)  # Send data using the 'data' parameter
page.post('https://www.example.com/api', json=data)  # Send data as JSON using the 'json' parameter
```

**Other Request Methods**

For less common request methods, access the internal `Session` object of `SessionPage` and use the `requests` library directly.

```python
session = page.session
response = session.head('https://www.example.com')  # Send a HEAD request
print(response.headers)
```

**Retrieving Page Information**

After accessing a web page, `SessionPage` offers properties and methods to extract information.

```python
page.get('https://www.example.com')
print(page.title)  # Print the page title
print(page.html)  # Print the raw HTML content
```

**Available Properties**

* `url`: The current URL.
* `url_available`: A boolean indicating whether the URL is reachable.
* `title`: The page title.
* `raw_data`: Raw bytes of the response content (equivalent to `Response.content`).
* `html`: The parsed HTML text.
* `json`: Parses the response content as JSON (if applicable).
* `user_agent`: The user-agent used for the request.

**Managing Operational Parameters**

`SessionPage` provides settings to control network requests and retries.

* `timeout`: Connection timeout (default 10 seconds).

```python
page = SessionPage(timeout=5)  # Set timeout during creation
page.timeout = 20  # Modify timeout
```

* `retry_times`: Number of retry attempts for failed connections (default 3).

```python
page.retry_times = 5
```

* `retry_interval`: Delay between retry attempts in seconds (default 2).

```python
page.retry_interval = 1.5
```

* `encoding`: Encoding format for the web page.

**Cookies Management**

* `cookies()`: Retrieves cookie information.

```python
for cookie in page.cookies(as_dict=False, all_domains=True):  # Retrieve cookies from all domains as a list
    print(cookie)
```

**Embedded Objects**

* `session`: The internal `requests.Session` object.
* `response`: The `requests.Response` object generated after a request, allowing access to raw response data.

**Element Information Retrieval**

`SessionPage` works with `SessionElement` objects, representing elements extracted from HTML.

**Example Element**

```html
<div id="myDiv" class="divClass">Hello World!
    <p>Paragraph text</p>
    <!-- This is a comment -->
</div>
```

**Available Properties**

* `html`: The element's outer HTML.
* `inner_html`: The element's inner HTML.
* `tag`: The element's tag name.
* `text`:  Formatted text content of the element, with line breaks removed.
* `raw_text`: The raw, unformatted text content of the element, preserving whitespace and line breaks.
* `texts()`: A list of text content from direct child nodes, optionally filtering for only text nodes.
* `comments`: A list of comments within the element.
* `attrs`: A dictionary of all attributes and their values.
* `attr()`: Retrieves the value of a specific attribute.
* `link`: The element's 'href' or 'src' attribute, if present.
* `page`: The `SessionPage` object containing the element.
* `xpath`: The absolute XPath of the element.
* `css_path`: The absolute CSS selector path of the element.

**Batch Information Access from Element Lists**

Methods like `eles()` return lists of elements, providing the `get` attribute for batch information retrieval.

```python
elements = page.eles('tag:div')
print(elements.get.texts())  # Get a list of text content from all div elements
```

**Available Methods**

* `get.attrs()`: Returns a list of values for a specific attribute from all elements.
* `get.links()`: Returns a list of 'href' or 'src' attributes from all elements.
* `get.texts()`: Returns a list of text content from all elements.

**Practical Example**

```python
from DrissionPage import SessionPage

page = SessionPage()
page.get('https://gitee.com/explore')

list_items = page('tag:ul@text():全部推荐项目').eles('tag:a')  # Find all links within a specific list

for item in list_items:  # Loop through the links
    print(item.tag, item.text, item.attribute('href'))  # Print tag name, text, and href attribute
```

**Page Settings**

`SessionPage` offers settings for controlling retry behavior, timeout, encoding, cookies, headers, proxies, and other aspects of the HTTP session.

**Example Usage**

```python
from DrissionPage import SessionPage

page = SessionPage()
page.set.cookies([{'name': 'cookie1', 'value': 'value1'}, {'name': 'cookie2', 'value': 'value2'}])  # Set cookies
page.set.headers({'User-Agent': 'MyCustomUserAgent'})  # Set headers
page.set.proxies(http='127.0.0.1:8080')  # Set proxies
```

**Available Settings Methods**

* `set.retry_times()`: Sets the number of retry attempts.
* `set.retry_interval()`: Sets the delay between retry attempts.
* `set.timeout()`: Sets the connection timeout.
* `set.encoding()`:  Sets the page encoding.
* `set.cookies()`: Sets one or more cookies.
* `set.cookies.clear()`: Clears all cookies.
* `set.cookies.remove()`: Removes a specific cookie.
* `set.headers()`:  Sets the entire headers dictionary.
* `set.header()`: Sets a specific header value.
* `set.user_agent()`:  Sets the user-agent.
* `set.proxies()`:  Sets proxy information.
* `set.auth()`: Sets authentication credentials.
* `set.hooks()`: Sets callback functions for various events.
* `set.params()`: Sets query parameters for requests.
* `set.verify()`: Enables or disables SSL certificate verification.
* `set.cert()`:  Sets SSL client certificates.
* `set.stream()`: Enables or disables streaming response content.
* `set.trust_env()`: Sets whether to trust environment variables.
* `set.max_redirects()`:  Sets the maximum number of redirects allowed.
* `set.add_adapter()`: Adds a custom adapter to handle specific URLs.

* `close()`: Closes the connection.

**Startup Configuration (`SessionOptions`)**

`SessionOptions` manages the initial configuration of a `SessionPage` object, allowing you to customize settings before creating the page object.

**Creating a `SessionOptions` Object**

```python
from DrissionPage import SessionOptions

so = SessionOptions()  # Create with default settings from the INI file
so = SessionOptions(read_file=False)  # Create with built-in defaults, ignoring the INI file
so = SessionOptions(ini_path='./my_config.ini')  # Create with settings from a specific INI file
```

**Setting Methods**

`SessionOptions` provides a range of methods to configure various settings, all returning the `SessionOptions` object itself for chaining.

* Header Management:
    * `set_headers()`: Sets the entire headers dictionary.
    * `set_a_header()`: Sets a specific header.
    * `remove_a_header()`: Removes a specific header.
    * `clear_headers()`: Clears all headers.
* Cookies Management:
    * `set_cookies()`: Sets one or more cookies.
* Other Settings:
    * `set_timeout()`: Sets the connection timeout.
    * `set_retry()`: Configures retry attempts and intervals.
    * `set_proxies()`: Sets proxy information.
    * `set_download_path()`:  Sets the default download directory.
    * `set_auth()`:  Sets authentication credentials.
    * `set_hooks()`: Sets event callback functions.
    * `set_params()`: Sets query parameters for requests.
    * `set_cert()`: Configures SSL client certificates.
    * `set_verify()`: Enables or disables SSL certificate verification.
    * `add_adapter()`: Adds a custom adapter for specific URLs.
    * `set_stream()`: Enables or disables streaming responses.
    * `set_trust_env()`:  Sets whether to trust environment variables.
    * `set_max_redirects()`: Sets the maximum number of redirects.

**Saving Settings**

* `save()`: Saves the current configuration to the INI file from which it was loaded or a specified path.
* `save_to_default()`: Saves the configuration to the default `configs.ini` file.

**ChromiumPage in Depth**

`ChromiumPage` is designed for browser control, providing methods to interact with web pages and manage browser functionalities.

**Creating a ChromiumPage Object**

You can create a `ChromiumPage` object using different methods, controlling how the browser is launched or taken over.

* **Direct Creation:** This uses the default configuration from the INI file or built-in defaults if the file doesn't exist.

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()
```

* **Specifying Port or Address:** You can take over a browser on a specific port, or if the port is free, launch a new browser on that port.

```python
page = ChromiumPage(9333)  # Take over or launch on port 9333
page = ChromiumPage('127.0.0.1:9333')  # Specify address and port
```

* **Creation through Configuration Information (`ChromiumOptions`):**  `ChromiumOptions` provides fine-grained control over the browser launch.

```python
from DrissionPage import ChromiumPage, ChromiumOptions

co = ChromiumOptions().set_browser_path(r'D:\chrome.exe')  # Set a custom browser path
page = ChromiumPage(addr_or_opts=co) 
```

* **Directly Specifying Address:** You can create a `ChromiumPage` object directly by providing the browser address in the 'ip:port' format. This will either take over an existing browser on that address or launch a new one using the default configuration.

```python
page = ChromiumPage(addr_or_opts='127.0.0.1:9333')
```

* **Creation with a Specific INI File:** You can use a custom INI file for configuration.

```python
from DrissionPage import ChromiumPage, ChromiumOptions

co = ChromiumOptions(ini_path=r'./custom_config.ini')
page = ChromiumPage(addr_or_opts=co)
```

**Taking Over an Open Browser**

`ChromiumPage` can take over an existing browser session if it's launched with the correct debugging port.

* **Programmatically Launched Browsers:** If you launch a browser using DrissionPage and don't close it, subsequent program runs will reuse that browser session.
* **Manually Opened Browsers:** You can enable remote debugging in a manually opened browser by adding `--remote-debugging-port=port_number` to the browser shortcut target. DrissionPage can then take over this browser by specifying the same port number.
* **Browsers Launched by BAT Files:** You can create a BAT file to launch the browser with remote debugging enabled, and then take over using DrissionPage.

**Multi-Browser Coexistence**

You can manage multiple browsers concurrently by configuring distinct ports and user data directories for each browser.

* **Specifying Independent Ports and Data Folders:** Use `ChromiumOptions` to set separate ports and user data paths for each browser instance.

```python
from DrissionPage import ChromiumPage, ChromiumOptions

co1 = ChromiumOptions().set_paths(local_port=9111, user_data_path=r'D:\data1')
co2 = ChromiumOptions().set_paths(local_port=9222, user_data_path=r'D:\data2')

page1 = ChromiumPage(addr_or_opts=co1)
page2 = ChromiumPage(addr_or_opts=co2)
```

* **Automatic Port Assignment (`auto_port()`):** The `auto_port()` method in `ChromiumOptions` dynamically assigns free ports and temporary user data directories for each browser, ensuring isolation but preventing reuse across program runs.

```python
co1 = ChromiumOptions().auto_port()
co2 = ChromiumOptions().auto_port()

page1 = ChromiumPage(addr_or_opts=co1)
page2 = ChromiumPage(addr_or_opts=co2)
```

* **Setting Automatic Assignment in the INI File:**  You can configure `auto_port()` permanently in the INI file.

```python
ChromiumOptions().auto_port(True).save()
```

**Using the System Browser User Directory**

By default, DrissionPage creates empty user directories for each browser instance. To leverage existing user profiles, plugins, and settings, use `ChromiumOptions.use_system_user_path()`.

```python
co = ChromiumOptions().use_system_user_path()
page = ChromiumPage(co)

# Alternatively, set it in the INI file for persistent use
ChromiumOptions().use_system_user_path().save()
```

**Creating a New Browser Profile**

To ensure a clean browser state without existing data, use `auto_port()` or manually specify a new port and an empty user data directory.

**Visiting Web Pages with `get()`**

Similar to `SessionPage`, `ChromiumPage` provides a `get()` method for navigating to URLs, with built-in retry mechanisms for handling network issues.

```python
page.get('https://www.example.com')
page.get('https://www.example.com', retry=1, interval=1, timeout=1.5)  # Configure retries and timeout
```

**Page Loading Modes**

DrissionPage offers different loading strategies to optimize automation efficiency.

* **`normal()` (Default):** Waits for the page to load completely, including resources.
* **`eager()`:** Stops loading after the DOM is ready, potentially skipping resource downloads.
* **`none()`:** Only waits for the initial connection, giving you full control over the loading process.

You can configure the loading mode using `ChromiumOptions`, the `set.load_mode` property of the page object, or the `set.load_mode.xxx()` methods at runtime.

```python
co = ChromiumOptions().set_load_mode('none')  # Set using ChromiumOptions
page = ChromiumPage(co)

page.set.load_mode.eager()  # Set at runtime
```

**Retrieving Page Information**

`ChromiumPage` provides various properties and methods to extract information from the browser.

* `html`: The page's HTML content (excluding iframes).
* `json`: Parses the response content as JSON (if the URL points to a JSON file).
* `title`: The page title.
* `user_agent`: The browser's user-agent string.
* `browser_version`: The browser's version.
* `save()`: Saves the current page as an MHTML file or PDF.

**Page Interaction**

`ChromiumPage` offers extensive capabilities to interact with browser pages, including navigation, element manipulation, script execution, cookie and cache management, and window control.

**Page Navigation**

* `get()`: Navigates to a URL.
* `back()`: Goes back in browsing history.
* `forward()`: Goes forward in browsing history.
* `refresh()`: Refreshes the page.
* `stop_loading()`: Forcefully stops page loading.

**Element Management**

* `add_ele()`: Creates and inserts a new element into the DOM, either visible or invisible.
* `remove_ele()`: Removes an element from the DOM.

**Script Execution**

* `run_js()`: Executes JavaScript code on the page.
* `run_js_loaded()`: Executes JavaScript after the page has loaded.
* `run_async_js()`: Executes JavaScript asynchronously.
* `run_cdp()`: Executes Chrome DevTools Protocol commands.
* `run_cdp_loaded()`: Executes Chrome DevTools Protocol commands after the page loads.

**Cookies and Cache Management**

* `set.cookies()`:  Sets one or more cookies.
* `set.cookies.clear()`: Clears all cookies.
* `set.cookies.remove()`:  Removes a specific cookie.
* `set.session_storage()`: Sets or removes session storage items.
* `set.local_storage()`: Sets or removes local storage items.
* `clear_cache()`: Clears browser cache, including session storage, local storage, cookies, and regular cache.

**Window Management**

DrissionPage allows you to control the browser window's size, position, state, and visibility.

* **Window State and Size:**
    * `set.window.max()`: Maximizes the window.
    * `set.window.mini()`: Minimizes the window.
    * `set.window.full()`: Switches to full-screen mode.
    * `set.window.normal()`: Restores the window to its normal state.
    * `set.window.size()`: Sets the window's width and height.
* **Window Position:**
    * `set.window.location()`: Sets the window's position on the screen.
* **Window Visibility:**
    * `set.window.hide()`: Hides the browser window (Windows only, requires `pypiwin32`).
    * `set.window.show()`: Shows the browser window.

**Page Scrolling**

DrissionPage provides methods to scroll the page vertically and horizontally.

* **Scrolling to Specific Positions:**
    * `scroll.to_top()`: Scrolls to the top of the page.
    * `scroll.to_bottom()`: Scrolls to the bottom of the page.
    * `scroll.to_half()`: Scrolls to the vertical middle of the page.
    * `scroll.to_rightmost()`:  Scrolls to the far right of the page.
    * `scroll.to_leftmost()`:  Scrolls to the far left of the page.
    * `scroll.to_location()`: Scrolls to specific x and y coordinates.
* **Incremental Scrolling:**
    * `scroll.up()`: Scrolls up by a specified number of pixels.
    * `scroll.down()`: Scrolls down by a specified number of pixels.
    * `scroll.right()`: Scrolls right by a specified number of pixels.
    * `scroll.left()`: Scrolls left by a specified number of pixels.
* **Scrolling to Element Visibility:**
    * `scroll.to_see()`: Scrolls until a specific element is visible.

**Scrolling Settings**

DrissionPage allows you to control the scrolling behavior to handle pages that use smooth scrolling.

* `set.scroll.smooth()`: Enables or disables smooth scrolling. It's generally recommended to disable smooth scrolling for automation stability.
* `set.scroll.wait_complete()`: Sets whether to wait for scrolling to complete before proceeding. This is useful when smooth scrolling is unavoidable.

**Handling Pop-up Messages**

DrissionPage provides methods to manage JavaScript alert boxes (alert, confirm, prompt).

* `handle_alert()`:
    * Handles alert boxes by accepting, dismissing, or inputting text.
    * Can wait for the alert box to appear before processing.
    * Option to retrieve the alert message without interacting with it.
    * Can handle subsequent alert boxes (useful for exit confirmations).
* `set.auto_handle_alert()`:  Enables automatic handling of alert boxes within a specific tab or globally for all tabs.

**Closing and Reconnecting**

* `disconnect()`: Disconnects the page object from the browser without closing the tab, freeing resources.
* `reconnect()`: Re-establishes the connection to the browser after a disconnection.
* `quit()`: Closes the browser.

**Element Information Retrieval**

`ChromiumElement` objects represent elements in the browser, inheriting all properties from `SessionElement` and offering additional browser-specific information.

**Size and Location**

`ChromiumElement` provides properties to retrieve the element's size and position within the page, viewport, and screen.

* **Size:**
    * `rect.size`: Element's width and height.
* **Location:**
    * `rect.location`: Coordinates of the element's top-left corner relative to the page.
    * `rect.midpoint`:  Coordinates of the element's center.
    * `rect.click_point`:  Coordinates where the `click()` method will target.
    * `rect.viewport_location`: Coordinates of the element's top-left corner relative to the viewport.
    * `rect.viewport_midpoint`: Coordinates of the element's center relative to the viewport.
    * `rect.viewport_click_point`:  Click target coordinates within the viewport.
    * `rect.screen_location`:  Coordinates of the element's top-left corner relative to the screen.
    * `rect.screen_midpoint`: Coordinates of the element's center relative to the screen.
    * `rect.screen_click_point`: Click target coordinates relative to the screen.
* **Corners:**
    * `rect.corners`:  Coordinates of all four corners of the element relative to the page.
    * `rect.viewport_corners`: Coordinates of all four corners relative to the viewport.

**Attributes and Content**

* `pseudo.before`: Text content of the element's `::before` pseudo-element.
* `pseudo.after`: Text content of the element's `::after` pseudo-element.
* `style()`: Retrieves the value of a specific CSS style property, including pseudo-element styles.
* `property()`: Retrieves the value of a JavaScript property.
* `shadow_root`:  Returns the `ShadowRoot` object if the element has a shadow DOM.

**Status Information**

`ChromiumElement` provides properties to access the element's current state.

* `states.is_in_viewport`:  Whether the element is visible in the viewport.
* `states.is_whole_in_viewport`: Whether the entire element is visible within the viewport.
* `states.is_alive`:  Whether the element is still present in the DOM (useful for dynamic pages).
* `states.is_checked`: Whether a checkbox or radio button is checked.
* `states.is_selected`: Whether an option in a `<select>` element is selected.
* `states.is_enabled`:  Whether the element is enabled.
* `states.is_displayed`: Whether the element is visible.
* `states.is_covered`: Whether the element is overlapped by another element.
* `states.is_clickable`: Whether the element can be clicked (based on size, visibility, and enabled state).
* `states.has_rect`:  Whether the element has size and location information.

**Saving Elements**

* `src()`: Retrieves the resource used by the element's `src` attribute (e.g., image data, script content).
* `save()`: Saves the resource retrieved by `src()` to a file.

**ShadowRoot Properties**

`ShadowRoot` objects represent shadow DOM roots, offering properties to access information within the shadow DOM.

* `tag`:  Returns 'shadow-root'.
* `html`: The outer HTML of the shadow root, including the `<shadow_root>` tag.
* `inner_html`: The HTML content within the shadow root.
* `page`: The page object containing the shadow root.
* `parent_ele`: The element to which the shadow root is attached.
* `states.is_enabled`: Whether the shadow root is enabled.
* `states.is_alive`:  Whether the shadow root is still present in the DOM.

**Element Interaction**

`ChromiumElement` provides methods to simulate user interactions with elements, such as clicking, typing, dragging, hovering, and modifying attributes.

**Clicking Elements**

* `click()`, `click.left()`: Simulate a left-click on the element. You can choose between simulated clicks, JavaScript clicks, or automatic selection based on element visibility and state.
* `click.right()`:  Simulates a right-click.
* `click.middle()`:  Simulates a middle-click, optionally returning a new tab object if the click opens a new tab.
* `click.multi()`:  Performs multiple left-clicks.
* `click.at()`: Clicks at a specific offset relative to the element's top-left corner.
* `click.to_upload()`: Triggers the file selection dialog and uploads files.
* `click.to_download()`: Triggers a download and returns a download task object.
* `click.for_new_tab()`: Clicks and waits for a new tab to appear, returning the new tab object.

**Entering Content**

* `clear()`:  Clears the element's text content.
* `input()`: Types text or key combinations into the element, optionally clearing the existing content beforehand. You can use the `Keys` class to input special keys and combinations.
* `focus()`: Sets the focus on the element.

**Dragging and Hovering**

* `drag()`: Drags the element to a new position relative to its original location.
* `drag_to()`:  Drags the element to another element or to specific coordinates.
* `hover()`:  Simulates hovering the mouse over the element, optionally at a specific offset.

**Modifying Elements**

* `set.innerHTML()`: Sets the element's inner HTML.
* `set.property()`:  Sets the value of a JavaScript property.
* `set.style()`:  Sets CSS styles for the element.
* `set.attr()`: Sets the value of an attribute.
* `remove_attr()`: Removes an attribute.
* `set.value()`:  Sets the element's value.
* `check()`: Checks or unchecks checkboxes or radio buttons.

**Executing JavaScript**

* `run_js()`:  Executes JavaScript code within the context of the element.
* `run_async_js()`: Executes JavaScript asynchronously.
* `add_init_js()`:  Adds initialization scripts to be executed before the page loads any other scripts.
* `remove_init_js()`:  Removes initialization scripts.

**Element Scrolling**

* `scroll`: Provides methods for scrolling within scrollable elements, similar to page scrolling methods.

**List Selection (`<select>`)**

The `select` attribute of a `<select>` element provides methods for interacting with drop-down lists.

* `select()`, `select.by_text()`: Selects options by their text content.
* `select.by_value()`:  Selects options by their value attribute.
* `select.by_index()`:  Selects options by their index (starting from 1).
* `select.by_locator()`: Selects options using a locator.
* `select.by_option()`:  Selects options by directly passing `ChromiumElement` objects representing the `<option>` elements.
* Methods for deselecting options are also available: `cancel_by_text()`, `cancel_by_value()`, `cancel_by_index()`, `cancel_by_locator()`, and `cancel_by_option()`.
* `select.all()`: Selects all options in a multi-select list.
* `select.clear()`:  Clears all selections.
* `select.invert()`:  Inverts the current selection.
* `select.is_multi`:  A boolean indicating whether the list is a multi-select list.
* `select.options`: A list of all `<option>` elements.
* `select.selected_option`:  The currently selected `<option>` element (for single-select lists).
* `select.selected_options`:  A list of all selected `<option>` elements (for multi-select lists).

**Waiting**

DrissionPage provides intelligent waiting mechanisms to enhance automation stability and efficiency. The `wait` attribute of page objects and element objects provides various waiting methods.

**Page Object Waiting Methods**

* `wait.load_start()`: Waits for the page to start loading.
* `wait.doc_loaded()`: Waits for the page document to load (usually handled automatically by `get()`).
* `wait.eles_loaded()`: Waits for one or more elements to load into the DOM.
* `wait.ele_displayed()`: Waits for an element to become visible.
* `wait.ele_hidden()`: Waits for an element to become hidden.
* `wait.ele_deleted()`:  Waits for an element to be removed from the DOM.
* `wait.download_begin()`:  Waits for a download to start.
* `wait.upload_paths_inputted()`: Waits for file paths to be filled in the upload dialog.
* `wait.new_tab()`: Waits for a new tab to open.
* `wait.title_change()`: Waits for the page title to change to include or exclude specific text.
* `wait.url_change()`:  Waits for the URL to change to include or exclude specific text.
* `wait.alert_closed()`: Waits for an alert box to close.
* `wait()`:  Waits for a specified number of seconds or a random duration within a range.

**Element Object Waiting Methods**

* `wait.displayed()`:  Waits for the element to become visible.
* `wait.hidden()`:  Waits for the element to become hidden.
* `wait.deleted()`: Waits for the element to be removed from the DOM.
* `wait.covered()`: Waits for the element to be overlapped by another element.
* `wait.not_covered()`: Waits for the element to no longer be overlapped.
* `wait.enabled()`: Waits for the element to become enabled.
* `wait.disabled()`: Waits for the element to become disabled.
* `wait.stop_moving()`:  Waits for the element to stop moving.
* `wait.clickable()`: Waits for the element to become clickable.
* `wait.disabled_or_deleted()`: Waits for the element to become either disabled or removed from the DOM.
* `wait()`: Waits for a specified number of seconds or a random duration within a range.

**File Uploads**

DrissionPage offers two methods for file uploads:

1. **Natural Interaction:** 
    * Use `element.click.to_upload()` to trigger the file selection dialog and automatically fill in the file paths.
    * This approach intercepts the dialog and avoids manual interaction, simplifying automation.

2. **Traditional Method:**
    * Locate the file input element (`<input type="file">`).
    * Use the `input()` method to enter the file path(s).
    * This approach is suitable for simple upload controls but might be less robust for dynamic or complex upload scenarios.

**Tab Management**

DrissionPage provides robust tab management capabilities, allowing you to create, access, switch between, and close tabs efficiently.

**Multi-Tab Usage**

Unlike Selenium, DrissionPage supports multiple tab objects, enabling concurrent tab operations without focus switching. This simplifies code and enhances stability.

* `new_tab()`: Creates a new tab, optionally navigating to a URL.
* `get_tab()`: Retrieves a tab object by its ID, index, title, URL, or type.
* `get_tabs()`:  Retrieves a list of tab objects matching specific criteria.
* `latest_tab`:  The tab object for the most recently activated tab.
* `click.for_new_tab()`:  Clicks an element that opens a new tab and returns the new tab object.
* `click.middle()`:  Simulates a middle-click, commonly used to open links in new tabs.

**Closing and Reconnecting Tabs**

* `close()`: Closes the tab associated with the tab object.
* `disconnect()`:  Disconnects the tab object from the browser without closing the tab.
* `reconnect()`:  Re-establishes the connection to the tab after a disconnection.
* `close_tabs()`:  Closes specific tabs or all tabs except the specified one.

**Activating Tabs**

* `set.tab_to_front()`:  Brings a tab to the foreground without changing the focus of the page object.
* `set.activate()`:  Activates the tab associated with the tab object.

**Multi-Tab Collaboration**

DrissionPage enables seamless interaction between multiple tabs, allowing you to collect data from different tabs without focus switching, enhancing efficiency and code clarity.

**iframe Handling**

DrissionPage simplifies interaction with iframes by providing direct access to elements within iframes without requiring explicit frame switching.

**Retrieving iframe Objects**

* `get_frame()`:  Retrieves an iframe object by its locator, index, ID, or name attribute.
* `get_frames()`:  Retrieves a list of iframe objects matching specific criteria.

**Finding Elements within iframes**

* **Direct Access for Same-Domain iframes:** You can directly locate elements within same-domain iframes using the page object's `ele()` and `eles()` methods. This eliminates the need for frame switching, simplifying code.
* **Accessing Elements in Cross-Domain iframes:** For cross-domain iframes, retrieve the iframe object using `get_frame()` and then use its `ele()` and `eles()` methods to locate elements within the iframe's context.

**ChromiumFrame**

`ChromiumFrame` objects represent iframes, behaving as both elements and page objects.

* **Element Properties:** `tag`, `html`, `inner_html`, `attrs`, `xpath`, `css_path`, `attr()`, `set.attr()`, `remove_attribute()`, and relative positioning methods.
* **Page Properties:** `url`, `title`, `get()`, `refresh()`, `active_ele`, `run_js()`, `scroll`, and `get_screenshot()`.

**Monitoring Network Data**

DrissionPage provides built-in listeners to monitor network data packets exchanged between the browser and web servers. This enables capturing dynamically loaded content and analyzing API requests.

**Example Usage**

* **Waiting and Capturing Packets:**

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()
page.listen.start('api.example.com/data')  # Start listening for packets matching the URL pattern
page.get('https://www.example.com')  # Navigate to the page

packet = page.listen.wait()  # Wait for a matching packet
print(packet.url)
```

* **Real-time Packet Capturing:**

```python
for packet in page.listen.steps():  # Iterate through captured packets in real time
    print(packet.url)
```

**Setting Targets and Starting Monitoring**

* `listen.start()`: Starts the listener and sets target patterns for capturing packets.
* `listen.set_targets()`:  Modifies target patterns during monitoring or sets them before starting.

**Waiting and Retrieving Packets**

* `listen.wait()`:  Waits for a specified number of matching packets to be captured.
* `listen.steps()`: Returns an iterator for real-time packet retrieval.
* `listen.wait_silent()`: Waits for specific requests to complete.

**Pausing and Resuming Monitoring**

* `listen.pause()`: Suspends monitoring.
* `listen.resume()`: Resumes monitoring.
* `listen.stop()`: Stops monitoring and clears the captured packet queue.

**DataPacket Object**

`DataPacket` objects represent captured network packets, providing access to request and response information.

**Request Object**

Contains details of the request, including the URL, method, headers, cookies, and POST data.

**Response Object**

Contains details of the response, including headers, body content, status code, and timing information.

**Action Chains**

Action chains allow you to create complex sequences of user interactions, such as mouse movements, clicks, and keyboard inputs.

**Using Action Chains**

* **Built-in `actions` Attribute:**  Access the action chain through the `actions` attribute of page objects. The actions will be executed after the page loads.
* **Creating New Objects (`Actions`):** Import the `Actions` class and create an object by passing in the page object. This approach doesn't wait for the page to load before executing actions.

**Mouse Movements**

* `move_to()`: Moves the mouse to the center of an element or to specific coordinates.
* `move()`: Moves the mouse relative to its current position.
* `up()`, `down()`, `left()`, `right()`:  Move the mouse in the respective directions.

**Mouse Buttons**

* `click()`: Simulates a left-click.
* `r_click()`: Simulates a right-click.
* `m_click()`:  Simulates a middle-click.
* `db_click()`: Simulates a double-click.
* `hold()`, `release()`: Simulate holding down and releasing the left mouse button.
* `r_hold()`, `r_release()`:  Hold down and release the right mouse button.
* `m_hold()`, `m_release()`: Hold down and release the middle mouse button.

**Scrolling**

* `scroll()`: Simulates scrolling the mouse wheel.

**Keyboard Input**

* `key_down()`:  Simulates pressing a keyboard key.
* `key_up()`:  Simulates releasing a keyboard key.
* `input()`: Types text or key combinations.
* `type()`:  Types text by simulating individual key presses and releases.

**Waiting**

* `wait()`:  Pauses the action chain for a specified duration.

**Screenshots and Videos**

DrissionPage allows you to capture screenshots of web pages and elements, as well as record videos of browser interactions.

**Page Screenshots**

* `get_screenshot()`:  Captures a screenshot of the page, either the entire page or the visible viewport, and saves it to a file or returns it as bytes or a base64-encoded string.

**Element Screenshots**

* `get_screenshot()`:  Captures a screenshot of a specific element.

**Page Recording**

* `screencast`:  Provides methods for recording browser interactions as videos or a sequence of images.
* Recording modes: `video_mode()`, `frugal_video_mode()`, `js_video_mode()`, `imgs_mode()`, `frugal_imgs_mode()`.
* `screencast.set_save_path()`: Sets the directory for saving recordings.
* `screencast.start()`:  Starts recording.
* `screencast.stop()`:  Stops recording.

**Browser Startup Configuration (`ChromiumOptions`)**

`ChromiumOptions` provides comprehensive control over browser launch parameters, preferences, extensions, and other settings.

**Creating a `ChromiumOptions` Object**

* `ChromiumOptions()`:  Creates an object with default settings loaded from the INI file.
* `ChromiumOptions(read_file=False)`: Creates an object with built-in defaults, ignoring the INI file.
* `ChromiumOptions(ini_path='./my_config.ini')`: Loads settings from a specific INI file.

**Command Line Arguments**

* `set_argument()`:  Sets a command-line argument for the browser launch.
* `remove_argument()`: Removes a command-line argument.
* `clear_arguments()`:  Clears all command-line arguments.

**Running Path and Port**

* `set_browser_path()`:  Sets the path to the browser executable.
* `set_tmp_path()`:  Sets the directory for temporary files.
* `set_local_port()`:  Sets the local port for the browser's debugging protocol.
* `set_address()`:  Sets the browser address in the 'ip:port' format.
* `auto_port()`: Enables automatic port assignment and launching a new browser profile.
* `set_user_data_path()`: Sets the path to the user data directory for browser profiles.
* `use_system_user_path()`:  Enables using the system's default browser user data directory.
* `set_cache_path()`:  Sets the browser's cache directory.
* `existing_only()`: Restricts DrissionPage to use only existing browser instances and prevents launching new ones.

**Using Extensions**

* `add_extension()`: Adds a browser extension to be loaded.
* `remove_extensions()`: Removes all extensions.

**User Profile Settings**

* `set_user()`: Sets the browser profile to use.
* `set_pref()`: Sets a preference in the user profile.
* `remove_pref()`: Removes a preference from the `ChromiumOptions` object.
* `remove_pref_from_file()`:  Removes a preference from the actual user profile file.
* `clear_prefs()`:  Clears all preferences.

**Operational Parameter Settings**

* `set_timeouts()`:  Sets timeout values for various operations.
* `set_retry()`: Configures retry attempts and intervals.
* `set_load_mode()`: Sets the page loading strategy.
* `set_proxy()`:  Sets the browser's proxy.
* `set_download_path()`:  Sets the default download directory.

**Other Settings**

* `headless()`: Enables or disables headless mode.
* `set_flag()`: Sets experimental features (Chrome flags).
* `clear_flags_in_file()`: Clears experimental features from the user profile file.
* `clear_flags()`: Clears experimental features from the `ChromiumOptions` object.
* `incognito()`:  Enables or disables incognito (private) mode.
* `ignore_certificate_errors()`: Ignores certificate errors.
* `no_imgs()`: Disables image loading.
* `no_js()`:  Disables JavaScript execution.
* `mute()`: Mutes audio.
* `set_user_agent()`:  Sets the user-agent string.
* `set_paths()`: A convenience method to set multiple path settings.

**Saving Settings to File**

* `save()`: Saves the configuration to an INI file.
* `save_to_default()`: Saves the configuration to the default `configs.ini` file.

**WebPage: Combining Modes**

`WebPage` integrates the capabilities of both `ChromiumPage` (browser control) and `SessionPage` (packet handling), providing a unified and flexible approach to web automation.

**Creating a WebPage Object**

* `WebPage()`:  Creates a `WebPage` object in d mode (browser control) by default, loading settings from the INI file.
* `WebPage('s')`: Creates a `WebPage` object in s mode (packet handling).
* `WebPage(chromium_options=co, session_or_options=so)`: Creates a `WebPage` object using custom configurations for both modes.

**Mode Switching**

* `mode`: Indicates the current mode (either 'd' or 's').
* `change_mode()`: Switches between d mode and s mode, synchronizing login information.

**Cross-Mode Functionality**

You can access functionalities of both modes regardless of the current mode, offering flexibility and efficiency.

* `cookies_to_session()`: Copies cookies from the browser to the `Session` object.
* `cookies_to_browser()`: Copies cookies from the `Session` object to the browser.

**Unique Features of WebPage**

* **MixTab:** The `get_tab()` method returns `MixTab` objects, which, like `WebPage`, support mode switching and offer most tab management capabilities.
* **Closing Objects:**
    * `close_driver()`: Closes the browser and switches to s mode.
    * `close_session()`: Closes the `Session` object and switches to d mode.
    * `close()`: Closes the current tab and `Session` object.
    * `quit()`:  Closes both the `Session` and `Driver` objects, as well as the browser.

**Simplified Writing**

DrissionPage offers simplified writing conventions for conciseness and readability.

* **Locator Syntax Simplification:** You can use shorter abbreviations for common locator components, such as `#` for ID, `.` for class, `tx` for text, `t` for tag, `x` for XPath, and `c` for CSS selectors.
* **Shadow Root Simplification:** `ele.shadow_root` can be shortened to `ele.sr`.
* **Relative Positioning Parameter Simplification:** You can directly pass the index as the first argument to relative positioning methods like `parent()`, `next()`, and `before()`.

**Advanced Tips and Tricks**

* **Custom Waits:** You can create custom waiting conditions using the `wait_until` function to pause execution until specific conditions are met.
* **Element Existence Checks:** The `NoneElement` object returned when an element is not found evaluates to `False` in a conditional statement, providing a convenient way to check for element existence.
* **Custom Exception Handling:** You can configure DrissionPage to throw an exception immediately when an element is not found using `Settings.raise_when_ele_not_found = True`.
* **Default Return Values for Missing Elements:**  Set a default return value for missing elements using `page.set.NoneElement_value()` to handle situations where an element might not exist without raising exceptions.
* **Filtering Element Lists:** DrissionPage provides methods to filter lists of elements based on visibility, enabled state, selection status, attributes, text content, and other criteria.
* **Batch Attribute and Text Retrieval:**  Use the `get.attrs()`, `get.links()`, and `get.texts()` methods to retrieve attribute values and text content from lists of elements efficiently.
* **Cross-iframe Element Access (Same Domain):**  For iframes within the same domain, you can directly access elements within the iframe using the page object's methods, eliminating the need for frame switching.

**Conclusion**

DrissionPage is a powerful and versatile Python library for web automation. Its unique combination of browser control and packet handling capabilities, coupled with its intuitive locator syntax, flexible waiting mechanisms, and advanced features like action chains, network monitoring, and iframe handling, makes it an excellent choice for a wide range of automation tasks, from simple data scraping to complex web interactions.

This comprehensive guide provides a detailed understanding of DrissionPage's components, methods, and workflows, empowering you to leverage its full potential for your web automation projects.

