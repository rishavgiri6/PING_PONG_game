# PING_PONG_game
Author-Rishav Giri
A new 2d version of the online ping-pong game using multithreading client-server architecture of Java coupled with TCP Socket Programming.

Also, the 2nd version is Flappy Pong:


#### The Game



This is the 2nd game we covered in our project This game is sort of a combination of Flappy Bird, Pong and Brick Breaker. 
Building Flappy Pong
=====

#### 

```processing
/********* VARIABLES *********/

// We control which screen is active by settings / updating
// gameScreen variable. We display the correct screen according
// to the value of this variable.
//
// 0: Initial Screen
// 1: Game Screen
// 2: Game-over Screen

int gameScreen = 0;

/********* SETUP BLOCK *********/

void setup() {
  size(500, 500);
}


/********* DRAW BLOCK *********/

void draw() {
  // Display the contents of the current screen
  if (gameScreen == 0) {
    initScreen();
  } else if (gameScreen == 1) {
    gameScreen();
  } else if (gameScreen == 2) {
    gameOverScreen();
  }
}


/********* SCREEN CONTENTS *********/

void initScreen() {
  // codes of initial screen
}
void gameScreen() {
  // codes of game screen
}
void gameOverScreen() {
  // codes for game over screen
}


/********* INPUTS *********/

public void mousePressed() {
  // if we are on the initial screen when clicked, start the game
  if (gameScreen==0) {
    startGame();
  }
}


/********* OTHER FUNCTIONS *********/

// This method sets the necessery variables to start the game  
void startGame() {
  gameScreen=1;
}
```
We built the basic structure and seperate different parts by the comment blocks.

As we see, we define different methods for display, which contains the contents the screen. In our draw block, we simply check the value of our `gameScreen` variable, and call the correct method accordingly.

In the `void mousePressed(){...}` part, we are listening to mouse clicks and if the active screen is 0, the initial screen, we call the `startGame()` method which starts the game as you'd expect. The first line of this method changes `gameScreen` variable to 1, the game screen.

The next step was to implement our initial screen. To do that, we will be editing the `initScreen()` method. Here it goes:

```processing
void initScreen() {
  background(0);
  textAlign(CENTER);
  text("Click to start", height/2, width/2);
}
```

Now our initial screen has a black background and a simple text, "Click to start", located in the middle and aligned to the center. But when we click, nothing happens. We haven't yet specified any content for our game screen. The method `gameScreen()` doesn't have anything in it, so we aren't covering the previous contents drawn from the last screen (the text) by having `background()` as the first line of draw. That's why the text is still there, eventhough the `text()` line is not being called anymore *(remember the moving ball which leaves a trace behind from the last part)*. The background is still black for the same reason. So let's go ahead and begin implementing the game screen.

```processing
void gameScreen() {
  background(255);
}
```



#### <u>Step 2 - Creating the ball & Gravity implementation</u>

Now,after we worked on the game screen. We firstly created our ball. We  defined variables for its coordinates, color and size because we might want to change those values later on. For instance, if we want to increate the size of the ball as the player scores higher so that the game will be harder. We will need to change its size, so it should be a variable. We define speed of the ball as well, after we implement the gravity.

First, we add:

```processing
...
int ballX, ballY;
int ballSize = 20;
int ballColor = color(0);
...
void setup() {
  ...
  ballX=width/4;
  ballY=height/5;
}
...
void gameScreen() {
  ...
  drawBall();
}
...
void drawBall() {
  fill(ballColor);
  ellipse(ballX, ballY, ballSize, ballSize);
}
```

We defined the coordinates as global variables, created a method that draw the ball called it in the game screen method. Only thing to pay attention here is, we **initiated** coordinates, but we defined them in `setup()`. The reason we did that is, we wanted the ball to start at one fourth from left and one fifth from top. There is no particular reason we want that, but that is a good point for the ball to start. So we needed to get the `width` and `height` of the sketch dynamically. The sketch size is defined in `setup()`, after the first line. `width` and `height` are not set before the `setup()` runs, that's why we couldn't achieve this if we defined the variables on top.

### Gravity

Now implementing gravity is easy actually, there are only a few tricks. Here is the implementation first:

