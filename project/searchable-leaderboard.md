+++
title = "Searchable Leaderboard"
date = 2018-02-18T13:14:22-05:00
thumbnail = "/img/dblb.png"
link = "http://stat.diabolis.net"
section1_title = "front-end frameworks"
section1_thumbnails = [
    "/img/tn_bs.png",
    "/img/tn_vj.png"
]
section2_title = "back-end technologies"
section2_thumbnails = [
    "/img/tn_nj.png",
    "/img/tn_ms.png"
]
panelcolor = "#7E4E4E"
+++

I made this for fun! When I crossed launching a gaming server off of my bucket list, I decided it would be interesting to keep a leaderboard of all the players who have played on it and their logged scores to date. 

The front-end is a <a class='projectpanel_body_content_link' href="https://vuejs.org/">Vue.js Single Page Application (SPA)</a> that uses a custom <a class='projectpanel_body_content_link' href="https://getbootstrap.com/">Bootstrap</a> theme for styling. The SPA consists of several Vue components that accept user input for searching and paging through logged scores, then issue the respective API calls to get the results requested by said user. A single component, DataTable, handles rendering the JSON returned by API calls into a responsive table. As this part of the application consists of client-side JavaScript, it pairs nicely with my preferred hosting solution at the moment, Amazon Web Services' Simple Storage Service (S3).

The back-end is the simple web API mentioned earlier, and it was written using <a class='projectpanel_body_content_link' href="https://nodejs.org/en/">Node.js and the <a class='projectpanel_body_content_link' href="https://github.com/mysqljs/mysql">MySQL library for Node.js</a>. Access to the database where the players' scores are kept is exposed through several 'get' routes. These routes correspond to requests for score resources that the front-end issues, where the request URI can consists of a 'player' or 'map' name, a 'limit' amount to state how many records to return, and a 'page' parameter for pagination. Middleware is used to process rows returned by a query to the MySQL database, and return JSON that the front-end can use to render a table. 