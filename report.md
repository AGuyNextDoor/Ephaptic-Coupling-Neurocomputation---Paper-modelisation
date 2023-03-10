# Modelisation Report

## Implementation Choices

### Library

The implementation of the neuron was done with the library [Brian](https://brian2.readthedocs.io/), using their `NeuronGroup` implementation allowing the creation of a group of neurons, sharing common properties such as :

- the model's equations
- firing's threshold
- reset values after firing
- refractory time
- any other externe variables.

This library was chosen arbitrarily, not knowing many other modelization libraries in python listed.

### Modelisation Type

A choice was to be made between using `SpatialNeuron`s and `NeuronGroup`s.

#### SpatialNeuron's Pros and Cons

`SpatialNeuron` allowed for the individual programming of compartments, very well suited for this exercise because of the different behaviors of the dendrite and the soma, and how they interact in between them and with other neurons on the side. This class also allowed for more precise programming of certain dimensions such as the dendrite's exact length, diameter, and position relative to the soma, making this class very well suited for the design of a single neuron.

But this class lacks options and performance when scaling to a larger set of neurons. The connection options between neurons (synaptic or with equations) are very limited compared to the design of the `NeuronGroup` classes. After discussing on the forum of brian ([link here](https://brian.discourse.group/t/grouping-a-large-number-of-spatialneuron-ressource/928/1)), another problem was pointed out to me for performance scaling. Although it is technically possible to design the network with the `SpatialNeuron` class, the performance scaling is very bad and would have not permitted to experiment efficiently at a large scale (200*18 -> 3600 neurons) with a long period (~[0.1; 5] seconds).

#### NeuronGroup

The group of neurons was generated, implementing the neuron's dendritic and somatic relationship directly with the circuit equations, Hodgkin–Huxley's equations, and Maxwell's equations and how they influence the dendritic's and somatic's electric potential (respectively $v_dend$ and $v_soma$).

It was possible to give each neuron an $x$ and $y$ attribute, mimicking the grid position of each neuron and their respective distance.

Using the `Synapses` class to create a connexion between each neuron allowed the calculation of the electric field contributions of all $i \neq j$ neurons.

## Personal Critical Assessment

### Model's assumptions

1. As suggested by the paper, the firing of neurons happens spontaneously with the only contribution of the potential of the surrounding electrical field. The *NMDA*, *KDR* and Leaky gates are passive in the model.
2. It was assumed that neurons in the middle of the grid, had more electric field contribution than those closer to the sides. This was analyzed in visualization and used as a verification of the "correct behavior" of the neuron population. It was verified that neurons in the center did have more electrical field potential contribution than those closer to the edges.
3. The Stacking Factor ($SF$) used in maxwell's equation local potential contribution $$V_{(k,l)\_e\_z} = SF \times \frac{\rho}{4 \pi r} \Sigma_{(i,j)} ( \frac{I_{(i,j)\_\text{tran\_d}}}{r_{(i,j)\_d\rarr(k,l)\_z}} + \frac{I_{(i,j)\_\text{tran\_s}}}{r_{(i,j)\_s\rarr(k,l)\_z}})$$ was removed because the `Synapses` class does a more precise, neuron-by-neuron, sum of all the electrical fields.
4. To my knowledge and comprehension, the factor $A_d$ and $A_s$ in the transmembrane potential equations $$C_m \frac{d V_{m\_d}}{dt} = - I_{NMDA} - I_{KDR} - I_{L_d} - \frac{g_{c\_sd}}{A_d}(V_{m\_d} - V_{m\_s})$$ and $$C_m \frac{d V_{m\_s}}{dt} = - I_{L_s} - \frac{g_{c\_sd}}{A_s}(V_{m\_d} - V_{m\_s})$$ wasn't clearly explained. From the context of the transmembrane potential and when analytically studying the dimensions of the equation, both factors were identified as the Area of their respective parts (data that was given earlier in the paper with the diameter of the soma and dendrite).
5. The constant value $k_B$ was not found in the table 3 as mentionned by the paper. In an implementation of the gating function found in examples of [Brian's documentation](https://brian2.readthedocs.io/en/stable/examples/frompapers.Brunel_Wang_2001.html?highlight=persistent#sample-specific-persistent-activity) (for the value `I_NMDA_rec`), the value used for $k_B$ is $0.062$ and is therefor used in this implementation.

### What's missing or to be improved

1. The potential accumulation is pretty fast, I believe due to maxwell's equation contribution creating a snowball effect. The lack of a refractory period of this behavior creates a fast stabilization of the neurons. At this date, I am not sure how exactly how to impact this behavior or if it is a good hint.
2. The dendritic currents and gates here are considered passive and therefore it was chosen to not include the rise and decay period of those gates because of a lack of time and a certain uncertainty about the consequences of the "passive" adjective used. I believe they should indeed be implemented at some point with a derived behavior.
3. Not a lot of test was done playing with the different Magnesium concentrations, but it is a 'simple' variable modification (the variable `Mg`). The magnesium behavior could be naturally implemented in the equations. This could allow comparing certain results with the paper. This was not tested or implemented by a lack of time and a prioritization of tasks.

## What's next

- More tests and verifications to explain (and most probably improve) certain behaviors such as : *(priority 1, difficulty 3)*
  - Stabilization around -55 mV before firing (mostly because of the leaky and other current passive values)
  - Some repetition of large droppings of current (probably due to a factor that is not at the right dimension)
  - When changing the number of neurons, the behavior of the grid changes quite a bit. More tests to see how this stabilizes at a larger scale and the implication this could have.
- Testing the grid's behavior with the different magnesium concentrations. *(priority 3, difficulty 1)*
- More visualization can be created to have a better total understanding of individual neurons as well at the complete neuron population. *(priority 2, difficulty 2)*
- The [dendrify](https://dendrify.readthedocs.io/en/latest/index.html) package suggested in the forum's comments was not explored for this exercise but could provide implementation simplification and rigor for some equations' behaviors. *(priority 3, difficulty 4)*
- The dendrify package seems to have an option to access to GPU computation via CUDA. This could be a nice improvement to accelerate large scale simulation and debugging.

## Help and resources

- Apart from one question on the official forum of the package Brian ([linked here](https://brian.discourse.group/t/grouping-a-large-number-of-spatialneuron-ressource/928)), no other online help or tool was used apart from software and scientific documentation.

**Resources** :

- [Scientific Paper](https://pubmed.ncbi.nlm.nih.gov/30295923/)
- [Brian's package documentation](https://brian2.readthedocs.io/en/stable/)
  - In [PDF](https://brian2.readthedocs.io/_/downloads/en/stable/pdf/)
- [Brian's Forum](https://brian.discourse.group/)
- Note : The priority and difficulty ladder helps with priorization and works the following way :
  - Priority 1 is low importance and 5 high importance.
  - Difficulty 1 is low difficulty and 5 is high difficult.
