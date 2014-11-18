font-embedding
==============

Experimenting with different methods of font embeding


PROTOTYPE NOTES

[5th November 2014]

<https://speakerdeck.com/andyhume/web-fonts-as-a-progressive-enhancement-ampersand-2013>

Font requirements 
= Never block rendering
= Avoid flash of fallback font
= Compress and subset fonts as much as possible
= Reduce HTTP requests to as few as possible
= Cache as agressively as browsers allow

Websites do not need to look the same in every browser

http caching is vital, localstorage works

Modern browser, supporting WOFF, with local storage available then display fonts, otherwise dont.
====> what are the support guidelines and mobile implications of this?

[6th November 2014]

I am coming to the conclusion that we should be using progressive enhancement for fonts, things dont need to look the same in every browser, and specifically since the fallback is so close. I think we should be targetting only A grade browsers, which means using WOFF and base 64 encoding may well be ok, since they are IE9 and up.

http://www.gorges.us/blog/google-fonts
- getting woff format and using it

http://www.fontsquirrel.com/tools/webfont-generator
- font generator to different formats!

https://github.com/typekit/webfontloader
- using this, browser support here https://github.com/typekit/webfontloader#browser-support

https://github.com/typekit/webfontloader#events
.wf-loading
.wf-active
.wf-inactive

font seems to reload even when you refresh the page, not just on force refresh.

Moving to s3 from Dropbox in order to test far future cache expires headers

base 64 encoding can be done here http://software.hixie.ch/utilities/cgi/data/data

set far future expire headers in s3 
cache headers here: https://console.aws.amazon.com/s3/home?region=us-east-1#
max-age=15778463

moved the script back to the head and its much snappier

[?] if we use base 64 encoding or we dont use the font loader, how can we reliably tell if the font is working or not?

[10th November 2014]

Marcos suggests using the BBC method

function cutsTheMustard(){
   return 'querySelector' in document && 'localStorage' in window && 'addEventListener' in window;
}

http://www.smashingmagazine.com/2014/09/08/improving-smashing-magazine-performance-case-study/
-> how smashing magazine do fonts

Their budget is:
  Speed Index <= 1000
  Ensure that all HTML, CSS and JavaScript fit within the first 14 KB
  And they use the Google Font Loader you showed me
I think they answer some of your doubts about the localstorage

Smashing magazine found "HTTP cache was quite unreliable"

Looks like smashing magazine had the same issues with the font loader that I did
"However, we discovered that after switching to WebFontLoader, we started seeing FOUT way too often, with HTTP cache being quite unreliable again, and that permanent switch from a fallback font to the web font being quite annoying, basically ruining the reading experience."


"One solution was to include the @font-face directive only on larger screens by wrapping it in a media query, thus avoiding loading web fonts on mobile devices and in legacy browsers altogether. (In fact, if you declare web fonts in a media query, they will be loaded only when the media query matches the screen size. So no performance hit there.)"
-> but they really wanted their fonts to work on mobile so ...

"The only other option was to improve the caching of fonts. We couldn’t do much with HTTP cache, but there was one option we hadn’t looked into: storing fonts in AppCache or localStorage. Jake Archibald’s article on the beautiful complexity of AppCache led us away from AppCache to experiment with localStorage, a technique that The Guardian’s team was using at the time."
-> http://alistapart.com/article/application-cache-is-a-douchebag
-> https://github.com/ahume/webfontjson

does this mean that if the only reason they went off appcache was because of Jakes experiences that the thing Jake has been working on tp replace appcache, could that be a goer?

with local storage it seems you can only cache 5Mb per domain
-> http://www.html5rocks.com/en/tutorials/offline/quota-research/
localStorage is much slower than HTTP cache, but it’s more reliable.


SMASING MAGAZINE LOGIC
	does the browser support localstorage?
		is the cookie set (the font has been stored) -> JavaScript snippet in the header before the body element
			load from localstorage
		else
			- load web fonts asynchronously after the page has started rendering
			- set cookie
	else 
		does the browser support WOFF?
			include fonts with good ol’ link href and, well, frankly just hope for the best — that the fonts will be properly cached and persist in the user’s browser cache.
		else
			we provide external URLs to fonts with older font mime types, hosted on Fontdeck.

	"We could have avoided the switch by just storing the fonts in localStorage on the first visit and have no switch during the first visit, but we decided that one switch is acceptable, because our typography is important to our identity."
	-> not an option for us either

	"Because the cache isn’t persistent in WebViews, fonts do load asynchronously in applications such as Tweetdeck and Facebook, yet they don’t remain in the cache once the window is closed. In other words, with every WebViews visit, the fonts are re-downloaded. Some old Blackberry devices seemed to clear cookies and delete the cache when the battery is running out. And depending on the configuration of the device, sometimes fonts do not persist in mobile Safari either."
	-> acceptable

OUR INTENDED LOGIC
	does the browser support localstorage?
		is the fontLoaded cookie set
			- load from localstorage
		else
			does the browser support WOFF?
				- load web fonts asynchronously (WOFF/WOFF2) after the page has started rendering with timeout to avoid never loading
				- set cookie if font loaded
			else
				-x no fonts loaded
	else 
		-x no fonts loaded


[11th November]

data:application/font-woff;charset=utf-8;base64,wOFF%00%01%00%00%00...) format('woff'); or data:application/font-woff,wOFF%00%01%00%00%00...) format('woff'); ?

http://software.hixie.ch/utilities/cgi/data/data my woff was wrong in format, the %00 shouldnt be there apparently






