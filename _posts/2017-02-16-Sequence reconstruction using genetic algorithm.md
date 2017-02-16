---
title: "Sequence reconstruction using genetic algorithm"
layout: post
---
# Sequence reconstruction using genetic algorithm

## Sequencing by hybridization

Sequencing by hybridization(SBH)는 지금의 NGS가 보편화되기 이전에 많이 사용되었던 sequencing 방법이다. 한
생물의 genome을 처음부터 끝까지 한번에 읽는 방법은 현재로써는 불가능하므로, restriction enzyme이나 shotgun
approach 등을 통해 genome을 여러 개의 DNA fragment로 나눈 뒤 fragment 각각을 sequencing하고,
assemble함으로써 간접적으로 genome 전체를 sequencing하게 된다. 이하는 sequencing by hybridization을
통해 조각난 DNA fragment 하나를 어떻게 sequencing하는지에 대한 이야기이다.

SBH를 간략히 설명하면 다음과 같다. 먼저 길이 l을 갖는 가능한 모든 oligonucleotide probe(총 $$4^l$$개)를
microarray에 심는다. 이런 oligonucleotide library를 만들기 굉장히 복잡할 것 같지만, linear number of
steps 안에 제작이 완료된다고 한다(이 방법에 대해서도 알아보면 흥미로울듯!). 여튼 이렇게 oligonucleotide library를
만들게 되면 이제 우리는 DNA fragment 안의 길이 l인 subsequence들을 감지할 수 있게 된다. DNA fragment를
fluorescent marker로 labeling하고 microarray에 hybridization시키면 DNA fragment의
subsequence에 해당하는 oligonucleotide에서만 형광이 발현되기 때문이다. Array상에서 이들 밝은 점들의 좌표를 알아내어서
대응하는 oligonucleotide들을 얻어낼 수 있고, 이것을 <b>spectrum</b>이라고 한다.

우리의 목표는 짧은 oligonucleotide들이 모여 있는 spectrum으로부터 DNA fragment를 복원해내는 것이다.

## Errors in spectrum

위의 방법으로 얻어낸 spectrum 상에는 Positive error와 Negative error의 크게 두 가지의 오류가 존재한다.

Positive error는 <b>DNA fragment에 포함되어 있지 않은 oligonucelotide가 나타나는</b> error로서,
noncomplementary한 oligonucleotide가 DNA fragment에 결합했을 때 발생한다. 또한 fluorescent
image 자체의 noise 때문에 발생하기도 한다.

Negative error는 <b>DNA fragment에 포함되어 있지만, spectrum에 나타나지 않는</b> error로서, DNA
fragment 상에 같은 subsequence가 중복되어 나타나거나, pH등의 조건이 잘 맞지 않아 완벽한 hybridization이
이루어지지 않았을 때 발생한다.

여기서는 positive error와 negative error를 모두 포함하는 erroneous spectrum을 가지고 DNA
fragment를 복원해볼 것이다.

## Genetic algorithm for sequence reconstruction

이 문제를 genetic algorithm으로 풀기 위하여, 몇 가지를 정의해야 한다.

### 해의 표현

Adjacency-based encoding을 사용한다. 즉, 임의의 해는 spectrum index의 permutation이며 i번째 위치에
index j가 있다는 것은, spectrum 상의 i번째 oligonucleotide 다음에 j번째 oligonucelotide가 뒤따른다는
것이다. 예를 들어, "ACTCTGG"라는 DNA fragment에서 아래와 같이 erroneous spectrum과 해가 주어졌을 때,

<p>
<center>{ACT, CAA, CTG, TCT, TGG, TTG}</center>
<center>4, 1, 5, 3, 6, 2</center>
</p>

해는 1번째 oligonucleotide 다음에 4번째가 오고, 즉 ACT 다음에 TCT가 오고, 2번째 다음에는 1번째, 즉 CAA 다음에
ACT가 오고...
라는 의미가 된다. 이렇게 완성된 cycle은 {ACT, TCT, CTG, TGG, TTG, CAA, ACT} 이다.