```processing
...
float gravity = 1;
float ballSpeedVert = 0;
...
void gameScreen() {
  ...
  applyGravity();
  keepInScreen();
}
...
void applyGravity() {
  ballSpeedVert += gravity;
  ballY += ballSpeedVert;
}
void makeBounceBottom(float surface) {
  ballY = surface-(ballSize/2);
  ballSpeedVert*=-1;
}
void makeBounceTop(float surface) {
  ballY = surface+(ballSize/2);
  ballSpeedVert*=-1;
}
// keep ball in the screen
void keepInScreen() {
  // ball hits floor
  if (ballY+(ballSize/2) > height) { 
    makeBounceBottom(height);
  }
  // ball hits ceiling
  if (ballY-(ballSize/2) < 0) {
    makeBounceTop(0);
  }
}
```
And the is the result:

<img src="http://i.imgur.com/cE4SKC5.gif" style="box-shadow: 0 0 10px lightgray">

 The variable we defined as `gravity` is just a numeric value that we add to `ballSpeedVert` on every loop. And `ballSpeedVert` is the vertical speed of the ball, which is added to the Y coordinate of the ball (`ballY`) on each loop. We watch the coordinates of the ball and make sure it stays in the screen. If we didn't, ball would fall to infinity. For now, our ball only moves vertically. So we watch the floor and ceiling boundries of the screen. With `keepInScreen()` method, we check if `ballY` (**+** the radius) is less than `height`, and similarly `ballY` (**-** the radius) is more than `0`. If the conditions don't meet, we make the ball bounce (from the bottom or the top) with `makeBounceBottom()` and `makeBounceTop()` methods. To make the ball bounce, we simply move the ball to exact location where it had to bounce and multiply the vertical speed (`ballSpeedVert`) with `-1` (multiplying with -1 changes the sign). When the speed value has a minus sign, adding Y coordinate the speed becomes `ballY + (-ballSpeedVert)`, which is `ballY - ballSpeedVert`. So the ball immediately changes its direction with the same speed. Then, as we add `gravity` to `ballSpeedVert` and `ballSpeedVert` has a negative value, it starts to get close to `0`, eventually becomes `0`, and starts increasing again. That makes the ball rise, rise slower, stop and start falling.

There is a problem though. The ball keeps bouncing. If this was a real world scenario, the ball would have faced a) air friction, b) friction everytime it touches a surface. That's the behaviour we want for our game. So implementing this is easy. We add the following:

```processing
...
float airfriction = 0.0001;
float friction = 0.1;
...
void applyGravity() {
  ...
  ballSpeedVert -= (ballSpeedVert * airfriction);
}
void makeBounceBottom(int surface) {
  ...
  ballSpeedVert -= (ballSpeedVert * friction);
}
void makeBounceTop(int surface) {
  ...
  ballSpeedVert -= (ballSpeedVert * friction);
}
```
And what we get is:

<img src="http://i.imgur.com/w8P97ii.gif" style="box-shadow: 0 0 10px lightgray">

As the name suggests, `friction` is the surface friction and `airfriction` is the friction of air. So obviously, `friction` has to apply each time the ball touches any surface. `aurfriction` however has to apply constantly. So that's what we did. `applyGravity()` method runs on each loop, so we take away `0.0001` percent of its current value from `ballSpeedVert` on every loop. `makeBounceBottom()` and `makeBounceTop()` methods runs when ball touches any surface. So in those methods, we did the same thing, only this time with `friction `.

#### <u>Step 3 - Creating the racket</u>

Now we needed a racket make the ball bounce on. We should be controlling the racket. Let's make it controllable with the mouse. Here are the codes first:

```processing
...
color racketColor = color(0);
float racketWidth = 100;
float racketHeight = 10;
...
void gameScreen() {
  ...
  drawRacket();
  ...
}
...
void drawRacket(){
  fill(racketColor);
  rectMode(CENTER);
  rect(mouseX, mouseY, racketWidth, racketHeight);
}
```
We defined the color, width and height of the racket as a global variable, we might want them to change while the gameplay. We implemented a method `drawRacket()` which does what its name suggests. We set the `rectMode` to center, so our racket is aligned to the center of our cursor.

Now that we created the racket, we have to make the ball bounce on it.

