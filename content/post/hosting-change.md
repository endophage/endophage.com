+++
date = "2017-01-29T23:55:55-08:00"
title = "Hosting Change"

+++

Disclaimer: though I am not unique, I should acknowledge that
I receive free hosting from both [netlify.com](https://netlify.com)
and [github.com](https://github.com) for my publicly available
content. /disclaimer

I don't write much, that's pretty obivous from the sparse number of posts
on this site, even less since I moved hosting and pruned some I didn't care
about keeping. I've had three writing/publishing setups prior to this current
iteration, and it's mainly the publishing side that gets in my way whenever
I think about doing more writing.

I started with a Wordpress blog back in 2009. I paid for some cheap shared
hosting, got cPanel based access, dropped in wordpress via the completely
awful interface and I was away. It's hardly surprising in hidsight but within
the first year, the site had been hacked via some wordpress vuln and a bunch
of spam posts had been thrown up. Fortunately it was just Wordpress and not
the hosting that was compromised and I was able to recover it by modifying
the changed (by the hacker) admin password directly in the database. Still,
this put me off Wordpress and shortly after I moved my blog to Posterous.

Posterous was a fantastic platform. The admin pages were slow as hell and it
relied on waaaaay to much JavaScript. However, they knew what was needed of
a hosted writing platform and it had all the features the typical amateur
and probably entry level pro needed at the time. You could construct your
posts offline in emails and send them in when they were ready, Posterous
would take care of posting links to all your social media platforms. For the 
most part though, it got out of your way and let you worry about what you
wanted to communicate.

Unfortunately, for those that don't know, Posterous was bought by Twitter in
what turned out to be a very confused marriage. Twitter seemingly had no
idea what it was planning to do with the service and soon after the acquisition,
shut it down (or hey, maybe this was their plan all along).

With that I took a hiatus from even attempting to maintain a blog of any sort.
It wasn't until I joined Docker a couple of years ago and, now working on open 
source projects I knew I could talk about publicly, I felt like trying to get
back to some writing.

### Static site generators

I knew that I wanted to go with a static site generator this time around.
Markdown is wonderfully simple for my purposes, I really don't need a fancy
WYSIWYG editor, and a static side sidesteps all the issues of user accounts
(at least, user accounts _I_ have to manage and worry about) that can be hacked.

I can write my markdown, version control it, push it somewhere like GitHub 
(with 2FA enabled) as a backup, generate a static set of HTML pages plus
supporting assets, and just upload those to some hosting space. As it seemed
like the popular tool for exactly this, I first tried out Jekyll; up until
I transitioned a few hours ago, I was still using Jekyll, though I hadn't
actually fired it up in a long time.

Jekyll does the job, but is kind of a pain. I know it works for a lot of people,
but with the number of systems I wanted to use it across, having Windows, Mac OS,
and Linux (Arch and Ubuntu) systems in my home, I felt I was always fighting with
it. Almost every time I went to write a new post, I'd get to the point where
I wanted to build the site and something would break. Eventually, I just gave up,
hence my lack of posts.

On top of that, I hadn't really needed to deal with cPanel much during my Wordpress
time. After the initial setup it was just there, I didn't have to actually see it.
With Jekyll I was back on cPanel based hosting (in an effort to keep my site as
cheap to host as possible), and now I was actually having to deal with cPanel.
I don't blame Jekyll for this in any way, it was just one more pain point that
stopped me writing as much as I would have liked; I'd mentally flinch at the thought
of having to fight with Jekyll and cPanel to get a post published.

## The latest iteration

If I want to write some posts, I want everything that isn't _writing posts_ to
get the hell out of my way. To that end, I'd been hearing good things about 
[Hugo](https://gohugo.io) as a static site generator, and more recently, 
Netlify as a build & hosting solution.

The main benefit with Hugo is that, being written in Go, there is a single binary
to be downloaded that will do everything required to build your site. That same
binary even includes a server for development purposes. This greatly simplified
the writing process, and if necessary, building too. All I had to do was pick a 
theme, and as you can see, I like to keep it pretty minimalist, so there were quite
a few options available to me.

Netlify has integrations with a wide array of generators, including Hugo, and will
connect to your VCS repository. Every time you push a change, Netlify will rebuild 
your site and publish the latest version. In the most basic teir, perfect for a 
personal site like this, it's completely free. Additionally, to add a LetsEncrypt
certificate was all of 3 clicks.

I only had to specify 3 parameters to Netlify to get my site up and published:

1. The GitHub repository
2. The build command; in this case `hugo` (literally, that is my entire build command)
3. The directory that would contain my static site after building; by default,
   Hugo puts the build site in a directory called `public` at the location you ran
   the build command.

While Posterous was great (in my opinion at least), I'm pretty damn happy with
this Hugo + Netlify combo. I frankly use social media less and less so that feature
isn't a big loss to me, and the simplicity of this publishing pipeline has 
high value to me.

Anyway, all that was just to say, if you're looking at moving or starting a
personal website, and terminals and markdown don't scare you, Hugo and Netlify 
make a great combination.