물론 해를 만들다 보면 길이가 spectrum의 개수보다 작은 cycle(subcycle)이 존재할 수 있다. 편의상 DNA fragment상에
tandem repeat은 없다고 가정하므로, 여기서는 이러한 해들은 '적합하지 않은 해'라고 생각하자.

### Fitness (적합도)

표현된 해의 각 위치에서 시작하는 subpath를 찾는다. 단, subpath의 길이는 원래 DNA fragment보다는 길면 안된다. 이 중
가장 많은 oligonucleotide를 포함하는 subpath가 포함하는 oligonucelotide의 개수가 적합도가 된다. 예를 들어,
위의 예에서 6개의 subpath는

<p>
<center>{ACT, TCT, CTG, TGG} = ACTCTGG</center>
<center>{TCT, CTG, TGG} = TCTGG</center>
<center>{CTG, TGG, TTG} = CTGTTG</center>
<center>{TGG, TTG} = TGGTTG</center>
<center>{TTG, CAA} = TTGCAA</center>
<center>{CAA, ACT, TCT} = CAACTCT</center>
</p>

가 되며, 적합도는 {ACT, TCT, CTG, TGG}에 4개의 oligonucleotide가 포함되어 있고 이것이 최대이므로 4이다.

### Selection (선택)

유전 알고리즘에서는 해의 population에서 적당한 해 두개를 '선택'하여 자식 해를 만들게 된다. 이 때, 좋은 해일수록 더 많이 선택되어
많은 자식 해를 만드는 것이 다음 세대에서의 전반적인 적합도를 올리는 방법이라 생각할 수 있다. 따라서 적당한 선택 방법을 사용하는 것이
중요하다. 여기서는 <i>stochastic remainder sampling without replacement</i>선택 방법을 사용하였다.
자세한 설명은 생략한다.

### Crossover (교차)

부모 해 두개가 선택되면, 부모 해를 어느 정도 반영하지만 새로운 특징을 갖는 자손이 태어나도록 하는 것이 다음 세대의 다양성을 높이는 데
좋다. 따라서 적당한 교차 방법을 사용하게 된다. 여기서는 <i>greedy crossover</i> 방법을 사용한다.

Greedy crossover 방법을 간략히 설명하면 다음과 같다.

가장 먼저 첫 번째 oligonucleotide는 spectrum 상에서 random하게 선택한다.

다음으로, 0.2의 확률로 나머지 spectrum 상에서 가장 '좋은' oligonucelotide를 두 번째로 한다. 여기서 가장 좋다는 것은
첫 번째 oligonucleotide와 가장 많이 겹친다는 것을 말한다.

나머지 0.8의 확률로 부모 해를 들여다본다. 부모 해에서 첫 번째 oligonucleotide 다음에 오는 두 oligonucleotide 중
더 '좋은' 것을 택한다. 이 때, subcycle을 만들지 않는 놈으로 골라야 한다. 만약 두 경우 모두 subcycle을 만든다면, 남아
있는 spectrum 상에서 임의로 선택한다.

이 과정을 반복하여 해의 크기가 spectrum의 크기와 같아지면, 두 부모로부터 하나의 자손을 만든 것이다!

### Initial population

Initial population은 uniform distribution에서 random하게 선택하되, subcycle이 없는 해들로 고른다.

## Implementation

### Preprocessing

먼저, sequence를 받아서 (erroneous) spectrum을 만들어주는 함수를 작성하자. generate_spectrum() 함수는
positive error rate과 negative error rate을 argument로 받아서 error를 발생시킨다.


{% highlight python %}
import random
import numpy as np
{% endhighlight %}


{% highlight python %}
def pop_random_element(lst):
    i = random.randrange(0, len(lst))
    return lst.pop(i)
{% endhighlight %}


{% highlight python %}
def generate_spectrum(seq, oligonucleotideLength, positiveErrorRate=0.1, negativeErrorRate=0.1):
    idealSpectrum = [seq[i:i+oligonucleotideLength] for i in range(len(seq) - oligonucleotideLength + 1)]
    erroneousSpectrum = []
    
    numCorrectSpectrum = int(len(idealSpectrum) * (1 - negativeErrorRate))
    numPositiveErrorSpectrum = int(len(idealSpectrum) * positiveErrorRate)

    _idealSpectrum = idealSpectrum[:]
    for _ in range(numCorrectSpectrum):
        erroneousSpectrum.append(pop_random_element(_idealSpectrum))
    
    for _ in range(numPositiveErrorSpectrum):
        randomSpectrum = generate_random_spectrum(oligonucleotideLength, idealSpectrum)
        erroneousSpectrum.append(randomSpectrum)
        
    # mess up spectrum
    random.shuffle(erroneousSpectrum)
        
    return erroneousSpectrum