```processing
...
int racketBounceRate = 20;
...
void gameScreen() {
  ...
  watchRacketBounce();
  ...
}
...
void watchRacketBounce() {
  float overhead = mouseY - pmouseY;
  if ((ballX+(ballSize/2) > mouseX-(racketWidth/2)) && (ballX-(ballSize/2) < mouseX+(racketWidth/2))) {
    if (dist(ballX, ballY, ballX, mouseY)<=(ballSize/2)+abs(overhead)) {
      makeBounceBottom(mouseY);
      // racket moving up
      if (overhead<0) {
        ballY+=overhead;
        ballSpeedVert+=overhead;
      }
    }
  }
}
```
And here is the result:

<img src="http://i.imgur.com/Pzollp3.gif" style="box-shadow: 0 0 10px lightgray">

So what `watchRacketBounce()` does is, it makes sure the racket and the ball collides. There are two things to check here, is the ball and the racket lined up a) **horizontally** and b) **vertically**. The first if statement checks if the X coordinate of the right side of the ball is greater than the X coordinate of the left side of the racket (and the other way around). If it is, second statement checks if the distance between the ball and the racket is smaller than or equal to the radius of the ball *(which means they are colliding)*. So if these conditions meet, `makeBounceBottom()` method gets called and the ball bounces on our racket (at `mouseY`, where the racket is).

We notice the variable `overhead` which is calculated by `mouseY - pmouseY` ? `pmouseX` and `pmouseY` variables store the coordinates of the mouse at the previous frame. As the mouse can move very fast, there is a good chance we might miss to detect the distance between the ball and the racket correctly in between frames if the mouse is moving towards the ball fast enough. So, we take the difference of the mouse coordinates in between frames and take that into account while detecting distance. The faster mouse is moving, the grater distance is acceptable. 

We also use `overhead` for another reason. We detect which way the mouse is moving by checking the sign of `overhead`. If overhead is negative, mouse was somewhere below in the previous frame so our mouse (racket) is moving up. In that case, we want to add an extra speed to the ball and move it a little further than regular bounce to simulate the effect of hitting the ball with the racket. If `overhead` is less than `0`, we add it to `ballY` and `ballSpeedVert` to make the ball go higher and faster. So the faster racket hits the ball, the higher and faster it will move up.


#### <u>Step 4 - Horizontal movement & Controlling the ball</u>

This section, we will add a horizontal movement to the ball. Then, we will make it possible to control the ball horizontally with our racket. Here we go:

```processing
...
// we will start with 0, but for we give 10 just for testing
float ballSpeedHorizon = 10;
...
void gameScreen() {
  ...
  applyHorizontalSpeed();
  ...
}
...
void applyHorizontalSpeed(){
  ballX += ballSpeedHorizon;
  ballSpeedHorizon -= (ballSpeedHorizon * airfriction);
}
void makeBounceLeft(float surface){
  ballX = surface+(ballSize/2);
  ballSpeedHorizon*=-1;
  ballSpeedHorizon -= (ballSpeedHorizon * friction);
}
void makeBounceRight(float surface){
  ballX = surface-(ballSize/2);
  ballSpeedHorizon*=-1;
  ballSpeedHorizon -= (ballSpeedHorizon * friction);
}
...
void keepInScreen() {
  ...
  if (ballX-(ballSize/2) < 0){
    makeBounceLeft(0);
  }
  if (ballX+(ballSize/2) > width){
    makeBounceRight(width);
  }
}
```
And the result is:

<img src="http://i.imgur.com/AYMTWci.gif" style="box-shadow: 0 0 10px lightgray">

The idea here is the same as the way we did for vertical movement. We created a horizontal speed variable, `ballSpeedHorizon`. We created a method to apply the horizontal speed to the `ballX` and take away the air friction. We added two more if statements to the `keepInScreen()` method which will watch the ball for hitting the left and right sides of the screen. Finally we created `makeBounceLeft()` and `makeBounceRight()` methods to handle the bounces from left and right.

Now that we added a horizontal speed to the game, we want to control the ball with the racket. As in the famous Atari game <a href="https://www.youtube.com/watch?v=MTnhlAlzb6Y">Breakout</a> and in all other brick breaking games, the ball should go left or right according to the point it hit the racket. The edges of the racket should give the ball more horizontal speed whereas the middle shouldn't have an effect. Codes first, lets go:

