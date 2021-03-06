# Rapid prototyping and testing of frontend perf optimization ideas using Webpagetest (WPT)

Couple of days back I was trying to figure out how to test and evaluate the impact of new performance optimizations without having to make server side changes. The optimization that came to mind was [Addy Osmani](https://twitter.com/addyosmani)'s [quicklink](https://github.com/GoogleChromeLabs/quicklink/issues) library. The question is how can one test this library on their page without having to go through a server side change to the base page HTML file. 

## What kind of optimizations can you test? 

There are two places you can inject a script into your page
* Option A - Once the browser decides its navigated to a new page and [fires the frameNavigated](https://github.com/WPO-Foundation/wptagent/blob/56b8c4931864cae6cabe38f2e70f5c9ec75a23a1/internal/devtools.py#L945) event  - this is if you inject the script into the “inject script” section of the WPT UI
* Option B - Once the page has completed loading - this is if you are using “WPT scripts“ to script out a user flow. 

So if you are looking to improve performance of the current page you will need to use option A, but if you are looking to improve performance of future pages by making optimizations to the current page then you should go with option B

Expert tip - insert a custom metric using performance.mark(‘wpt-inject’) to figure out when the JS completed execution.

## Webpagetest to the rescue !

Webpagetest(WPT) scripts are extremely powerful to setup a user flow for testing and you have the ability to inject JavaScripts into the page. So below are the steps on how you can leverage the quicklink library to test how much of an improvement you can see for your website. 

### Setting up the prototype via wpt scripts
In order to use the quicklink library on your page, you will need to first download the file, then inject it into the page and then initialize it. Below is the script that does just that.
```javascript
var script = document.createElement('script'); 
script.type = 'text/javascript'; 
script.onload = function () {
quicklink.listen(); quicklink.prefetch(['https://www.foo.com/product1']); 
}; 
script.src = 'https://cdnjs.cloudflare.com/ajax/libs/quicklink/2.0.0-alpha/quicklink.umd.js'; 
document.getElementsByTagName('body')[0].appendChild(script);
```
* The first thing you need to do is remove line breaks and include it as one single line. Note to make sure you have space between the end of each line ```; ```
```javascript 
var script = document.createElement('script'); script.type = 'text/javascript'; script.onload = function () {quicklink.listen(); quicklink.prefetch(['https://www.foo.com/product1']); }; script.src = 'https://cdnjs.cloudflare.com/ajax/libs/quicklink/2.0.0-alpha/quicklink.umd.js'; document.getElementsByTagName('body')[0].appendChild(script);
```

* The second thing you need to do is to verify if the script is doing what it needs to. In my case I needed this script to be injected right before the close of the ```</body>``` tag of the HTML. I can do this by running the same single line script snippet within the console section of my Chrome browser. If the script is inserted successfully then you know WPT also be able to do the same. 

### Creating the WPT script 
The goal of quicklinks is to safely prefetch future pages users will potentially visit. To do this you will need to do the following 
a) Load the page you wish to initiatize quicklink
b) Allow the library to figure out what are the links within the view port and then check if the device conditions for prefetching are met. If they are then, the library goes ahead and prefetches those urls. 
c) Allow time for the browser to fetch those urls, this would be similar to a user browsing the page. 

Below is the WPT script for the sequence above 
```
logData 0
navigate https://www.foo.com/<the-page-you-wish-to-inject-the-script>/
execAndWait var script = document.createElement('script'); script.type = 'text/javascript'; script.onload = function () {quicklink.listen(); quicklink.prefetch(['https://www.foo.com/<page-you-wish-to-prefetch>']);  }; script.src = 'https://cdnjs.cloudflare.com/ajax/libs/quicklink/2.0.0-alpha/quicklink.umd.js'; document.getElementsByTagName('body')[0].appendChild(script);
Sleep 10 
logData 1
navigate https://www.foo.com/<page-you-wish-to-prefetch>
```

### Where to add the WPT script
You can add the WPT script outline above into the "script" section of webpagetest.org site. Once it's added go ahead and run your tests

## Gotchas 
* Webpagetest does not show the fetches it made to prefetch cache. So don't worry if it's missing from the waterfall.

## Feedback and Results
Personally, I feel WPT is an amazing tool to rapidly test new ideas as long as it involves modifying the HTML file. I am currently testing some of the new ideas out there and will post results of our testing soon. 

## Credits 
* Addy Osmani - For the quicklink library
* Pat Meenan - For an awesome webpage testing platform



