It seems like there's a new JS framework every day. Let's find out if that's actually the case.

I hacked up a quick [library](https://github.com/rhoffmann8/ghapi.js) to interact with the Github JSONP API and search for all JS repositories created on today's date. Let's see if it's true for today.

#### Today's results for "javascript framework":

<div id="frameworks_table">
  <span class="loading-text">Loading...</span>
</div>

#### Bonus: Today's results for "opinionated":

<div id="opinionated_table">
  <span class="loading-text">Loading...</span>
</div>

<script src="https://cdn.rawgit.com/rhoffmann8/ghapi.js/master/ghapi.js"></script>
<script type="text/javascript">
var today = new Date(),
    todayStr = today.toISOString().substr(0,10);

var buildTable = function(tableId, data) {
  var $thead = $('<thead />')
    .append($('<tr />')
      .append($('<th />')
        .text('Name')
      ).append($('<th />')
        .text('Description  ')
      ).append($('<th />')
        .text('URL')
      )

    )
  var $tbody = $('<tbody />');

  if (!data.items.length) {
    $tbody.html('No results (yet)!');
  }

  data.items.forEach(function(item) {
    $tbody.append(
      $('<tr />')
        .append($('<td />')
          .text(item.name)
        ).append($('<td />')
          .text(item.description)
        ).append($('<td />')
          .text(item.html_url)
        )
    );
  });

  $("#"+tableId).html(
    $('<table />', {
      'class': 'results-table'
    })
    .append($thead)
    .append($tbody)
  );
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

//ghapi script seems to load async despite not having async attr
var checkApi = function() {
  if (typeof GHApi != 'undefined') {
    clearInterval(interval);
    getFrameworks();
  }
};
var interval = setInterval(checkApi, 1000)
</script>

<style>
.results-table {
  font-size: 0.9em;
  word-break: break-word;
  border-spacing: 0;
  border-collapse: collapse;
}
.results-table th, 
.results-table td {
  min-width: 75px;
  padding: 5px;
}
.results-table tbody tr:nth-child(2n) {
  background: #FFF;
}
.results-table td {
  border: 1px solid #B9B9B9;
}
</style>