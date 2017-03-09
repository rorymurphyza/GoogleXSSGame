# GoogleXSSGame
Solutions to the Google XSS game. I've put in notes on any tricks or thoughts I had while doing this, so hopefully this helps if (like me) this is the first time you are doing something like this.

Warning, contains spoilers so be careful if you are playing the games. 

##Challenge 1
We are presented with a simple text entry box on a website. A quick check shows that the text box does no filtering or validation and creates a URL link with exactly what is written in to it.
This means we can inject an HTML elemnt straight away. The easiest element to inject is:
```HTML
<script>alert("Winner!")</script>
```
Enter this directly in to the textbox and you should be fine.
Alternately, you can enter this as part of the URL:
```HTML
https://xss-game.appspot.com/level1/frame?query=<script>alert("Winner!")</script>
```

##Challenge 2
We now have a blog type website, where we can enter text that gets displayed as a new entry. 
Entering our solution from Challenge 1 simply creates a new blog post with our entry in the `<blockquote>` tags. We know that this won't work for us. Instead of a direct injection, let's try an error injection to see if we can abuse JavaScript a little bit. Using the `<img>` tag, let's see if we can use an `onerror` as suggested in the hints:
```JavaScript
<img src="this.gif" onerror=alert("Winner!") />
```
If you enter this as your new blog post, you should see the alert pop up and show that you have outsmarted it. 

##Challenge 3
Looking at how we can manipulate this, we see that the only place we can inject a new element is directly into the URL bar. Taking a look at the JavaScript running here, we can see that this is where the input into the URL bar is processed:
```JavaScript
<script>
      function chooseTab(num) {
        // Dynamically load the appropriate image.
        var html = "Image " + parseInt(num) + "<br>";
        html += "<img src='/static/level3/cloud" + num + ".jpg' />";
        $('#tabContent').html(html);
 
        window.location.hash = num;
 
        // Select the current tab
        var tabs = document.querySelectorAll('.tab');
        for (var i = 0; i < tabs.length; i++) {
          if (tabs[i].id == "tab" + parseInt(num)) {
            tabs[i].className = "tab active";
            } else {
            tabs[i].className = "tab";
          }
        }
 
        // Tell parent we've changed the tab
        top.postMessage(self.location.toString(), "*");
      } 
      ...
    </script>
```
The `chooseTag(num)` function is where the input is processed, so let's see if we can abuse this. This function is going to insert an HTML element with a title and the picture link. Interestingly, the title part has the `num` input being passed through a `parseInt(num)` function, but the link to the picture does not. We know that we cannot inject anything here, as the `parseInt(num)` function will just return a NaN response. However, looking at the next line, we see that the `num` parameter is simply used there without being tested or escaped. This looks like a great place to break in.

If we insert a `1` as the `num` parameter, we expect the website to operate as usual. However, if we can manipulate what is inserted here, we could potentially cause an injection like we did in the previous challenge. Looking at the link that sources the `img` in the JavaScript, let's make it look like this:
```HTML
<img src="/static/level3/cloud1" onerror="alert("Winner!")/>
```
To do this, we just needed to add the input `1' onerror='alert("Winner!")'/>` to the URL.
I found that this doesn't work in Firefox as firefox seems use URL encoding to replace the ' and " symbols. I'm not 100% sure why, but Chrome doesn't do this.

##Challenge 4
Let's take a look at what we have here. We have a freetext box (that takes anything) which kicks off a timer to display a message. The box functions correctly when we put in a number and anything else just causes the spinner to keep going and going and going. Looking at what happens when we press the button, we see that we put a call to the `startTimer()` function and then draw the spinner using the `onload` feature of an image. This looks like a fantastic place to try break.

The HTML element that is drawn looks like this for an input of 60:
```HTML
<img src="/static/loading.gif" onload="startTimer('60');">
```
We want to manipulate it to look like this:
```HTML
<img src="/static/loading.gif" onload="startTimer(''); alert('Winner!');">
```
So we can simply insert the string `'); alert('Winner!'` into the textbox to break it.

##Challenge 5

Here we have a very simple web app that will grab an input email address and sign it up for a service. Looking at the code running on the site, you can see that although there is a nice textbox that looks like it will be a great place to break in, nothing in actually done with the text input. However, looking at the scripting here, we see that on each page the next page is displayed in the URL. Looking closer, whatever is put in this URL is passed through a decision tree and eventually ends up as an `href` HTML link. Now this looks like something we can use.

The easiest thing to do here would be to create a fake link and an onClick event, similar to what we would have done before. The name of the challenge is "Breaking protocol" though, so maybe there is another way. The `href` tag actually allows us to insert a JavaScript function in place of a link and this sounds like something we can abuse nicely.
A little bit of playing around shows that we can insert this:
```HTML
javascript:alert("Winner!")
```
This will then turn the link that is created into a starter for the JavaScript function which will display the alert.

##Challenge 6
Time for the last bit of this game. We now want to load an external JavaScript file into the site. With this, it means we can do all sorts of naughty things. From what we have done before, we can see that the URL attack is the way to go. The parameter `frame#` serves as a placeholder for whatever script we want loaded into the service. The service will attempt to retrieve this as part of the normal scriptloading process in the `head` section of the HTML.

Before we start trying to get a file loaded, we need to have a file to load. I don't have my own server running where I can host the file so I'm going to go with the hint of using the google.com/jsapi service. If we navigate to http://www.google.com/jsapi?callback=foo, we see a very long file that ends in `foo();`. With a bit of guesswork, we can see that if we use the parameter `callback=alert` we will get back a JavaScript file which contains the line `alert();`. This should be exactly what we want.

Putting the link `http://www.google.com/jsapi?callback=alert` directly into the URL of the website, we now trigger a warning that says that we cannot use some containing the http:// string. Looking at the piece of the website that triggers this warning, we see that it is quite a simple validator that specifically looks for the string "http://...". Perhaps we could get past this by simply not putting the "http:" part into our link?

```HTML
//www.google.com/jsapi?callback=alert
```
This gets past the validator and loads the JavaScript file that we looked at earlier. 

#Conclusion
If you are reading this and know of any other similar challenges, please let me know.
