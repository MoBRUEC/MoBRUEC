## Take contents from the Power Automate RSS feed and post it to Teams using an Adaptive Card 

E-Mails are nice and all, but sometimes a Teams message would be a lot more convenient.
Let's start with the Power Automate feed (https://flow.microsoft.com/en-us/blog/feed/) and as soon as a new entry is detected, have it run through HTML To Text:
![HTML to Text will help with the special characters not getting in the way!](/images/12.png)

Now let's use that text for our Adaptive Card, but before we do we are going to get rid of any line breaks.
![Composing content for a new Adaptive Card](/images/13.png)
In the compose step, the expression would therefore be:

``` 
replace(variables('myInputString'),decodeUriComponent('%0A'),' ')
```

As for designing the Adaptive Card, all you need is some valid JSON generated via [the Adaptive Card Designer](https://www.adaptivecards.io/designer/). Visually drag & drop what you need and then insert the data properties you want to add wherever you want them to add.
*Important* though: Make sure to set "version" to 1.2! (At least that was necessary as of 2nd of Feb 2023.)
Otherwise the Adaptive Card will not render in Teams. Check out the documentation to find out which version is supported in Teams currently at the time of you reading this.
![Composing content for a new Adaptive Card](/images/14.png)


Want to learn more? Check out my many other articles:

**LinkedIn** > [LinkedIn Articles about Business & Tech](https://www.linkedin.com/today/author/mbrueckner)

**Medium** > [Articles about detailed topics around Cloud Computing and more](https://medium.com/@mohammedbrueckner)

**DevTo** > [Even more detailed articles on topics like using AI Services](https://dev.to/mrbrue)