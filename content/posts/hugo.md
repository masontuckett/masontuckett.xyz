+++
date = '2025-03-10T00:38:44-07:00'
title = 'Hugo'
hideReply = true
author = "Mason Tuckett"
description = "Primer on Hugo Static Site Generator"
tags = [
    "hugo",
    "markdown",
    "html",
]
+++
For my personal website, I opted to use a static site generator.

Not only would this allow me to automate content creation—it avoids the needless clunkiness of content management systems such as WordPress.

For the modern web, it makes no sense to implement a _largely static blog_ through a content management system; it unnecessarily increases your [attack surface](https://www.cvedetails.com/product/4096/Wordpress-Wordpress.html?vendor_id=2337_).

That being said, there are a plethora of static site generators and frameworks available—but many ultimately defeat the purpose of a true static site generator.

Hugo fit my use case perfectly—boasting incredible flexibility while remaining lightweight and highly straightforward.

## What is Hugo?

Hugo is a static site generator written in Go, designed to be both effortlessly hackable and extensive.

Content is written in Markdown and compiled into static HTML files—according to predefined templates/taxonomies.

Hundreds of premade themes are [readily available](https://themes.gohugo.io/)—ranging from simple, lightweight blogs to ready-to-deploy e-commerce platforms.

## Hackability

Hugo operates using a robust template system—streamlining the process of content generation.

Configuration files (YAML, TOML, JSON) are used to configure/manage content and settings—enabling dynamic content generation and layered customization.

Hugo's shortcode system is used in conjunction with its template system, allowing users to create reusable content/functionality within Markdown files. 

Shortcodes are by far Hugo's most important feature, supporting in-depth [dynamic functions](https://gohugo.io/functions/)—ranging from simplistic individual CSS blocks to JS/TS integration. 

### Here is an Example Showcasing Conditional Logic
```md
{{ if isset .Params "title" }}
    <title>{{ .Params.title }}</title>
{{ else }}
    <title>{{ .Site.title }}</title>
{{ end }}
```

## Closing - My Thought Process

I see far too many IT professionals publishing subpar blogs—often touting frameworks that are wholly unnecessary for their use case.

While platforms such as WordPress are a sound choice, I see many professionals opting for frameworks that appear professional or _"technically advanced."_

But their integration only scratches the surface—not fully utilizing the framework's capabilities; __why use such complex systems rudimentarily?__

It does not impress me to have an extensive homelab just to use technologies _you believe make you appear technically accomplished_.

For my use case, I wanted something secure yet simple to host my projects while automating some content creation; security is my end goal.
