# HW 2 (2018311639 Yeo Hyunseung)

Built-in modules time, random, logging are used. Also, imported Maze class from src.maze.

    import time
    import random
    import logging
    from src.maze import Maze
    
## q_learning_RL

    class q_learning_RL(Solver):
    
initiallizing function:

    def __init__(self, maze, quiet_mode=False, neighbor_method="fancy"):
        logging.debug('Class Q learning ctor called')
        self.name = "Q learning"
        super().__init__(maze, neighbor_method, quiet_mode)  
    
    
The number of episode, alpha, gamma, epsilon is set as 1000, 0.1, 0.9, 0.1 each.

    def solve(self):
        maze = self.maze
        grid = self.maze.grid
        
        n_episode = 1000
        alpha = 0.1
        gamma = 0.9
        epsilon = 0.1


get_reward(): Given function to get reward by iteration

        def get_reward(k_curr, l_curr, k_next, l_next, maze):
            
            if grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                reward = -1
           
            elif (k_next, l_next) == maze.exit_coor:
                reward = 100
            else:
                reward = -0.1
            return reward



q_learning(maze, n_episode, alpha, gamma, epsilon): Q-value updating function

First, initialize q table all in zeros, and set maximum iteration as 400 times.

        def q_learning(maze, n_episode, alpha, gamma, epsilon):
            q_table = [[[0 for action in range(4)] for col in range(20)] for row in range(20)]         # Q_table for 20*20 states with 4 actions
            
            episode = 0
            max_iteration = 400

During each episode, the function will initialize current state as entry state. And make the actions list. The action 'up', 'down', 'left', 'right' will be on the index of 0, 1, 2, 3 of the list.

            while episode < n_episode:               
                k_curr, l_curr = self.maze.entry_coor
                k_next, l_next = k_curr, l_curr
                actions = ['up', 'down', 'left', 'right']
                
For each episode, initialize the iteration time as 0. The string action will indicate the action on each iteration.     

                iteration = 0
                while (k_curr, l_curr) != self.maze.exit_coor:                    
                    action = ''
                    
Neighbours is the list of possible next state in the maze. if the row or column of the current state is 0 or 19, it's action will be restricted in 2 or 3 directions.             

                    neighbours = list()
                    if not k_curr < 1:
                        neighbours.append((k_curr-1, l_curr))
                    else:
                        neighbours.append(None)
                    if not k_curr > 18:
                        neighbours.append((k_curr+1, l_curr))
                    else:
                        neighbours.append(None)
                    if not l_curr < 1:
                        neighbours.append((k_curr, l_curr-1))
                    else:
                        neighbours.append(None)
                    if not l_curr > 18:
                        neighbours.append((k_curr, l_curr+1))
                    else:
                        neighbours.append(None)

                   
Epsilon-greedy method:
Epsilon is set as 0.1. if the random number between 0~1 is less than 0.1(i.e., with the probability of 0.1), the action will be random. 

                    if random.random() < epsilon:
                        while action == '':
                            neighbourlist = list()
                            for neighbour in neighbours:
                                if not neighbour == None:
                                    neighbourlist.append(neighbour)

                            k_next, l_next = random.choice(neighbourlist)
                            
                            # print("next state: " + str(k_next) + ", " + str(l_next) )
                            if (k_next == k_curr - 1) and (l_next == l_curr):
                                action = 'up'
                            elif (k_next == k_curr + 1) and (l_next == l_curr):
                                action = 'down'
                            elif (k_next == k_curr) and (l_next == l_curr-1):
                                action = 'left'
                            elif (k_next == k_curr) and (l_next == l_curr+1):
                                action = 'right'
                                
Else, the action will be decided with  the highest q-value.

                    else:
                        actionlist = list()
                        for act in actions:
                            if actions.index(act) == q_table[k_curr][l_curr].index(max(q_table[k_curr][l_curr])):
                                actionlist.append(act)
                        #print(actionlist)
                        action = random.choice(actionlist)
                        if action == 'up' and neighbours[0]!= None:
                            k_next, l_next = neighbours[0]
                        elif action == 'down'and neighbours[1]!= None:
                            k_next, l_next = neighbours[1]
                        elif action == 'left'and neighbours[2]!= None:
                            k_next, l_next = neighbours[2]
                        elif action == 'right'and neighbours[3]!= None:
                            k_next, l_next = neighbours[3]

                                 

                    
                    
                    

