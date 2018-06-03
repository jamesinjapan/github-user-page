---
layout: post
title:  "Making a Javascript Calculator"
date: 2018-06-03 08:00:00 +09:00
date: 2018-06-03 08:00:00 +09:00
comments: true
category: [projects, javascript]
---
There is no better way to learn programming than to get stuck into some projects. So as a way to build my own portfolio and to improve my skills as a whole, I will write a few posts about the process of making small projects. Today I will make a basic javascript calculator on CodePen.

[Check it out before we begin!](https://codepen.io/jamesinjapan/full/dKGyWP/)

![My Javascript Calculator](/assets/javascript-calculator.jpg "Javascript Calculator")

Specification
-------------

What do we need from our calculator? Well, it should have a number pad to enter numbers with, a decimal point button, equals button, along with functions for addition, subtraction, multiplication and division.

Like a regular calculator we will want the user to be able to add to the product of a previous function, as well as keep a number in memory.

Additionally, it might be nice to have a backspace function for when you accidentally enter the wrong number, and maybe also the ability to make the current number positive or negative.

Setting up the frontend
-----------------------

So let's first make a basic interface.

Let's make a container div to house five row divs of five button divs, in the familiar calculator style. In each of the button divs we will place plain text which will act as the value of our button. We will also need a place to for the number display, so let's add another row to the top.

For identifying what function we want to use for our calculations, let's make every button that doesn't get printed to the display have an id that we can refer to later.

{% highlight html linenos %}
<div id="container">
  <div id="display" class="row">
    JAVASCRIPT CALCULATOR
  </div>
  <div class="row">
    <div id="backspace" class="button">►</div>
    <div id="recallFromMemory" class="button">MR</div>
    <div id="addToMemory" class="button">M+</div>
    <div id="changeSign" class="button">±</div>
  </div>
  <div class="row">
    <div class="button">7</div>
    <div class="button">8</div>
    <div class="button">9</div>
    <div id="division" class="button">÷</div>
  </div>
  <div class="row">
    <div class="button">4</div>
    <div class="button">5</div>
    <div class="button">6</div>
    <div id="multiplication" class="button">×</div>
  </div>
  <div class="row">
    <div class="button">1</div>
    <div class="button">2</div>
    <div class="button">3</div>
    <div id="subtraction" class="button">−</div>
  </div>
  <div class="row">
    <div class="button">0</div>
    <div class="button">.</div>
    <div id="calculate" class="button">=</div>
    <div id="addition" class="button">+</div>
  </div>
</div>
{% endhighlight %}

Next, let's add some CSS Flexbox to center all our text values in their buttons and to ensure we have a nice big interface to work with. 

{% highlight css linenos %}
#container {
  width: 300px;
  margin: auto;
}

.row {
  display: flex;
  justify-content: space-evenly;
  align-content: center;
  height: 75px;  
}

.button {
  display: flex;
  justify-content: center;
  flex-direction: column;
  text-align: center;
  width: 75px;
  height: 75px;
  border: 1px #333 solid;
  font-size: 2em;
}

#display {
  display: flex;
  justify-content: center;
  flex-direction: column;
  text-align: center;
  border: 1px #333 solid;
  font-size: 2em;
  height: 75px;
}
{% endhighlight %}

Now that we have a functional interface, let's start building up the backend.

Setting up the display
----------------------

Let's think about the basic operations that our code will need to conduct. The first thing we want is for the value of a pressed button to be displayed at the top of the calculator. A second button press should append the value to the display so we can chain numbers together.

To do that, we will need to have our code watching for any button presses, and that means we want to use [addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener). 

We have 20 buttons on our calculator, so we don't want to create an event listener for each button, so let's create a single event listener for the entire container div.

{% highlight js linenos %}
var calculatorContainer = document.getElementById('container');
var calculatorDisplay = document.getElementById('display');
calculatorContainer.addEventListener('click', processClick, false);
{% endhighlight %}

There are two variables we want to hold onto: one, the container of the calculator, and two, the display of the calculator.

Then we add an event listener that will wait for clicks inside the container and then run processClick() for each click.

{% highlight js linenos %}
function processClick(e) {
  if (e.target.className === 'button') {
    let clickedButtonValue = e.target.innerHTML;
    let currentDisplayValue = calculatorDisplay.innerHTML;
    calculatorDisplay.innerHTML = currentDisplayValue.concat(clickedButtonValue);
  }
}
{% endhighlight %}

