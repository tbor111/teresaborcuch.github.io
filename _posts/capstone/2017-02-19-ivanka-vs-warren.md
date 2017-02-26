---
layout: post
title: Ivanka Trump vs Elizabeth Warren in the News
category: capstone
tags: [Highcharts, sentiment analysis, NLP, graphs]
---

# Plotting with Highcharts, Part 2

The intention behind the SentimentPlotter class was to create visualizations that could help me understand which news articles were responsible for fluctuations in sentiment on a particular topic. I also thought it might be interesting to compare sentiment on a topic between different sources and take a look at the titles of articles with the highest and lowest scores.

When I was looking at the most commonly mentioned people for each news source, I noticed that Ivanka Trump was the number four mention for Fox News, but ranked well below 50 in both *The Times* and *The Washington Post*. Correcting for relative numbers of articles from each source, that's **12** times as many mentions as *The Times* and **9** times as many as *The Post*. In addition, the average sentiment score for articles mentioning Ivanka were 4 times and twice as high for *The Times* and *The Post*, respectively. This called for an investigation.

This plot shows the sentiment scores for all Fox News articles mentioning the term "Ivanka" published between February 6th and February 18th.

<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/highcharts-more.js"></script>
<script src="https://code.highcharts.com/modules/exporting.js"></script>

<div id="ivanka-fox-sent-container" style="height: 400px; margin: auto; min-width: 400px; max-width: 600px"></div>

