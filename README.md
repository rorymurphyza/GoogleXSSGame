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
If you enter this as you new blog post, you should see the alert pop up and show that you have outsmarted it. 
