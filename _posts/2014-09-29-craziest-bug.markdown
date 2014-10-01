---
layout: post
title:  My craziest bug in web development(and the funniest one)
date:   2014-09-29
categories: technical
---

Bugs are common companions for web developers, it’s as if they live and die with us.

As a novice, some of my previous challenges were mostly result of inexperience, but this particular bug is definitely one of its kind and I think it’s worth writing about. 

So I was building a website that allows user to upload photos to display on the site. It’s built with Ruby on Rails, using Amazon S3 for photo storage; carrierwave, sidekiq and Redis-To-Go for photo processing. I used Heroku for hosting, with one web dyno and one worker dyno. 

Two days ago I started to test photo uploading feature on my iphone5 by browsing the site with iOS Safari. Surprisingly, I’ve discovered that everytime I click “submit photo” button; it seems as if the photo upload process could never complete, and server would just timeout itself after a while. 

<img src="https://s3-us-west-1.amazonaws.com/stephensxu.github.io/my_craziest_bug/error.jpg" height="300" width="170" alt="">

In my server logs, it would display a “code=H15 dec=’Idle connection” error message from heroku router; then after a minute or so redis would also yell timeout error plus a cluster of stack trace:

<img src="https://s3-us-west-1.amazonaws.com/stephensxu.github.io/my_craziest_bug/server_error.png" height="270" width="750" alt="">

In a working scenario, the web dyno should have received the uploading and then unload the process to worker dyno for processing. However none of these server responded, instead it was heroku router. 

First I thought the server might have timed out due to file size was too big. So I conducted experiments with a 45KB tiny photo on my home wi-fi connection. 

Nope, still same result. That’s weird! I was certain that the same photo, uploading with same network connection worked fine with desktop chrome uploading. “It must be mobile!” it’s yelling in my head. Jesse warned me to not make conclusion prematurely and instead tried to figure out at what stage did the server timeout. I did that by wrapping my photo#create method in photos_controller with a bench method like so:

{% highlight ruby linenos %}

def bench(label)
 t0 = Time.now
 puts("[#{t0}] Before: #{label}")

 result = yield

 t1 = Time.now
 puts("[#{t1}] After: #{label}")
 puts("Total: %0.2f seconds" % [t1 - t0])

 result
end

