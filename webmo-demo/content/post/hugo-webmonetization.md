---
title: "SF DeFi Hackathon: Hugo + Web Monetization"
author: Sabine Bertram
date: 2019-11-11T19:08:37-05:00
image: h+w.png
draft: false
---

**by Sabine Bertram, originally posted on [coil.com](https://coil.com/p/sabinebertram/SF-DeFi-Hackathon-Hugo-Web-Monetization/mAownjGgI)**

I recently made my way back to San Francisco to attend San Francisco Blockchain Week 2019. The final event of this week was the [DeFi hackathon](https://hackathon.sfblockchainweek.io/). But after an amazing week of [Crypto Economics Security Conference](https://cesc.io/), [Macro.WTF](https://macro.wtf/), and [Epicenter Conference](https://sfblockchainweek.io/), not to forget about the numerous meetups and Halloween parties, I was pretty exhausted and not in the mood for a hackathon. I didn't want to commit to a team or a project and I also didn't have an idea myself. So on Friday night, after the opening ceremony, I called it a day and went back to my bed and breakfast for a good night's sleep. 

The first thought that popped into my mind the next morning was "What if could easily web monetize a Hugo static site?" It may seem weird that this is the first thing I would think of but considering the facts that (a) I also visited my friends at [Coil](https://coil.com/) during the week and that (b) I have been trying to convince my fiancé (who I call every morning) to enable web monetization on [his blog](https://papierlos-studieren.net/), this is just putting two and two together. 

![Web Monetization](/images/webmo.png)

As most of the readers probably know, [web monetization](https://webmonetization.org/) is a standard. It is a JavaScript browser API which allows the creation of a payment stream from the reader to the content creator. The user needs a browser extension like [Coil](https://coil.com/) or [Minute](https://github.com/interledgerjs/minute), or they are ahead of their time and use the [Puma Browser](https://www.pumabrowser.com/). These will check the website for the existence of a meta tag called `"monetization"` which includes the [payment pointer](https://interledger.org/rfcs/0026-payment-pointers/). 

Full of energy, I headed back to the hackathon, googling [Hugo themes](https://themes.gohugo.io/) on the bus and texting my friend Aaron about the idea. By the time I got to the venue, I had already decided on the Hugo theme I wanted to play with - [Newsroom](https://themes.gohugo.io/newsroom/) by [Weru](https://github.com/onweru). My knowledge on Hugo in general was very limited, though. I have helped my fiancé create his page and I played with one on my own a while ago (mainly because my PhD supervisor keeps nagging me to finally create a personal website), but I wasn't too familiar with the whole framework. Luckily, [documentation](https://gohugo.io/documentation/) is excellent so I had a page up and running in no time. 

![Hugo](/images/hugo.png)

Just for those of you who don't know this famous static website generator: [Hugo](https://gohugo.io/) makes it super easy for average Joe to create a beautiful website. Content creators write all their posts in markdown and all configurations are made in a global `config.toml` file. A theme is imported as a git submodule, which takes care of all the styling. 

## The Hack

The first step was to fork my chosen theme Newsroom and to enable web monetization in general. This was quite easy and only required an HTML template that was loaded in the page header and included the meta tag:
```html
<meta name="monetization" content='{{ .Site.Params.monetization }}'>
```
To make it more user-friendly, the payment pointer is a variable set in the `config.toml`:
```toml
[params]
  monetization = "$twitter.xrptipbot.com/sabinebertram_"
```
I created a demo site, imported my tweaked theme, set the payment pointer in the `config.toml`, and voilà, micropayments were flowing!

![Streaming micropayments](/images/pic1.png)

I could have stopped here and that would have been the smallest hack in history, but I deliberately chose a blog theme to also allow for exclusive content. And here is where it became tricky. First, I thought I would simply let the blog post creator wrap the entire part that is supposed to be exclusive in a `div`-tag. However, this has 2 disadvantages: (a) the user may not know what a `div`-tag is, and (b) Hugo doesn't recognize the markdown within the `div`-tag and thus completely destroys the layout. So that was not an option. 

Alternative 2 was to include just one line above the content that is supposed to be exclusive in the markdown:
```html
<div id="exclusive"></div>
```
I realize that this solution doesn't solve problem (a), but it is just one line that can be copy-pasted by the blogger (and let's face it, we are always copy-pasting stuff). Everything that is included below this line is made exclusive by the magic of JavaScript. How, you ask? 

As mentioned before, web monetization is a JavaScript browser API. If web monetization is enabled on a site, i.e. a meta tag including the payment pointer exists, tools like the Coil extension or the PumaBrowser create the variable `document.monetization`, which includes the state of monetization, `pending` or `started`. They also emit an event called `monetizationstart` whenever the stream of micropayments is starting. 

My code filters all tags below the exclusive `div` tag and hides this content by default. Whenever the `monetizationstart` event is emitted, it shows the content plus a little thank you note. 

![Exclusive content shown](/images/pic3_circles.png)

If `document.monetization` does not exist or the payment stream does not start, the content stays hidden.

![Exclusive content hidden](/images/pic2_circles.png)

I am aware of the fact that this is not the most elegant solution and that a clever person can still access the exclusive content without paying by using the developer console of the browser. If you are a passionate and skilled front-end developer, feel free to improve that! Here are the repositories:

* the theme: https://github.com/sabinebertram/hugo-webmonetization-theme
* the demo: https://github.com/sabinebertram/hugo-webmonetization-demo

The demo is still live and can be found here: https://hugowebmonetization.netlify.com/

*Thanks to [Aaron Soskin](https://twitter.com/AaronSoskin) for all the images, the great blog post on the demo page, as well as supporting me during the hack!*

## What is next for Hugo + Web Monetization?

<div id="exclusive"></div>

First, I want to make exclusive content work on every platform. Currently, unlocking does not work on PumaBrowser even though payments are streamed.

Second, I want to clean up my JavaScript code. After all, this was a hackathon and I don't know much front-end JavaScript. 

Finally, I want to create a pull request to the Hugo github repository to include web monetization in their [internal templates](https://gohugo.io/templates/internal/). They currently already support Google Analytics, Disqus, OpenGraph, and Twitter Cards. This would allow every content creator to enable web monetization by simply including their payment pointer in the `config.toml`. No tweaking of templates necessary. 