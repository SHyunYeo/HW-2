# HW 2 (2018311639 Yeo Hyunseung)

Built-in modules csv, math, Counter are used. Also, matplotlib is used for plotting the learning curve at TASK 3.

    import csv
    import math
    from collections import Counter
    import matplotlib.pyplot as plt





## q_learning_RL

    class q_learning_RL(Solver):
    def __init__(self, maze, quiet_mode=False, neighbor_method="fancy"):
        logging.debug('Class Q learning ctor called')

        self.name = "Q learning"
        super().__init__(maze, neighbor_method, quiet_mode)  
    
    def solve(self):
        maze = self.maze
        grid = self.maze.grid
        
        n_episode = 10000
        alpha = 0.1
        gamma = 0.9
        epsilon = 0.1


        def get_reward(k_curr, l_curr, k_next, l_next, maze):
            
            if grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                reward = -1
           
            elif (k_next, l_next) == maze.exit_coor:
                reward = 100
            else:
                reward = -0.1
            return reward

        def q_learning(maze, n_episode, alpha, gamma, epsilon):
            q_table = [[[0 for action in range(4)] for col in range(20)] for row in range(20)]         # Q_table for 20*20 states with 4 actions
            
            episode = 0
            reached_goal = 0
            
            #learningcurve_y = list()
            
            max_iteration = 400

            while episode < n_episode:
                print("The agent reached goal "+ str(reached_goal) + " times")
                # initial state
                
                k_curr, l_curr = self.maze.entry_coor
                k_next, l_next = k_curr, l_curr
                actions = ['up', 'down', 'left', 'right']
                
                iteration = 0

                print('""""""""""""""""""""""""episode  ' + str(episode) + '"""""""""""""""""""""""""')
                
                print(q_table)
            
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

                    #for neighbour in neighbours:
                    #    if neighbour != None and grid[k_curr][l_curr].is_walls_between(maze.grid[neighbour[0]][neighbour[1]]):
                     #       print(actions[(neighbours.index(neighbour))]+" is wall")

                    
                    print('++++++++ Iteration '+str(iteration)+': '+"current state: " + str(k_curr) + ',' + str(l_curr) + '+++++++++')


                    #epsilon-greedy method
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
                        #print("action: random..... ")
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

                        #print("q values are "+str(q_table[k_curr][l_curr]))
                

                    
                    
                    


                    
                    reward = get_reward(k_curr, l_curr, k_next, l_next, maze)        # reward
                    
                    Max_q_value = max(q_table[k_next][l_next])                              # max Q(s', a')


                    print("q value: "+str(q_table[k_curr][l_curr]))
                    #print("reward: "+str(reward))
                    #print("Maximum Q value: "+str(Max_q_value))
                    
                    # updating Q-value
                    q_table[k_curr][l_curr][actions.index(action)] = round(((1-alpha)*q_table[k_curr][l_curr][actions.index(action)] + alpha*(reward+gamma*Max_q_value)), 10)
                    #print("updated q value: "+str(q_table[k_curr][l_curr][actions.index(action)]))
                    if (k_next, l_next) == self.maze.exit_coor and not grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                        reached_goal = reached_goal + 1
                        
                        print('=================================GGGGGGGGGGOOOOOOOOOOOAAAAAAAAAAALLLLLLLLLLLL==============================')
                        print(self.maze.exit_coor)
                    
                    # move to next state

                    
                    if maze.grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                        print("Wall is on the path")
                    else:
                        k_curr = k_next
                        l_curr = l_next
                    
                    #print('++++++++next state: ' + str(k_curr) + ',' + str(l_curr) + '+++++++++')
                    iteration = iteration + 1
                    
                    if iteration > max_iteration:
                        break;
                episode = episode +1
        
                           
            return q_table
                    
        def q_learning_path(maze, q_table):
            path = list()
            #print(q_table)
            # initial state
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


                
                                          

                
                print('++++++++ Iteration '+str(iteration)+': '+"current state: " + str(k_curr) + ',' + str(l_curr) + '+++++++++')


                
                
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

                #print("q values are "+str(q_table[k_curr][l_curr]))
                print(action)
                
                ########################### if current state is in path, add state in path as true ######################################## 

            
                reward = get_reward(k_curr, l_curr, k_next, l_next, maze)        # reward
            
                Max_q_value = max(q_table[k_next][l_next])                              # max Q(s', a')


            
                q_table[k_curr][l_curr][actions.index(action)] = round(((1-alpha)*q_table[k_curr][l_curr][actions.index(action)] + alpha*(reward+gamma*Max_q_value)), 10)
                #print("updated q value: "+str(q_table[k_curr][l_curr][actions.index(action)]))
                if (k_next, l_next) == self.maze.exit_coor and not grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                    print('=================================GGGGGGGGGGOOOOOOOOOOOAAAAAAAAAAALLLLLLLLLLLL==============================')
                    print(self.maze.exit_coor)
                # move to next state

            
                if maze.grid[k_curr][l_curr].is_walls_between(maze.grid[k_next][l_next]):
                    print("Wall is on the path")
                else:    
                    k_curr = k_next
                    l_curr = l_next
                    path.append(((k_curr, l_curr), False))  # Add coordinates to part of search path

        
            
                #print('++++++++next state: ' + str(k_curr) + ',' + str(l_curr) + '+++++++++')
                iteration = iteration + 1
                if iteration > 400:
                    break



            

            return path



            
    
        
        q_table = q_learning(maze, n_episode, alpha, gamma, epsilon)
        path = q_learning_path(maze, q_table)


        time_start = time.time()
        if not self.quiet_mode:
            print("Number of moves performed: {}".format(len(path)))
            print("Execution time for algorithm: {:.4f}".format(time.time() - time_start))
        
        
        return path