def generate_random_spectrum(oligonucleotideLength, idealSpectrum):
    randomSpectrum = random_spectrum(oligonucleotideLength)
    while randomSpectrum in idealSpectrum:
        randomSpectrum = generate_random_spectrum(oligonucleotideLength)
    
    return randomSpectrum
    
def random_spectrum(oligonucleotideLength):    
    return "".join([["A", "T", "G", "C"][random.randrange(0, 4)] for _ in range(oligonucleotideLength)])
{% endhighlight %}


{% highlight python %}
SEQUENCE = "ACTCAGGTACGCTATTCTATGGTCGAGCGTCC"
OLIGONUCLEOTIDE_LENGTH=10
MAX_LENGTH=len(SEQUENCE)
spectrum = generate_spectrum(SEQUENCE, oligonucleotideLength=OLIGONUCLEOTIDE_LENGTH, positiveErrorRate=0.2, negativeErrorRate=0.2)

spectrum[:5]
{% endhighlight %}




    ['TACGCTATTC', 'TCGAGCGTCC', 'TCAGGTACGC', 'AGGTACGCTA', 'ATGTGGCTTT']



성능을 위해 약간의 preprocessing을 거친다. OVERLAP은 두 oligonucelotide 간의 overlap을 미리 계산해 둔
matrix이며, index는 oligonucleotide를 정수로 변환하기 위한 dictionary이다.


{% highlight python %}
def preprocess(spectrum):
    overlap = preprocess_overlap(spectrum)
    index = preprocess_index(spectrum)
    
    return overlap, index

def preprocess_overlap(spectrum):
    OVERLAP = [[-1] * len(spectrum) for _ in range(len(spectrum))]
    def get_overlap(predecessor, successor):
        oligonucleotideLength = len(predecessor)
        overlap = oligonucleotideLength - 1

        for i in reversed(range(oligonucleotideLength - 1)):
            if predecessor[-overlap:] != successor[:overlap]:
                overlap -= 1
            else:
                break

        return overlap

    for i in range(len(spectrum)):
        for j in range(len(spectrum)):
            OVERLAP[i][j] = get_overlap(spectrum[i], spectrum[j])
    
    return OVERLAP

def preprocess_index(spectrum):
    return dict(zip(spectrum, range(len(spectrum))))
{% endhighlight %}

Wrapper를 만든다.


{% highlight python %}
def index_of(oligonucleotide):
    return INDEX[oligonucleotide]

def get_overlap(predecessor, successor):    
    return OVERLAP[index_of(predecessor)][index_of(successor)]
{% endhighlight %}

### Fitness

Fitness function을 구현해보자. 위의 설명 그대로 구현하면 아래와 같다.


{% highlight python %}
def fitness(solution):
    maxFitness = 0
    cycle = generate_cycle(solution)
    for i in range(len(cycle)):
        fit, subpath = evaluate_subpath_starting_from(cycle, i)
        maxFitness = max(maxFitness, fit)
    
    return maxFitness

def generate_cycle(solution):
    cycle = []
    i = 0
    
    while True:
        cycle.append(spectrum[i])
        i = solution[i]
        if i == 0:
            return cycle

def evaluate_subpath_starting_from(cycle, start):
    curr = start
    subpath = cycle[curr]
    score = 1
    length = len(subpath)
    
    prev = curr
    curr = (curr + 1) % len(cycle)
    
    while True:
        # compute overlap with accumulated subpath and next oligonucleotide
        overlap = get_overlap(cycle[prev], cycle[curr])
        
        # if the subpath can be extended, extend it!
        extendedLength = length + (OLIGONUCLEOTIDE_LENGTH - overlap)
        if extendedLength <= MAX_LENGTH:
            score += 1
            length = extendedLength
            subpath += cycle[curr][overlap:]
            
            prev = curr
            curr = (curr + 1) % len(cycle)
        else:
            break
    
    return score, subpath
{% endhighlight %}