processClick() will look at the element that has been clicked and determine if it has the class name "button". If it does, then it will take the text from the button div and then append the text of the display to match.

However, this means that when we first press our buttons we will end up with our display's opening text (JAVASCRIPT CALCULATOR) being appended with whatever numbers we enter, so let's add a function to clear the display and then include a line in processClick() to clear the display if the opening text is there.

First let's capture the display text on load. Note I am using trim to remove any whitespace from either side of the value.

{% highlight js linenos %}
var openingDisplayText = calculatorDisplay.innerHTML.trim();
{% endhighlight %}

Then we will edit processClick to use this variable to check if the currentDisplayValue (also now trimmed) matches the openingDisplayText, then call a function to clear the display (which returned a blank string so that the currentDisplayValue is also blanked off).

{% highlight js linenos %}
function processClick(e) {
  if (e.target.className === 'button') {
    let clickedButtonValue = e.target.innerHTML.trim();
    let currentDisplayValue = calculatorDisplay.innerHTML.trim();
    if (currentDisplayValue === openingDisplayText) {
      currentDisplayValue = clearDisplay();
    }
    calculatorDisplay.innerHTML = currentDisplayValue.concat(clickedButtonValue);
  }
}

function clearDisplay() {
  clearedValue = '';
  calculatorDisplay.innerHTML = clearedValue;
  return clearedValue;
}
{% endhighlight %}

By making clearDisplay a function, we can use it everytime we refresh the display after making a calculation or pressing an operator button. 

Finally, for the handling of the display, let's cap the amount of digits to 12, like most physical calculators do, by changing the last line of our the processClick function to:

{% highlight js linenos %}
if (currentDisplayValue.length < 12) {
  calculatorDisplay.innerHTML = currentDisplayValue.concat(clickedButtonValue); 
}
{% endhighlight %}

Adding operations
-----------------

A calculator isn't very useful if it doesn't let us perform operations on our numbers, so let's change that. First, make the simple functions that will act as our operations:

{% highlight js linenos %}
function addition(firstValue, secondValue) {
  return firstValue + secondValue;
}

function subtraction(firstValue, secondValue) {
  return firstValue - secondValue;
}

function multiplication(firstValue, secondValue) {
  return firstValue * secondValue;
}

function division(firstValue, secondValue) {
  return firstValue / secondValue;
}
{% endhighlight %}

These are pretty much what you would expect: take two values and return the result after calculating with an appropriate operator. 

Next we need to think about how we pass these values to the function. Performing a calculation with a calculator requires a minimum of four button presses: the first number, the operator, the second number, and equals. So we need to store at least the first number and operator until the user presses the equals button.

Let's create variables to handle this: 

{% highlight js linenos %}
var firstValue, operator, secondValue; 
{% endhighlight %}

All the buttons that shouldn't append to the display have ids with the name of the function we expect to use, so we can modify our processClick function so that clicks from divs with ids assign the display value to firstValue and operation to operator.

{% highlight js linenos %}
function processClick(e) {
  if (e.target.className === 'button' && e.target.id === '') {
    ...
  } else if (e.target.className === 'button') {
    prepareOperation(e.target.id);
  }
}

function prepareOperation(elementId) {
  operator = elementId;
  firstValue = getDisplayValue();
  clearDisplay();
}

function getDisplayValue() {
  let currentDisplayValue = calculatorDisplay.innerHTML.trim();
  return parseFloat(currentDisplayValue);
}
{% endhighlight %}

So, if the clicked element is a button but doesn't have an id, processClick will pass the element's id over to prepareOperation. For now, this new function just assings the operator and display value to our variableList object.

Next we need to consider what happens when we have our first value and operators queued up and we enter a second value and press the equals button. This is the most simple interaction a user will have with a calculator. Let's treat the equals operator (id: calculate) as a condition within prepareOperation.

{% highlight js linenos %}
function prepareOperation(elementId) {
  if (elementId == 'calculate') {
    secondValue = getDisplayValue();
    let function_name = window[operator];
    if (typeof function_name === 'function') {
      let result = function_name(firstValue, secondValue);
      calculatorDisplay.innerHTML = result;
      resetVariableList();
    }
  } else {
    operator = elementId;
    firstValue = getDisplayValue();
    clearDisplay();
  }  
}

