---
title: Adventures in Python- Mars Rover
tags: python
date: 2023-09-06 17:46:55
excerpt: >-
    I learned a bit of Python this week. Mostly though I felt like I was just figuring out how to translate between Java or C# to the Python equivalent. It was really interesting to see how certain concepts play out across the languages. It was a lot of fun and I wanted to take it a little further. I'm going to do this by implementing some of the TDD Katas I've been using for Java and C# into Python. First up Mars Rover...
---


I learned a bit of Python this week. Mostly though I felt like I was just figuring out how to translate between Java or C# to the Python equivalent. It was really interesting to see how certain concepts play out across the languages. It was a lot of fun and I wanted to take it a little further. I'm going to do this by implementing some of the TDD Katas I've been using for Java and C# into Python. First up Mars Rover...

> The code for this kata is [here](https://github.com/nickmza/python). I've tried to commit after each passing test if you want to follow along.

> Python Friends: I am  aware that this is probably not idiomatic Python - please point out more Pythony? Pythonesque? approaches in the comments.

# Mars Rover Kata

Here's a summary of the problem from [Kata Log](https://kata-log.rocks/mars-rover-kata):

- You are given the initial starting point (x,y) of a rover and the direction (N,S,E,W) it is facing.
- The rover receives a character array of commands.
- Implement commands that move the rover forward/backward (f,b).
- Implement commands that turn the rover left/right (l,r).
- Implement wrapping at edges. But be careful, planets are spheres.
- Implement obstacle detection before each move to a new square. - If a given sequence of commands encounters an obstacle, the rover moves up to the last possible point, aborts the sequence and reports the obstacle.

# Starting with the grid
I've decided to model the world as a grid and have this grid contain all the information about the world such as obstacles etc. My thinking is that the Rover can then ask the Grid about its environment. 

I started by creating some test data. I'm using X to indicate an Obstacle. I also decided to add ? for Aliens and * for Resources. This is not part of the original kata but I thought it could be fun to add new behaviors to handle these in future katas.

```
....................
...............?....
...X......*.........
....................
.............X......
...*................
....................
........X...........
...?.........*......
....................
```

For my first test I wanted to check to see if the file could be loaded and contained the correct information. I broke this into 3 smaller tests:
{% codeblock lang:python %}
    def test_board_parse(self):
        inputFile = rover.readFile()
        self.assertTrue(len(inputFile) == 10)
    
    def test_parse_board_dimension(self):
        inputFile = rover.readFile()
        grid = rover.parseBoard(inputFile)
        self.assertTrue(grid.width == 20)
        self.assertTrue(grid.height == 10)

    def test_obstacles_loaded(self):
        inputFile = rover.readFile()
        grid = rover.parseBoard(inputFile)
        obstacles = grid.GetObstacles()
        self.assertEquals(len(obstacles),3)
{% endcodeblock %}

The first just checks to see if the loaded file has the correct number of lines. The second checks that the dimensions are correct. The last one checks that the correct number of obstacles were loaded. I used this approach so that I could limit the amount of implementation code I had to write for each test. 

> Looking at this now I see that the assert of the 3rd test is weak. I should also assert that the 3 obstacles are in the expected locations.


In terms of the implementation code there's nothing too complicated. One interesting part is this:

{% codeblock lang:python %}
class CellFactory():
    def createCell(self, symbol, row: int, column: int) -> Cell:
        match symbol:
            case ".":
                return None
            case "*":
                return Resource(row, column)
            case "X":
                return Obstacle(row, column)
            case "?":
                return Alien(row, column)

def parseBoard(input: list) -> Grid:
    grid = Grid(len(input[0]), len(input))

    row = 1
    col = 1

    factory = CellFactory()
    for line in input:
        for symbol in line:
            cell = factory.createCell(symbol,row, col)
            if(cell != None):
                grid.cells.append(cell)
            col+=1
        col = 1
        row += 1

    return grid
{% endcodeblock %}

In this code I initialise my Grid and then iterate through each line and each character in the input file. I use a  [factory](https://en.wikipedia.org/wiki/Factory_method_pattern) to create the corresponding class for each character and add it to a list. I considered creating a multidimensional array here but I can't think of any specific value that would be added at this stage. Down the line there may be performance implications but for now this is simple and works. I've also decided to not create any object for 'empty' cells on the grid. 

Finally for the Grid I implement tests to see if I can get back the Cell at a particular location:
{% codeblock lang:python %}
def test_get_cell(self):
    grid = self.getBoard()
    cell = grid.get_cell(3,4)
    self.assertTrue(isinstance(cell, rover.Obstacle))
    self.assertEqual(cell.row, 3)
    self.assertEqual(cell.column, 4)
{% endcodeblock %}

The implementation introduced me to Python's Filter operation:

{% codeblock lang:python %}

    def get_cell(self, row, column) -> Cell:
        cells = list(filter(lambda cell: cell.row == row and cell.column == column, self.cells))
        if(len(cells) > 0):
            return cells[0]
        else:
            return None

{% endcodeblock %}

To get back a specific cell I create a Filter passing a lambda and the collection to filter. Items that meet the condition defined in the lambda will be included in the filter. I then have to pass the Filter to a List so that I can inspect the results. I'm guessing this works in a similar fashion to Linq in C# where the expression is only evaluated once it's iterated over or accessed. If there is no specific cell returned we return None to indicate that the cell is 'empty'. Looking at this now I see I need to add a validation for when the requested Cell is outside of the range of the Grid.

# The Rover

At this point we have Grid that the Rover can use to understand the environment. The kata asks that we give the Rover co-ordinates and a direction. I wanted to start by checking that we throw an exception if the Rover is placed off the grid. 

{% codeblock lang:python %}

    def test_init_rover_fail_if_invalid(self):
        inputFile = rover.readFile()
        grid = rover.parseBoard(inputFile)

        try:
            r = rover.Rover(grid,99,99, rover.CompassDirection.North)
            self.assertTrue(False)
        except:
            pass

{% endcodeblock %}

Next I wanted to check that the Rover can parse a stream of Commands. I pass the command string to the Rover and then get back its current buffer of commands. We want to ensure there are the correct number of commands and that they have been translated correctly.

{% codeblock lang:python %}

    def test_command_parsing(self):
        inputFile = rover.readFile()
        grid = rover.parseBoard(inputFile)    
        r = rover.Rover(grid,1,1, rover.CompassDirection.South)
        commands = "FFFLFF"
        r.load_commands(commands)
        commandBuffer = r.commands
        self.assertEquals(len(commandBuffer), 6)
        self.assertEquals(commandBuffer[5], rover.Commands.Forward)

{% endcodeblock %}

To implement this I re-used the pattern for loading the Grid:

{% codeblock lang:python %}

    def get_command(self, command: str):
        match command:
            case "F":
                return Commands.Forward
            case "B":
                return Commands.Backward
            case "L":
                return Commands.Left
            case "R":
                return Commands.Right
    
    def load_commands(self, commands: str):
        for command in commands:
            self.commands.append(self.get_command(command))

{% endcodeblock %}

With this in place I can now try to move the Rover:

{% codeblock lang:python %}

def test_move_rover(self):
    inputFile = rover.readFile()
    grid = rover.parseBoard(inputFile)    
    r = rover.Rover(grid,1,1, rover.CompassDirection.South)
    commands = "FFFLFF"
    r.load_commands(commands)
    r.execute_commands()

    self.assertEquals(r.row, 4)
    self.assertEquals(r.column, 3)

{% endcodeblock %}

To implement the movement we iterate through the commands executing each one. 

> This is a really simple implementation that does not meet the full requirements of the kata **but** it meets the criteria of this test. Later on when we add tests for collision detection it will force me to revisit this code.

{% codeblock lang:python %}

def execute_commands(self):
    for command in self.commands:
        match command:
            case Commands.Forward:
                self.move_rover_forward()
            case Commands.Backward:
                self.move_rover_backward() 
            case Commands.Left:
                self.turn_rover_left()  
            case Commands.Right:
                self.turn_rover_right()  

{% endcodeblock %}

Moving forward/backward involves increasing the Rover's current Row/Column based on it's current direction. Turning involves changing the current direction. To make this easier I modelled Direction as an Enum so we can just add/remove 1 from the current value to change the direction. Here's the code:

{% codeblock lang:python %}

    def change_direction(self, change: int):
        current = self.direction.value
        current += change
        if(current <= 0):
            current = 4
        if(current > 4):
            current = 1
        self.direction = CompassDirection(current)

    def turn_rover_left(self):
        self.change_direction(-1)

    def turn_rover_right(self):
        self.change_direction(1)

{% endcodeblock %}

# Next Steps
At this point we have the Rover on the Grid and able to move around. Next steps will be to implement wrapping and collision detection. So far Python has been pretty accessible. Documentation is good and there are plenty of examples online. I do have the feeling that there are probably different ways to do things that are specific to Python but hopefully I'll learn some of these as we go. For now I just need to stick to Python's naming conventions and stop myself from finishing every line with ';'. 