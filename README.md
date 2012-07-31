# clojure-encog

Clojure wrapper for the encog (v3) machine-learning framework .

-from the official encog website:
---------------------------------
"Encog is an open source Machine Learning framework for both Java and DotNet. Encog is primarily focused on neural networks and bot programming. It allows you to  create many common neural network forms, such as feedforward perceptrons, self organizing maps, Adaline, bidirectional associative memory, Elman, Jordan and Hopfield networks and offers a variety of training schemes."

-from me:
---------
Encog has been around for almost 5 years, and so can be considered fairly mature and optimised. Apart from neural-nets, version 3 introduced SVM and Bayesian classification. With this library, which is a thin wrapper around encog, you can construct and train many types of neural nets in less than 10 lines of pure Clojure code. The whole idea, from the start, was to expose the user as little as possible to the Java side of things, thus eliminating any potential sharp edges of a rather big librabry like encog. Hopefully I've done a good job...feel free to try it out, and more importantly, feel free to drop any comments/opinions/advice/critique etc etc...

P.S.: This is still work in progress. Nonetheless the neural nets and training methods are pretty much complete - what's left at this point is data-models, randomization and the bayesian stuff...aaaa also I'm pretty sure we need tests :) ...  


## Usage

The jar(s)?
-------------------

As usual, it lives on clojars. Just add:
``` clojure
[org.encog/encog-core "3.1.0"]   ;official encog 3.1 release 
[clojure-encog "0.4.0-SNAPSHOT"] ;my code
```
to your :dependencies and you 're good to go...


Quick demo:
-------------

Ok, most the networks are already functional so let's go ahead and make one. Let's assume that for some reason we need a feed-forward net with 32 input neurons, 1 output neuron (classification), and 2 hidden layers with 50 and 10 neurons respectively...We don't really care about the activation function at this point because we are not going to do anything useful with this particular network.

``` clojure
(def network      ;;def-ing it here for demo purposes
    (make-network {:input   32
                   :output  1
                   :hidden [50 10]} ;;2 hidden layers
                  (make-activationF :sigmoid) 
                  (make-pattern     :feed-forward)))
                  
;;this is actually the neural pattern I used for my final year project at uni!                  
```
...and voila! we get back the complete network initialised with random weights.

Most of the constructor-functions (make-something) accept keyword based arguments. For the full list of options refer to documentation or source code. Don't worry if you accidentaly pass in wrong parameters to a network e.g wrong activation function for a specific net-type. Each concrete implementation of the 'make-network' multi-method ignores arguments that are not settable by a particular neural pattern!

Of course, now that we have the network we need to train it...well, that's easy too!
first we are going to need some dummy data...

``` clojure
(let [xor-input [[0.0 0.0] [1.0 0.0] [0.0 0.1] [1.0 1.0]]
      xor-ideal [[0.0] [1.0] [1.0] [0.0]] 
      dataset   (make-data :basic-dataset xor-input xor-ideal)
      trainer   ((make-trainer :back-prop) network dataset)])

;;notice how 'make-trainer' returns a function which itself expects some argumets.
;;in this case we're using simple back-propagation as our training scheme of preference.
;;feed-forward nets can be used with a variety of activations/trainers.
```
as soon as you have that, training is simply a matter of:
``` clojure
(train trainer 0.01 500 [(RequiredImprovementStrategy. 5)])
;train expects a training-method , error tolerance, iteration limit, strategies (a possibly empty vector)
```

and that's it really!
after training finishes you can start using the network as normal. For more in depth instructions consider looking at the 2 examples found in the examples.clj ns. These include the classic xor example (trained with resilient-propagation) and the lunar lander example (trained with genetic algorithm) from the from encog wiki/books.

In general you should always remember:
- Most (if not all) of the constructor-functions (e.g. make-something) accept keywords for arguments. The documentation tells you
exactly what your options are. Some constructor-functions return other functions (closures) which then need to be called again with potentially extra arguments, in order to get the full object. 

- 'make-network' is a big multi-method that is responsible for looking at what type of neural pattern has been passed in and dispatching the appropriate method. This is the 'spine' of creating networks in clojure-encog.

- NeuroEvolution of Augmenting Topologies (NEAT) don't need to be initialised as seperate networks like all other networks do. Instead, we usually initialise a NEATPopulation which we then pass to NEATTraining via 
``` clojure
((make-trainer :neat) some-function-name true/false population)  ;OR
((make-trainer :neat) some-function-name true/false 2 1 1000)    ;if we want a brand new population with default parameters
```     

- Simple convenience macros do exist for evaluating quickly a trained network and also for implementing the CalculateScore class which is needed for doing GA or simulated-annealing training.

- Ideally, check the source when any 'strange' error occurs. You don't even have to go online - it's in the jar!

Other stuff...
----------------
Developed using Clojure 1.4 and leiningen2.
Should work with 1.3 but not lower than that!
It seems that the stdout problem persists in leiningen2 as well... (repl hangs unless aot and "lein2 run")


This is still work in progress...If you're going to do any serious ML job with it, be prepared to write some Java simply because not everything has been wrapped. The plan is not to have to write any Java code by version 1.0. 

## License

Copyright © 2012 Dimitrios Piliouras

Distributed under the Eclipse Public License, the same as Clojure.