<script>
Highcharts.chart('ivanka-fox-sent-container', {
  chart: {
    zoomType: 'x'
  },
  title: {
    text: 'Sentiment Score of Fox Articles Containing "Ivanka"'
  },
  xAxis: [{
    type: 'datetime',
    allowDecimals: false,
    title: {
      text: 'date',
      scalable: false
    }
  }],
  yAxis: {
    labels: {
      format: '{value}'
    },
    title: {
      text: 'Sentiment Scores'
    }
  },

  tooltip: {
    shared: false
  },
  series: [{"data": [[1486339200000, 0.012893597017457003], [1486512000000, 0.0085798846609315294], [1486598400000, 0.015615816349790106], [1486684800000, -0.0044300847782622662], [1486771200000, 0.004827019520220804], [1486857600000, 0.014874702333123888], [1486944000000, 0.0019028678979650592], [1487116800000, 0.00040940148478960797], [1487203200000, 0.0075649819524816822], [1487376000000, 0.009115185632635284], [1487462400000, 0.015845936775213093]], "type": "spline", "name": "Mean Score"},
  {"whiskerLength": 0, "name": "Range", "color": "#FF0000", "data": [[1486339200000, 0.012893597017457003, 0.012893597017457003], [1486512000000, -0.0054573760883033255, 0.01598094835094451], [1486598400000, -0.005328642326226954, 0.031396791339973154], [1486684800000, -0.01865927755869861, 0.013729353708328201], [1486771200000, -0.008867256863921261, 0.023691408224302952], [1486857600000, 0.011551055796055797, 0.01861143341289374], [1486944000000, -0.028873109319340035, 0.024545518397108116], [1487116800000, -0.007898392331262702, 0.008717195300841918], [1487203200000, 0.00521702822404353, 0.009912935680919834], [1487376000000, -0.006810582439776544, 0.01925656918346977], [1487462400000, 0.015845936775213093, 0.015845936775213093]], "stemWidth": 3, "type": "errorbar"},
  {"name": "Macy\'s facing pressure to drop Ivanka Trump line ", "color": "#FF0000", "data": [[1486339200000, 0.012893597017457003]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Report: T.J. Maxx Tells Employees to Discard Ivanka Merchandise Signs ", "color": "#FF0000", "data": [[1486512000000, -0.0054573760883033255]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "\'Judicial Tyranny Has Run Amok\': Malkin Says Court\'s Ruling Is a \'Wake-Up Call\' ", "color": "#FF0000", "data": [[1486598400000, -0.005328642326226954]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Krauthammer on Flynn-Russia Report: \'There Is No Crime\' ", "color": "#FF0000", "data": [[1486684800000, -0.01865927755869861]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Report: Russia Considering Sending Snowden to US to \'Curry Favor\' with Trump ", "color": "#FF0000", "data": [[1486771200000, -0.008867256863921261]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "O\'Reilly on \'Watters\' World\': The Far Left \'Just Hates\' President Trump ", "color": "#FF0000", "data": [[1486857600000, 0.011551055796055797]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Franken Claims Some GOP Senators Are Worried About Trump\'s Mental State ", "color": "#FF0000", "data": [[1486944000000, -0.028873109319340035]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Whoopi Goldberg defends Tiffany Trump ", "color": "#FF0000", "data": [[1487116800000, -0.007898392331262702]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Tiffany Trump said she would \'love to\' sit with Whoopi Goldberg at fashion show ", "color": "#FF0000", "data": [[1487203200000, 0.00521702822404353]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Women Boycott Nordstrom After Retailer\'s Decision to Drop Ivanka\'s Line ", "color": "#FF0000", "data": [[1487376000000, -0.006810582439776544]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "\'Golden Girls\'-Themed Cafe Opens in New York City ", "color": "#FF0000", "data": [[1487462400000, 0.015845936775213093]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false},
  {"name": "Macy\'s facing pressure to drop Ivanka Trump line ", "color": "#FF0000", "data": [[1486339200000, 0.012893597017457003]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Donald Trump says Ivanka treated \'unfairly\' by Nordstrom amid more retailers dropping her merchandise ", "color": "#FF0000", "data": [[1486512000000, 0.01598094835094451]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "BREAKING: Appeals Court Upholds Suspension of Trump\'s Immigration Order ", "color": "#FF0000", "data": [[1486598400000, 0.031396791339973154]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "\'We\'ll Win That Battle\': Trump Could File New Travel Executive Order on Monday ", "color": "#FF0000", "data": [[1486684800000, 0.013729353708328201]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Author\'s Book Analyzes Why 69 Percent of Divorces Are Initiated by Women ", "color": "#FF0000", "data": [[1486771200000, 0.023691408224302952]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Conway: Melania Trump Has Been a \'Great Friend and Great Role Model\' ", "color": "#FF0000", "data": [[1486857600000, 0.01861143341289374]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "This Single Mom Dressed as a Man for Son\'s Dads-Only Event at His School ", "color": "#FF0000", "data": [[1486944000000, 0.024545518397108116]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "TX Woman on 8-Year Sentence for Illegal Voting: They Made an Example of Me ", "color": "#FF0000", "data": [[1487116800000, 0.008717195300841918]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Tiffany Trump Accepts Whoopi\'s Invitation to Sit Together at NY Fashion Week ", "color": "#FF0000", "data": [[1487203200000, 0.009912935680919834]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "These Dads Dancing Ballet with Their Daughters Will Steal Your Heart ", "color": "#FF0000", "data": [[1487376000000, 0.01925656918346977]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "\'Golden Girls\'-Themed Cafe Opens in New York City ", "color": "#FF0000", "data": [[1487462400000, 0.015845936775213093]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}

  ]
});
</script>

And for comparison, here is the graph of the NYT articles mentioning "Ivanka":

<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/highcharts-more.js"></script>
<script src="https://code.highcharts.com/modules/exporting.js"></script>

<div id="nyt-ivanka-sent-container" style="height: 400px; margin: auto; min-width: 400px; max-width: 600px"></div>

<script>
Highcharts.chart('nyt-ivanka-sent-container', {
  chart: {
    zoomType: 'x'
  },
  title: {
    text: 'Sentiment Score of NYT Articles Containing "Ivanka"'
  },
  xAxis: [{
    type: 'datetime',
    allowDecimals: false,
    title: {
      text: 'date',
      scalable: false
    }
  }],
  yAxis: {
    labels: {
      format: '{value}'
    },
    title: {
      text: 'Sentiment Scores'
    }
  },

  tooltip: {
    shared: false
  },
  series: [{"data": [[1486425600000, 0.016025179097353594], [1486512000000, 0.0021263937593822176], [1486598400000, 0.0063346125657030459], [1486684800000, -0.0049397663205696394], [1486771200000, 0.0040972363826734118], [1486857600000, -0.0016320863034611616], [1486944000000, -0.0070087836013007828], [1487030400000, 0.0067621499063050712], [1487116800000, -0.013022376163081243], [1487203200000, 0.0046427424273604231], [1487289600000, -0.0075793031671242874], [1487376000000, 0.0065314191600546207], [1487462400000, 0.027127567865876424]], "type": "spline", "name": "Mean Score"},
  {"whiskerLength": 0, "name": "Range", "color": "#FF0000", "data": [[1486425600000, 0.009764556906164668, 0.02228580128854252], [1486512000000, -0.005573071136106141, 0.005493427715215642], [1486598400000, -0.004691742514640724, 0.016201041009099595], [1486684800000, -0.023066126521418748, 0.012770579958953518], [1486771200000, 0.004097236382673412, 0.004097236382673412], [1486857600000, -0.0049361189327681585, 0.004139537133231448], [1486944000000, -0.007008783601300783, -0.007008783601300783], [1487030400000, 0.003371136212365749, 0.010340864258895601], [1487116800000, -0.013022376163081243, -0.013022376163081243], [1487203200000, 0.004642742427360423, 0.004642742427360423], [1487289600000, -0.007579303167124287, -0.007579303167124287], [1487376000000, 0.004865355342616356, 0.008197482977492886], [1487462400000, 0.027127567865876424, 0.027127567865876424]], "stemWidth": 3, "type": "errorbar"},
  {"name": "A Conservative Climate Solution: Republican Group Calls for Carbon Tax", "color": "#FF0000", "data": [[1486425600000, 0.009764556906164668]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "In Istanbul, Surprise That Trump Towers Complex Is Linked to Trump", "color": "#FF0000", "data": [[1486512000000, -0.005573071136106141]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump, the Courts and Nordstrom", "color": "#FF0000", "data": [[1486598400000, -0.004691742514640724]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Am I Imagining This?", "color": "#FF0000", "data": [[1486684800000, -0.023066126521418748]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trumps Economic Cabinet Is Mostly Bare. This Man Fills the Void.", "color": "#FF0000", "data": [[1486771200000, 0.004097236382673412]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump Sons Forge Ahead Without Father, Expanding and Navigating Conflicts", "color": "#FF0000", "data": [[1486857600000, -0.0049361189327681585]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Power of Disruption", "color": "#FF0000", "data": [[1486944000000, -0.007008783601300783]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Struggle Inside The Wall Street Journal", "color": "#FF0000", "data": [[1487030400000, 0.003371136212365749]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Hate Group Numbers in U.S. Rose for 2nd Year in a Row, Report Says", "color": "#FF0000", "data": [[1487116800000, -0.013022376163081243]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Full Transcript and Video: Trump News Conference", "color": "#FF0000", "data": [[1487203200000, 0.004642742427360423]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "A Jewish Reporter Got to Ask Trump a Question. It Didnt Go Well.", "color": "#FF0000", "data": [[1487289600000, -0.007579303167124287]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "In Praise of Hypocrisy", "color": "#FF0000", "data": [[1487376000000, 0.004865355342616356]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Might Ivanka Trump Speak up if Her Father Guts the Arts?", "color": "#FF0000", "data": [[1487462400000, 0.027127567865876424]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false},
  {"name": "Melania Trump Inc. Imperiled", "color": "#FF0000", "data": [[1486425600000, 0.02228580128854252]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Ivanka Trump Reported to Have Stepped Down as Murdoch Trustee", "color": "#FF0000", "data": [[1486512000000, 0.005493427715215642]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump Tells Xi Jinping U.S. Will Honor One China Policy", "color": "#FF0000", "data": [[1486598400000, 0.016201041009099595]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "When the Fire Comes", "color": "#FF0000", "data": [[1486684800000, 0.012770579958953518]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trumps Economic Cabinet Is Mostly Bare. This Man Fills the Void.", "color": "#FF0000", "data": [[1486771200000, 0.004097236382673412]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "On S.N.L., Alec Baldwin and Melissa McCarthy Take On Trump and Sean Spicer", "color": "#FF0000", "data": [[1486857600000, 0.004139537133231448]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Power of Disruption", "color": "#FF0000", "data": [[1486944000000, -0.007008783601300783]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "No More Rules of the Game", "color": "#FF0000", "data": [[1487030400000, 0.010340864258895601]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Hate Group Numbers in U.S. Rose for 2nd Year in a Row, Report Says", "color": "#FF0000", "data": [[1487116800000, -0.013022376163081243]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Full Transcript and Video: Trump News Conference", "color": "#FF0000", "data": [[1487203200000, 0.004642742427360423]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "A Jewish Reporter Got to Ask Trump a Question. It Didnt Go Well.", "color": "#FF0000", "data": [[1487289600000, -0.007579303167124287]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trumps Dual Roles Collide With Openings in Dubai and Vancouver", "color": "#FF0000", "data": [[1487376000000, 0.008197482977492886]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Might Ivanka Trump Speak up if Her Father Guts the Arts?", "color": "#FF0000", "data": [[1487462400000, 0.027127567865876424]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}
  ]
});

</script>

One caveat about interpreting these graphs is that "Ivanka" literally has to appear only once in the entire article body for it to be included in the data. She may not be the subject of the article at all.

Now, let's look at Fox News articles mentioning "Warren." Although the majority of these articles are likely about Senator Elizabeth Warren, it's possible that millionaire Warren Buffett might also be included.

<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/highcharts-more.js"></script>
<script src="https://code.highcharts.com/modules/exporting.js"></script>

<div id="fox-warren-sent-container" style="height: 400px; margin: auto; min-width: 400px; max-width: 600px"></div>

<script>
Highcharts.chart('fox-warren-sent-container', {
  chart: {
    zoomType: 'x'
  },
  title: {
    text: 'Sentiment Score of Fox Articles Containing "Warren"'
  },
  xAxis: [{
    type: 'datetime',
    allowDecimals: false,
    title: {
      text: 'date',
      scalable: false
    }
  }],
  yAxis: {
    labels: {
      format: '{value}'
    },
    title: {
      text: 'Sentiment Scores'
    }
  },

  tooltip: {
    shared: false
  },
  series: [{"data": [[1486339200000, 0.015667484554174937], [1486425600000, -0.0093909834823826099], [1486512000000, -0.0040683085704669953], [1486598400000, -0.0079095518801175716], [1486857600000, 0.014874702333123888], [1486944000000, 0.0074587902701816807], [1487116800000, 0.011549685660681036], [1487376000000, 0.011188775392158479]], "type": "spline", "name": "Mean Score"},
  {"whiskerLength": 0, "name": "Range", "color": "#FF0000", "data": [[1486339200000, 0.015667484554174937, 0.015667484554174937], [1486425600000, -0.02526555496642395, 0.01567273151659359], [1486512000000, -0.03014353918824808, 0.017887966595637565], [1486598400000, -0.019255695508389143, 0.014587458366191996], [1486857600000, 0.011551055796055797, 0.01861143341289374], [1486944000000, -0.009061513136073791, 0.02390893822329171], [1487116800000, 0.011549685660681036, 0.011549685660681036], [1487376000000, 0.006882140234132234, 0.015495410550184725]], "stemWidth": 3, "type": "errorbar"},
  {"name": "Kennedy: \'Out of Touch\' Pelosi Just Trying to Hold on to Power ", "color": "#FF0000", "data": [[1486339200000, 0.015667484554174937]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Curt Schilling: Sen. Warren \'Represents Everything We Hate About Politics\' ", "color": "#FF0000", "data": [[1486425600000, -0.02526555496642395]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Hannity Blasts \'Fake Pocahontas\' Warren & \'Obstructionist Democrats\' ", "color": "#FF0000", "data": [[1486512000000, -0.03014353918824808]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Sean Hannity: Phony Native American Liz Warren leads Dems twisted anti-Trump charge ", "color": "#FF0000", "data": [[1486598400000, -0.019255695508389143]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "O\'Reilly on \'Watters\' World\': The Far Left \'Just Hates\' President Trump ", "color": "#FF0000", "data": [[1486857600000, 0.011551055796055797]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Here Are All the Times Sunday\'s Grammy Awards Got Political ", "color": "#FF0000", "data": [[1486944000000, -0.009061513136073791]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Relax, Trump is stone cold sane ", "color": "#FF0000", "data": [[1487116800000, 0.011549685660681036]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "I\'m a Democrat (and ex-CIA) but the spies plotting against Trump are out of control ", "color": "#FF0000", "data": [[1487376000000, 0.006882140234132234]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false},
  {"name": "Kennedy: \'Out of Touch\' Pelosi Just Trying to Hold on to Power ", "color": "#FF0000", "data": [[1486339200000, 0.015667484554174937]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Pence Casts Tie-Breaking Vote to Confirm Education Secy. Betsy DeVos ", "color": "#FF0000", "data": [[1486425600000, 0.01567273151659359]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Jessica Alba on the two biggest challenges she faces at home ", "color": "#FF0000", "data": [[1486512000000, 0.017887966595637565]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Kellyanne Conway on Ivanka Trump\'s Fashion Line: \'Go Buy It Today!\' ", "color": "#FF0000", "data": [[1486598400000, 0.014587458366191996]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Conway: Melania Trump Has Been a \'Great Friend and Great Role Model\' ", "color": "#FF0000", "data": [[1486857600000, 0.01861143341289374]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The dirty secret smart Democrats know (but won\'t admit) about Trump ", "color": "#FF0000", "data": [[1486944000000, 0.02390893822329171]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Relax, Trump is stone cold sane ", "color": "#FF0000", "data": [[1487116800000, 0.011549685660681036]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Politics seeped into NY Fashion Week ", "color": "#FF0000", "data": [[1487376000000, 0.015495410550184725]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}]
});

</script>

Here are the NYT articles mentioning "Warren." It's also important to note here that the sentiment of the article might not necessarily reflect the sentiment the author feels toward the search term. For instance, the most negative article is entitled "The Shameful Silencing of Elizabeth Warren in the Senate." Its sentiment score is -0.03, but I'd speculate that it was the silencing that earned the negative score, rather than Elizabeth Warren herself.

<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/highcharts-more.js"></script>
<script src="https://code.highcharts.com/modules/exporting.js"></script>

<div id="nyt-warren-sent-container" style="height: 400px; margin: auto; min-width: 400px; max-width: 600px"></div>

<script>
Highcharts.chart('nyt-warren-sent-container', {
  chart: {
    zoomType: 'x'
  },
  title: {
    text: 'Sentiment Score of NYT Articles Containing "Warren"'
  },
  xAxis: [{
    type: 'datetime',
    allowDecimals: false,
    title: {
      text: 'date',
      scalable: false
    }
  }],
  yAxis: {
    labels: {
      format: '{value}'
    },
    title: {
      text: 'Sentiment Scores'
    }
  },

  tooltip: {
    shared: false
  },
  series: [{"data": [[1486339200000, 0.030550193776498244], [1486425600000, 0.0059771854074820857], [1486512000000, -0.0013482515727537034], [1486598400000, 0.0084482055144543804], [1486684800000, 0.0056052336456628039], [1486771200000, 0.0058862322336146076], [1486857600000, 0.0077633195161757543], [1486944000000, 0.0041968928405249628], [1487030400000, 0.0033148693785134079]], "type": "spline", "name": "Mean Score"},
  {"whiskerLength": 0, "name": "Range", "color": "#FF0000", "data": [[1486339200000, 0.030550193776498244, 0.030550193776498244], [1486425600000, -0.014740629350118044, 0.025859594955697326], [1486512000000, -0.031268166085045235, 0.00986241334666112], [1486598400000, -0.003610253752973803, 0.01669348637739443], [1486684800000, -0.0015618664266787918, 0.017052253374662048], [1486771200000, 0.004097236382673412, 0.008432742545153723], [1486857600000, -0.017705964631922, 0.039381642976002385], [1486944000000, -0.007008783601300783, 0.022765495664105063], [1487030400000, 0.003314869378513408, 0.003314869378513408]], "stemWidth": 3, "type": "errorbar"},
  {"name": "Trump to Individual Investors: Caveat Emptor", "color": "#FF0000", "data": [[1486339200000, 0.030550193776498244]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Republican Senators Vote to Formally Silence Elizabeth Warren", "color": "#FF0000", "data": [[1486425600000, -0.014740629350118044]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Shameful Silencing of Elizabeth Warren in the Senate", "color": "#FF0000", "data": [[1486512000000, -0.031268166085045235]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Consumer Watchdog Faces Attack by House Republicans", "color": "#FF0000", "data": [[1486598400000, -0.003610253752973803]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "What Happened to Elizabeth Warren Has Roots in Racism", "color": "#FF0000", "data": [[1486684800000, -0.0015618664266787918]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trumps Economic Cabinet Is Mostly Bare. This Man Fills the Void.", "color": "#FF0000", "data": [[1486771200000, 0.004097236382673412]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Review: S.N.L. Targets Trump Again, With a Hint of Exhaustion", "color": "#FF0000", "data": [[1486857600000, -0.017705964631922]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Power of Disruption", "color": "#FF0000", "data": [[1486944000000, -0.007008783601300783]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Yankees Are Rebuilding Everywhere as Spring Training Starts", "color": "#FF0000", "data": [[1487030400000, 0.003314869378513408]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false},
  {"name": "Trump to Individual Investors: Caveat Emptor", "color": "#FF0000", "data": [[1486339200000, 0.030550193776498244]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Mike Pence, the Tiebreaker", "color": "#FF0000", "data": [[1486425600000, 0.025859594955697326]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Elizabeth Warren Was Told to Be Quiet. Women Can Relate.", "color": "#FF0000", "data": [[1486512000000, 0.00986241334666112]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Elizabeth Warren Persists", "color": "#FF0000", "data": [[1486598400000, 0.01669348637739443]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Ford to Invest $1 Billion in Artificial Intelligence Start-Up", "color": "#FF0000", "data": [[1486684800000, 0.017052253374662048]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Are Democrats Falling Into Trumps Trap?", "color": "#FF0000", "data": [[1486771200000, 0.008432742545153723]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Adele Dominates the Grammys; Beyonc Stops the Show", "color": "#FF0000", "data": [[1486857600000, 0.039381642976002385]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Best and Worst of the Grammys", "color": "#FF0000", "data": [[1486944000000, 0.022765495664105063]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Yankees Are Rebuilding Everywhere as Spring Training Starts", "color": "#FF0000", "data": [[1487030400000, 0.003314869378513408]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}
  ]
});

</script>

So overall, during this (somewhat limited) date range, Fox News reported more positively on Ivanka Trump than the NYT did, and more positively than it did on Elizabeth Warren. And conversely, as we might have expected, *The Times* reported more positively on Warren than it did on Ivanka.