Call the parameter of reward from get_reward and maximum q-value of next state.
                    
                    reward = get_reward(k_curr, l_curr, k_next, l_next, maze)        # reward                    
                    Max_q_value = max(q_table[k_next][l_next])                              # max Q(s', a')


The q-value is updated by Q updating rule: Q(s,a) = (1-alpha)* Q(s,a)+alpha* (r(s,a)-Qmax(s',a')

                    # updating Q-value
                    q_table[k_curr][l_curr][actions.index(action)] = round(((1-alpha)*q_table[k_curr][l_curr][actions.index(action)] + alpha*(reward+gamma*Max_q_value)), 10)
                    
                    
If there is no wall between current state and next state, the agent will move to next state. Else, it will stay on current state.

                    # move to next state
                    
                    if maze.grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                        print("Wall is on the path")
                    else:
                        k_curr = k_next
                        l_curr = l_next
                    
After the iteration, iteration number will be increased as 1. If next state is the goal state, maximum iteration will be adjusted as iteration(when the agent finally arrived goal state) + 10 to shorten the calculation. If it exceed the maximum iteration number, loop will break.

                    iteration = iteration + 1
                    if (k_next, l_next) == self.maze.exit_coor and not grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                        max_iteration = iteration +10
                    else:
                        max_iteration = 500
                    if iteration > max_iteration:
                        break;
                        
After the whole episode, episode number will be increased as 1. When all episodes are over, the function will return learned q_table.

                episode = episode +1     
            return q_table
                   
                   
q_learning_path is function finding the path by given q-table. The algorithm is similar with q_learning. First, initialize q table all in zeros, and set maximum iteration as 400 times. For each episode, initialize the iteration time as 0. The action will be decided by highest Q-value. If there is repeated highest q values, the action will be chosen as random among them.                                      
        
        def q_learning_path(maze, q_table):
            path = list()            
            k_curr, l_curr = self.maze.entry_coor
            k_next, l_next = k_curr, l_curr
            actions = ['up', 'down', 'left', 'right']                
            iteration = 0
            k_next, l_next = k_curr, l_curr
            
            while (k_curr, l_curr) != self.maze.exit_coor:
                action = ''
                neighbours = list()
                if not k_curr < 1:
                    neighbours.append((k_curr-1, l_curr))
                else:
                    neighbours.append(None)
                if not k_curr > 18:
                    neighbours.append((k_curr+1, l_curr))
                else:
                    neighbours.append(None)
                if not l_curr < 1:
                    neighbours.append((k_curr, l_curr-1))
                else:
                    neighbours.append(None)
                if not l_curr > 18:
                    neighbours.append((k_curr, l_curr+1))
                else:
                    neighbours.append(None)
 
  
                actionlist = list()
                for act in actions:
                    if actions.index(act) == q_table[k_curr][l_curr].index(max(q_table[k_curr][l_curr])):
                        actionlist.append(act)
                
                action = random.choice(actionlist)
                if action == 'up' and neighbours[0]!= None:
                    k_next, l_next = neighbours[0]
                elif action == 'down'and neighbours[1]!= None:
                    k_next, l_next = neighbours[1]
                elif action == 'left'and neighbours[2]!= None:
                    k_next, l_next = neighbours[2]
                elif action == 'right'and neighbours[3]!= None:
                    k_next, l_next = neighbours[3]


                reward = get_reward(k_curr, l_curr, k_next, l_next, maze)        # reward
                Max_q_value = max(q_table[k_next][l_next])                              # max Q(s', a')           
                q_table[k_curr][l_curr][actions.index(action)] = round(((1-alpha)*q_table[k_curr][l_curr][actions.index(action)] + alpha*(reward+gamma*Max_q_value)), 10)
                
                # move to next state            
                if maze.grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                    print("Wall is on the path")
                else:    
                    k_curr = k_next
                    l_curr = l_next
                    path.append(((k_curr, l_curr), False))  # Add coordinates to part of search path
                    
                iteration = iteration + 1
                if iteration > 400:
                    break
            return path

After the 400 times of iteration, the function will end and return the path.
            
At the end of the solve function, it will execute the q_learning and q_learning_path with input parameters and return path, finally.
        
        q_table = q_learning(maze, n_episode, alpha, gamma, epsilon)
        path = q_learning_path(maze, q_table)

        time_start = time.time()
        if not self.quiet_mode:
            print("Number of moves performed: {}".format(len(path)))
            print("Execution time for algorithm: {:.4f}".format(time.time() - time_start))
        return path
        
        