### Selection

Selection은 stochastic remainder sampling without replacement를 이용한다. 구현은 아래와 같다.


{% highlight python %}
def stochastic_remainder_sampling_without_replacement_selection(population):
    matingPool = []
    normalizedFitnesses = get_normalized_fitnesses(population)
    fill_mating_pool_with_integer_part(matingPool, normalizedFitnesses)
    fill_mating_pool_with_fractional_part(matingPool, normalizedFitnesses)
    
    matingPool = [population[i] for i in matingPool]
    return matingPool

def get_normalized_fitnesses(population):
    fitnesses = [fitness(solution) for solution in population]
    meanFitness = sum(fitnesses) / len(fitnesses)
    
    normalizedFitnesses = [fitness / meanFitness for fitness in fitnesses]
    return normalizedFitnesses

def fill_mating_pool_with_integer_part(matingPool, normalizedFitnesses):
    for i, fitness in enumerate(normalizedFitnesses):
        matingPool.extend([i] * int(fitness))

def fill_mating_pool_with_fractional_part(matingPool, normalizedFitnesses):
    fractionalParts = [fitness - int(fitness) for fitness in normalizedFitnesses]
    for i, fractionalPart in enumerate(fractionalParts):
        if biased_coin_toss_successed(fractionalPart):
            matingPool.append(i)

def biased_coin_toss_successed(fractionalPart):
    return random.random() < fractionalPart
{% endhighlight %}

### Crossover

Crossover는 greedy crossover를 사용한다. 구현은 아래와 같다.


{% highlight python %}
def get_child_population(population):
    matingPool = stochastic_remainder_sampling_without_replacement_selection(population)
    
    childPopulation = []
    while len(childPopulation) < len(population):
        parents = get_random_pair(matingPool)
        child = greedy_crossover(parents)
        
        childPopulation.append(child)
    
    return childPopulation

def get_best_fitness_and_solution(population):
    bestFitness, bestSolution = max([(fitness(solution), solution) for solution in population], key=lambda x: x[0])
    
    return bestFitness, bestSolution

def greedy_crossover(parents, p=0.1):
    remainingSpectrum = spectrum[:]
    chromosomeLength = len(parents[0])
    child = [-1] * chromosomeLength
    
    firstOligonucleotide = pop_random_element(remainingSpectrum)
    predecessor = firstOligonucleotide  # randomly choose first oligonucleotide
    for _ in range(chromosomeLength - 1):
        dice = random.random()
        if dice < p:
            bestSuccessor = find_best_successor_from_remaining_spectrum(child, predecessor, remainingSpectrum)
        else:
            bestSuccessor = find_best_successor_from_parents(child, predecessor, parents, remainingSpectrum)
        child[index_of(predecessor)] = index_of(bestSuccessor)
        predecessor = bestSuccessor
    
    child[child.index(-1)] = index_of(firstOligonucleotide)
    
    return child

def get_random_pair(population):
    while True:
        i1 = random.randrange(0, len(population))
        i2 = random.randrange(0, len(population))
        if i1 != i2:
            return population[i1], population[i2]
    
def has_subcycle(chromosome):
    chromosomeLength = len(chromosome)
    for i in range(chromosomeLength):
        visited = [False] * chromosomeLength        
        curr = i
        while curr != -1:
            if visited[curr] and all(visited):
                return False
            elif visited[curr]:
                return True
            
            visited[curr] = True
            curr = chromosome[curr]
            
    return False

def find_best_successor_from_remaining_spectrum(child, predecessor, remainingSpectrum):
    bestIndex, bestSuccessor = max([(i, successor) for i, successor in enumerate(remainingSpectrum)],
                                  key=lambda x: get_overlap(predecessor, x[1]))
    
    remainingSpectrum.pop(bestIndex)
    return bestSuccessor

