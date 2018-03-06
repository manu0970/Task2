# Task2
Create a Line chart add dots and interaction (whenever you click on the dots display information).

# Steps to reproduce the sample

## Create index.html
Index is responsible for joining all javascript and css files.
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
  </body>
  <link rel="stylesheet" href="./styles.css" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/4.5.0/d3.min.js" charset="utf-8"></script>  
  <script src="./main.js"></script>
</html>
```
## Create styles.css
```css
body { font: 12px Arial;}

path { 
    stroke: steelblue;
    stroke-width: 2;
    fill: none;
}

.axis path,
.axis line {
    fill: none;
    stroke: grey;
    stroke-width: 1;
    shape-rendering: crispEdges;
}

circle {
  fill: green;
  stroke: green;
  stroke-width: 2;
}

div.tooltip {	
    position: absolute;			
    text-align: center;			
    width: 120px;					
    height: 28px;					
    padding: 2px;				
    font: 12px sans-serif;		
    background: lightsteelblue;	
    border: 0px;		
    border-radius: 8px;			
    pointer-events: none;			
}
```
## Import data.csv
```csv
month,sales
10-Oct-16,6500
1-Nov-16,5400
1-Dec-16,3500
1-Jan-17,9000
1-Feb-17,8500
```
## Set up lite-server
First of all it is necessary to have installed nodejs.
https://nodejs.org/es/
From the project folder (where is index.html) run console, and type:
```cmd
npm init
```
Next step:
```cmd
npm install lite-server --save-dev
```
...and add a "script" entry within your project's package.json file:
```json
  "scripts": {    
    "dev": "lite-server"
  },
```
Finally, and whenever you want to visualize the web page type:
```cmd
npm start
```
For more information about "lite-server":

https://www.npmjs.com/package/lite-server

## Create main.js
The variables are declared. They allow creating a "svg" with certain "margins" and a scale determined by "x" and "y".
```javascript
let margin = null,
    width = null,
    height = null;

let svg = null;
let x, y = null; // scales
```
All functions are also initialized, which are explained below.
```javascript
setupCanvasSize();
appendSvg("body");
refreshChart();
```
The refreshChart () function changes the format of the data. Then call the drawChart () function that initializes all other functions.
```javascript
function refreshChart() {
  d3.csv("data.csv", function(error, data) {  
    if (error) throw error;

    // parse the date / time
    var parseTime = d3.timeParse("%d-%b-%y");    

    data.forEach(function(d) {
          d.month = parseTime(d.month);
          d.sales = +d.sales;
      });

    clearCanvas();
    drawChart(data);
  });
}
```
```javascript
function drawChart(totalSales) {
  setupXScale(totalSales);
  setupYScale(totalSales);
  appendXAxis(totalSales);
  appendYAxis(totalSales);
  appendLineCharts(totalSales);
  appendTooltipCharts(totalSales)
}
```
This function sets margins, width and height.
```javascript
function setupCanvasSize() {
  margin = {top: 20, left: 80, bottom: 20, right: 30};
  width = 960 - margin.left - margin.right;
  height = 520 - margin.top - margin.bottom;
}
```
A new element of type svg is appended to the html document. Its attributes such as width, height and position are configured.
```javascript
function appendSvg(domElement) {
  svg = d3.select(domElement).append("svg")
              .attr("width", width + margin.left + margin.right)
              .attr("height", height + margin.top + margin.bottom)
              .append("g")
              .attr("transform",`translate(${margin.left}, ${margin.top})`);

}
```
The data is scaled to the pixels on the screen.
- The X axis passes the date to pixels, since both are numbers is a linear scaling.
- The Y axis passes the number of sales to pixels, since both are numbers is a linear scaling.
```javascript
function setupXScale(totalSales)
{
  x = d3.scaleTime()
      .range([0, width])
      .domain(d3.extent(totalSales, function(d) { return d.month}));
}

function setupYScale(totalSales)
{
  var maxSales = d3.max(totalSales, function(d, i) {
    return d.sales;
  });

  y = d3.scaleLinear()
        .range([height, 0])
        .domain([0, maxSales]);
}
```
Next, the lines of the X and Y axes are added to svg. And move to their respective positions. Now it is possible to see the axes without data.
```javascript
function appendXAxis(totalSales) {
  svg.append("g")
    .attr("transform",`translate(0, ${height})`)
    .call(d3.axisBottom(x));
}

function appendYAxis(totalSales) {
  svg.append("g")
  .call(d3.axisLeft(y));
}
```
The next step draws the line of the graph by x and y.
```javascript
function appendLineCharts(totalSales)
{

  var valueline = d3.line()
                    .x(function(d) { return x(d.month); })
                    .y(function(d) { return y(d.sales); });
                    
  svg.append("path")
  .data([totalSales])
  .attr("class", "line")
  .attr("d", valueline);
}
```
A pop-up window appears each time the mouse is placed on the point.
```javascript
function appendTooltipCharts(totalSales)
{
  var div = d3.select("body").append("div")	
  .attr("class", "tooltip")				
  .style("opacity", 0);

    svg.selectAll("dot")	
    .data(totalSales)			
.enter().append("circle")								
    .attr("r", 5)		
    .attr("cx", function(d) { return x(d.month); })		 
    .attr("cy", function(d) { return y(d.sales); })		
    .on("mouseover", function(d) {		
        div.transition()		
            .duration(200)		
            .style("opacity", .9);		
        div.html(d.month.toDateString() + "<br/>"  + d.sales + ' sales')	
            .style("left", (d3.event.pageX) + "px")		
            .style("top", (d3.event.pageY - 28) + "px");	
        })					
    .on("mouseout", function(d) {		
        div.transition()		
            .duration(500)		
            .style("opacity", 0);	
    })
}
```
## Create the table

```diff
  var div = d3.select("body").append("div")	
  .attr("class", "tooltip")				
  .style("opacity", 0);

+  var table = d3.select("body").append("table")
+  .attr("width", width + margin.left + margin.right);				
+  table.html( '<tr>     <th>Date</th>      <th>Sales</th>     </tr>' +
+             '<tr>     <td>- (Click any point) </td>      <td>-</td>     </tr>');

    // Add the scatterplot
    svg.selectAll("dot")	
```
```diff
    .on("mouseout", function(d) {		
        div.transition()		
            .duration(500)		
            .style("opacity", 0);	
    })
+    .on("click", function(d) {				
+      table.html( '<tr>     <th>Date</th>      <th>Sales</th>     </tr>' +
+                  `<tr>     <td> ${d.month.toDateString()} </td>     <td> ${d.sales} </td>   </tr>`)	  
+      });
}
```
```css
table {
  /* border-collapse: collapse;
   */
  margin-left: 80px;
  margin-right: 30px;
}

th, td {
  text-align: left;
  padding: 8px;
}

tr:nth-child(even){background-color: #f2f2f2}

th {
  background-color: #4CAF50;
  color: white;
}
```
