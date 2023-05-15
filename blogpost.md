---
title: "Ensure proper link resolution in all your fleek hosted sites."
category: "Guides"
date: "2023-05-15 10:00:00"
desc: "Sharing some pro deployment tips for the most popular frameworks"
thumbnail: ".images/general/best-practices-ipfs.png"
alt: "IPFS links resolution"
image: ".images/general/best-practices-ipfs.png"
cannonical: "https://blog.fleek.co/posts/ipfs-links-resolution/"
---

Gonza here from the Customer Support team at Fleek! Being a power user of the fleek platform, I've deployed multiple user's sites that suffer from the same issue: **The links aren't properly resolving on IPFS.** This can be due to many different reasons, so let's cover those to ensure your paths will always resolve properly.

Let's say you saw a really cool template for a blog and you want to deploy it to IPFS via fleek. For this example we will use [Vercel's Blog Starter Kit](https://vercel.com/templates/next.js/blog-starter-kit).

For the sake of simplicity, we will be using the UI app.fleek.co provides to deploy our site. Once you've cloned the repo, head to the hosting tab and input your build settings:

![Build settings](https://i.gyazo.com/4358f2069972fe9ac39832fea3ad2454.png)

Since we need to export this site, we will modify our `package.json` to include an export script, so the final build is static. Your `scripts` section should then look like this:

```
"scripts": {
  "dev": "next",
  "build": "next build",
  "start": "next start",
  "export": "next export",
  "typecheck": "tsc"
}
```

While we are at it, let's create a `next.config.js` file on the root of the blog-starter project to meet the 'next export' requirements:

```
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  images: {
    unoptimized: true,
  },  
}
module.exports = nextConfig
```

This will ensure the images load properly after being exported. Once that's done, deploy your site and it should look something like this: 
https://bold-heart-0892.on.fleek.co

At first glance the site seems to be correctly deployed and the routes work as intended. But what happens when you try to refresh a subpage , or access it directly ?

If we try to access https://bold-heart-0892.on.fleek.co/posts/dynamic-routing directly, we will most likely get the following error:

```ipfs resolve -r /ipfs/bafybeigg6r5ayu2eybrbi2ggdohxejqjtflxmrvgi4szo3o4reuwf6v4sy/posts/dynamic-routing: no link named "dynamic-routing" under Qma3aXUsxB1ketUS7zvCT1ZT4J87FyCf4x8YSuNgoxyCr2```

This is a really bad experience for anyone using the site. But don't worry, we can fix it! By using **trailing slashes**, we ensure that the URLs of our website are properly resolved. Specifically, when we use trailing slashes, we are indicating to the IPFS network that the URL represents a directory, rather than a file. This is important because **IPFS uses a content-addressed system**, which means that each piece of content is identified by a unique hash. Let's add it to our `next.config.js`:

```
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  images: {
    unoptimized: true,
  },  
  trailingSlash: true,
}
module.exports = nextConfig
```

***Note:** If you were using React, instead of adding `trailingSlash: true`, you would replace `BrowserRouter` with `HashRouter` in your `index.js` and `App.js` to achieve the same behaviour.*

Now let's try to deploy again. I will use a separate site to compare the behaviour, but you can just update the example site:
https://super-surf-8983.on.fleek.co/

Notice that this time, I'm able to refresh the subpages, and access them directly without errors: 
https://super-surf-8983.on.fleek.co/posts/dynamic-routing/

But what would happen if one of your links expire, or a user queries for a page that doesn't exist ? 
https://super-surf-8983.on.fleek.co/about

```ipfs resolve -r /ipfs/bafybeiebjbtdhxcw6eburuuah4bonyx5ekt2jgeapie37oist4illxcgga/about: no link named "about" under QmRoAdU8EWMv2BJqCpMxS37MznZdA1ZdBqnnQxpnCUwPvk```

We would get this pesky IPFS error again. So how do we ensure that our site will never display this error ? 

#### ***Enter the `_redirects` file.***

This feature enables support for redirects, single-page applications, custom 404 pages, and moving to IPFS-backed hosting without breaking existing links. It is very easy to implement, we only need to **add a file called _redirects to the public folder of our project**. You can read more about the implementation cases [here](https://docs.ipfs.tech/how-to/websites-on-ipfs/redirects-and-custom-404s/#evaluation), for our example, let's redirect all invalid links to our 404 page

The content of the `_redirects` file should look like this:
`/* /404.html 404`

Once we update our site with this change, we can now access https://proud-wave-0766.on.fleek.co/about and instead of seeing an intimidating IPFS error message, we will see our custom 404 page, ready to catch any broken or rotten links. You can customize your 404 page however you want, for instance, you could also choose to send the user to your homepage instead:

`/* /index.html 301`

A wide array of possibilites opens up with IPFS _redirects support. We've covered multiple options to ensure proper link resolution on your IPFS site. How you handle your site's pages and links is up to you now.
