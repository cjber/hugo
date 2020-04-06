+++
author = "Cillian Berragan"
title = "Agent Based Model: Liverpool Crime"
date = "2019-12-12"
tags = [
    "python",
    "abm",
]
categories = [
"Portfolio"
]
+++

An agent based model framework, written in python, demonstrating the interaction between crime and police officers in the Liverpool region. Completed as part of the GEOG5995M University of Leeds module: Programming for Social Scientists.

<!--more-->

---
<p align="center">
<font size="5">
<a href="https://github.com/cjber/abm_liv"> Repository</a></br>

---

<p align="center">
<a href="https://github.com/cjber/abm_liv"> Documentation</a>
</font>
</p>

---


Either directly download as a zip from the "Clone or download" button. Or:

```bash
git clone https://github.com/cjber/abm_liv
```

then run the program with:

```bash
unzip abm_liv.zip
cd abm_liv/
pip install --user -r requirements.txt
python main.py
```

## Final Model

![](img/model.gif)

## Intention of the Software

This model demonstrates a selection of unmoving crime points (red), with a selection of moving police points (blue) contained within a bounding polygon, in this case Liverpool Authority District. Crime points are taken directly from the police.uk API, and are given as a random subset of the total crime data available for the year 2018.

Police aim for the crime they are nearest to, and once they reach a certain proximity this crime is considered solved, and takes a green colour. Police ignore any crime that has been solved.

Additionally, the Liverpool Authority District has been split into grids of equal size, grid colour changes based on whether an unsolved crime, or police point is present for each frame. The presence of unsolved crime reduces the value of a grid, giving it a red colour, while the presence of a police point increases the value of a grid, giving it a green colour. Crime points have a chance to be randomly added with each iteration, this is printed to the console.

The model itself broadly demonstrates the natural occurrence of crime in Liverpool, as most runs produce crime points clustered towards the city centre (North East). While police units are randomly spaced when initiated. Often the model when ran will cause police units to chase crime towards the city centre. If an unsolved crime is present further out, it is often left ignored by police as it does not become a priority to target, as police units chase only the nearest crime point.

The goal of this model design was to demontrate the possibility of utilising true geometric areas to act as a barrier for agents. By taking true crime points for the polygon used, the effect of suitable police location may be assessed practically. Through this, the model also demonstrates the API querying made available through the police.uk website.

## UML Diagram

Given below is a UML diagram which demonstrates the interaction between python Classes and any class level functions included in this program.

### Classes

![](img/classes.png)

### Modules

![](img/packages.png)

