---
layout: post
title: "A framework a day..."
description: ""
category: 
tags: [javascript]
---
{% include JB/setup %}

It seems like there's a new JS framework every day. Let's find out if that's actually the case.

<!--more-->

I hacked up a quick [library](https://github.com/rhoffmann8/ghapi.js) to interact with the Github API and search for all JS repositories created on today's date. Let's see if it's true for today.

Today's results for "javascript framework":

<table id="frameworks_table" class="results-table">
	<thead>
		<tr>
			<th>Name</th>
			<th>Description</th>
			<th>URL</th>
		</tr>
	</thead>
	<tbody></tbody>
</table>

Bonus: Today's results for "opinionated":

<table id="opinionated_table" class="results-table">
	<thead>
		<tr>
			<th>Name</th>
			<th>Description</th>
			<th>URL</th>
		</tr>
	</thead>
	<tbody></tbody>
</table>

And yes, I realize I more or less wrote a framework in order to get these lists of frameworks.

<script src="https://cdn.rawgit.com/rhoffmann8/ghapi.js/master/ghapi.js"></script>
<script type="text/javascript">
var today 		= new Date(),
	todayStr 	= today.toISOString().substr(0,10);

var buildTable = function(tableId, data) {
	var table = document.getElementById(tableId),
		tbody = table.getElementsByTagName('tbody')[0],
		i, row, el;

	if (!data.items.length) {
		tbody.innerHTML = "No results (yet)!";
		return;
	}

	for (i = 0; i < data.items.length; i++) {
		row = document.createElement('tr');

		el = document.createElement('td');
		el.innerHTML = data.items[i].name;
		row.appendChild(el);

		el = document.createElement('td');
		el.innerHTML = data.items[i].description;
		row.appendChild(el);

		el = document.createElement('td');
		el.innerHTML = data.items[i].html_url;
		row.appendChild(el);		

		tbody.appendChild(row);
	}
};

var getFrameworks = function() {
	// example chaining params
	GHApi.open('/search/repositories')
		.query('javascript framework')
		.addParam('order', 'desc')
		.addFilter('language', 'javascript')
		.addFilter('created', '>'+todayStr)
		.setCallback(function(meta, data) {

			buildTable('frameworks_table', data);

			getOpinionators();
		})
		.submit();
};

var getOpinionators = function() {
	// example passing params in submit
	GHApi.submit('/search/repositories', {
		q: 'opinionated',
		order: 'desc'
	},{
		created: '>'+todayStr,
	}, function(meta, data) {
		buildTable('opinionated_table', data);
	});
};

getFrameworks();
</script>

<style>
.results-table th, 
.results-table td {
	font-family: "Roboto", "Helvetica" sans-serif;
}
.results-table th {
	padding: 5px 5px;
	font-size: 14px;
}
.results-table td {
	padding: 0px 5px;
	background: #EEE;
}
</style>