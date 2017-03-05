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
