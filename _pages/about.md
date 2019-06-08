---
layout: single
permalink: /about/
title: "About me"
author_profile: true
---

I'm Félix Luginbühl, a data analyst, R developer and [Shiny app](https://shiny.rstudio.com/) architect. I build data tools for organizations.

In my spare time, I developed two R packages:

- [wikisourcer](https://lgnbhl.github.io/wikisourcer){:target="_blank"} to download public domain works from Wikisource.
- [polyglot](https://lgnbhl.github.io/polyglot){:target="_blank"} to learn foreign language vocabulary in the R console.

{% capture notice-text %}
* For updates of recent blog posts, follow me on [Twitter](https://twitter.com/lgnbhl).
* For reproducing my data analysis, go on my [Github page](https://github.com/lgnbhl/blogposts).
* Curious about what I can do for your organisation? Have a look at my [Project page](https://felixluginbuhl.com/project/).
{% endcapture %}

<div class="notice--info">
  {{ notice-text | markdownify }}
</div>

<a href="https://www.linkedin.com/in/felixluginbuhl" class="btn btn--danger">Contact me</a>

<form
            class="contact-form d-flex flex-column align-items-center"
            action="https://formspree.io/felix.luginbuhl@protonmail.ch"
            method="POST"
          >
            <h4 class="mb-1 form-group w-75">Want to work with me?</h4>
            <p class="mt-0 form-group w-75">Let me know about your projects and ideas.</p>
            <div class="form-group w-75">
              <input
                type="name"
                class="form-control"
                placeholder="Your name"
                name="name"
                required
              />
            </div>
            <div class="form-group w-75">
              <input
                type="email"
                class="form-control"
                placeholder="Email address"
                name="name"
                required
              />
            </div>

            <div class="form-group w-75">
              <textarea
                class="form-control"
                type="text"
                placeholder="Type a message..."
                rows="7"
                name="name"
                required
              ></textarea>
            </div>

            <button type="submit" class="btn btn-submit btn-default w-75">Send message</button>
          </form>
