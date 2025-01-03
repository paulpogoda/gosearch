<p align='center'>
<img src='img/gosearch-logo.png' height=50% width=50%><br>
<i>This project heavily relies on contributors, please see <a href="#contributing">Contributing</a> for more details.</i><br>
<code>git clone https://github.com/ibnaleem/gosearch.git && cd gosearch && go build && ./gosearch [username]</code>
</p>

<p align="center">
  <img src="https://github.com/ibnaleem/gosearch/actions/workflows/go.yml/badge.svg?event=push" alt="GitHub Actions Badge"> <img src="https://img.shields.io/github/last-commit/ibnaleem/gosearch"> <img src="https://img.shields.io/github/commit-activity/w/ibnaleem/gosearch"> <img src="https://img.shields.io/github/contributors/ibnaleem/gosearch"> <img alt="GitHub forks" src="https://img.shields.io/github/forks/ibnaleem/gosearch"> <img alt="GitHub repo size" src="https://img.shields.io/github/repo-size/ibnaleem/gosearch"> <img alt="GitHub License" src="https://img.shields.io/github/license/ibnaleem/gosearch">
</p>
<hr>

## What is `GoSearch`?
<p align='center'>
<img src='img/1.png' height=80% width=80%><br>
<img src='img/2.png' height=80% width=80%><br>
<img src='img/3.png' height=80% width=80%><br>
</p>

`GoSearch` is an efficient and reliable OSINT tool designed for uncovering digital footprints associated with a given username. It's fast, straightforward, and dependable, enabling users to track an individual's online presence across multiple platforms. `GoSearch` also integrates data from HudsonRock's Cybercrime Intelligence Database to provide insights into cybercrime-related information. It also taps into [`BreachDirectory.org`](https://breachdirectory.org)'s database offering access to a comprehensive list of data breaches, plain-text and hashed passwords linked to the username. This tool is ideal for those needing accurate, no-frills results when investigating online identities.