function resetVariableList() {
  firstValue = undefined;
  secondValue = undefined;
  operator = undefined;
}
{% endhighlight %}

So when the elementId is "calculate", we grab the secondValue, and then we look for the operator in the current window which allows us to check if it is a valid function and execute it. Then when we are done we assign the result to the display and reset our variables.

However, now we have a number in the display and when we next press a number, it simply gets appended to the result rather than starting a new calculation. So let's revisit how we handle this and use a boolean flag to determine when to allow appends and when we should clear the screen.

{% highlight js linenos %}
var clearDisplayFlag = true;

function processClick(e) {
  if (e.target.className === 'button' && e.target.id === '') {
    let clickedButtonValue = e.target.innerHTML.trim();
    let currentDisplayValue = calculatorDisplay.innerHTML.trim();
    if (clearDisplayFlag) {
      currentDisplayValue = clearDisplay();
      clearDisplayFlag = false;
    }
    if (currentDisplayValue.length < 12) {
     calculatorDisplay.innerHTML = currentDisplayValue.concat(clickedButtonValue); 
    } 
  } else if (e.target.className === 'button') {
      prepareOperation(e.target.id);
  }
}

function clearDisplay() {
  clearedValue = '';
  calculatorDisplay.innerHTML = clearedValue;
  return clearedValue;
}

function prepareOperation(elementId) {
  if (elementId == 'calculate') {
    ...
  } else {
    operator = elementId;
    firstValue = getDisplayValue();
    clearDisplayFlag = true;
  }  
}

function resetVariableList() {
  firstValue = undefined;
  secondValue = undefined;
  operator = undefined;
  clearDisplayFlag = true;
}
{% endhighlight %}

So now everytime the page loads it starts with the clearDisplayFlag being true, solving the problem of appending to our title. Then we also toggle the flag to true when we reset our variables after pressing the equals button. Finally, after the processClick function calls clearDisplay, it toggles the flag to false to allow digits to append again.

Chaining operations
-------------------

When you use a calculator, you are not just limited to a single calculation, instead you can continue to operate on the results of previous calculations. For example, if I press 1 + 1 + 1 + 1 and then =, I should get 4. In our calculator right now, all we end up with are reassignments of the first value and operator until the final 1 is added as a second value, so we get 2. 

We should be calculating the result, sending it to the display and assigning it to the first value (along with the operator) so we can continue the chain. This means we need to rewrite our prepareOperation function.

{% highlight js linenos %}
function prepareOperation(elementId) {
  if (firstValue) {
    result = performCalculation(elementId); 
    if (result) {
      calculatorDisplay.innerHTML = result;
      if (elementId === 'calculate') {
        resetVariableList();
      } else {
        firstValue = result;
        operator = elementId;
        clearDisplayFlag = true;
      }
    }
  } else {
    ...
  }  
}

function performCalculation() {
  secondValue = getDisplayValue();
  let function_name = window[operator];
  if (typeof function_name === 'function') {
    return function_name(firstValue, secondValue);
  } else {
    resetVariableList();
    return false;
  }
}
{% endhighlight %}

Now prepareOperation checks if the firstValue is defined, and if it is then it runs the calculation (now in a separate performCalculation function), then it will refresh some or all of the variables depending on the last operator button pressed.

So now we can tally and chain our operations. Let's add some functions for our final buttons: memory recall, add to memory, sign toggling and backspace.

Changing the display value
--------------------------

Before we do anything else, let's tidy up processClick. This will be where we need our final four functions to be called from so they don't get processed as a calculation. Let's also extract some of the other condition results into their own functions and change processClick so that it uses a switch condition to make it more readable.

{% highlight js linenos %}
function processClick(e) {
  if (e.target.className === 'button') {
    switch (e.target.id) {
      case 'backspace':
        backspace();
        break;
      case 'changeSign':
        changeSign();
        break;  
      case '':
        appendValue(e.target.innerHTML.trim());
        break;
      default: 
        prepareOperation(e.target.id);
    }
  }
}

function appendValue(newValue) {
  let currentDisplayValue = calculatorDisplay.innerHTML.trim();
  if (clearDisplayFlag) {
    currentDisplayValue = clearDisplay();
    clearDisplayFlag = false;
  }
  if (currentDisplayValue.length < 12) {
    calculatorDisplay.innerHTML = currentDisplayValue.concat(newValue); 
  } 
}
{% endhighlight %}

