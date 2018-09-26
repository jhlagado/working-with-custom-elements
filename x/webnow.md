
## Web app perform zone 2 — progressive JPEGs and tricks to halve your image load

![](https://cdn-images-1.medium.com/max/3840/1*25nV0DHqpDDCz10_g2pJkQ.png)

## Performance, performance, performance

In my [last talk on Lighthouse integration](https://itnext.io/web-app-perform-zone-1-lighthouse-ci-integration-3b06841770c2) we grilled Australian websites under a slow 3G network and kicked off a performance improvement journey by integrating [Lighthouse](https://github.com/GoogleChrome/lighthouse) into code review. Here I am going to give a few tips on performant images. 

Before the main topic:

## Performance is not a developer fuss.

* [Pinterest rebuilt their pages for performance realizing a 40% reduction in perceived wait times, thus increasing both search engine traffic and sign-ups by 15%](https://medium.com/@Pinterest_Engineering/driving-user-growth-with-performance-improvements-cfc50dafadd7).

* [By cutting average page load time by 850 milliseconds, COOK found they were able to increase conversions by 7%, decrease bounce rates by 7%, and increase pages per session by 10%](https://www.nccgroup.trust/uk/about-us/resources/cook-real-user-monitoring-case-study/?style=Website+Performance&resources=Case+Studies).

If you are interested, there is an in-depth article on Why Performance matters. Teleport is [here](https://developers.google.com/web/fundamentals/performance/why-performance-matters/). Web developers care about images. If you have a performance budget in your project or in your mind, No. 1 priority should be given to images. As — 
>  According to the [**HTTP Archive](http://httparchive.org/)**, 60% of the data transferred to fetch a web page is images composed of JPEGs, PNGs and GIFs. As of July 2017, images accounted for [**1.7MB](http://httparchive.org/interesting.php#bytesperpage)** of the content loaded for the 3.0MB average site.

Yes, images are half of the web page load. And compared with JavaScript bundle which we spend days and nights trying to split it into meaningful chunks and to lazy load, image processing is transparent and static. Today I am sharing a few of the low hanging fruit with respect to images:

## Fruit 1 — Progressive JPEGS

When a baseline JPEG loaded, as we see most of the time the image appears as a top-to-bottom scan. A progressive JPEG by comparison is a series of scans of increasing quality. Check B-JPEG VS P-JPEG below:

![Top-to-bottom scan reveals image gradually. User will need to wait for full image download. ](https://cdn-images-1.medium.com/max/2048/1*VA0-5sIgdCCna-2v5rKpKw.png)

![Progressive JPEG unveils itself in a series of scans. User doesn’t need to wait if user is not interested in image details](https://cdn-images-1.medium.com/max/2048/1*kazfZ1stBEURYwmk_VTy7A.png)

As you can see, progressive images give the user a perception that the image loading is “visually completed”. This will give user a feeling that page loading is fast. This conclusion is not subjective. There are a number of big names that put progressive jpegs under their belts:

* [**Twitter.com ships Progressive JPEGs](https://www.webpagetest.org/performance_optimization.php?test=170717_NQ_1K9P&run=2#compress_images)** with a baseline of quality of 85%. They measured user perceived latency (time to first scan and overall load time) and found overall, PJPEGs were competitive at addressing their requirements for low file-sizes, acceptable transcode and decode times.

* [**Facebook ships Progressive JPEGs for their iOS app](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/)**. They found it reduced data usage by 10% and enabled them to show a good quality image 15% faster.

* [**Yelp switched to Progressive JPEGs](https://engineeringblog.yelp.com/2017/06/making-photos-smaller.html)** and found it was in part responsible for ~4.5% of their image size reduction savings. They also saved an extra 13.8% using MozJPEG.

From the numbers, you can see it is faster but not really that impressive. So I did a demo project to experiment:

I built a demo site from [create-react-app](https://github.com/facebook/create-react-app) template and added [material-UI Carousel](https://demos.creative-tim.com/material-kit-react). I used [image-min webpack plugin](https://github.com/Klathmon/imagemin-webpack-plugin#readme) to make images progressive. Turning on slow 3G network throttling in Chrome, I get following scans:

![image(bg6) show initial coarse scan as 36 kb loaded](https://cdn-images-1.medium.com/max/5628/1*G962D1fKcS71qb7RSCUg8g.png)

![with secondary scan(68kb) loaded, image looks almost same with full image on 15 inch screen](https://cdn-images-1.medium.com/max/5610/1*6N52CAXZaKTXej7HsBsMvQ.png)

![image keeps replacing itself with better scan, till a final scan with full size 239kb](https://cdn-images-1.medium.com/max/5598/1*ef9sL_HjS4sLzxAfmZje4w.png)

Pictures above are quite self-explainatory. With the second scan of 68kb, you almost get the full details of the 239kb image. That’s actually a 71% of initial page load reduction. For proactive users who don’t care about image details, the user can use initial scan of 36kb as full image and react. That is 84% less. 

So Why NOT turn progressive? Progressive JPEGs are useful under slow network conditions, amusing at normal network and stupidly simple to make. Only a few lines of code can make this difference. Check my commit to apply this changes for all the details. You only need to install a plugin and add it to your build process.
[**feat(app): add imagemin to create progressive image · hurricanew/webnowtalk@0bc7da9**
*Contribute to hurricanew/webnowtalk development by creating an account on GitHub.*github.com](https://github.com/hurricanew/webnowtalk/commit/0bc7da903ff85b1d9b69a64272ba4bec532925da)

## Fruit 2 — [Guetzli](https://github.com/google/guetzli) Compression

Ok, lets see if we can go further on JPEG. For image formats, developers are picky and reserve JPEG as first choice for large images. 

![Jpeg format is the one if no animation and fine details are needed. ](https://cdn-images-1.medium.com/max/2000/1*sOA0gY49OXw3xIQ-sAcy7A.png)

However, is a JPEG image rock solid on every pixel? I came across a “magic” tool called Guetzli. It amazes me in two aspects:

1. I could not pronounce it. I can’t find anywhere where the word “Guetzli” is used.

2. It can halve your JPEG images with no visual loss. 

Talking is cheap, lets act. With OSX: brew install guetzli Then navigate to images folder, run guetzli --quality 84 [origin file name].jpg [new file name].jpg And here is the before/after JPEGs with full zoom:

![Guetzli compressed image (left) has no visual difference from original](https://cdn-images-1.medium.com/max/5256/1*de14xuohXAJTgxh-Z58RWA.png)

**Size? Origin is 257 kb and compressed is 135kb, that is 47% less!**

You may have noticed I have set the quality to 84 which is the bottom of quality range [Guetzli](https://github.com/google/guetzli) allows. It shaved half of my JPEG. It just worked. How does this magic happen? According to [Guetzli](https://github.com/google/guetzli) official definition:
>  [**Guetzli](https://github.com/google/guetzli)** is a promising, if slow, perceptual JPEG encoder from Google that tries to find the smallest JPEG that is perceptually indistinguishable from the original to the human eye. It performs a sequence of experiments that produces a proposal for the final JPEG, accounting for the psychovisual error of each proposal. Out of these, it selects the highest-scoring proposal as the final output.

Actually [**Guetzli](https://github.com/google/guetzli) **performs a sequence of compression solutions and has a score system to pick the best one. It is democratic. Processing takes time and because it is once-off process, it is totally harmless. 

Of course automation is a must and there is [webpack guetzli plugin](https://github.com/imagemin/imagemin-guetzli) for it. 

## Fruit 3— SVGO the SVG sweetener

Whether you like it or not, SVG is the king for small images now. Small size, vector, xml code which gives accessibility and an SEO boost. Is SVG the end of the evolution? As a performance paranoid, you should challenge everything. 
>  SVG files, especially those exported from various editors, usually contain a lot of redundant and useless information. This can include editor metadata, comments, hidden elements, default or non-optimal values and other stuff that can be safely removed or converted without affecting the SVG rendering result.

So SVG as text based source file, is just like HTML. It has a lots of room to improve. 

Staging time for SVGO -> **SVG O**ptimizer is a Nodejs-based tool for optimising SVG vector graphics files. It’s goal is simple, reduce svg weight as much as possible while keeping it looking the same. Let's see what SVGO can do. We have this lovely tiger SVG logo here from illustrator. The size is 5.198 kb. 

![](https://cdn-images-1.medium.com/max/2000/1*qxFWXZIA16BsUVpbyK2Zpw.png)

We need to install SVGO as npm module:

npm i -g svgo

svgo [your file].svg -o [your optimized file].svg 

![svgo shaved off your svg by 49% with no loss](https://cdn-images-1.medium.com/max/2000/1*Xm4CeJKybkobj1LGf4-l1w.png)

Bingo! I get 50% discount. My tiger logo is just as lovely as it used to be.

Looking at the difference, SVGO is not only squeezing white space, it's also removing unused meta data and standardised attributes. Therefore increasing readability. 

![not only squeezing white space, it also remove unused meta data, standardised attributes.](https://cdn-images-1.medium.com/max/5656/1*BiOlqAKPp-yQb1ilUPJicA.png)

And I know you will suffocate without an automated solution. Check [SVGO main page](https://github.com/svg/svgo). Heaps of plugins available there.

That’s it. If you think this article is awfully boring and forget what you have read so far. No worries, you just remember to:

* use webpack/whatever tool you favourite to make your JPEG images progressive

* use Guetzli to halve your large JPEG images

* use SVGO to tidy up your SVG icons

I have tested these tools on production images from sites with high traffic such as banks. You can still get 5 to 40 % off even on highly optimised image assets like these. 

Thanks for reading. So your fruit breakfast is over and in a couple of hours you will be able to halve your project image load. Every web app should have a performance budget. Apply these techniques to your project today and bask in your performance glory! 

I am writing a series posts for performance and follow me if you think it is helpful.
>  In retrospect, all revolutions seem inevitable. Beforehand, all revolutions seem impossible . — *Michael McFaul*

Other siblings of this post so far:
[**Web app perform zone 1 — lighthouse CI integration**
*Blah before the topic*itnext.io](https://itnext.io/web-app-perform-zone-1-lighthouse-ci-integration-3b06841770c2)


