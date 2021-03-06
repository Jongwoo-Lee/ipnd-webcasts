Using Object Oriented Programming to Animate Objects
===

# Introduction

This Webcast session will talk about the concept of animation and how Object Oriented Programming is perfectly suited to animate objects on the screen. Animating objects is a classic example for the use of Object Oriented Programming (OOP) because we can easily think of real objects in the real world and how each object can move and interact with the world independent from any other object that exists.

At the end of the Webcast session, we will understand:

* The theory of animation and how to implement animation in a program
* How to create circle objects and animate these objects using OOP design
* Learn about edge collision to prevent the circle objects from moving off the screen

# Animation

Animation is just a series of still pictures shown in very rapid succession. If we think how old film movies work, we can understand that the film it self is made up of very large wheels of pictures that are rapidly displayed to people. Therefore to animate an onscreen object, we need to display pictures of an object in rapid succession.

Let's start off with this starter code for this Webcast session:

```python
# Object Oriented Programming Example with Bouncing Balls using Tkinter GUI library: Starter Code

from Tkinter import *

# Define Settings, Globals, and other helper objects

PROGRAM_NAME = 'Bouncy'
FPS = 60
MS_PER_FRAME = int(1.0/FPS * 1000)            # Will equal 16 milliseconds

class Application(Frame):

  canvas_width = 800
  canvas_height = 800

  def __init__(self):
    Frame.__init__(self, master=None)
    self.grid()
    self.__createWidgets()
    self.__createCanvasObjects()
    self.__animate()

  def __createWidgets(self):
    """Creates all widgets for the Application frame"""
    canvas_options = {'width' : Application.canvas_width,
                      'height': Application.canvas_height,
                      'bg'    : 'white'
                     }

    self.__canvas = Canvas(self, canvas_options)
    self.__canvas.pack()

  def __createCanvasObjects(self):
    """Initialize all of our objects to draw on the canvas"""
    self.__data = {}

    self.__data['circleColor'] = 'blue'
    self.__data['circleLeft'] = 50
    self.__data['circleTop'] = 50
    self.__data['circleSize'] = 50


  def __animate(self):
    # Update all object positions based on their current state: collisions for example
    self.__updateAll()

    # Clear canvas and redraw all objects with their new positions
    self.__redrawAll()

    # This is the key to our animation. We wait MS_PER_FRAME and call our animate function again
    self.__canvas.after(MS_PER_FRAME, self.__animate)

  def __updateAll(self):
    self.__data['circleLeft'] += 250.0/FPS

  def __redrawAll(self):
    """Deletes existing pixels from canvas and redraws everything"""
    self.__canvas.delete(ALL)

    self.__canvas.create_oval(self.__data['circleLeft'],
                  self.__data['circleTop'],
                  self.__data['circleLeft'] + self.__data['circleSize'],
                  self.__data['circleTop'] + self.__data['circleSize'],
                  fill=self.__data['circleColor'],
                  width = 0)

def main():
  app = Application()
  app.winfo_toplevel().title(PROGRAM_NAME)

  # Start application
  app.mainloop()

if __name__ == '__main__':
  main()
```

We'll be using the Tkinter library that comes standard in Python to create a Graphical User Interface application (GUI). The entire application has three stages:

* Initialize GUI components and widgets
* Create the objects to be shown on a Canvas object; this is where we draw all of our graphics.
* Animate the objects

We can see these steps in the Application constructor here:

```python
  def __init__(self):
    Frame.__init__(self, master=None)
    self.grid()
    self.__createWidgets()
    self.__createCanvasObjects()
    self.__animate()
```

Let's go into the `self.__animate()` function. We assume that the audience is already familiar with Object Oriented Programming in Python. Note that the name of the function is `__animate`. The naming convention to put two underscore characters in front of the function tells us that we want this function to be private and we should not access this function outside of the Application object.

Here is the code for the `__animate` function:

```python
def __animate(self):
    # Update all object positions based on their current state: collisions for example
    self.__updateAll()

    # Clear canvas and redraw all objects with their new positions
    self.__redrawAll()

    # This is the key to our animation. We wait MS_PER_FRAME and call our animate function again
    self.__canvas.after(MS_PER_FRAME, self.__animate)
```

There are three distinct stages to animation:

* Update position of all our objects. If an object collides with the edge of the canvas or gets input from a user, we want to update the x and y positions of all of our objects before we re-draw the objects on our canvas.
* In order to give the illusion of animation, for every frame, we need to clear the canvas and delete all pixels that were painted on the canvas in the last frame. This the main reason why we call the widget that we draw on 'Canvas' because in traditional painting, once we paint on a canvas, the color stays there permanently. Without wiping away the existing pixels on the canvas, the pixels will never go away and can only get drawn over. Once we clear the canvas, we then proceed to redraw all of our objects with their updated positions on the canvas.
* We wait a few milliseconds to call our `__animate` function again. This is the key to animation in programming. We program a function that gets called repeatedly every few milliseconds. In order to achieve smooth 60 frames per second animation, we need to wait approximately 16.667 milliseconds before we move onto the next frame. Since the `after` function can only take in integers, we will wait 16 milliseconds before the next frame.

# Creating Objects for Animation

Looking at the starter code, we know that if we want to add in new objects, we would need to create the object, add in code to update its position every frame, and then add in code to redraw the object. Imagine we want to create 50 objects. Without OOP design, we would be forced to write a few hundred lines of code that are very similar to each other.

This method would be error-prone because we have to copy and paste a lot of code and violates the DRY principal (Don't-Repeat-Yourself) in programming. We want to use the computer to take care of all of the details of creating and keeping track of our objects for us.

Therefore we will design our objects with OOP design in mind. For this case, we will create two classes: a Shape class and a Circle class.

## The Shape Class

We want to create a general class called Shape that specific objects such as a circle, square, polygon, and triangle can inherit. This goes back to the DRY principle because all of these shapes share similar characteristics such as **position, velocity, width, height, etc.** Furthermore, if we were to add in any new features to our shapes, we can simply add in our feature in the Shape class and the rest of our shapes will inherit the new feature.

Here is how we start to define the Shape class:

```python
class Shape(object):

# Private variables
  x_pos, y_pos = 0, 1

  def __init__(self, vel, width, height, color, canvas_width, canvas_height, x=None, y=None):
    """Constructor for Shape. Randomly assigns a position of the shape in the canvas and
       assigns the velocity list [x,y] as a velocity vector to the shape.
    """
    self.x = random.randint(0,canvas_width - width) if not x else x
    self.y = random.randint(0,canvas_height - height) if not y else y
    self.width = width
    self.height = height
    self.color = color
    self.vel = list(vel)                      # We need to make a copy of the list or else
                                              # there will be reference errors
```

The constructor tells us that whenever we create a new object, we will create a new object with the position x and y, width, height, color, and velocity. Velocity will be a 2-dimensional list that contains the magnitude and direction that will help us calculate the position of our object for any frame.

We now want to program some helpful functions that will help us get and set information in our Shape object:

```python
  # Getters and Setters
  def get_vel(self):
    return list(self.vel)

  def set_vel(self, vel):
    self.vel = list(vel)

  def get_position(self):
    return tuple([self.x,self.y])

  def set_position(self,x,y):
    self.x = x
    self.y = y
```

Next we want to create our animation methods to help us update the position of our object and help us detect whether our object collided with the edge of the canvas

```python
  # Animation methods-------------------------------------------------------------------------------
  def update(self, canvas):
    """Updates x and y position of shape based on the shape's current state."""
    self.x += self.vel[Shape.x_pos]/float(FPS)
    self.y += self.vel[Shape.y_pos]/float(FPS)
```

This small function is the work horse of our entire animation because upon the call to `update()`, the x and y position of our object changes due to the velocity vector. Notice how we are dividing x and y components of our velocity by the variable `FPS`. This is purely used as a convenience factor for us when we set the velocity. When we set the velocity, it helps us to think in pixels per second. This conversion will convert the rate from pixels per second to pixels per frame.

## The Circle Class

We can now dive into the specific functions that make up the Circle class.

```python

class Circle(Shape):
  """Inherits from Shape class that contains that creates circle objects on the canvas and handles
  collisions between Circle objects
  """

  def __init__(self, vel, width, height, color, canvas_width, canvas_height, x=None,y=None):
    assert width == height, 'Width and Height need to be the same!'
    Shape.__init__(self, vel, width, height, color, canvas_width, canvas_height,x,y)

    self.radius = width/2
    self.center = (self.x + self.radius, self.y + self.radius)
```

We can see that we are inheriting the Shape class from the first line `class Circle(Shape):`. We are also overriding the Constructor method that Shape has. Since our shape is now a circle, there are two properties that would be useful for us to know: radius and center point.

The center point is the point at the center of the circle. Knowing the center of the circle is useful to calculate collisions and perform other calculations on the circle, so we need to add in a center attribute for all of our circle objects.

We also override the update function as well:

```python
  def update(self, canvas):
    """Updates the circle's center position in addition to its x and y position."""
    Shape.update(self, canvas)
    self.center = (self.x + self.radius, self.y + self.radius)
```

We can now move on to two new functions that are specific to our Circle class:

```python
@classmethod
  def create_random_circle(cls):
    """Creates a circle with random size, velocity, and colors"""

    min_width = 20
    max_width = 80
    min_speed = -400
    max_speed = 400

    colors = ['orange','blue','red','purple','cyan','yellow','magenta']

    width = random.randint(min_width, max_width)
    height = width

    vel = [random.randint(min_speed, max_speed),
           random.randint(min_speed, max_speed)]

    color = colors[random.randint(0,len(colors) - 1)]

    return cls(vel,width,height,color,Application.canvas_width,Application.canvas_height)
```

`create_random_circle` is a class method that takes in a reference to the class itself and does not belong to any object of the Circle class. This means that this function is at the class level and we call this class method like this: `Circle.create_random_circle()`.

Class methods are useful because we do not need to create instances of the same function per object, improving our efficiency.

We calculate our random variables and then return a new Circle object by invoking the Circle's constructor function:

`return cls(vel,width,height,color,Application.canvas_width,Application.canvas_height)`

`cls` is a reference to the Circle class itself.

Our final function to program is the draw function:

```python
def draw(self, canvas):
    """Takes in a canvas object and will draw a circle on the canvas object."""
    canvas.create_oval(self.x, self.y,
                       self.x + self.width, self.y + self.height,
                       fill = self.color,
                       width = 0)
                       
```

We use the `create_oval` method to actually draw the circle and fill it with the object's color

# Edge Collision

In order to detect whether the object has hit the edge of our canvas, we create a bounding box that is an imaginary box that covers our circle:

![](images/bound_box1.png)

If we were to analyze this bounding box further, we can see that we have four points that we can use in order to see whether the edge of the canvas crosses the edge of one of our bounding boxes

![](images/bound_box2.png)

Therefore, we can say that for any edge on the canvas, if the object's edge is at or beyond the edge of the canvas, we can say that the object collides with the edge and therefore we should change our velocity to have the object bounce back towards the inside of the canvas.

For example, to check whether the bounding box collides with the top edge of the canvas, we do:

`if obj.y <=0:`

The canvas coordinate system starts with (`0,0`) on the upper left corner of the canvas and (`canvas_width, canvas_height`) will be the lower right hand corner of the canvas:

![](images/canvas.png)

Therefore if we want to check whether the object edge is at or beyond the top edge of the canvas, we check whether `obj.y` is less than or equal to 0

If we want to check whether the bottom of the bounding box collides with the bottom edge of the canvas, we do:

`if obj.y + obj.height >= canvas.height`

Therefore our edge collision function can be:

```python
def edge_collision_check(self, canvas, canvas_width, canvas_height):
    """Assumes bounding box is rectangular in shape. Checks whether the edges of the bounding
    box intersect the edges of the canvas. If the shape is at the edge of the canvas, function
    updates the velocity vectors to bounce the shape away from the edge.
    """

    # Check top and bottom edges
    if self.y <= 0 or self.y + self.height >= canvas_height:
      self.vel[Shape.y_pos] = -self.vel[Shape.y_pos]

    # Check left and right edges
    if self.x <= 0 or self.x + self.width >= canvas_width:
      self.vel[Shape.x_pos] = -self.vel[Shape.x_pos]

```

# Putting Everything Together

Once we've created our classes, we can simply instantiate our objects and call the update methods for each object. Here is the final code with updated object creation and object update methods.  
               
```python
# Object Oriented Programming Example with Bouncing Balls using Tkinter GUI library: Final Code
from Tkinter import *
import random

# Define Settings, Globals, and other helper variables

PROGRAM_NAME = 'Bouncy'
FPS = 60
MS_PER_FRAME = int(1.0/FPS * 1000)            # Will equal 16 milliseconds

# Define Classes------------------------------------------------------------------------------------

class Shape(object):

  # Private variables
  x_pos, y_pos = 0, 1

  def __init__(self, vel, width, height, color, canvas_width, canvas_height, x=None, y=None):
    """Constructor for Shape. Randomly assigns a position of the shape in the canvas and
       assigns the velocity list [x,y] as a velocity vector to the shape.
    """
    self.x = random.randint(0,canvas_width - width) if not x else x
    self.y = random.randint(0,canvas_height - height) if not y else y
    self.width = width
    self.height = height
    self.color = color
    self.vel = list(vel)                      # We need to make a copy of the 												list or else
                                              # there will be reference errors

  # Getters and Setters
  def get_vel(self):
    return list(self.vel)

  def set_vel(self, vel):
    self.vel = list(vel)

  def get_position(self):
    return tuple([self.x,self.y])

  def set_position(self,x,y):
    self.x = x
    self.y = y

  # Animation methods-------------------------------------------------------------------------------
  def update(self, canvas):
    """Updates x and y position of shape based on the shape's current state."""
    self.x += self.vel[Shape.x_pos]/float(FPS)
    self.y += self.vel[Shape.y_pos]/float(FPS)

  def edge_collision_check(self, canvas, canvas_width, canvas_height):
    """Assumes bounding box is rectangular in shape. Checks whether the edges of the bounding
    box intersect the edges of the canvas. If the shape is at the edge of the canvas, function
    updates the velocity vectors to bounce the shape away from the edge.
    """

    # Check top and bottom edges
    if self.y <= 0 or self.y + self.height >= canvas_height:
      self.vel[Shape.y_pos] = -self.vel[Shape.y_pos]

    # Check left and right edges
    if self.x <= 0 or self.x + self.width >= canvas_width:
      self.vel[Shape.x_pos] = -self.vel[Shape.x_pos]

class Circle(Shape):
  """Inherits from Shape class that contains that creates circle objects on the canvas and handles
  collisions between Circle objects
  """

  def __init__(self, vel, width, height, color, canvas_width, canvas_height, x=None,y=None):
    assert width == height, 'Width and Height need to be the same!'
    Shape.__init__(self, vel, width, height, color, canvas_width, canvas_height,x,y)

    self.radius = width/2
    self.center = (self.x + self.radius, self.y + self.radius)

  @classmethod
  def create_random_circle(cls):
    """Creates a circle with random size, velocity, and colors"""

    min_width = 20
    max_width = 80
    min_speed = -400
    max_speed = 400

    colors = ['orange','blue','red','purple','cyan','yellow','magenta']

    width = random.randint(min_width, max_width)
    height = width

    vel = [random.randint(min_speed, max_speed),
           random.randint(min_speed, max_speed)]

    color = colors[random.randint(0,len(colors) - 1)]

    return cls(vel,width,height,color,Application.canvas_width,Application.canvas_height)

  def update(self, canvas):
    """Updates the circle's center position in addition to its x and y position."""
    Shape.update(self, canvas)
    self.center = (self.x + self.radius, self.y + self.radius)

  def draw(self, canvas):
    """Takes in a canvas object and will draw a circle on the canvas object."""
    canvas.create_oval(self.x, self.y,
                       self.x + self.width, self.y + self.height,
                       fill = self.color,
                       width = 0)

  def circle_collision_check(self, objects):
    """Bonus: Add in our circle collision check to calculate collisions amongst circle objects.
       Reference article to read: http://simonpstevens.com/articles/vectorcollisionphysics
    """
    pass

# Define Main GUI Application-----------------------------------------------------------------------

class Application(Frame):

  canvas_width = 800
  canvas_height = 800

  def __init__(self):
    Frame.__init__(self, master=None)
    self.grid()
    self.__create_widgets()
    self.__create_canvas_objects()
    self.__animate()

  def __create_widgets(self):
    """Creates all widgets for the Application Frame."""
    canvas_options = {'width' : Application.canvas_width,
                      'height': Application.canvas_height,
                      'bg'    : 'white'
                     }

    self.__canvas = Canvas(self, canvas_options)
    self.__canvas.pack()

  def __create_canvas_objects(self):
    """Initialize all of our objects to draw on the canvas."""
    self.__objects = []

    # Create 10 random circles to bounce around
    for num in range(10):
      self.__objects.append(Circle.create_random_circle())

  def __animate(self):
    """The animation function that updates all positions, redraws all objects and calls itself
    again.
    """
    # Update all object positions based on their current state: collisions for example
    self.__updateAll()

    # Clear canvas and redraw all objects with their new positions
    self.__redrawAll()

    # This is the key to our animation. We wait MS_PER_FRAME and call our animate function again
    self.__canvas.after(MS_PER_FRAME, self.__animate)

  def __updateAll(self):
    """Updates all object positions and vectors depending on where they are at any point of the
    canvas.
    """
    for obj in self.__objects:
      obj.edge_collision_check(self.__canvas, Application.canvas_width, Application.canvas_height)
      obj.circle_collision_check(self.__objects)
      obj.update(self.__canvas)

  def __redrawAll(self):
    """Deletes all existing objects from canvas and redraws everything"""
    self.__canvas.delete(ALL)

    for obj in self.__objects:
      obj.draw(self.__canvas)
      
def main():
  # Setup application
  app = Application()
  app.winfo_toplevel().title(PROGRAM_NAME)

  # Start application
  app.mainloop()

if __name__ == '__main__':
  main()
```