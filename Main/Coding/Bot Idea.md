I was planning on making a bot, then I had some medical complications, but I think it still might be a cool and pretty simple idea if anyone else wants to try it out.

under genetic algorithms (GAs, like NEAT or NSGA II), there are methods of training a network to encode the weights of ANNs through a calculation (its called indirect encoding, for ex HyperNEAT-LEO). This means that you can store a way to calculate all of the weights & biases in the network, with another smaller network.

specifically given the limited token count, its possible to encode pretty large ANNs into your chess bot, which even if it doesn't work the best, could be interesting to see.

anyways, random comments over, gl to everyone participating!

HyperNEAT paper for anyone interested
https://axon.cs.byu.edu/~dan/778/papers/NeuroEvolution/stanley3**.pdf
MSS, builds off of HyperNEAT
https://ntnuopen.ntnu.no/ntnu-xmlui/handle/11250/2656670
CMOEA, builds off of MSS, a little complicated but could work better
https://arxiv.org/abs/1807.03392

Source: https://discord.com/channels/1132289356011405342/1132353219889217708/1137944819415646298

Sure!

Couple definitions first:
          -GA stands for genetic algorithm, it's basically just a way of training network weights without gradient descent. Basically RL stuffs
          -NEAT is a GA
          -A ANN is a artificial neural network comprised of a single activation function (usually its like RELU, or sigmoid, or tanh, etc)
          -A CPPN is a neural network comprised of multiple activation functions (usually gaussian, sin, sigmoid, tanh, etc). Theres some interesting theory behind these its pretty cool

So, using NEAT, you evolve a ANN directly by mutating the ANN over and over. Basically, this leads to networks that are basically overly complicated, and incredibly dense. It has been scientifically shown that there are some other issues because of that, which lead to the development of HyperNEAT.

Using HyperNEAT, you use NEAT (or some other GA) to evolve a CPPN (mutate it over and over). The CPPN then is used to determine the weights between each connection. CPPN and ANN substrate below, hopefully this helps a lil:

Source: https://discord.com/channels/1132289356011405342/1132353219889217708/1138161265588576397