First, backspace. We can use substring to remove the last character from the display. 

{% highlight js linenos %}
function backspace() {
  let currentDisplayValue = calculatorDisplay.innerHTML.trim();
  let backspacedValue = currentDisplayValue.length - 1;
  calculatorDisplay.innerHTML = currentDisplayValue.substring(0, backspacedValue); 
}
{% endhighlight %}

Next, change sign. This is probably the easiest. We will need to check if the first character in the display is a minus sign and if not, prepend one, or else remove it. 

{% highlight js linenos %}
function changeSign() {
  let currentDisplayValue = calculatorDisplay.innerHTML.trim();
  if (currentDisplayValue[0] == '-') {
    calculatorDisplay.innerHTML = currentDisplayValue.substring(1);
  } else {
    calculatorDisplay.innerHTML = '-'.concat(currentDisplayValue);
  }
}
{% endhighlight %}

Now, lets handle the memory-related buttons, as they add some extra complications to our user interface.

Memory operations
-----------------

Most calculators have a little M on the display to tell you that you have a value in memory. We currently don't have a place for that on our display, so let's add one. I found this one on [pxhere](https://pxhere.com/en/photo/13595).

![Background for my Javascript Calculator](https://i.imgur.com/uteTbQQ.jpg "Javascript Calculator Background")

{% highlight html linenos %}
<div id="container">
  <div id="displayContainer" class="row">
    <div id="memoryIcon">M</div>
    <div id="display" class="row">JAVASCRIPT CALCULATOR</div>
  </div>
  ...
</div>
{% endhighlight %}

So we've rejigged the display into a display_container, and display, and then added the memory_icon div. Let's put the "M" text in the div while we overlay the icon over the display with some CSS trickery, then we'll use "display: none;" to hide it away.

{% highlight css linenos %}
#container {
  position: relative;
  width: 400px;
  border: 1px #333 solid;
}

#displayContainer {
  border: 1px #333 solid;
}

#display {
  display: flex;
  justify-content: center;
  flex-direction: column;
  text-align: center;
  font-size: 2em;
  height: 75px;
}

#memoryIcon {
  position: absolute;
  left: 5px;
  font-size: 1em;
  font-weight: bold;
  display: none;
}
{% endhighlight %}

This floats the icon above the display and does not cause any offset in the display. Now we just need our Javascript to toggle the visibility of the icon and store the value in memory. We also want to ensure that it doesn't work until a number has been entered into the calculator: we don't want it to save "JAVASCRIPT CALCULATOR".

{% highlight js linenos %}
var firstValue, operator, secondValue, memoryValue;

function addToMemory() {
  if (calculatorDisplay.innerHTML.trim() != 'JAVASCRIPT CALCULATOR') {
    memoryValue = getDisplayValue();
    memoryIcon.style.display = 'block';
    clearDisplayFlag = true;
  }
}
{% endhighlight %}

Okay, now onto our final function, recallFromMemory. We want this to check whether memoryValue has a value assigned and then bring it onto the display. 

{% highlight js linenos %}
var firstValue, operator, secondValue, memoryValue;

function addToMemory() {
  if (calculatorDisplay.innerHTML.trim() != 'JAVASCRIPT CALCULATOR') {
    memoryValue = getDisplayValue();
    memoryIcon.style.display = 'block';
    clearDisplayFlag = true;
  }
}
{% endhighlight %}]

And those are all the functions we had planned. There are other things we could add: pi, exponentials, percentages, anything you might want, in fact. But let's stop there and just quickly style the calculator into something we can be happy with.

Finishing touches
-----------------

I won't go into too much detail about the styling of the final design. I wanted to have something that looked like a physical object on a desk, so first I added a desk image to the background.

{% highlight css linenos %}
html {
  background:  url('https://i.imgur.com/uteTbQQ.jpg') no-repeat center center fixed; 
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
}
{% endhighlight %}

