# CookieMonster - A Burp Suite Extension

#### This extension registers custom session handling rule actions which can be used to...
- easily add client-side cookies to Burp's cookie jar on demand when Burp cannot otherwise detect them, for example if they have been obfuscated by a WAF
- selectively remove specific cookies from Burp's cookie jar
- empty Burp's cookie jar, for example to ensure only the desired cookies (or no cookies at all) are available to Burp

## Overview

![CookieMonster screenshot](https://github.com/bao7uo/CookieMonster/raw/master/images/title_screenshot.png)

CookieMonster is a Burp Suite Extension which allows web application security testers to register various types of cookie-related session handling actions to be performed by the Burp session handling rules.

The extension can be used to add obfuscated WAF client-side cookies to Burp's cookie jar. If Burp cannot detect updates to WAF cookies and update the Burp cookie jar, then after a short time the WAF will block any requests from Burp which don't contain the updated value. This makes it very difficult to use important Burp features such as the Scanner and Intruder.

CookieMonster defeats the WAF by generating a generic PhantomJS script and then calls the PhantomJS binary with the necessary parameters to run the script which loads the web page and waits for the JavaScript to set the cookie, which is then returned by PhantomJS and picked up by the Burp extension. Tests showed that calling the PhantomJS binary was quicker than using Selenium etc. and also it means there are less dependencies.

The other action types are the removal of specific named cookies from Burp's cookie jar, and the ability to empty the whole jar. These additional features add some helpful flexibility when using more complex session handling rulesets, to ensure the session remains valid by avoiding problematic cookies or to ensure specific application code-paths are properly tested.

## Requirements

- Configure Burp Extender Python Environment (Jython)
- PhantomJS which can either be downloaded and installed from http://phantomjs.org/download.html

## Demo

A lab page is provided (hosted on GitHub Pages) to try out the extension.

#### Step-by-step

- Load CookieMonster extension in Burp
- By default the extension contains the correct settings for the demo, so just click 'Add a new "Get cookies" session handler with these settings for these cookies'
- Then go to Burp's project options tab, navigate to the Sessions Handling Rules section and click on the options cog icon to load the JSON options file from CookieMonster's demo folder (https://github.com/bao7uo/CookieMonster/raw/master/demo/CookieMonster_test.json). This will add the session handling rule and actions
- Go to Burp's repeater tab and paste the contents of the txt file from the demo folder (https://github.com/bao7uo/CookieMonster/raw/master/demo/Repeater.txt)
- Add the correct target hostname (pages.bao7uo.com) and port 80 to Repeater

![CookieMonster Demo screenshot](https://github.com/bao7uo/CookieMonster/raw/master/images/demo_screenshot.png)

If a cookie appears and the value changes every time you click "Go" in repeater, then the demo is working. As you will be able to see, the JavaScript response from the server is obfuscated, but CookieMonster is obtaining the correct value and placing it in the cookie jar, and Burp is then adding it to the Repeater Request.

## Usage

### Burp session rules best practice

- It is preferable to use Burp's session management rules to check whether the session is valid so that this extension is only invoked when necessary, to avoid unnecessary PhantomJS requests. Any unnecessary requests could reduce Active Scan or Intruder Attack performance when they are relying on this extension.

### CookieMonster settings for obtaining cookies

The settings/fields shown in the following screenshot are explained in this section.

![CookieMonster settings screenshot](https://github.com/bao7uo/CookieMonster/raw/master/images/settings_screenshot.png)

#### Set cookies to be valid for domain

- The cookies that are obtained by PhantomJS and added to Burp's cookie jar will be scoped to this domain

#### Path to PhantomJS binary

- Download and install PhantomJS from http://phantomjs.org/download.html
- Browse for the PhantomJS binary to populate this field

#### Optional PhantomJS arguments

- Allows PhantomJS to use a proxy server if necessary
- Check out http://phantomjs.org/api/command-line.html to see what options are available

#### Obtain cookies from this URL

- This is the URL that PhantomJS will open to obtain the cookies from

#### Milliseconds that PhantomJS should wait for cookies to be set

- Sometimes the client-side Javascript can take time to run, so this value sets a delay to wait for the cookie to become available to PhantomJS
- During testing, 500 ms seemed to be a reliable delay, but this value can be tuned as appropriate

#### Cookies to obtain

- Add the name of each desired cookie to the list

#### Adding the session handling action

- Once all of the above fields have been populated, click the 'Add a new "Get cookies" session handler' button to register the session handling action with Burp.

#### Using the rule

- In Burp's project options, sessions, rules, add a new rule action which invokes a Burp extension, choosing your new registered action
- Ensure that the rule is located before an appropriate 'Use cookies from Burp's cookie jar' rule
- For best results, it is usually appropriate to add a 'Check session is valid' rule action, located before the 'Get cookies' rule action.
- The .json file in the demo folder of this repository provides a simple example of a rule with working actions

## CookieMonster Roadmap

This project is still under development.

#### Potential future improvements:
- Improve UI
- Nomenclature
- MVC pattern
- Exception handling

#### Potential future features may include:
- Send target URL to repeater
- Burp session management configuration profiles (using load/saveConfigAsJson)
- Allow cookie names to be specified as a regex, or to add all cookies found by PhantomJS
- Get other types of data/fields from phantom

## Contribute
Contributions, feedback and ideas will be appreciated.

## License notice

Copyright (C) 2017 Paul Taylor

See LICENSE file for details.