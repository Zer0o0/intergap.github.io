---
layout: post
title: 使用MEGA软件构建系统发育进化树
tags: [sequence, toolkits, pipline]
thumbnail-img: /assets/img/mds/lifetree.png
---

系统发育进化树（phylogenetic tree），也叫系统进化树或者进化树，主要是利用树状分支图来表示个物种或基因/蛋白间的亲缘关系。MEGA（Molecular Evolutionary Genetics Analysis）软件是一款集序列比对、序列分析与系统进化树构建于一体的开源软件，具有分析效率高、操作简单和功能一体化等优点，适合少量序列数据的分析。

### MEGA构建进化树步骤

需求：给定一个蛋白质，已知蛋白序列和基因序列，找到同源蛋白并构建进化树。

1. 利用NCBI工具[blastp](https://blast.ncbi.nlm.nih.gov/Blast.cgi)搜索目标蛋白的同源蛋白;
2. 根据blast结果和背景知识筛选搜索的到的蛋白，具体为blast的E value、Percent Identity数值以及物种信息等；
3. 下载fasta文件，注意blast得到的序列是不完整的蛋白序列，可以根据Protein数据库ID，利用[Batch Entrez](https://www.ncbi.nlm.nih.gov/sites/batchentrez)工具批量下载序列文件；
4. 使用MEGA构建进化树：
4.1 导入下载的fasta文件（DATA模块）
4.2 使用ClustalW或者MUSCLE算法进行多序列比对，比对完成后可以对冗余序列进行裁剪对齐
4.3 点击Data按钮，完成Phylogenetic Analysis
4.4 计算最优建树模型（MODELS模块）
4.4 构建进化树（PHYLOGENY模块），可选择ML、NJ等方法，参数设置Bootstrap method对树进行检验，ML方法可选择4.4的最优模型
4.5 进化树的美化（如树的形状和标签字体字号等）和输出

**Bootstrap方法**会以原序列为蓝本随机重组生成新的序列，重复估算树模型。如果原序列计算得到的分枝在新Bootstrap中依然频繁出现，则该分枝的可信度高，分枝在Bootstrap中出现的频率就是表征分枝可信度的参数。在构建进化树的结果中，会出现Original Tree和Bootstrap Consensus Tree，其中  
Original Tree是步长检验构建的N次株树中的最优系统树。未经过多棵树合并，所以Original Tree上有计算得到的距离数据，可以精确地表征两个基因的亲缘远近，MEGA形成的Original Tree上也有频率参数，实际来自Bootstrap Consensus Tree的对应分枝；  
Bootstrap Consensus Tree 是很多次Bootstrap得到的平均结果，它不包含进化距离信息（在设置View时无法调用，也没有意义），分枝上的数字代表该分枝的频率参数，即经步长检验有百分之几的树具有这根树枝，反应了该树枝的可信度。另外，它的拓扑结构也可能与Original Tree有较大差异。

Ref: [Building Phylogenetic Trees from Molecular Data with MEGA](https://academic.oup.com/mbe/article/30/5/1229/992850?login=true)

---

### 算法比较

- 基于距离的算法

距离法先将比对后的序列转化为序列间两两成对的距离矩阵，而后以矩阵为基础计算出树分支的顺序和枝长。常见的距离算法有UPGMA (unweighted pair-group method with Arithmetic mean)、 FM (Fitch-Margoliash)、ME (Minimum Evolution)以及NJ (Neighbor-Joining)等。

UPGMA方法现已不常用；FM和ME方法计算速度较慢，不适用于物种数目较多的情况；NJ方法在建树中比较常用，优点是重建的树相对准确，假设少，计算速度快，只得到一棵树，缺点主要表现在将序列上的所有位点同等对待，且所分析序列的进化距离不能太大，所以NJ法适用于进化距离不大，信息位点少的短序列。

- 最大简约法，MP (Maximum parsimony)

与其他建树方法相比，MP方法无需引入处理核苷酸或者氨基酸替代时所必需的假设(替代模型)。同时，MP方法对于分析某些特殊的分子数据(如插入序列和插入／缺失)有用。在分析的序列位点上没有回复突变或平行突变，且被检验的序列位点数很大的时候，MP方法能够获得正确的(真实)系统树。但MP方法推导的树不是唯一的，在分析序列上存在较多的回复突变或平行突变，而被检验的序列位点数又比较少的时候，最大简约法可能会出现建树错误。故MP方法适用于序列残基差别小，具有近似变异率，包含信息位点比较多的长序列。

- 最大似然法，ML (Maximum likelihood)

ML方法对所有可能的系统发育树都计算似然函数，似然函数值最大的那棵树即为最可能的系统发育树。利用最大似然法来推断一组序列的系统发生树，需首先确定序列进化的模型，如Jukes—Cantor模型、Kimura二参数模型及一般二参数模型等。在进化模型选择合理的情况下，ML法是与进化事实吻合最好的建树算法，其缺点是计算强度非常大，极为耗时。

- 贝叶斯推断法 BI (Bayesian Inference)

近年来发展起来的一种新的利用贝叶斯演绎法预测种系发生史的系统进化分析方法，它既保留了最大似然法的基本原理，又引进了马尔科夫链的蒙特卡洛方法(markov chain monte carlo process)，来模拟演化树的较晚期可能性分布，并使计算时间大大缩短。  
贝叶斯法的优点在于：推导系统树、评估系统树的不确定性、检测选择作用、比较系统树、参考化石记录计算分歧时间和检测分子钟。贝叶斯法得到的系统进化树不需要利用自引导法进行检验，其后验概率直观地反映了系统进化树的可信程度，是一种系统进化分析的好方法，它既能根据分子进化的现有理论和各种模型用概率重建系统进化关系，又克服了最大似然法计算速度慢、不适用于大数据集样本的缺陷。贝叶斯法可以选择适当的模型来拟合数据，它和最大似然法相似，都是选定一个进化模型，然后通过程序搜索模型和序列数据一致的最优系统树。但二者基本的不同在于，最大似然法是以观察数据的最大概率来拟合系统树，贝叶斯法是通过系统树对数据及进化模型的最大拟合概率而得到系统树；最大似然法给出的是数据的概率，而贝叶斯法给出的是模型的概率；最大似然法搜索单一的最相似系统树，贝叶斯法得到的是具有大致相等似然的系统树集合。另外，通过贝叶斯法分析得到的结果很容易解释，系统树分支上的数值就表明了该分支的概率，而且通过贝叶斯法，我们可以利用复杂的碱基替代模型快速而有效地分析大的数据。

**总结一下：**
常用构树方法的比较甄选从上述我们可以了解到，重建系统发生树的方法有很多，也各有优缺点。因此在实际操作中，往往需要根据自己的研究需要联合使用不同的构树方法以获得最佳分析结果。比较以上几种主要的构树方法，一般情况下，若有合适的分子进化模型可供选择，用最大似然法构树获得的结果较好；对于近缘物种序列，通常情况下使用最大简约法；而对于远缘物种序列，一般使用邻接法或最大似然法。对于相似度很低的序列，邻接法往往出现长枝吸引(branch attraction)现象，有时严重干扰进化树的构建。对于各种方法重建进化树的准确性，Hall (2005)认为贝叶斯法最好，其次是最大似然法，然后是最大简约法。其实如果序列的相似性较高，各种方法都会得到不错的结果，模型间的差别也不大。邻接法和最大似然法是需要选择模型的。蛋白质序列和DNA序列的模型选择是不同的。蛋白质序列的构树模型一般选择Poissoncorrection(泊松修正)，而核酸序列的构树模型一般选择Kimura2-parameter (Kimura一2参数)。如果对各种模型的理解并不深入，最好不要使用其他复杂的模型。参数的设置推荐使用缺省的参数。在重建进化树过程中，均需选择bootstrap进行树的检验。一般bootstrap的值>70，则认为重建的进化树较为可靠。如果bootstrap的值太低，则有可能进化树的拓扑结构有错误，进化树是不可靠的。因此，一般推荐用两种以上不同的方法构建进化树，如果所得到的进化树类似，且bootstrap值总体较高，则得到的结果较为可靠。通常情况下，只要选择了合适的方法和模型，构出的树均是有意义的，研究者可根据自己研究的需要选择最佳的树进行分析。

### 建树基本步骤

- 数据准备

目前，构建生命之树常用的数据包括形态数据和分子数据。形态数据主要通过对形态性状编码来获取；分子数据主要通过公共数据库GenBank下载或实验获取。选择合适的DNA片段对系统发育关系重建至关重要。如果所选基因的进化速率太慢，提供的系统发育信息不足, 系统发育关系可能得不到很好的解决；如果所选基因的进化速率太快，正确的系统发育信息常常会被大量的非同源相似信号淹没。

- 序列拼接

为了提高序列的准确性, 往往需要对所测正反向序列进行拼接和校正, 常用的拼接软件有Contig Express、Geneious、Sequencher等。

- 序列比对

为了保证序列的同源性和所得系统发育关系的可靠性，需要对原始序列进行比对和校正。自动比对序列的软件包括Clustal 、MAFFT、MUSCLE等; 手工校对序列的软件有BioEdit 、Se-Al 、Geneious等。

- 校正有争议的位点

保守区选择是系统发育分析过程中一个重要的步骤，对于信息位点足够多的建树序列，该步骤更是必不可少。常用的软件为Gblock。

- 模型选择

在建树之前，通常要对矩阵的最佳模型进行评估。常用的软件有ModelTest 、MrModelTest、jModelTest等。ModelTest包含56种DNA替代模型，MrModelTest包含24种MrBayes中可用的模型, 而jModelTest包含88种模型。熟悉各建树模型的优点与不足，根据数据特点有针对性地利用不同的模型，可以减少建树过程中出现的偏差。

- 选择建树方法

如上所述。

- 树的显示与美化

常用的编辑和显示树图的软件有TreeView、FigTree、MEGA、ITOL、R包（ggtree、APE）等。

- 进化树枝长的含义

不同的建树方法得到的进化枝长也不一样，  
NJ：表示遗传距离；  
MP：性状状态变换的替换数；  
ML/BI：该分枝上的相对进化数量（遗传变异量。

- 时间树？？

[TimeTree of Life](http://www.timetree.org/)

### 参考
[系统发育树构建算法和软件](https://zhuanlan.zhihu.com/p/101144418)