## Installation & Usage
```
$ git clone https://github.com/ibnaleem/gosearch.git && cd gosearch
```
```
$ go build
```
For Unix:
```
$ ./gosearch <username>
```
I recommend adding the `gosearch` binary to your `/bin` for universal use:
```
$ sudo mv gosearch ~/usr/bin
```
For Windows:
```
C:\Users\***\gosearch> gosearch.exe <username>
```
## Use Cases
GoSearch allows you to search [breachdirectory.org](https://breachdirectory.org) for compromised passwords associated with a specific username. To fully utilise GoSearch, follow these steps:

1. Obtain a **free** API key from `https://rapidapi.com/rohan-patra/api/breachdirectory`.
2. Include the API key in the command arguments like this:
```
$ gosearch [username] [api-key]
```
GoSearch will automatically generate popular email addresses for a given username.

## Why `GoSearch`?
`GoSearch` is inspired by [Sherlock](https://github.com/sherlock-project/sherlock), a popular username search tool. However, `GoSearch` improves upon Sherlock by addressing several of its key limitations:

1. Sherlock is Python-based, which makes it slower compared to Go.
2. Sherlock is outdated and lacks updates.
3. Sherlock sometimes reports false positives as valid results.
4. Sherlock frequently misses actual usernames, leading to false negatives.

The primary issue with Sherlock is false negatives—when a username exists on a platform but is not detected. The secondary issue is false positives, where a username is incorrectly flagged as available. `GoSearch` tackles these problems by colour-coding uncertain results as yellow which indicates potential false positives. This allows users to easily filter out irrelevant links. If there's enough demand, we might implement an option to report only confirmed results or focus solely on detecting false negatives.

## Contributing
`GoSearch` relies on the [data.json](https://raw.githubusercontent.com/ibnaleem/gosearch/refs/heads/main/data.json) file which contains a list of websites to search. Users can contribute by adding new sites to expand the tool’s search capabilities. This is where most contributions are needed. The format for adding new sites is as follows:

```json
{
  "name": "Website name",
  "base_url": "https://www.website.com/profiles/{}",
  "url_probe": "optional, see below",
  "errorType": "errorMsg/status_code/profilePresence/unknown",
  "errorMsg/errorCode": "errorMsg",
  "cookies": [
    {
      "name": "cookie name",
      "value": "cookie value"
    }
  ]
}
```

Each entry should include a clear and concise website name to facilitate manual searches, helping avoid duplicate submissions.

### `base_url`
The `base_url` is the URL `GoSearch` uses to search for usernames, unless a `url_probe` is specified (see [`url_probe`](#url_probe)). Your first task is to identify the location of user profiles on a website. For example, on Twitter, user profiles are located at the root path `/`, so you would set `"base_url": "https://twitter.com/{}"`. The `{}` is a *placeholder* that `GoSearch` will automatically replace with the username when performing the search.

For example, if you run the query `./gosearch ibnaleem`, `GoSearch` will replace the `{}` placeholder with "ibnaleem", resulting in the URL `https://shaffan.dev/user/ibnaleem`, assuming the `base_url` is set to `https://shaffan.dev/user/{}`. This allows `GoSearch` to automatically generate the correct URL to check for the user's profile.

### `url_probe`
In some cases, websites may block direct requests for security reasons but offer an API or alternate service to retrieve the same information. The `url_probe` field is used to specify such an API or service URL that checks username availability. Unlike the `base_url`, which is used to directly search for profile URLs, the `url_probe` generates a different API request, and GoSearch will display the API URL in the terminal instead of the profile URL.

For example, Duolingo profiles are available at `https://duolingo.com/profile/{}`, but to check if a username is available, Duolingo provides an API URL: `https://www.duolingo.com/2017-06-30/users?username={}`. If we used the `url_probe` as the `base_url`, the terminal would show something like `https://www.duolingo.com/2017-06-30/users?username=ibnaleem` instead of the user profile URL `https://duolingo.com/profile/ibnaleem`, which could confuse users. This distinction helps keep the process clearer and more intuitive, especially for those who may be less familiar with programming.

### `errorType`
There are 4 error types
1. `status_code` - a specific status code that is returned if a username does not exist (typically `404`)
2. `errorMsg` - a custom error message the website displays that is unique to usernames that do not exist
3. `profilePresence` a custom message the website displays that is unique to usernames that exist.
4. `unknown` - when there is no way of ascertaining the difference between a username that exists and does not exist on the website

#### `status_code`
The easiest to contribute, simply find an existing profile and make a request with the following code:
```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "os"
)

func MakeRequest(url string) {
    resp, err := http.Get(url)
    if err != nil {
        log.Fatal(err)
    }

    defer resp.Body.Close()

    fmt.Println("Response:", resp.Status)
}

func main() {
    var url string = os.Args[1]
    MakeRequest(url)
}
```
```
$ go build
```
```
$ ./request https://yourwebsite.com/username
Response: 200 OK
```
Where username is the existing username on the website. Then, make the same request with a username that does not exist on the website:
```
$ ./request https://yourwebsite.com/usernamedoesnotexist
Response: 404 Not Found
```
Copy and set `errorCode`, the field under `errorType`, as the code that's printed to the terminal (in this case it's `404`).
#### `errorMsg`
This is more tricky, so what you must do is download the response body to a file. Luckily I've already written the code for you:
```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
)

func MakeRequest(url string) {
	resp, err := http.Get(url)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(body))
	os.WriteFile("response.txt", []byte(body), 0644)
}

func main() {
	url := os.Args[1] // Take URL as argument from command line
	MakeRequest(url)
}
```
```
$ go build
```
```
./test https://website.com/username
```
Once again, the first username corresponds to an existing account, while the second username is for an account that does not exist. Be sure to rename `response.txt` to avoid having my code overwrite it.
```
$ mv response.txt username_found.txt
```
```
$ ./test https://website.com/username_does_not_exist
```
```
$ mv response.txt username_not_found.txt
```
You’ll need to analyse the response body of `username_not_found.txt` and compare it with `username_found.txt`. Look for any word, phrase, HTML tag, or other unique element that appears only in `username_not_found.txt`. Once you've identified something distinct, add it to the `errorMsg` field under the `errorType` field. Keep in mind that `errorType` can only have one field below it: either `errorCode` or `errorMsg`, **but not both**. Below is *incorrect*:
```json
{
    "errorType": "status_code",
    "errorCode": 404,
    "errorMsg": "<title>Hello World</title>"
}
```
#### `errorMsg`
The exact opposite of `errorMsg`; instead of analysing the `username_not_found.txt`'s response body, analyse the `username_found.txt`'s response body to find any word, phrase, HTML tag or other unique element that only appears in `username_found.txt`. Set `errorType: profilePresence` and set the `errorMsg` to what you've found.
#### `"unknown"`
Occasionally, the response body may be empty or lack any unique content in both the `username_not_found.txt` and `username_found.txt` files. After trying cookies, using the `www.` subdomain, you are left with no answers. In these cases, set the `errorType` to `"unknown"` (as a string) and include a `404` `errorCode` field underneath it.
#### `cookies`
Some websites may require cookies to retrieve specific data, such as error codes or session information. For example, the website `dzen.ru` requires the cookie `zen_sso_checked=1`, which is included in the request headers when making a browser request. To test for cookies and analyze the response, you can use the following Go code:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func MakeRequest(url string) {
	client := &http.Client{}

	// Create a new HTTP GET request
	req, err := http.NewRequest("GET", url, nil)
	if err != nil {
		log.Fatalf("Error creating request: %v", err)
	}

	// Create the cookie
	cookie := &http.Cookie{
		Name:  "cookie_name",
		Value: "cookie_value",
	}

	// Add the cookie to the request
	req.AddCookie(cookie)

	// Send the request
	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("Error making request: %v", err)
	}
	defer resp.Body.Close()

	// Output the response status
	fmt.Println("Response Status:", resp.Status)
}

func main() {
	// Ensure URL is provided as the first argument
	if len(os.Args) < 2 {
		log.Fatal("URL is required as the first argument.")
	}
	url := os.Args[1]
	MakeRequest(url)
}
```

When testing cookies, check the response status and body. For example, if you always receive a `200 OK` response, try adding `www.` before the URL, as some websites redirect based on this:

```
$ curl -I https://pinterest.com/username
HTTP/2 308
...
location: https://www.pinterest.com/username
```

```
$ curl -I https://www.pinterest.com/username
HTTP/2 200
```

Additionally, make sure to use the above code to analyse the response body when including the `www.` subdomain and relevant cookies.

To contribute, follow the template above, open a PR, and I'll merge it if `GoSearch` can successfully detect the accounts.

## LICENSE
This project is licensed under the GNU General Public License - see the [LICENSE](https://github.com/ibnaleem/gosearch/blob/main/LICENSE) file for details.
