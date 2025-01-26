---
title: 'How to Render jQuery Flot Charts through AngularJS'
tags: [javascript, AngularJS, jQuery]
categories: blog

comments: true
---

There are lots of javascript data visualization and charting libries out there, but only a couple bother to minimize the IE8 compatability headache. [Flot Charts](http://www.flotcharts.org) seems to work nicely for my purposes on this particular project. I need bar charts and donut charts, both of which Flot handles well.

Thing is, I really want to use [AngularJS](http://angularjs.org) for the project I'm working on right now. I don't know of any Angular-native charting libraries (though that would be a killer addition to [Angular UI](https://github.com/angular-ui)). Now I'm faced with the added layer of wrapping a jQuery plugin inside of Angular functions. What?

Luckily AngularJS can render Flot through a unique feature called a [directive](http://docs.angularjs.org/guide/directive). Stackoverflow user jm- [got me started](http://stackoverflow.com/questions/13103671/how-to-integrate-flot-with-angularjs) in the right direction on this. I've expanded his example in my own JSFiddle, which [renders a bar chart through AngularJS](http://jsfiddle.net/LC5JE/).

If you've worked with Flot before in regular ol' jQuery, then you're probably familiar with the [Flot API](https://github.com/flot/flot/blob/master/API.md). If you're new to Flot, then give the API a quick read. You can customize pretty much anything in your graph/chart with it.

Here's some example DOM markup:

```html
<div ng-app="App">
  <div ng-controller="barChartController">
    <chart ng-model="data"></chart>
  </div>
</div>
```

Then the AngularJS:

```javascript
var App = angular.module('App', []);

App.controller('barChartController', function ($scope) {
  var daftPoints = [[0, 4]],
    punkPoints = [[1, 20]];

  var data1 = [
    {
      data: daftPoints,
      color: '#00b9d7',
      bars: {
        show: true,
        barWidth: 1,
        fillColor: '#00b9d7',
        order: 1,
        align: 'center',
      },
    },
    {
      data: punkPoints,
      color: '#3a4452',
      bars: {
        show: true,
        barWidth: 1,
        fillColor: '#3a4452',
        order: 2,
        align: 'center',
      },
    },
  ];

  $scope.data = data1;
});

App.directive('chart', function () {
  return {
    restrict: 'E',
    link: function (scope, elem, attrs) {
      var chart = null,
        options = {
          xaxis: {
            ticks: [
              [0, 'Daft'],
              [1, 'Punk'],
            ],
          },
          grid: {
            labelMargin: 10,
            backgroundColor: '#e2e6e9',
            color: '#ffffff',
            borderColor: null,
          },
        };

      var data = scope[attrs.ngModel];

      // If the data changes somehow, update it in the chart
      scope.$watch('data', function (v) {
        if (!chart) {
          chart = $.plot(elem, v, options);
          elem.show();
        } else {
          chart.setData(v);
          chart.setupGrid();
          chart.draw();
        }
      });
    },
  };
});
```

### How to get this to work on IE8

1. Add <code>id="ng-app"</code> to the document's root element in conjunction with <code>ng-app</code> attribute

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:ng="http://angularjs.org" id="ng-app" ng-app="App">
     ...
   </html>
   ```

2. Load the [AngularUI IE shiv](https://github.com/angular-ui/ui-utils/tree/master/modules/ie-shiv), at the bottom of your javascript file list, in an IE8-targeted comment, and declare the <code>chart</code> directive. IE8 won't load that `<chart>` element without this.

   ```html
   <script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
   <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.0.7/angular.min.js"></script>
   <script src="js/application.js"></script>
   ...
   <!--[if lte IE 8]>
     <script>
       // Declare custom directive names and load shiv for AngularJS in here. Or else IE8 bummertown.
       window.myCustomTags = ['chart'];
     </script>
     <script src="js/ie-shiv.js"></script>
   <![endif]-->
   ...
   <body>
     ...
   </body>
   ```

```

```