```processing
void watchRacketBounce() {
  ...
  if ((ballX+(ballSize/2) > mouseX-(racketWidth/2)) && (ballX-(ballSize/2) < mouseX+(racketWidth/2))) {
    if (dist(ballX, ballY, ballX, mouseY)<=(ballSize/2)+abs(overhead)) {
      ...
      ballSpeedHorizon = (ballX - mouseX)/5;
      ...
    }
  }
}
```

Results as:

<img src="http://i.imgur.com/ST9H2MZ.gif" style="box-shadow: 0 0 10px lightgray">

Adding that simple line to the `watchRacketBounce()` did the job. What we did is, we found the distance of the point that ball hits the racket from the center of the racket with `ballX - mouseX`. Then, we make it the horizontal speed. The actual difference is too much, so I gave a few shots and figured that one tenth of the value feels the most neatural.


#### <u>Step 5 - Creating the walls</u>

Our sketch looks more like a game at each step. In this step, we  add walls coming towards us, just like in Flappy Bird. 

```processing
...
int wallSpeed = 5;
int wallInterval = 1000;
float lastAddTime = 0;
int minGapHeight = 200;
int maxGapHeight = 300;
int wallWidth = 80;
color wallColors = color(0);
// This arraylist stores data of the gaps between the walls. Actuals walls are drawn accordingly.
// [gapWallX, gapWallY, gapWallWidth, gapWallHeight]
ArrayList<int[]> walls = new ArrayList<int[]>();
...
void gameScreen() {
  ...
  wallAdder();
  wallHandler();
}
...
void wallAdder() {
  if (millis()-lastAddTime > wallInterval) {
    int randHeight = round(random(minGapHeight, maxGapHeight));
    int randY = round(random(0, height-randHeight));
    // {gapWallX, gapWallY, gapWallWidth, gapWallHeight}
    int[] randWall = {width, randY, wallWidth, randHeight}; 
    walls.add(randWall);
    lastAddTime = millis();
  }
}
void wallHandler() {
  for (int i = 0; i < walls.size(); i++) {
    wallRemover(i);
    wallMover(i);
    wallDrawer(i);
  }
}
void wallDrawer(int index) {
  int[] wall = walls.get(index);
  // get gap wall settings 
  int gapWallX = wall[0];
  int gapWallY = wall[1];
  int gapWallWidth = wall[2];
  int gapWallHeight = wall[3];
  // draw actual walls
  rectMode(CORNER);
  fill(wallColors);
  rect(gapWallX, 0, gapWallWidth, gapWallY);
  rect(gapWallX, gapWallY+gapWallHeight, gapWallWidth, height-(gapWallY+gapWallHeight));
}
void wallMover(int index) {
  int[] wall = walls.get(index);
  wall[0] -= wallSpeed;
}
void wallRemover(int index) {
  int[] wall = walls.get(index);
  if (wall[0]+wall[2] <= 0) {
    walls.remove(index);
  }
}
```

And that resulted as:

<img src="http://i.imgur.com/6nB23Hp.gif" style="box-shadow: 0 0 10px lightgray">

he data we keep in the arrays are for the gap wall (ie. the gap between two walls). The arrays contain the values:

```processing
[gap wall X, gap wall Y, gap wall width, gap wall height]
```

The actual walls are drawn based on the gap wall values.  We have two base methods to manage the walls, `wallAdder()` and `wallHandler`.

`wallAdder()` method simply adds new walls in every `wallInterval` millisecond to the arraylist. We have a global variable `lastAddTime` which stores the time when the last wall was added *(in milliseconds)*. If the current millisecond `millis()` minus the last added millisecond `lastAddTime` is larger than the our interval value `wallInterval`, that means it is time to add a new wall. Random gap variables are than generated based on the global variables defined at the very top. Then a new wall (integer array that stores the gap wall data) is added into the arraylist and the `lastAddTime` is set to the current millisecond `millis()`.

