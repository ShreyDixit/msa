# Multi-perturbation Shapley value Analysis (MSA)
<img align="left" src="images/Artboard%202.jpg" alt="msa logo" width="300"> 

**TLDR**: A Game theoretical approach for calculating the contribution of each element of a system (here network models of the brain) to a system-wide description of the system. The classic neuroscience example: How much each brain region is causally relevant to an arbitrary cognitive function. 

## Motivation & such:
MSA is developed by Keinan and colleagues back in 2004, the motivation for them was to have a causal picture of the system by lesioning its elements. The method itself is not new, if not the first, it was among one of the earliest ones used by neuroscientists to understand the brain. The reasoning is quite simple, let us study broken systems to see what's missing both from the brain and the behavior (or cognition) and assume that region was causally necessary for the emergence of that cognitive/behavioral state. What MSA does is to see this necessity as contribution. If the brain region is indeed the seed of this cognitive function (whatever this means) then its contribution should be very high while other regions will have near zero contribution. Having this in mind then we can see the whole scenario as a cooperative game in which a coalition of players work together and obtain some divisible outcome, then the question is quite the same. How to divide the outcome to the players in a "fair" way such that the most "important" player gets the biggest chunk. Shapley value is then that chunk! It is the result of a mathematically rigorous and axiomatic procedure that derives who should get how much from all possible combinations of coalitions and all ordering in which players can enter the game. Translating it to neuroscience, it derives a ranking of contributions from a dataset of all possible combinations of lesions. This means 2<sup>N</sup> lesions (assuming lesions are binary, either perturbed or not), which N is the number of brain regions. 

As you probably noticed this won't be feasible to calclulate as for example, it means a total number of 4,503,599,627,370,496 lesion combinations, assuming the brain is organized as Broadmann said, i.e., with 52 regions. So we estimate! For a more detailed description visit:

[Keinan, Alon, Claus C. Hilgetag, Isaac Meilijson, and Eytan Ruppin. 2004. “Causal Localization of Neural Function: The Shapley Value Method.” Neurocomputing 58-60 (June): 215–22.
](https://www.sciencedirect.com/science/article/abs/pii/S0925231204000426?via%3Dihub)

And

[Keinan, Alon, Ben Sandbank, Claus C. Hilgetag, Isaac Meilijson, and Eytan Ruppin. 2006. “Axiomatic Scalable Neurocontroller Analysis via the Shapley Value.” Artificial Life 12 (3): 333–52.](https://direct.mit.edu/artl/article/12/3/333/2530/Axiomatic-Scalable-Neurocontroller-Analysis-via)

## Installation:
Start with making a virtual environment and a Python version `3.9` (still not tested on how much backward-compatible it is). Then clone the repository (`git clone https://github.com/kuffmode/msa.git`, move to the `msa` folder (`cd msa`), and run `pip install .` In case the requirements weren't installed automatically you can also use the command `pip install -r requirements.txt` and then let me know so I can fix it!
## How it works:
Here you can see a schematic representation of how the algorithm works (interested in math instead? check the papers above). Briefly, all MSA needs from you is a list of players and a game function. The players can be your nodes, for example, brain regions or indices in a connectivity matrix, or links between them as tuples. It then shuffles them to produce N orderings in which they can join the game. This can end with repeating permutations if the set is small but that's fine don't worry! MSA then produces a "combination space" in which it produces all the combinations the player should form coalitions. Then it uses your game function and fills the contributions of those coalitions. The last step is to perform a Shapley integration and isolate each player's contribution in that given permutation. Repeating this for all permutations produces a contribution table (shapley table) and you'll get your shapley values by averaging over permutations so the end result is a value per element/player. To get a better grasp of how this works in code, check the minimal example in the examples folder.

<img src="images/Artboard%201.jpg" alt="msa unbiased sampling algorithm"> 

## How it works in Python:
I tried to make the package compact and easy-to-use but still there are a few things to keep in mind. Please take a look at the examples but just to give a flavor let's start working with the set ABCD as we have in the above picture.

Importing will be just: 
```
from core import msa, utils as ut
```
Then we define some elements and generate the permutation space:
```
nodes = ['A', 'B', 'C', 'D']
permutation_space = msa.make_permutation_space(n_permutations=1000, elements=nodes)
```
This results in a list of tuples, our permutation space that has 1000 permutations in it, here are the top 5 ones:
```
[('D', 'C', 'A', 'B'),
 ('A', 'D', 'C', 'B'),
 ('D', 'A', 'B', 'C'),
 ('D', 'B', 'C', 'A'),
 ('A', 'D', 'C', 'B')]
```
Then we use this to produce our combination space:
```
combination_space = msa.make_combination_space(permutation_space=permutation_space)
```
And a quick look of what's inside:
```
[frozenset({'D'}),
 frozenset(),
 frozenset({'C', 'D'}),
 frozenset({'A', 'C', 'D'}),
 frozenset({'A', 'B', 'C', 'D'}),
 frozenset({'A'}),
 frozenset({'A', 'D'}),
 frozenset({'A', 'B', 'D'}),
 frozenset({'B', 'D'}),
 frozenset({'B', 'C', 'D'}),
 frozenset({'C'}),
 frozenset({'A', 'B'}),
 frozenset({'A', 'B', 'C'}),
 frozenset({'A', 'C'}),
 frozenset({'B', 'C'}),
 frozenset({'B'})]
```
As you can see eventhough the permutation space has 1000 permutations, the combination space is exhausted because the total number of possible combinations is 2<sup>4</sup> or 16. Now here's the trick, we need to assign values to these combinations (coalitions) by keeping them intact while every other element is perturbed. In other words, the contribution of coalition `{'B', 'C'}` is isolated if we lesion `{'A', 'D'}` before playing the game. So what we do (and is not in the figure above) is to produce the "complement space" of the combination space:
```
complement_space = msa.make_complement_space(combination_space=combination_space, elements=nodes)
```
that is the difference of what's in the combination space in that coalition and what is not:

```
[('C', 'B', 'A'),
 ('C', 'D', 'B', 'A'),
 ('B', 'A'),
 ('B',),
 (),
 ('C', 'D', 'B'),
 ('C', 'B'),
 ('C',),
 ('C', 'A'),
 ('A',),
 ('D', 'B', 'A'),
 ('C', 'D'),
 ('D',),
 ('D', 'B'),
 ('D', 'A'),
 ('C', 'D', 'A')]
```
As you can see, for example when combination is `{'D'}` the corresponding complement is `('C', 'B', 'A')`. Note the difference in types, combination space is an `OrderedSet` of `frozenset`s so the Shapley value calculations are quicker while complement space is an `OrderedSet` of `Tuples` So handling it in your objective function is easier. Speaking of, let's make the worst objective function that just produces random values regardless of what's what (see the example `on ground-truth models.ipynb` for a more elaborate version.)[(see the example `on ground-truth models.ipynb` for a more elaborate version.)](https://github.com/kuffmode/msa/blob/main/examples/on%20ground-truth%20models.ipynb)
```
def rnd(complements):
	return np.random.randint(1, 10)
```
We'll next play the games and aquire the contributions as follows:
```
contributions, lesion_effects = msa.take_contributions(elements=nodes,
                                        combination_space=combination_space,
                                        complement_space=complement_space,
                                        objective_function=rnd)
```
Both `contributions` and `lesion_effects` are the same values just addressed differently. For example, if the contribution of coalition `{'B', 'C'}` is 5 points then you can also say the effect of lesioning coalition `{'A', 'D'}` is 5 points. This by itself is not that informative but if you know the contribution of the grand coalition (intact system) then you can claim that the effect of lesioning `{'A', 'D'}` is a drop of some performance from x to 5.

Lastly, you can calculate Shapley values like:
```
shapley_table = msa.make_shapley_values(contributions=contributions,permutation_space=permutation_space)
```
Which gives you a `pd.DataFrame` to work with.

## The Interface:
To make things easier, msa comes with an interface function:

```
shapley_table, contributions, lesion_effects = msa.interface(multiprocessing_method='joblib',
                                                             elements=regions,
                                                             n_permutations=1000,
                                                             objective_function=rnd,
                                                             n_parallel_games=-1,
                                                             random_seed=1)
```
For this one, all you have to do is to provide your elements, the objective function, and specify some parameters. For example, you can choose between two different multiprocessing toolboxes `joblib` and `ray` to distribute `msa.take_contributions` over `n_parallel_games`. Specifying a `random_seed` is encouraged for reproducibility but the default is `None`.
 
## TODO (Contribution):
- More estimation methods.
- Integrating `ray` with `ray cluster`.
- Providing built-in objective functions for common neural network libraries.

## Cite:
```
@misc{msa,
  author = {Fakhar, Kayson},
  title = {msa: A compact Python package for Multiperturbation Shapely value Analysis},
  year = {2021},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/kuffmode/msa}},
}
```
