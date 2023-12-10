---
title: CRM Parameter Browser
date: 2017-03-09 00:00:00 +0300
categories: [Dynamics 365]
tags: [dynamics365, plugin]     # TAG names should always be lowercase
img_path: /assets/img/posts/2017-03-09-crm-parameter-browser
---

A few months ago I read a [question in Stack Overflow](https://stackoverflow.com/questions/40281492/how-to-know-what-inputparameters-values-are-possible-in-dynamics-crm-plugin-cont) where someone was trying to find out all the available Input/Output parameters in plugins. Unfortunately, there's no single source that provides the full list of parameters (until now) and the suggested approach was to take a look of the specific request/response in the MSDN (they're listed under the "Properties" section).

There's nothing wrong with that answer, but I started to think that it would be great to actually have that list containing all the the possible parameters. Sometimes I need to work with some messages that I'm not very familiar with and I have to spend some time investigating the request and response in the MSDN and/or debugging the step to see what information is available in the plugin context.

# Parameter Browser

![ParameterBrowser](1-placeholder.png)
_ParameterBrowser_

# Extraction Query
I based my query in the one available in [Michael Palmer's blog](https://xrmpalmer.wordpress.com/2013/05/27/crm2011-plugin-inputparameter-and-outputparameter-helper/). I added a couple of columns and played a little bit with the types' name (to make them more readable) but basically the query is the same.

You can have a look of it [here](https://gist.github.com/fedejousset/f2cc28665dc7740e31d5bdbc0bf376af).

<script src="https://gist.github.com/fedejousset/f2cc28665dc7740e31d5bdbc0bf376af.js"></script>

# Final comments

The project is available in GitHub <a href="https://github.com/fedejousset/CRMParameterBrowser">GitHub</a> open to any contribution. It was not possible for me to test that every parameter in every message is 100% accurate so please let me know if you spot any error with the input/output parameters so I can fix it.