`wallHandler()` loops through the current walls that is in the arraylist. And for each item at each loop, it calls `wallRemover(i)`, `wallMover(i)` and `wallDrawer(i)` by the index value of the arraylist. These methods do what their name suggests. `wallDrawer()` draws the actual walls based on the gap wall data. It grabs the wall data array from the arraylist, and calls `rect()` method to draw the walls to where they should actually be. `wallMover()` method grabs the element from the arraylist, changes its X location based on the `wallSpeed` global variable. Finally, `wallRemover()` removes the walls from the arraylist which are out of the screen. If we didn't do that, Processing would have treated them as they are still in the screen. And that would have been a huge loss in performance. So when a wall is removed from the arraylist, it doesn't get drawn on the next loops.

The final challenging thing left to do is to detect the collusions of the ball and the walls.

```processing
void wallHandler() {
  for (int i = 0; i < walls.size(); i++) {
    ...
    watchWallCollision(i);
  }
}
...
void watchWallCollision(int index) {
  int[] wall = walls.get(index);
  // get gap wall settings 
  int gapWallX = wall[0];
  int gapWallY = wall[1];
  int gapWallWidth = wall[2];
  int gapWallHeight = wall[3];
  int wallTopX = gapWallX;
  int wallTopY = 0;
  int wallTopWidth = gapWallWidth;
  int wallTopHeight = gapWallY;
  int wallBottomX = gapWallX;
  int wallBottomY = gapWallY+gapWallHeight;
  int wallBottomWidth = gapWallWidth;
  int wallBottomHeight = height-(gapWallY+gapWallHeight);

  if (
    (ballX+(ballSize/2)>wallTopX) &&
    (ballX-(ballSize/2)<wallTopX+wallTopWidth) &&
    (ballY+(ballSize/2)>wallTopY) &&
    (ballY-(ballSize/2)<wallTopY+wallTopHeight)
    ) {
    // collides with upper wall
  }
  
  if (
    (ballX+(ballSize/2)>wallBottomX) &&
    (ballX-(ballSize/2)<wallBottomX+wallBottomWidth) &&
    (ballY+(ballSize/2)>wallBottomY) &&
    (ballY-(ballSize/2)<wallBottomY+wallBottomHeight)
    ) {
    // collides with lower wall
  }
}
```
`watchWallCollision()` method gets called for each wall on each loop. We grab the coordinates of the gap wall, calculate the coordinates of the actual walls (top and bottom) and we check if the coordinates of the ball collides with the walls.

#### <u>Step 5 - Health and score</u>

Now that we can detect the collisions of the ball and the walls, we can decide on the game mechanics. After some tuning to the game, I managed to make the game remotely playable. But still, it is very hard. My first thought on the game was to make it like Flappy Bird, when the ball touches the walls, game ends. But then I realised it would be impossible to play. So here is what I thought:

There should be a health bar on top of the ball. The ball should lose health while it is touching the walls. With this logic, it doesn't make sense to make the ball bounce back from the walls. So when the health is 0, the game should end and we should switch to game over screen. So here we go:  


```processing
int maxHealth = 100;
float health = 100;
float healthDecrease = 1;
int healthBarWidth = 60;
...
void gameScreen() {
  ...
  drawHealthBar();
  ...
}
...
void drawHealthBar() {
  noStroke();
  fill(236, 240, 241);
  rectMode(CORNER);
  rect(ballX-(healthBarWidth/2), ballY - 30, healthBarWidth, 5);
  if (health > 60) {
    fill(46, 204, 113);
  } else if (health > 30) {
    fill(230, 126, 34);
  } else {
    fill(231, 76, 60);
  }
  rectMode(CORNER);
  rect(ballX-(healthBarWidth/2), ballY - 30, healthBarWidth*(health/maxHealth), 5);
}
void decreaseHealth(){
  health -= healthDecrease;
  if (health <= 0){
    gameOver();
  }
}
```

And here is a simple run:

<img src="http://i.imgur.com/I5fgLTf.gif" style="box-shadow: 0 0 10px lightgray">

We created a global variable `health` to keep the health of the ball. And then created a method `drawHealthBar()` which draws two rectangles on top of the ball. First one is the base health bar, other is the active one that shows the current health. The width of the second one is dynamic, and is calculated by `healthBarWidth*(health/maxHealth)`, the ratio of our current health with respect to the with of the health bar. Finally, the fill colors are set according to the value of health. 

Last but not least, **scores**... Let's begin.