bench("photoscontroller#create) { @photo = @kitchen.photos.build(photo_params) }

{% endhighlight %}


Tried to upload from iphone again. Nop; not only the server timed out as it had done before; none of the debugging statement in the bench instrumentation got printed out. That means those lines were never executed!

So now I know that the request data never got to my photos_controller and, most likely, it has never reached my web application at all. The timeout must have happened at the heroku web-server level.  What could have caused the request to choke up at heroku server, and how can I explain the redis errors? Is my iphone sending an unreadable request? To my understanding, a HTTP request sent from mobile browser should not be structurally different from a HTTP request sent from desktop browser. 

Next, I switched my web server from WEBrick to Thin, no luck. 

Next, I switched from Wi-Fi to LTE network, still no luck.

Next, I used chrome browser on my iphone5 to upload the same 45kb photo. WAIT IT WORKED! But only works for that one particular photo, all other attempts failed. WHAT! At least on Safari I could consistently reproduce the problem, now this just made life more difficult.

Finally, I tried to upload photo with the xcode iOS simulator Safari browser. Everything worked perfectly fine in the xcode simulator. 

Hmm...those inconsistent results made it really hard to isolate the problem. By now I’m kind of running out of good ideas, other than to try intercepting the request sent by my iphone and look at header and body of the request to find some clues.

Before I went that route, I asked Jesse to perform the same upload action with same 45KB photo with his phone. Well, surprised again! Everything worked fine on his phone, with different sized photos and for both wi-fi connection and LTE. 

“F MY PHONE!” We were both saying. Jesse was on iOS 8.0.2 and I was on iOS 8.0. Later that night I tried with an iphone running on iOS 7, an ipad mini and a Samsung galaxy SIII; they all worked fine. It became apparent that the timeout problem was unique to my phone only. But which configuration could have caused the difference? What if other people have the same configuration with me and go through same experiences when uploading photo?

Both me and Jesse were very curious about the root cause of this issue. So we decided to go ahead and set up a proxy server on my desktop so we can intercept my iphone http traffics, open up the hood and see what exactly is going on with the upload request.

We decided to use <a href="http://mitmproxy.org/index.html">mitmproxy</a>, since Charles proxy isn’t free. Mitmproxy is relatively easy to set up with brew and has a “somewhat intuitive” UI. Instructions of how to use it can be found <a href="http://blog.just2us.com/2012/05/sniff-iphone-http-traffic-using-mitmproxy/">here</a>.

Now I’m armed with a rather powerful tool which would give me much deeper visibility into the conversation between the iphone browser and my server, we’re ready to investigate!

First step is to capture the request that’s causing error, one sent with my iphone. Now with a proxy server in the middle, the timeout situation is slightly different than previously, but both server and safari timed out nevertheless. My mitmproxy did capture the request, this is what the HTTP request looks like:

<img src="https://s3-us-west-1.amazonaws.com/stephensxu.github.io/my_craziest_bug/ios8.0_safari_upload_http.png" height="450" width="750" alt="">

Everything looked normal at first glance. But notice that the body of the request, under the line says “Form data:”, there are four fields. The third field where it says “photo[picture]:” seem to be empty. This is suppose to be the field that contains the actual encoded data of the photo. At first we thought it might be a mitmproxy UI design decision that would hide this data. So we moved on to look at the response part, which look like this:

<img src="https://s3-us-west-1.amazonaws.com/stephensxu.github.io/my_craziest_bug/ios8.0_safari_upload_http_response.png" height="454" width="750">

So indeed there is a response from the server, with a x-Runtime of 2.02 seconds to generate the response. This confirms that the request(or at least part of it) did reach heroku and received a response. At this point it’s still not apparent to us why would this cause server to timeout. So I configured my desktop google chrome to use mimproxy, uploaded a photo and captured a request in this working scenario, which look like this:

<img src="https://s3-us-west-1.amazonaws.com/stephensxu.github.io/my_craziest_bug/desktop_chrome_upload_http.png" height="454" width="750">

Haha! Now the difference became obvious. Notice with the desktop chrome request, the “photo[picture]” field in request body DOES contain data, which confirms that if there are data included in this field it would have been displayed by mitmproxy. 

Looking back to the previous iOS request body, the entire “photo[picture]” field is completely empty, while under the header field “Content-Length”, it says “77646”. Content-Length describe the length of the file in bytes that should have been sent with the request. Basically, when the server saw Content-Length of 77646, it’s expecting the request body to contain 77646 bytes of data; however since the request body “photo[picture]” is empty and size of request data is smaller than the stated amount,, the server just hang around waiting for the rest until it times out. The server was saying to client: “Hey! You said you were gonna send 77646 bytes, but I only got 17482 bytes, where is the rest?” The request data was missing and the server received an incomplete request that it doesn't know how to process. 

Now we’ve gotten to the bottom of the issue, we know that for some reason iOS Safari was not including the uploaded file in the request body but told the server to expect something it never sent. After some googling, we found out this was indeed a bug in iOS 8.0 Safari and have since been fixed by the 8.0.2 upgrade 2 days ago:

<img src="https://s3-us-west-1.amazonaws.com/stephensxu.github.io/my_craziest_bug/ios_8.0.2_update_notes.png" height="382" width="750">

Oh well. Had I read this upgrade notes before going through all these hustles, maybe it would have saved efforts; but if time reverse back I’d rather to go through this process again. This was a great opportunity for me to learn how to use a proxy server for debugging purpose, to do some in-depth analysis of HTTP request and response cycle, and to learn one of my most important lessons in web development so far. I hope you find this useful as well.

Questions? Email me at stephensxub@gmail.com

<a href="{{ site.url }}">Back Home</a>
