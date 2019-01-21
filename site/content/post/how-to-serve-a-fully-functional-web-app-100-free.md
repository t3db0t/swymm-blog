---
title: How to Serve a Fully-Functional Web App 100% Free
date: '2019-01-20T18:57:48-05:00'
---
When I began work on [Swymm](https://www.swymm.org), I wasn't going to make the same mistake I made with the last web app prototype I made—spending money on it.

I'm a product designer and prototyper, but with a lifetime of programming experience. I can build just about anything, but _my goal is to invent something and see if it works,_ not just write a bunch of code. And if you don't know if something is going to work, you better not spend much money on it! In this light, I try to select technologies and platforms that give me as much out-of-the-box as possible.

And unless you have a really good reason, you shouldn't spend one cent on a prototype, and with the options now at our disposal, you don't have to. Usually the costs come with serving the app; you want real people to use your app online, so you need a backend to run the thing. Don't you?

Once I understood the simple idea behind the semi-hyped 'serverless' approach, it felt like coming out of a coma. All software, arguably, is a user interface coupled to data. On the web, our UIs run off JavaScript in a browser. One way or another, the UI is fed data from some kind of store, i.e. a Database.

# A Brief History of AJAX

Historically, the UI was _generated_ on the backend (i.e. PHP) by making database requests and constructing HTML from those responses. There was essentially no application other than the machinery of the backend. And then—

> In 1998, the Microsoft Outlook Web Access team developed the concept behind the XMLHttpRequest scripting object. It appeared as XMLHTTP in the second version of the MSXML library, which shipped with Internet Explorer 5.0 in March 1999. [Source](https://en.wikipedia.org/wiki/Ajax_(programming)#History)

IE was unequivocally a nightmare, and yet it brought us one of the most important advances in web application development: a way to move data around without being one and the same with the backend. This finally allowed us to separate the browser from the backend and conceive of a "browser application."

Then JQuery came along in 2006 and not only made it easier to manipulate the DOM, but wrap these new `XMLHttpRequest` calls into a highly developer-friendly API. This opened the floodgates to better and better iterations of what was then called "Web 2.0".

> "The first glimmerings of Web 2.0 are beginning to appear, and we are just starting to see how that embryo might develop. The Web will be understood not as screenfuls of text and graphics but as a transport mechanism, the ether through which interactivity happens."—[Darcy DiNucci, 1999, "Fragmented Future"](http://darcyd.com/fragmented_future.pdf)

The key word here is interactivity: a user's choices and actions have immediate effects on the UI—and critically, vice versa: the UI could show new data automatically, giving the user better information, faster.

But the backend was still around, of course: that's the thing you made requests to! What would you do without a backend?

# Enough History, Get to the Money

Screw the backend. Seriously, what do you really need it to do? If your UI is completely in the browser, you _serve it statically and make database calls directly from it!_

If you've already been doing this, it sounds obvious. It wasn't completely obvious to me when I first considered it. But the devil's in the details:

* Wait, who serves the database?
* How do you connect to the database?
* How do you authenticate safely?
* How do you form your database queries?

So let me cut to the chase with my particular solution, which I'm overall extremely happy with (with a few caveats).

## This is not a sales pitch. I have zero interest in promoting any particular vendor. This is my current personal solution.

## Serving the UI

First things first: I had heard about [Netlify](https://netlify.com) out and about, which is a very simple and effective static web host that uses git pushes to automatically update your site. If your application is basically a bunch of JavaScript that gets and sends data from somewhere else, why not just serve the thing the good old-fashioned way, with a single HTTP call? (Yes, there are caveats, namely SEO, let's save that for later)

### Cost: $0.00 for one user

Side note: if you shun build chains or are just doing quick-and-dirty prototyping, you can load all your libraries from places like [cdnjs](https://cdnjs.com/) and [unpkg](https://unpkg.com/).

## Which Database, and Who Serves It?

The first thing I tried for Swymm was [Firebase](https://firebase.google.com/) (or more specifically, FireStore). Great design, easy to use, free for a generous amount of calls and data, and **critically, you can connect to it directly from a browser.** So I went fairly far with this, writing a bunch of scripts to scrape [Wikidata](https://wikidata.org).

And then I discovered a very simple but disastrous limitation: no full-text search. (They [propose instead](https://firebase.google.com/docs/firestore/solutions/search) you use a 3rd-party search service, but I was trying to keep things as contained and simple as possible)

Not only that, but I was starting to realize my queries were going to need to get pretty sophisticated. I freaked out a little bit since I had invested a good amount of time into that platform, and then a coworker asked me why I wasn't just using MongoDB.

"I...don't...know...?" My experience with Mongo had been via Meteor, and I had inherited some old prejudices against it. But when I did a new round of research, I discovered that MongoDB's own hosting service, [Atlas](https://www.mongodb.com/cloud/atlas), is actually really robust. Still, though, what was great about Firebase was the direct browser connection. I was committed to no backend, so now what?

I then discovered MongoDB's [Stitch](https://www.mongodb.com/cloud/stitch) offering, which they describe as a "Backend as a Service." You may scoff, but that's a pretty good description.

Stitch supplies the direct-browser connection I was hoping for, along with a user management and auth scheme (because nobody should have to write that shit from scratch _shudder_).

I dove in right away and was really happy and excited, and then—fuck me, Stitch doesn't offer full-text search either. Mongo does, but Stitch doesn't have it in their API. WTF!?

[They say they'll eventually add it](https://groups.google.com/forum/#!topic/mongodb-stitch-users/ovkXk86Zw5Y), but I had to rub my temples, roll up my sleeves and take a whack at another core aspect of the serverless approach: lambdas.

### Cost: $0 for up to 488.28125 GB-seconds/month of transmitted data!?

## Lambda Lambda Lambda

Well, that's what AWS calls them, a name that goes all the way back to LISP! In 1936 a genius mathematician named Alonzo Church invented something called the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus), which was a consistent way of writing down mathematical functions that nest inside of each other. It's basically a string-rewriting system, but it turned out to be monumentally important to computer science. [He used the letter lambda randomly, apparently](https://cs.stackexchange.com/questions/55917/what-does-the-lambda-in-lambda-calculus-stand-for#answer-57465).

So a "lambda" is a function. A function takes in some information and puts out other information. Such as—hey! A database call! AWS introduced this idea in November of 2014, and made the serverless approach pop up overnight.

Now, maybe it's just me, but AWS is so fucking complicated. I get a headache just thinking about thinking about it. If I have to set up a whole API Gateway just to talk to my db... forget it.

But I needed a way to make a regular MongoDB query that gives me the `{$text: {$search:"blorp"}}` feature. I ended up using a [Google Cloud Function](https://cloud.google.com/functions/), which I don't really recommend honestly. The UI is janky as hell. But it works, and that's pretty important! Of course then I discovered that Netlify is [now offering a simplification of AWS lambdas](https://www.netlify.com/features/functions/), so I'd love to try that at some point.

One important detail about using lambdas to talk to your database: [reuse your database connections](https://cloud.google.com/functions/docs/bestpractices/networking), otherwise you can easily max out your connection pool and your beta users are gonna be out of luck.

Bear in mind that you will likely be changing and improving the design of your function rapidly, and they take time to redeploy every time. Annoying to test. It's also one more thing to maintain, so if and when Stitch gets a text search API, I'll be ditching my cloud function faster than a speeding neutrino.

# And that, believe it or not, is that

That's it: serve your Javascript app statically and find a hosted database with a browser API/SDK. Hopefully I saved some work for you.

This is all free up to a point, obviously; each service has a tier at which you will have to shell out some $. But our purpose here is design and prototyping. Even at the free tiers, you can serve a fully-fledged app to quite a lot of users!

# It Can't Be That Great, Can It?

Search Engine Optimization is the one major (potential) caveat with serverless applications. Search engines rely on a bunch of html text floating around on the web that it can crawl. If your application gets 100% of its content _after_ the application has loaded, how can a crawler get its claws on your juicy information?

The answer, broadly speaking, is [prerendering](https://www.netlify.com/blog/2016/11/22/prerendering-explained/), but this is where my expertise starts to fade. I will probably need to do something like this eventually with Swymm, if I want particular timeline searches to be indexed on Google.

But for now—I hope you enjoyed this microtreatise on serverless applications! Ciao!

—t3db0t, Swymm.org Founder
