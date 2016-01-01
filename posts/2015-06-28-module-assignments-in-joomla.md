Steven Matz made his MLB debut for the Mets earlier today. [I think the kid's going to be all right.](http://espn.go.com/new-york/mlb/story/_/id/13165823/steven-matz-new-york-mets-pitches-8th-goes-3-3-4-rbis-record-setting-debut?ex_cid=espnapi_public)

A couple of weeks ago I made a [Joomla component](https://github.com/rhoffmann8/joomla-menumodules) for managing menu-module relationships from a per-menu perspective rather than a per-module one. Since this required research on how Joomla records these relationships I figured it'd be worth sharing this information. 

Joomla uses a table, `#__modules_menu` with two columns, `menuid` and `moduleid`. When configuring the Menu Assignment section of a Joomla module there are four options to choose from in the "Module Assignment" dropdown:

* On all pages
* No pages
* Only on the pages selected
* On all pages except those selected

The rules for storing rows in the `#__modules_menu` table for each selection are as follows:

* For all pages, store one row with this module id and **0** as the menu id
* For no pages, do not store any row with this module id
* For only selected pages, store one row per menu id with this module id
* For on all pages except selected, store the *negative* menu id of each selection with this module id

For example, for a module with an id of 10, if I had three menu items -- 1, 2, and 3 -- and I chose only selected with 1 selected, the table would look like this:

moduleid | menuid
:---------:|:-------:
10       | 1

If I chose all except selected with menus 2 and 3 selected, the table would look like this:

moduleid | menuid
:---------:|:-------:
10       | -2
10       | -3

Pretty simple, but considering I could not find this information in any official documentation I'm hoping I can save someone else a trip through the Joomla source code.