def find_best_successor_from_parents(child, predecessor, parents, remainingSpectrum):
    successors = [spectrum[parent[index_of(predecessor)]] for parent in parents]
    bestSuccessor, bestOverlap = None, -1
    
    for successor in successors:
        tempChild = child[:]
        tempChild[index_of(predecessor)] = index_of(successor)
        if has_subcycle(tempChild) or successor not in remainingSpectrum:
            continue
            
        overlap = get_overlap(predecessor, successor)
        if overlap > bestOverlap:
            bestSuccessor = successor
            bestOverlap = overlap
    
    if not bestSuccessor:
        bestSuccessor = pop_random_element(remainingSpectrum)
    else:
        remainingSpectrum.pop(remainingSpectrum.index(bestSuccessor))
    
    return bestSuccessor
{% endhighlight %}

### Initial population

Initial population의 generation은 간단하다.


{% highlight python %}
def generate_initial_population(populationSize, solutionSize):
    population = []
    for _ in range(populationSize):
        solution = np.random.permutation(range(solutionSize))
        while not is_feasible_solution(solution):
            solution = np.random.permutation(range(solutionSize))
        
        population.append(solution)
    
    return population

def is_feasible_solution(solution):
    visited = [False] * len(spectrum)
    i = 0
    
    while True:
        visited[i] = True
        i = solution[i]
        if i == 0:
            break
    
    return all(visited)
{% endhighlight %}

### Run


{% highlight python %}
from tqdm import tqdm
{% endhighlight %}


{% highlight python %}
def run_algorithm(populationSize, solutionSize, iteration):
    bestFitness, bestSolution = -1, None
    population = generate_initial_population(populationSize=populationSize, solutionSize=solutionSize)
    
    for _ in range(iteration):
        fit, solution = get_best_fitness_and_solution(population)
        if fit > bestFitness:
            bestFitness, bestSolution = fit, solution
        population = get_child_population(population)
    
    return bestFitness, bestSolution

def get_best_subpath(solution):
    cycle = generate_cycle(solution)
    maxFitness, bestSubpath = max([evaluate_subpath_starting_from(cycle, i) for i in range(len(cycle))], key=lambda x: x[0])
    return bestSubpath
{% endhighlight %}


{% highlight python %}
# parameters
SEQUENCE = "TCCCTGAAGTTGGGACAAAATTGAACAAACAAGATGAGTTTGGTGCTTTGGAAAGTGTGAAAGCTGCTAGTGAACTATATTCTCCTTTATCAGGAGAAGTAACTGAAAT"
OLIGONUCLEOTIDE_LENGTH = 10
POSITIVE_ERROR_RATE = 0.2
NEGATIVE_ERROR_RATE = 0.2
MAX_LENGTH=len(SEQUENCE)

# generate spectrum
spectrum = generate_spectrum(SEQUENCE, \
                             oligonucleotideLength=OLIGONUCLEOTIDE_LENGTH, \
                             positiveErrorRate=POSITIVE_ERROR_RATE, \
                             negativeErrorRate=NEGATIVE_ERROR_RATE)

print("MAX_LENGTH : %d, SPECTRUM SIZE: %d" % (MAX_LENGTH, len(set(spectrum))))

OVERLAP, INDEX = preprocess(spectrum)

bestFitness, bestSolution = run_algorithm(populationSize=50, solutionSize=len(spectrum), iteration=20)

optimalFitness = int((len(SEQUENCE) - OLIGONUCLEOTIDE_LENGTH + 1) * (1 - NEGATIVE_ERROR_RATE))
print("Best solution with %d oligonucelotides results in %s (Optimal %d)" % (bestFitness, get_best_subpath(bestSolution), optimalFitness))
{% endhighlight %}

    MAX_LENGTH : 109, SPECTRUM SIZE: 100
    Best solution with 80 oligonucelotides results in CCCTGAAGTTGGGACAAAATTGAACAAACAAGATGAGTTTGGTGCTTTGGAAAGTGTGAAAGCTGCTAGTGAACTATATTCTCCTTTATCAGGAGAAGTAACTGAAAT (Optimal 80)
    

길이 109의 sequence를 잘 복원해냈다! 다음에는 이 알고리즘을 가지고 실제 sequence를 복원하는
