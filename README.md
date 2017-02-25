## Website Performance Optimization portfolio project

### Getting started

Read the following instructions to get the web portfolio project running on the browser.

1. Clone and download the project's repo:
~~~~
https://github.com/ikhancode/Udacity-FEWD-Nanodegree---Web-Optimization.git
~~~~
2. To inspect the site, you can run a local server

  ```bash
  $> cd /path/to/your-project-folder
  $> python3 -m http.server
  ```

3. Open a browser and visit localhost:8000
4. Now run ngrok on a separate terminal

  ``` bash
  $> cd /path/to/your-project-folder
  $> ./ngrok http 8000
  ```

5. Copy the public URL ngrok gives you on your browser and project's homepage should show up.

### Meeting specifications
####PageSpeed Score
#####Critical Rendering Path
`index.html` achieves a PageSpeed score of at least 90 for Mobile and Desktop.
######Optimization
Was able to get a PageSpeed Insights score of 95 and 94 mobile and desktop respectively by doing the following:
  * Compressed / optimized all images. Also resized pizzeria image and saved it separately with the original image. Original one is used in pizza.html and small (resized) one is used index.html.
  * Added a media query attribute to use print.css only when the screen is in print mode.
  * Removed online requesting of web fonts and instead added an async script at the bottom of the body which avoids render blocking
  * Removed the requesting of style.css file. Inlined it instead.
  * Added async attribute in for requesting analytics.js and perfmatters.js and moved their tags at the bottom of the body section

####Getting rid of jank
#####Frame rate
  Optimizations made to `views/js/main.js` make `views/pizza.html` render with a consistent frame-rate at 60fps when scrolling.
######Optimization
  * Took this DOM traversal outside the loop and assigned it to a variable where it is computed only once. After these optimizations, the scrolling was acquiring 60fps
~~~~
var randPiz = document.getElementById("randomPizzas");
~~~~
* Refactored this function. Comments in the code below.
~~~~
// Moves the sliding background pizzas based on scroll position
function updatePositions() {
  frame++;
  window.performance.mark("mark_start_frame");
  //replaced querySelectorAll method by much faster method, getElementsByClassName.
  var items = document.getElementsByClassName("mover");

  /*This lets the function look up length of items once instead of looking up in every iteration of the loop.
  The function would run tad efficiently this way.*/
  var itemsLength = items.length;

  /*To prevent the hassle of constantly declaring and assigning the variable phase in every iteration, I placed
  the phase variable outside the loop, assigning it an empty array used the loop for only pushing values in this variable.*/
  var phase = [];

  /*Put this property lookup in a variable outside the loop so that it is only computed once instead of looking up in
  every iteration of the loop. This makes the loop run faster.*/

  var topPos = document.body.scrollTop / 1250;

  //Since the phases repeat, only first 5 phases are needed to be calculated.
  for (var i = 0; i < 5; i++) {
    phase.push(Math.sin((topPos) + (i % 5)));
  }
  for (var i = 0; i < itemsLength; i++) {
    items[i].style.left = items[i].basicLeft + 100 * phase[i % 5] + 'px';
  }
  // User Timing API to the rescue again. Seriously, it's worth learning.
  // Super easy to create custom metrics.
  window.performance.mark("mark_end_frame");
  window.performance.measure("measure_frame_duration", "mark_start_frame", "mark_end_frame");
  if (frame % 10 === 0) {
    var timesToUpdatePosition = window.performance.getEntriesByName("measure_frame_duration");
    logAverageFrame(timesToUpdatePosition);
  }
}
  ~~~~
  * Assigned this DOM traversal in a variable outside the loop to avoid repeated traversing in the loop (thus making the loop run faster). Also replaced querySelector method with a much faster one, getElementByID
~~~~
var movPiz = document.getElementById("movingPizzas1");
~~~~

#####Computational Efficiency
Time to resize pizzas is less than 5 ms using the pizza size slider on the views/pizza.html page. Resize time is shown in the browser developer tools.
######Optimization
  * Refactored the janky, FSL loop, thus refactoring the whole function. Comments in the code below. After these optimisations, time to resize pizzas was way below 1ms!
  ~~~~
  // Iterates through pizza elements on the page and changes their widths
  // Refactoring perf of this function: Cameron's way
  function changePizzaSizes(size) {
    switch(size) {
      case "1":
        newWidth = 25;
        break;
      case "2":
        newWidth = 33.3;
        break;
      case "3":
        newWidth = 50;
        break;
      default:
        console.log("error in changePizzaSizes")
    }

    /*
    Previously, this DOM traversal was called and computed thrice in every iteration of the for loop.
    I assigned this traversal in a variable (randomPizzas) outside the loop so that it is only queried
    once (using a much faster method, getElementsByClassName) instead of querying in the loop thrice per iteration.
    */
    var randomPizzas = document.getElementsByClassName("randomPizzaContainer");
    /*
    I then removed all irrelevant calculations for pixel width and simplified the loop to be a one-liner assigning
    widths only as percentages. This fixed the problem of forced synch layout.
    */
    for (var i = 0; i < randomPizzas.length; i++) {
      randomPizzas[i].style.width = newWidth + "%";
    }
  }
  ~~~~
  *Main thing I changed here is to make the function dynamically set the amount of pizza's depending on the size of user's screen
  ~~~~
  // Generates the sliding pizzas when the page loads.
document.addEventListener('DOMContentLoaded', function() {
  var cols = 8;
  var s = 256;
  var elem = "";

  /*Assigned this DOM traversal in a variable outside the loop to avoid repeated traversing in the
  loop (thus making the loop run faster). Also replaced querySelector method with a much faster one, getElementByID.*/
  var movPiz = document.getElementById("movingPizzas1");

   // Dynamic way of setting number of pizzas needed to cover the user's screen
  var rows = window.screen.height / s;
  var perfNumPizzas = Math.ceil(rows * cols);

  for (var i = 0; i < perfNumPizzas; i++) {
    elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    movPiz.appendChild(elem);
  }
  updatePositions();
});
  ~~~~

  * Replaced querySelector method by much faster method, getElementByID in changeSliderLabel function
  ~~~~
  function changeSliderLabel(size) {
    switch(size) {
      case "1":
        document.getElementById("pizzaSize").innerHTML = "Small";
        return;
      case "2":
        document.getElementById("pizzaSize").innerHTML = "Medium";
        return;
      case "3":
        document.getElementById("pizzaSize").innerHTML = "Large";
        return;
      default:
        console.log("bug in changeSliderLabel");
    }
  }
  ~~~~
  and in determineDx function
  ~~~~
  var windowWidth = document.getElementById("randomPizzas").offsetWidth;
  ~~~~