The three main scripts are `main.py` which contains the GUI and all functions relating to the running of the model, `crime.py` containing the Crime agent, and `police.py` containing the Police agent. Additionally the second UML image shows that the `api.py` script is connected to `main.py`, this indicates the reading in of the crime data through the API directly into the main program. The `data_clean.py` script is not connected to the main program, and as such is not necessarily required to simply run the model. It is included for the processing of the bounds data, and may be modified as outlined in the **Developer Guide** section of the program [documentation](https://cjber.github.io/abm_liv/).

## Issues during development

* When querying the police.uk API on program startup, occasionally the request will be rejected. This originally meant that the program would fail. However, a solution was devised in which a successful query will save the data as a csv into the data subdirectory. The next run of the program will first query the police.uk API, and if the query is rejected it will fallback to the "cached" police data and use this for the crime points instead:

```python
print('Request sent.')
if response.status_code != 200:
    print("API lookup fail; using cached data.")
    df = pd.read_csv("./data/crime_cached.csv")
else:
    ## process and write to file
```

* Originally there were no checks to determine whether both crime and police points originally appeared within the bounds of the polygon. This meant that often points were not visible. This was solved through using the `geopandas.within()` geometric function and a while loop:


```python
    while True:
        # int for random row
        i = random.randint(0, len(crime_api) - 1)
        x = crime_api['x'].loc[i]
        y = crime_api['y'].loc[i]

        # convert point to geodatafame
        df = pd.DataFrame({'x': [x], 'y': [y]})
        geom = gpd.points_from_xy(df.x, df.y)
        gdf = gpd.GeoDataFrame(df, geometry=geom)

        # find if point falls within polygon
        within = int(gdf.within(self.bounds))
        # while loop breaks only if point is within
        if within == 1:
            self.x = gdf['x']
            self.y = gdf['y']
            self.geom = gdf['geometry']
            break
```

In this code example, the while loop does not break until the x and y variables provide a point which falls `within` the bounds of the `self.bounds` variable (the bounding polygon).

## Testing

This project primarily makes use of the `mypy` optional static type checking syntax for ensuring that correct variable types were used when assigned, and output in both global assignments and functions.

An effort was made to write unit tests using the `unittest` module, however it was unclear what the benefit would be for this object orientated agent approach. Perhaps a greater understanding of unit testing was required. The attempt may be accessed through the `test/` directory. Doctests were used sparingly, as generally functions used in this project do not return anything.

For an example of a doctest see `crime.py` containing the `Crime` object with the `distance_between` function. To test the doctests for this script run:

```
python -m doctest -v crime.py
```

## Concepts

The foundation of an agent based model are the 'agents', which in this demonstration move around randomly, within their own environment. 

First some initial variables were created:

```python
agents = []
num_of_agents = 10
num_of_iterations = 100
```

To move the agents around randomly, the python `random` library was imported and used to create random `xy` coordinates within a 100x100 grid.

```python
import random  # random number library

# random xy coordinates in 100x100 grid
for i in range(num_of_agents):
    agents.append([random.randint(0, 99), random.randint(0, 99)])

for j in range(num_of_iterations):
    for i in range(num_of_agents):
        if random.random() < 0.5:
            agents[i][0] = (agents[i][0] + 1) % 100
            #  print("adding 0 1") #  checking the values
        else:
            agents[i][0] = (agents[i][0] - 1) % 100
            #  print("taking 0 1")

        if random.random() < 0.5:
            agents[i][1] = (agents[i][1] + 1) % 100
            #  print("adding 1 1")
        else:
            agents[i][1] = (agents[i][1] - 1) % 100
            #  print("taking 1 1")

        #  check limits are followed
        if agents[i][1] > 99 or agents[i][1] < 0 or \
                agents[i][0] > 99 or agents[i][0] < 0:
            print("Warning; outside limits")
```
To ensure that if a agent is placed outside the 100x100 grid, the modulus operator, `%`, gives the remainder of the calculation `agents[i][i] +/- 1 / 100`, if the new agent position is below 0 or above 100, i.e. the value will wrap around to 99 with an agent position of -1, or 1 with an agent position of 101. Included in the above code are some simplistic debugging techniques, `print("adding X X")` indicates which random movement was being added in a particular iteration while `print("warning; outside limits")` was included to indicate whether the modulus operator was correctly limiting the agent limits.

These agents may now be plotted using the `matplotlib` library. The `num_of_agents` variable is used here to iterate through the plot, and create the specified number of agents:

```python
import matplotlib.pyplot as plt

plt.ylim(0, 99)
plt.xlim(0, 99)
for i in range(num_of_agents):
    plt.scatter(agents[i][1], agents[i][0])
plt.show()
```

The output of this code is a graph with random agent locations, the position of these agents changed each time the plot is rendered.

### Object-Orientated Programming

Building on the concept of 'agents' and random movement, it is possible to use object orientated programming to create an agent `Class`. From now on, the Agent class may be moved into a separate python script, and the script called in using the `import` function. A minimal example of this current Class is given:

```python
class Agent():

    def __init__(self, environment, agents, x=None, y=None):
        self.x = random.randint(0, len(environment[0]))
        self.y = random.randint(0, len(environment))

        self.agents = agents
        self.store = 0

        self.environment = environment

    def move(self, environment):
        height = len(environment)
        width = len(environment[0])

           if random.random() < 0.5:
                self.x = (self.x + rand) % width
            else:
                self.x = (self.x - rand) % width
            if random.random() < 0.5:
                self.y = (self.y + rand) % height
            else:
                self.y = (self.y - rand) % height
```

Classes are appropriate for developing agent based model frameworks as they enable the creation of new types of objects, which there may be multiple instances of. Additionally, classes in python may have methods which enable the modification of the class state.

With this agent class is is now possible to create agents in an object orientated manner:

```python
# agentframework.py contains the Agent Class
# and is within the same directory as the main script
import agentframework 

num_of_agents
agents = []

# Make the agents.
for i in range(num_of_agents):
    agents.append(agentframework.Agent())

# Move the agents.
for j in range(num_of_iterations):
    for i in range(num_of_agents):
        agents[i].move()
```

### Environment

The environment was created from a text file with comma separated values ranging from 0 to ~250, with 300 values per line, and 300 lines. This was read into python forming a two dimensional array:

```python
environment = []
with open('in.txt') as csv_file:
    csv_reader = csv.reader(csv_file, delimiter=',')
    for row in csv_reader:
        rowlist = []
        for value in row:
            rowlist.append(int(value))
        environment.append(rowlist)
```

## Animating

The next step is to animate the model, and allow a number of iterations to see how it develops over time. First initial parameters may be set, one method for user input is the use of command line arguments. This may be achieved through the `sys` import.

```python
import sys

num_of_agents = int(sys.argv[1])
num_of_iterations = int(sys.argv[2])
neighbourhood = int(sys.argv[3])
```

This allows when running the program for parameters to be selected in the form: `python main.py 10 10 10`. 

To create the animation, both the `matplotlib.pyplot` and `matplotlib.animation` libraries were required, the total imports used now include:

```python
import random
import matplotlib.pyplot as plt
import csv
import sys
import agentframework
import importlib

# typically accepted convention
from matplotlib.animation import FuncAnimation
```

Additional functions allow for the running of the animation, in this example agents move randomly, and eat the environment. An `update()` function is created which runs the class functions, and updates the plot each frame. The `gen_function()` specifies the number of frames to run, which in this case is a maximum of 1000, or until the stopping condition is met.


```python
fig = plt.figure()


def update(frame_number):
    for j in range(num_of_iterations):
        for i in range(num_of_agents):
            agents[i].move(environment)
            agents[i].eat()
            fig.clear()
            global carry_on

    if random.random() < 0.0001:
        carry_on = False
        print("stopping condition")

    for i in range(num_of_agents):
        plt.imshow(environment, vmin=min_val, vmax=max_val,
                   cmap='copper')
        plt.scatter(agents[i].x, agents[i].y, c='green', alpha=0.6)
        plt.axis('off')


def gen_function(b=[0]):
    a = 0
    global carry_on
    while (a < 1000) & (carry_on):
        yield a
        a = a + 1


animation = FuncAnimation(fig, update, interval=1, repeat=False,
                               frames=gen_function)
plt.show()
```