```processing
...
void gameOverScreen() {
  background(0);
  textAlign(CENTER);
  fill(255);
  textSize(30);
  text("Game Over", height/2, width/2 - 20);
  textSize(15);
  text("Click to Restart", height/2, width/2 + 10);
}
...
void wallAdder() {
  if (millis()-lastAddTime > wallInterval) {
    ...
    // added another value at the end of the array
    int[] randWall = {width, randY, wallWidth, randHeight, 0}; 
    ...
  }
}
void watchWallCollision(int index) {
  ...
  int wallScored = wall[4];
  ...
  if (ballX > gapWallX+(gapWallWidth/2) && wallScored==0) {
    wallScored=1;
    wall[4]=1;
    score();
  }
}
void score() {
  score++;
}
void printScore(){
  textAlign(CENTER);
  fill(0);
  textSize(30); 
  text(score, height/2, 50);
}
```

We needed to score when the ball passes a wall. But we need to add maximum 1 score per wall. Meaning, if the ball passes a wall than goes back and passes it again, another score shouldn't be added. To achieve this, we added another variable to the gap wall array within the arraylist. The new variable stores `0` if the ball didn't yet pass that wall and `1` if it did. Then, we modified the `watchWallCollision()` method. We added a condition that fires `score()` method and marks the wall as passed when the ball passes a wall which is not passed before.

 We need to set all the variables we used to their initial value, and restart the game. 

```processing
...
public void mousePressed() {
  ...
  if (gameScreen==2){
    restart();
  }
}
...
void restart() {
  score = 0;
  health = maxHealth;
  ballX=width/4;
  ballY=height/5;
  lastAddTime = 0;
  walls.clear();
  gameScreen = 0;
}
```

Let's add some more colors.

<img src="http://i.imgur.com/dJMUkO4.gif" style="box-shadow: 0 0 10px lightgray">



Porting Flappy Pong for web with p5js
=====


<a href="http://p5js.org/" target="_new">p5js</a> is a Javascript interpreter for processing. It is not a simple library that is capable of running processing codes, instead, p5js takes in pure Javascript codes. Our task was to convert Processing codes into Javascript following p5js format. The library has a set of functions and a syntax similar to Processing, and we have to do certain changes to our code to make them compatible with Javascript. 

First of all, we created a simple `index.html` and add `p5.min.js` to our header. We also created another file called `flappy_pong.js` which will house our converted codes.

```html
<html>
	<head>
		<title>Flappy Pong</title>
		<script tyle="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.4.19/p5.min.js"></script>
		<script tyle="text/javascript" src="flappy_pong.js"></script>
		<style>
			canvas {
				box-shadow: 0 0 20px lightgray;
			}
		</style>
	</head>
	<body>
	</body>
</html>
```

Our strategy while converting the codes should be copying and pasting all our codes into `flappy_pong.js` and then making all the changes. And that's what I did. And here are the steps I took to update the codes:

- Javascript is an untyped languages (ie. there is no type declarations like `int` and `float`). So we need to change all the type declarations to `var`.

- There is no `void` in Javascript. We should change all to `function`
- We need to remove the type declarations of arguments of functions. (ie. `function wallMover(var index) {` to `function wallMover(index) {` )
- There is no `ArrayList` in Javascript. But we can achieve the same thing using Javascript objects. We made the following changes:

	- ```ArrayList<int[]> walls = new ArrayList<int[]>();``` to ```var walls = [];```

	- ```walls.clear();``` to ```walls = [];```
	- ```walls.add(randWall);``` to ```walls.push(randWall);```
	- ```walls.remove(index);``` to ```walls.splice(index,1);```
	- ```walls.get(index);``` to ```walls[index]```
	- ```walls.size()``` to ```walls.length```
- Change the declaration of the array ```var randWall = {width, randY, wallWidth, randHeight, 0};``` to ```var randWall = [width, randY, wallWidth, randHeight, 0];```
- Remove all `public` keywords.
- Move all `color(0)` declarations into `function setup()` because `color()` will not be defined before `setup()` call.
- Change `size(500, 500);` to `createCanvas(500, 500);`
- Rename `function gameScreen(){` to something else like `function gamePlayScreen(){` because we already have a global variable named `gameScreen`. When we were working with processing, one was `void` and one was an `int`. But Javascript confuses these since it is untyped.
- Same thing goes for `score()`. I renamed it to `addScore()`.








