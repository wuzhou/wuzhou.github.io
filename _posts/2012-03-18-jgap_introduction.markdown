---
author: wuzhou
comments: false
date: 2012-03-18 09:03:12+00:00
layout: post
slug: jgap_introduction
title: 使用JGAP实现遗传算法（GA）编程入门
wordpress_id: 73
categories:
- 编程
tags:
- Java
- JGAP
---

这是一篇翻译过来的文章。

原文：[http://jcraane.blogspot.com/2009/02/introduction-to-genetic-algorithms-with.html](http://jcraane.blogspot.com/2009/02/introduction-to-genetic-algorithms-with.html)

出于兴趣的原因，我对遗传算法（GA）进行了自学。当我第一次知道JGAP这个项目时，我便对遗传算法产生了兴趣。根据JGAP项目网站上的介绍，JGAP是一个在Java环境下可以用来实现遗传算法（Genetic Algorithms）和遗传编程（Genetic Programming）的组件。对于菜鸟来说，想一下子对于遗传算法的所有概念有一个全面的了解是一件不容易的事。但是，在使用JGAP之前，最好还是要了解遗传算法的相关概念。这篇文章主要通过实例介绍了使用JGAP实现遗传算法的方法。在下一篇文章中将展示如何使用遗传编程（GP）解决问题。<!-- more -->

什么是遗传算法？John R. Koza给出了如下的定义：


<blockquote>The genetic algorithm is a probabilistic search algorithm that iteratively transforms a set (called a population) of mathematical objects (typically fixed length binary character strings), each with an associated fitness value, into a new population of offspring objects using the Darwinian principle of natural selection and using operations that are patterned after naturally occurring genetic operations, such as crossover (sexual recombination) and mutation.

(维基百科的定义：对于一个最优化问题，一定数量的候选解（称为个体）的抽象表示（称为染色体）的种群向更好的解进化。传统上，解用二进制表示（即0和1的串），但也可以用其他表示方法。进化从完全随机个体的种群开始，之后一代一代发生。在每一代中，整个种群的适应度被评价，从当前种群中随机地选择多个个体（基于它们的适应度），通过自然选择和突变产生新的生命种群，该种群在算法的下一次迭代中成为当前种群.)</blockquote>


在遗传算法中，可行解被称之为染色体（chromosome）。每个染色体中含有固定数量的基因（gene）。每个基因代表了可行解的不同部分。在遗传算法的进化（evolution）过程中，不同的可行解之间会进行交叉（crossover）和变异（mutation），从而产生更好的可行解。进化都是在一定数量的可行解之中完成的。这个一定数量的可行解叫做基因型(genotype) ，在基因型中含有一定数量的染色体。在进化过程中，使用自然选择（natural selection）原理决定哪些染色体更有可能遗传到下一代。自然选择的标准就叫做适应度（fitness）。具有更好适应度的染色体，将更有可能被遗传到下一代。在JGAP中适应度的确定取决于用户如何去实现fitness function。

通过学习JGAP提供的文档，用户自己可以就可以使用GAP来实现上述的概念了。不过，通过下面的例子你将会更好地了解JGAP是如何工作的。下面首先来介绍我们将要试图解决的问题。通过对例子的介绍，你会对上述的一些关于遗传算法的概念有清晰的了解。

一个搬家公司使用定制好的搬运箱（这些箱子是固定大小的，不同类型的箱子容积不同）来搬运东西，这些箱子将被放在货车里进行运输。为节省运输成本，要尽量减少使用的货车数量。问题陈诉：给定一定数量的不同容量的箱子，求最佳的运输方案，使得需要的货车数量最少？下面的例子将使用JGAP实现遗传算法来解决这个问题。

首先，我们可以考虑下面这种情况：有5个箱子，体积分别为1、4、2、2、2（立方米），而货车的容积是4立方米，如果我们不改变箱子的排列顺序，直接按这种顺序进行运输，那么货车的运输情况是：
Van   Boxes          Space wasted
Van1 Box1            3
Van2 Box4            0
Van3 Box2, Box2  0
Van4 Box2            2
适应度：（3+2）× 4 = 20。参考 fitness function 部分对在这个问题中适应度函数的解释。

在上面这种方案中需要4辆货车。但是，当我们使用箱子的总体积除以每辆汽车的容积来计算所需的货车数量，最佳的货车数量是11 / 4 = 2.75。应为车的数量不可能是小数，所以最佳的货车数量为3。最佳的运输安排为：1、2、2、2、4。在这个运输安排的基础上会有下面的分析结果：
Van   Boxes            Space wasted
Van1 Box1, Box 2   1
Van2 Box2, Box 2   0
Van3 Box4              0
适应度： 1 × 3 = 3.

在实现解决方案之前，还有几个准备步骤必须要完成。这些步骤对于使用遗传算法解决问题有很重要的意义。

1.定义在特定问题范畴下遗传算法的表述。把要放进货车的箱子使用一个数组来表示。遗传算法的目标就是要找出在这个数组所表示的这些箱子的最佳运输顺序。染色体表示可行解并含有固定长度的基因。在这个例子中每个可行解含有一系列的索引，这些索引代表了数组中的箱子。为了表示这些索引，我们可以使用IntegerGene。正如之前提到的，每个基因表示了可行解的一部分。在这个例子中，在可行解中的基因数量和箱子的数量一样。通过遗传算法这些基因会被优化为代表最佳运输方案的顺序。具体来说，如果有50个箱子那么染色体将含有50个基因，每个基因的初始值都代表了数组中的箱子，在这个例子中基因的初始值应该在0到49之间。

2.确定适应度函数。适应度函数将决定确定可行解的优良程度。在这个问题的情景下，需要的货车越少，浪费的空间越小，解决方案越优。

3.确定运行时需要的参数。在这个案例中需要的种群大小为50，总进化次数为5000次。所以，基因型含有50个染色体，并进行5000次遗传。这些参数要根据不同实验和问题进行选择。

4.确定终止条件。当进化了5000次或者已经达到了最优的使用货车辆数。这个最佳辆数可以用箱子的体积总和除以货车容积并取整得到（以为不存在小数辆货车）。


## 初始化


Box类含有volume（体积）属性。在这个例子中我们创造125个体积在0.25到3.00立方米的箱子。这些箱子将会被存在数组中。下面的代码展示了是如何创建这些箱子的：

    
    Random r = new Random(seed);
    this.boxes = new Box[125];
    for (int i = 0; i < 125; i++) {
    Box box = new Box(0.25 + (r.nextDouble() * 2.75));
    box.setId(i);
    this.boxes[i] = box;
    }


在开始配置JGAP之前我们首先要实现fitness function（适应度函数）。因为适应度函数确定哪些种群更有可能遗传到下一代，所以适应度函数是遗传算法中最终要的一部分。这个问题中的适应度函数可以这样实现：

    
    package nl.jamiecraane.mover;
    import org.jgap.FitnessFunction;
    import org.jgap.IChromosome;
    /**
    * Fitness function for the Mover example. See this
    * {@link #evaluate(IChromosome)} for the actual fitness function.
    */
    public class MoverFitnessFunction extends FitnessFunction {
    private Box[] boxes;
    private double vanCapacity;
    public void setVanCapacity(double vanCapacity) {
    this.vanCapacity = vanCapacity;
    }
    public void setBoxes(Box[] boxes) {
    this.boxes = boxes;
    }
    /**
    * Fitness function. A lower value value means the difference between
    the
    * total volume of boxes in a van is small, which is better. This means
    a
    * more optimal distribution of boxes in the vans. The number of vans
    needed
    * is multiplied by the size difference as more vans are more expensive.
    */
    @Override
    protected double evaluate(IChromosome a_subject) {
    double wastedVolume = 0.0D;
    double sizeInVan = 0.0D;
    int numberOfVansNeeded = 1;
    for (int i = 0; i < boxes.length; i++) {
    int index = (Integer) a_subject.getGene(i).getAllele();
    if ((sizeInVan + this.boxes[index].getVolume()) <=
    vanCapacity) {
    sizeInVan += this.boxes[index].getVolume();
    } else {
    // Compute the difference
    numberOfVansNeeded++;
    wastedVolume += Math.abs(vanCapacity - sizeInVan);
    // Make sure we put the box which did not fit in this van in the next van
    sizeInVan = this.boxes[index].getVolume();
    }
    }
    // Take into account the number of vans needed. More vans
    //produce a higher fitness value.
    return wastedVolume * numberOfVansNeeded;
    }
    }


在上面的适应度函数将会遍历可行解中的全部基因（每个基因代表了数组中箱子的序号），然后计算在这个可行解所表示的运输安排下需要的车的辆数。适应度是基于每辆货车中所浪费的空间，最后适应度是用总的浪费空间的体积乘以所需的货车数得到。这样的话需要的货车越多适应度就越差。在最开始提出的简单例子中，第一个可行解的适应度是20，第二个经过优化过的可行解的适应度是3。另外一个需要解释的概念是等位基因（Allele）。在上面的代码中就使用到了getAllele方法。等位基因只是基因的值的另一种说法，因为所有IntegerGene中每个基因的值都是Integer类型。

下面是开始设置JGAP的时候了：

    
    private Genotype configureJGAP() throws InvalidConfigurationException {
    Configuration gaConf = new DefaultConfiguration();
    // Here we specify a fitness evaluator where lower values means a better
    //fitness
    Configuration.resetProperty(Configuration.PROPERTY_FITEVAL_INST);
    gaConf.setFitnessEvaluator(new DeltaFitnessEvaluator());
    // Only use the swapping operator. Other operations makes no sense here
    // and the size of the chromosome must remain constant
    gaConf.getGeneticOperators().clear();
    SwappingMutationOperator swapper = new SwappingMutationOperator(gaConf);
    gaConf.addGeneticOperator(swapper);
    // We are only interested in the most fittest individual
    gaConf.setPreservFittestIndividual(true);
    gaConf.setKeepPopulationSizeConstant(false);
    gaConf.setPopulationSize(50);
    // The number of chromosomes is the number of boxes we have. Every
    //chromosome represents one box.
    int chromeSize = this.boxes.length;
    Genotype genotype;
    // Setup the structure with which to evolve the solution of the problem.
    // An IntegerGene is used. This gene represents the index of a box in the
    //boxes array.
    IChromosome sampleChromosome = new Chromosome(gaConf, new
    IntegerGene(gaConf), chromeSize);
    gaConf.setSampleChromosome(sampleChromosome);
    // Setup the fitness function
    MoverFitnessFunction fitnessFunction = new MoverFitnessFunction();
    fitnessFunction.setBoxes(this.boxes);
    fitnessFunction.setVanCapacity(VOLUME_OF_VANS);
    gaConf.setFitnessFunction(fitnessFunction);
    // Because the IntegerGenes are initialized randomly, it is neccesary to
    //set the values to the index. Values range from 0..boxes.length
    genotype = Genotype.randomInitialGenotype(gaConf);
    List chromosomes = genotype.getPopulation().getChromosomes();
    for (Object chromosome : chromosomes) {
    IChromosome chrom = (IChromosome) chromosome;
    for (int j = 0; j < chrom.size(); j++) {
    Gene gene = chrom.getGene(j);
    gene.setAllele(j);
    }
    }
    return genotype;
    }


通过上面的代码就配置好了JGAP。JGAP提供的Javadoc很值得细细阅读。在这个种群中含有50个染色体，每个染色体中的基因数量与数组中盒子的数量一样。因为在这个例子中适应度数值越小越好，所以我们使用了theDeltaFitnessEvaluator。

下面开始种群进化。种群进化的次数是5000次。当达到最佳的优化方案是进化也会停止。下面的代码展示了如何进化解决方案：

    
    private void evolve(Genotype a_genotype) {
    int optimalNumberOfVans = (int) Math.ceil(this.totalVolumeOfBoxes /
    VOLUME_OF_VANS);
    LOG.info("The optimal number of vans needed is [" + optimalNumberOfVans +
    "]");
    double previousFittest =
    a_genotype.getFittestChromosome().getFitnessValue();
    numberOfVansNeeded = Integer.MAX_VALUE;
    for (int i = 0; i < NUMBER_OF_EVOLUTIONS; i++) {
    if (i % 250 == 0) {
    LOG.info("Number of evolutions [" + i + "]");
    }
    a_genotype.evolve();
    double fittness = a_genotype.getFittestChromosome().getFitnessValue();
    int vansNeeded =
    this.numberOfVansNeeded(a_genotype.getFittestChromosome().getGenes()).size(
    );
    if (fittness < previousFittest && vansNeeded < numberOfVansNeeded) {
    this.printSolution(a_genotype.getFittestChromosome());
    previousFittest = fittness;
    numberOfVansNeeded = vansNeeded;
    }
    // No more optimal solutions
    if (numberOfVansNeeded == optimalNumberOfVans) {
    break;
    }
    }
    IChromosome fittest = a_genotype.getFittestChromosome();
    List<Van> vans = numberOfVansNeeded(fittest.getGenes());
    printVans(vans);
    this.printSolution(fittest);
    }


因为我们在JGAP设置中把preserveFittest属性设为了true，所以我们可以使用getFittestChromosome()方法得到最优的染色体也就是最优解。最优染色体中的125个基因多代表的数字中的箱子，这些基因表示了以怎样的顺序来运输盒子。具体的进化过程都是由JGAP来完成的。适应度函数决定了那些染色体更有可能在进入下一代种群。这样不断循环直到达到最优解或者近似最优解。适应度函数的重要性就表现在他决定了如何选择染色体。下是这个例子的运行结果：

    
    The total volume of the [125] boxes is [210.25989987666645] cubic metres.
    The optimal number of vans needed is [49]
    Number of evolutions [0]
    Fitness value [4123.992085987977]
    The total number of vans needed is [63]
    Fitness value [3458.197333300851]
    The total number of vans needed is [61]
    Fitness value [3138.2899569572887]
    The total number of vans needed is [60]
    Fitness value [2865.5105375433063]
    The total number of vans needed is [59]
    Fitness value [2562.282028584251]
    The total number of vans needed is [58]
    Fitness value [2267.7135196251966]
    The total number of vans needed is [57]
    Fitness value [1981.8050106661412]
    The total number of vans needed is [56]
    Fitness value [1704.5565017070858]
    The total number of vans needed is [55]
    Fitness value [1479.769464870246]
    The total number of vans needed is [54]
    Number of evolutions [250]
    Fitness value [1215.9278601031112]
    The total number of vans needed is [53]
    Fitness value [1002.6487336510297]
    The total number of vans needed is [52]
    Number of evolutions [500]
    Fitness value [774.5352329142294]
    The total number of vans needed is [51]
    Number of evolutions [750]
    Number of evolutions [1000]
    Fitness value [535.1696373214758]
    The total number of vans needed is [50]
    Number of evolutions [1250]
    Number of evolutions [1500]
    Number of evolutions [1750]
    Number of evolutions [2000]
    Number of evolutions [2250]
    Fitness value [307.8713731063958]
    The total number of vans needed is [49]
    Van [1] has contents with a total volume of [4.204196540671411] and
    contains the following boxes:
    Box:0, volume [2.2510465109421443] cubic metres.
    Box:117, volume [1.9531500297292665] cubic metres.
    Van [2] has contents with a total volume of [4.185233471369987] and
    contains the following boxes:
    Box:17, volume [1.0595047801111055] cubic metres.
    Box:110, volume [0.5031165156303853] cubic metres.
    Box:26, volume [2.6226121756284955] cubic metres.
    Van [3] has contents with a total volume of [4.312990612147265] and
    contains the following boxes:
    Box:91, volume [1.8897555340430203] cubic metres.
    Box:6, volume [2.423235078104245] cubic metres.
    ...Further output omitted


从上面的输出可以看出：进化到达2250代到2500代时达到了最佳优化方案。程序同时输出了最佳解决方案的具体内容。完整的源代码下载地址：http://code.google.com/p/jc-examples/ 。对应的文件是ga-moving-example-1.0.jar.


## 结论


遗传算法是一个令人兴奋的技术，但是在开始使用它之前需要花点力气掌握它的相关概念。这篇文章简单的介绍了遗传算法的相关概念和使用JGAP实现遗传算法。在下一篇文章中将会使用实例介绍如何进行遗传编程。

遗传算法中的银弹法则。下面的一些问题可以考虑使用遗传算法来解决：



	
  * 问题的解很难找到，但是问题的解的优劣是可以判断的。

	
  * 问题解的搜索范围很大、很复杂或者难以理解。

	
  * 问题的近似最佳接是可以被接受的。

	
  * 问题没有可用的数学分析方法。


虽然我不是这个领域的专家，但是欢迎提问，我将会尽力解答。


## 资源：


[1] Field guide to genetic programming
[2] JGAP
[3] [http://en.wikipedia.org/wiki/Genetic_algorithm](http://en.wikipedia.org/wiki/Genetic_algorithm)
[5] [http://www.genetic-programming.org/](http://www.genetic-programming.org/)
[6] [http://code.google.com/p/jc-examples/](http://code.google.com/p/jc-examples/)
[7] [http://en.wikipedia.org/wiki/Knapsack_problem](http://code.google.com/p/jc-examples/)
[8] [http://en.wikipedia.org/wiki/Bin_packing_problem](http://en.wikipedia.org/wiki/Bin_packing_problem)

PDF 版下载：[http://115.com/u/9738515](http://115.com/u/9738515)