To make the calculator fit its swanky background, it had to look a little less flat and matte. I used the [CSSMatic Gradient Generator](https://www.cssmatic.com/gradient-generator) with colors eyedropped from the MacBook on the left of the image, and then I tweaked a similar shadow effect from [Vanseo Design](https://vanseodesign.com/css/shadows/) until I got what I was looking for.

{% highlight css linenos %}
#container {
  position: relative;
  width: 400px;
  background: rgba(255,255,255,1);
  background: -moz-linear-gradient(-45deg, rgba(255,255,255,1) 0%, rgba(255,255,255,1) 47%, rgba(218,218,219,1) 100%);
  background: -webkit-gradient(left top, right bottom, color-stop(0%, rgba(255,255,255,1)), color-stop(47%, rgba(255,255,255,1)), color-stop(100%, rgba(218,218,219,1)));
  background: -webkit-linear-gradient(-45deg, rgba(255,255,255,1) 0%, rgba(255,255,255,1) 47%, rgba(218,218,219,1) 100%);
  background: -o-linear-gradient(-45deg, rgba(255,255,255,1) 0%, rgba(255,255,255,1) 47%, rgba(218,218,219,1) 100%);
  background: -ms-linear-gradient(-45deg, rgba(255,255,255,1) 0%, rgba(255,255,255,1) 47%, rgba(218,218,219,1) 100%);
  background: linear-gradient(135deg, rgba(255,255,255,1) 0%, rgba(255,255,255,1) 47%, rgba(218,218,219,1) 100%);
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#ffffff', endColorstr='#dadadb', GradientType=1 );
  margin: auto;
  padding: 20px 10px;
  font-family: Cabin, sans-serif;
  font-weight: bold;
  border: 2px solid black;
  box-shadow: 4px 10px 10px 1px black,
              inset 2px 2px 2px 1px #9ea499,
              inset -2px -2px 2px 1px #a3aaa0;
}
{% endhighlight %}

Next, I wanted my display to look like a LCD screen. Unfortunately, there aren't many good LCD webfonts around that don't require you to repackage them into your site, but I used Google's popular Roboto font. 

To give the display an inset look, I tweaked the numbers on [Julie Ann Horvath's The Perfect Inset Input CSS](https://gist.github.com/nrrrdcore/3309046).

{% highlight css linenos %}
#displayContainer {
  display: flex;
  justify-content: flex-end;
  height: 50px;
  margin: 10px 10px 20px 10px;
  padding: 10px;
  background: #7d8a74;
  border-radius: 3px;
  border: 1px solid transparent;
  border-top: none;
  border-bottom: 2px solid #DDD;
  box-shadow: inset 0 1px 2px rgba(0,0,0,.39), 0 -1px 1px #FFF, 0 1px 0 #FFF;
  font-family: Roboto, sans-serif;
  font-weight: bold;
}

#display {
  font-size: 25px;
  line-height: 50px;
}

#memoryIcon {
  position: absolute;
  left: 27px;
  top: 32px;
  font-size: 1em;
  font-weight: bold;
  display: none;
}
{% endhighlight %}

I wanted the digits to be big and bold, but the title ended up being too big to fit on a single line. To fix this, I made the font-size much smaller in the CSS file and adjusted the font size of the display in the clearDisplay function.

{% highlight js linenos %}
calculatorDisplay.style.fontSize = '48px';
{% endhighlight %}

The final thing I wanted to do was the give the buttons some padding and a slightly extruded look. Here I used the [CSS3 Button Generator at CSS Portal](http://www.cssportal.com/css3-button-generator/) to get the desired effect.

{% highlight css linenos %}
.button {
  display: flex;
  justify-content: center;
  flex-direction: column;
  width: 90px;
  height: 50px;
  font-size: 30px;
  text-align: center;
  line-height: 50px;
  text-shadow: 1px 1px 0px #EBDDE0;
  box-shadow: 1px 1px 1px #BEE2F9;
  -moz-border-radius: 4px;
  -webkit-border-radius: 4px;
  border-radius: 4px;
  border: 2px outset #E6E6E6;
  background: #63B8EE;
  background: linear-gradient(top,  #FFFFFF,  #DADADB);
  background: -ms-linear-gradient(top,  #FFFFFF,  #DADADB);
  background: -webkit-gradient(linear, left top, left bottom, from(#FFFFFF), to(#DADADB));
  background: -moz-linear-gradient(top,  #FFFFFF,  #DADADB);
}
{% endhighlight %}

Wrapping up
-----------

So there it is, a functioning calculator made from scratch in HTML, CSS and Javascript. I hope you like what you see and remember to keep working on small projects to build your experience.