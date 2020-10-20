# particle_motion
## A program designed to model a particle's 2D trajectory under multiple forces - and something fun.
    from ipywidgets import interactive
    import matplotlib.pyplot as plt
    import random

### Just creating a nice rectangle shape for later. Also, a visualized standard number generator.
    class Box:
        def __init__(self, left, right, bot, top):
            self.left = left
            self.right = right
            self.top = top
            self.bot = bot
        
    def random_num_gen(A, B, M):
        numbers = [1,2,3,4,5,6,7,8,9]
        start = random.choice(numbers)
        lis = []

        for result in range(6):
            new = ((start * A) + B) % M
            start = new
            lis.append(start)
        choice = random.choice(lis)

        return choice

### created functions to flip the particle so it will bounce off the sides of our container correctly. These are based off of standard kinematics.
    def flip_x(side, pos_f, pos_i, vel_i, vel_f):
        pos_f[0] = side
        time = (pos_i[0] - side)/vel_i[0]
        vel_f[0] *= -1
        pos_f[0] = pos_i[0] + vel_f[0]*time

    def flip_y(pos_f, pos_i, vel_i, vel_f, acc, universe):
        pos_f[1] = universe.bot
        time = -pos_i[1]/vel_i[1]
        vel_f[1] = -(vel_i[1] - acc[1]*time)
        pos_f[1] = pos_i[1] + vel_f[1]*time
    
### we create a shift function for a teleporter, this moves the particle left or right (whether p or m is positive or negative) by the amount of x or y
    def shift(x, y, p, m, pos_f):
        pos_f[0] += x * p
        pos_f[1] += y * m
    
### The nuts and bolts of a teleporter for our particle. In our wormhole box, we run a random number generator, and the number returned triggers the particle to be shifted somewhere else in the 'universe'
    def teleporter(wormhole, pos_f):
        if wormhole.left <= pos_f[0] <= wormhole.right and wormhole.bot <= pos_f[1] <= wormhole.top:
            x = random_num_gen(4,2,9)
            print(x)
            if x == 0:
                shift(5, 12, 1, 1, pos_f)
            elif x == 1:
                shift(25, 11, -1, 1, pos_f)
            elif x == 2:
                shift(10, 5, 1, -1, pos_f)
            elif x == 3:
                shift(9, 8, 1, 1, pos_f)
            elif x == 4:
                shift(20, 10, -1, -1, pos_f)
            elif x == 5:
                shift(12, 7, 1, -1, pos_f)
            elif x == 6:
                shift(10, 14, -1, 1, pos_f)
            elif x == 7:
                shift(17, 8, -1, -1, pos_f)
            elif x == 8:
                shift(18, 25, 1, 1, pos_f)
            else:
                pass
        else:
            pass
        
#### The move function details all of the forces that the particle will go under, and then moves the particle.
    def move(pos_i, vel_i, g, dt, wormhole, universe, mass, k, viscous):
#### Create our final position and velocity variables, initializing them as our previous position so they can be moved.
        vel_f = [vel_i[0],vel_i[1]]
        pos_f = [pos_i[0],pos_i[1]]
    
#### we apply our standard viscous force with a second power velocity. If the particle slows enough and switches direction the viscous vector will switch as well.
        if vel_f[0] >= 0:
            viscous[0] = -(vel_f[0] ** 2)/mass
        else:
            viscous[0] = (vel_f[0] ** 2)/mass
        if vel_f[1] >= 0:
            viscous[1] = -(vel_f[1] ** 2)/mass
        else:
            viscous[1] = (vel_f[1] ** 2)/mass
       
#### For a central force, I am creating a standard spring oscillation form. The farther the particle gets from the center, y = 25, then the stronger the force becomes.
        center = (pos_f[1] - 25)
        ar = [0, -(k * center)]
    
#### we sum our gravitational, viscous, and central force accelerations, and then apply the method of finite differences
        acc = [g[0] + viscous[0] + ar[0], g[1] + viscous[1] + ar[1]]
        vel_f[0] = vel_i[0] + acc[0] * dt
        vel_f[1] = vel_i[1] + acc[1] * dt
        pos_f[0] = pos_i[0] + vel_i[0] * dt
        pos_f[1] = pos_i[1] + vel_i[1] * dt

#### Comparisons to apply our flips. If the particle hits a wall, it flips.
        if universe.bot < pos_f[1] < universe.top:
            pass
        else:
            flip_y(pos_f, pos_i, vel_i, vel_f, acc, universe)

        if universe.left < pos_f[0] < universe.right:
                pass
        elif pos_f[0] > (universe.right / 2):
            flip_x(universe.right, pos_f, pos_i, vel_i, vel_f)
        elif pos_f[0] < (universe.right / 2):
            flip_x(universe.left, pos_f, pos_i, vel_i, vel_f)


        teleporter(wormhole, pos_f)


        return (pos_f, vel_f)

### The bread and butter of our simulation, our run statement. We define our variables and then perform a while loop for N turns. Our graph ends up as a list of X and Y and is graphed.
    def run_sim(k=0.5, gy=-4.81):
        wormhole = Box(25,30,15,20)
        universe = Box(0,50,0,50)
        dt = 0.25
        i = 0
        mass = 25
        pos_i = [1, 40]
        vel_i = [5, 0]
        g = [0, gy]
        X = []
        Y = []
        viscous = [0,0]

        while i < 75:
            pos_i, vel_i = move(pos_i, vel_i, g, dt, wormhole, universe, mass, k, viscous)
            X.append(pos_i[0])
            Y.append(pos_i[1])
            i+=1

        fig , ax = plt.subplots(figsize = (4, 4))
        time.sleep(0.1)
        plt.scatter(X,Y)
        plt.plot(X,Y)
        plt.xlim(-1, 51)
        plt.ylim(-1, 51)
        fig.canvas.draw()
    
### we create and interactive graph to alter the spring coefficient and the power of gravity to see their effect on the particle.
    attributes = interactive(run_sim, k=(0.1,2,0.1), gy=(-10.0, -1.0, 0.5), continuous_update=True)
    attributes
