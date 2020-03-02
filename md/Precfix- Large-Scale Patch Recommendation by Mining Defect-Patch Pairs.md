# Precfix: Large-Scale Patch Recommendation by Mining Defect-Patch Pairs

## ABSTRACT

Patch recommendation is the process of identifying errors in software systems and suggesting suitable fixes for them. Patch recommendation can significantly improve developer productivity by reducing both the debugging and repairing time. Existing techniques usually rely on complete test suites and detailed debugging reports, which are often absent in practical industrial settings. In this paper, we propose Precfix, a pragmatic approach targeting large-scale industrial codebase and making recommendations based on previously observed debugging activities. Precfix collects defect-patch pairs from development histories, performs clustering, and extracts generic reusable patching patterns as recommendations. We conducted experimental study on an industrial codebase with 10K projects involving diverse defect patterns. We managed to extract 3K templates of defect-patch pairs, which have been successfully applied to the entire codebase. Our approach is able to make recommendations within milliseconds and achieves a false positive rate of 22% confirmed by manual review. The majority (10/12) of the interviewed developers appreciated Precfix, which has been rolled out to Alibaba to support various critical businesses.

补丁推荐是指识别软件系统中的错误并建议适当的补丁的过程。通过减少调试和修复时间，补丁推荐可以显著提高开发人员的工作效率。现有的技术通常依赖于完整的测试套件和详细的调试报告，而这些在实际的工业环境中通常是不存在的。在本文中，我们提出Precfix，这是一种针对大型工业代码库的实用方法，并根据之前观察到的调试活动提出建议。Precfix从开发历史中收集缺陷-补丁对，执行集群，并提取通用的可重用补丁模式作为建议。我们对一个包含不同缺陷模式的10K项目的工业代码库进行了实验研究。我们成功地提取了缺陷-补丁对的3K模板，并成功地应用于整个代码库。我们的方法能够在几毫秒内给出建议，并通过人工检查获得22%的误阳性率。大多数(10/12)受访开发者对Precfix表示赞赏，该公司已经向阿里巴巴推出了Precfix，以支持各种关键业务。

## 1 INTRODUCTION

Patch recommendation is the process of identifying errors in software systems and suggesting suitable fixes. Recent studies show that on average 49.9% of software developers’ time has been spent in debugging and about half of the development costs are associated with debugging and patching [5, 11, 49]. Automated patch recommendation can significantly reduce developers’ debugging efforts and the overall development costs, improving software quality and system reliability.

[
[5] Tom Britton, Lisa Jeng, Graham Carver, Paul Cheak, and Tomer Katzenellenbogen. 2013. Reversible Debugging Software. Technical Report. Judge Business School, University of Cambridge. 
[11] Luca Gazzola, Daniela Micucci, and Leonardo Mariani. 2019. Automatic Software Repair: A Survey. Transactions on Software Engineering 45, 1 (2019), 34–67.
[49] Undo Software. 2014. Increasing Software Development Productivity with Reversible Debugging.
]

补丁推荐是识别软件系统中的错误并建议适当修复的过程。最近的研究表明，软件开发人员平均49.9%的时间花在了调试上，大约一半的开发成本与调试和打补丁有关[5,11,49]。自动补丁推荐可以显著减少开发人员的调试工作和整体开发成本，提高软件质量和系统可靠性。

Recommending patches automatically is a challenging task, especially for large-scale industrial codebase. Many state-of-the-art techniques from the literature make assumptions on the existence of auxiliary development artifacts such as complete test suites and detailed issue tracking and debugging reports, which may not be readily available in the day-to-day development environment. To better understand the specific challenges in applying existing techniques in the development environment of Alibaba, we investigated the development practices and the entire codebase at Alibaba, ended up extracting a sample benchmark set which includes over 10,000 software projects, spanning over 15 million commits and 30 million files. We have made the following observations through the study.

自动推荐补丁是一个具有挑战性的任务，特别是对于大规模的工业代码库。许多来自文献的最新技术对辅助开发工件的存在做出了假设，例如完整的测试套件和详细的问题跟踪和调试报告，这些在日常的开发环境中可能并不容易获得。为了更好地理解在阿里巴巴的开发环境中应用现有技术所面临的具体挑战，我们调查了阿里巴巴的开发实践和整个代码库，最终提取了一个样本基准集，其中包括超过10,000个软件项目，超过1,500万份提交和3,000万份文件。我们通过研究得出了以下观察结果。

First, the project codebase often has insufficient documentation and manual labeling of defects and patches is hardly possible, which makes accurate patch mining and generation difficult. For example, recent studies proposed approaches based on clone detection [36], patch mining [17], information retrieval [59], and machine learning [6, 26] for fault localization and repair, which naturally require a large amount of labeled data. On the other hand, existing methods for automatic defect and patch identification suffer from inaccuracy. For instance, the SZZ algorithm [48] only achieves less than 25% accuracy on our benchmark set, which is mainly due to the inaccurate/missing bug reports and log messages.

[
[36] Mathieu Nayrolles and Abdelwahab Hamou-Lhadj. 2018. CLEVER: Combining Code Metrics with Clone Detection for Just-In-Time Fault Prevention and Resolution in Large Industrial Projects. In International Conference on Mining Software Repositories. 153–164.
[17] Jiajun Jiang, Yingfei Xiong, Hongyu Zhang, Qing Gao, and Xiangqun Chen. 2018. Shaping Program Repair Space with Existing Patches and Similar Code. In International Symposium on Software Testing and Analysis. 298–309.
[59] Jian Zhou, Hongyu Zhang, and David Lo. 2012. Where Should the Bugs be Fixed? More Accurate Information Retrieval-Based Bug Localization Based on Bug Reports. In International Conference on Software Engineering. 14–24.
[6] Zimin Chen, Steve Kommrusch, Michele Tufano, Louis-Noël Pouchet, Denys Poshyvanyk, and Martin Monperrus. 2018. SequenceR: Sequence-to-Sequence Learning for End-to-End Program Repair. arXiv:cs.SE/1901.01808
[26] Jian Li, Pinjia He, Jieming Zhu, and Michael R. Lyu. 2017. Software Defect Prediction via Convolutional Neural Network. In International Conference on Software Quality, Reliability and Security. 318–328. 
[48] Jacek Śliwerski, Thomas Zimmermann, and Andreas Zeller. 2005. When do Changes Induce Fixes?. In International Conference on Mining Software Repositories. 1–5.
]

首先，项目的代码库通常没有足够的文档，并且很难手工标记缺陷和补丁，这使得精确的补丁挖掘和生成变得非常困难。例如，最近的研究提出了基于克隆检测[36]、patch挖掘[17]、信息检索[59]和机器学习[6,26]的故障定位和修复方法，这些方法自然需要大量的标记数据。另一方面，现有的缺陷自动识别和补丁识别方法也存在不准确的问题。例如，SZZ算法[48]在我们的基准集上只能达到不到25%的准确率，这主要是由于错误报告和日志消息不准确/缺失造成的。

Second, the patch recommendation process has to be highly responsive in suggesting patches, in order to be useful in the everyday development routine. However, many existing fault localization and patch generation techniques require dynamic execution of test suites. For example, the spectrum-based [1, 18] and mutationbased [35, 40] fault localization techniques both assume strong test cases and utilize test execution results to identify defects. The generate-and-validate approaches [7, 22, 32] for patch generation search for candidate patches and validate their correctness using test suites. The problem with these techniques is that the test suites in our benchmark set may not be complete and the test execution often takes significant amount of time, making responsive online patch recommendation impossible.

[
[1] Rui Abreu, Peter Zoeteweij, and Arjan JC Van Gemund. 2007. On the Accuracy of Spectrum-Based Fault Localization. In Testing: Academic and Industrial Conference Practice and Research Techniques-MUTATION. 89–98. 
[18] James A Jones and Mary Jean Harrold. 2005. Empirical Evaluation of the Tarantula Automatic Fault-Localization Technique. In International Conference on Automated Software Engineering. 273–282.
[35] Seokhyeon Moon, Yunho Kim, Moonzoo Kim, and Shin Yoo. 2014. Ask the Mutants: Mutating Faulty Programs for Fault Localization. In International Conference on Software Testing, Verification and Validation. 153–162.
[40] Mike Papadakis and Yves Le Traon. 2015. Metallaxis-FL: Mutation-Based Fault Localization. Software Testing, Verification and Reliability 25, 5-7 (2015), 605–628. 
[7] Valentin Dallmeier, Andreas Zeller, and Bertrand Meyer. 2009. Generating Fixes From Object Behavior Anomalies. In International Conference on Automated Software Engineering. 550–554.
[22] Dongsun Kim, Jaechang Nam, Jaewoo Song, and Sunghun Kim. 2013. Automatic Patch Generation Learned from Human-Written Patches. In International Conference on Software Engineering. 802–811. 
[32] Fan Long and Martin Rinard. 2016. Automatic Patch Generation by Learning Correct Code. In Symposium on Principles of Programming Languages. 298–312.
]

其次，补丁推荐过程必须在建议补丁时具有高度的响应性，以便在日常开发例程中发挥作用。然而，许多现有的故障定位和补丁生成技术需要测试套件的动态执行。例如，基于谱的[1,18]和基于突变的[35,40]故障定位技术都假设了强大的测试用例，并利用测试执行结果来识别缺陷。生成和验证补丁的方法[7,22,32]搜索候选补丁并使用测试套件验证它们的正确性。这些技术的问题是，我们的基准测试集中的测试套件可能不完整，测试执行通常需要大量时间，这使得响应性在线补丁推荐变得不可能。

Finally, many fault localization and patch generation techniques focus on specific domains such as concurrency [30, 31], HTML content generation [46], and memory leaks [10]. Yet, our benchmark set covers a variety of business scenarios from diverse application domains, including e-commerce, logistics, finance, cloud computing, etc. These domain-specific techniques may work well in their targeted cases, but they are often not generalizable to the diverse applications from the company codebase.

[
[30] Haopeng Liu, Yuxi Chen, and Shan Lu. 2016. Understanding and Generating High Quality Patches for Concurrency Bugs. In Symposium on Foundations of Software Engineering. 715–726.
[31] Peng Liu, Omer Tripp, and Charles Zhang. 2014. Grail: Context-Aware Fixing of Concurrency Bugs. In Symposium on Foundations of Software Engineering. 318–329.
[46] Hesam Samimi, Max Schafer, Shay Artzi, Todd Millstein, Frank Tip, and Laurie Hendren. 2012. Automated Repair of HTML Generation Errors in PHP Applications using String Constraint Solving. In International Conference on Software Engineering. 277–287.
[10] Qing Gao, Yingfei Xiong, Yaqing Mi, Lu Zhang, Weikun Yang, Zhaoping Zhou, Bing Xie, and Hong Mei. 2015. Safe Memory-Leak Fixing For C Programs. In International Conference on Software Engineering. 459–470.
]

最后，许多故障定位和补丁生成技术都集中在特定的领域，如并发性[30,31]、HTML内容生成[46]和内存泄漏[10]。然而，我们的基准集涵盖了来自不同应用领域的各种业务场景，包括电子商务、物流、金融、云计算等。这些领域特定的技术可能在它们的目标案例中工作得很好，但是它们通常不能推广到来自公司代码库的各种应用程序。

The above mentioned characteristics of the codebase and development environment make the adoption of existing techniques unsuitable. In this paper, we propose a pragmatic patch recommendation approach Precfix, with customized improvements in terms of both the precision and efficiency when applied on large-scale industrial codebase. First, Precfix does not rely on labeled defects or patches, which are difficult to obtain in practice. Instead, we automatically mine a large number of defect-patch pairs from historical changes. To improve the accuracy of the mining results, we introduce optimizations which take into account the characteristics of typical developer behaviors in committing bug fixing changes. We also allow developers in the loop to provide feedback on the quality of the recommended patches, and the feedback is used to improve the precision of future recommendations. Second, since the defects and patches are mined from the entire company codebase and we use generic features when processing and clustering them, Precfix is generally applicable to all company projects written in different languages, handling different business logic, and deployed on different platforms. Finally, Precfix consists of an offline patch discovery phase and an online patch recommendation phase. The online phase is designed to be extremely responsive and can finish recommending patches within milliseconds in practice. We found that being scalable and efficient is extremely important for patch recommendation tools to be integrated into day-to-day interactive and repetitive development tasks, such as code reviewing.

上述代码库和开发环境的特点使得采用现有技术变得不合适。在本文中，我们提出了一种实用的补丁推荐方法Precfix，当应用于大规模的工业代码库时，在精度和效率方面进行了定制改进。首先，Precfix不依赖于标记的缺陷或补丁，这些在实践中很难获得。相反，我们会自动地从历史变更中挖掘出大量的缺陷-补丁对。为了提高挖掘结果的准确性，我们引入了一些优化，这些优化考虑了提交bug修复更改时典型开发人员行为的特征。我们还允许开发人员在循环中对推荐的补丁的质量提供反馈，这些反馈用于提高将来推荐的精度。其次，由于缺陷和补丁是从整个公司的代码库中挖掘出来的，并且我们在处理和集群它们时使用通用特性，所以Precfix通常适用于用不同语言编写的所有公司项目，处理不同的业务逻辑，并部署在不同的平台上。最后，Precfix包括一个离线补丁发现阶段和一个在线补丁推荐阶段。在线阶段的设计是非常敏感的，在实践中可以在几毫秒内完成推荐补丁。我们发现，对于将补丁推荐工具集成到日常交互和重复的开发任务(如代码审查)来说，可伸缩性和效率是非常重要的。

Precfix has been implemented and deployed as an internal web service in Alibaba. It is also integrated as a part of the code review process and provides patch recommendations whenever developers commit new changes to the codebase. We evaluated the effectiveness and efficiency of Precfix, and demonstrate its usefulness on a large-scale industrial benchmark with over 10,000 projects, spanning over more than 15 million commits and 30 million files.

Precfix已经在阿里巴巴内部实现并部署为一项内部web服务。它还集成为代码审查过程的一部分，并在开发人员提交对代码库的新更改时提供补丁建议。我们评估了Precfix的有效性和效率，并在超过10,000个项目的大型工业基准测试中展示了它的有效性，这些项目涉及超过1,500万次提交和3,000万个文件。

Contributions.We make the following contributions in this paper. 
- We propose Precfix— a semi-automated patch recommendation tool for large scale industrial codebase. 
- Precfix implements customized optimizations in defect-patch pair mining and clustering, which help improve the accuracy of patch recommendation over existing techniques. 
- Precfix managed to extract 3K defect templates from 10K projects. Our approach is able to make recommendations within milliseconds and achieves a false positive rate of 22% confirmed by manual review. 
- We conducted a small-scale user study and the majority (10/12) of the interviewed developers appreciated Precfix, which has been rolled out to Alibaba to support various critical businesses.

贡献。我们在本文中做出了以下贡献。

- 我们建议Precfix -一个半自动的补丁推荐工具，为大规模的工业代码库。
- Precfix在缺陷-补丁对挖掘和聚类中实现了自定义优化，这有助于提高与现有技术相比的补丁推荐的准确性。
- Precfix设法从10K项目中提取3K个缺陷模板。我们的方法能够在几毫秒内给出建议，并通过人工检查获得22%的误阳性率。
- 我们进行了一项小规模的用户研究，大多数(10/12)受访开发者对Precfix表示赞赏。Precfix已经在阿里巴巴推出，以支持各种关键业务。

## 2 RELATEDWORK

The techniques presented in this paper intersect with different areas of research. In this section, we compare Precfix with fault localization, automated patch generation, and patch recommendation.

本文介绍的技术与不同的研究领域相交叉。在本节中，我们将Precfix与故障定位、自动补丁生成和补丁推荐进行比较。

### 2.1 Fault Localization

Fault localization [41, 51, 54, 60] is the activity of identifying the locations of faults in a program. Many different types of fault localization techniques have been proposed. Spectrum-based fault localization [1, 18] utilizes test coverage information to pinpoint faulty program or statistical techniques. For example, Tarantula [18] uses a homonym ranking metric to calculate the suspiciousness values of statements, which are calculated according to the frequency of the statements in passing and failing test cases. Mutation-based fault localization [35, 40] mutates a program and runs its test cases, using the test results to locate faults. For example, Metallaxis [40] generates a set of mutants for each statement, assigns each mutant a suspiciousness score, and aggregates the scores to yield the suspiciousness of the statement. Other faults localization techniques identify locations of faults in some alternative ways, including dynamic program dependency analysis [2, 45], stack trace analysis [53, 55], conditional expressions mutation analysis [58], information retrieval [59], and version history analysis [23, 44].

[
[41] Spencer Pearson, Jose Campos, Rene Just, Gordon Fraser, Rui Abreu, Michael D. Ernst, Deric Pang, and Benjamin Keller. 2017. Evaluating and Improving Fault Localization. In International Conference on Software Engineering. 609–620. 
[51] Shaowei Wang, David Lo, Lingxiao Jiang, Lucia, and Hoong Chuin Lau. 2011. Search-Based Fault Localization. In International Conference on Automated Software Engineering. 556–559.
[54] W. Eric Wong, Ruizhi Gao, Yihao Li, Rui Abreu, and Franz Wotawa. 2016. A Survey on Software Fault Localization. Transactions on Software Engineering 42, 8 (2016), 707–740.
[60] Daming Zou, Jingjing Liang, Yingfei Xiong, Michael D Ernst, and Lu Zhang. 2019. An Empirical Study of Fault Localization Families and Their Combinations. Transactions on Software Engineering (2019), To appear.
1
18
35
40
2
45
53
55
58
59
23
44
]

故障定位[41,51,54,60]是识别程序中故障位置的活动。人们提出了许多不同类型的故障定位技术。基于频谱的故障定位[1,18]利用测试覆盖信息来查明故障程序或统计技术。例如，Tarantula[18]使用同源性排名度量来计算语句的可疑值，这些可疑值是根据通过和失败测试用例中语句的频率来计算的。基于突变的故障定位[35,40]使程序发生突变并运行其测试用例，使用测试结果定位故障。例如，Metallaxis[40]为每个语句生成一组突变体，为每个突变体分配一个可疑性分数，并聚合这些分数以生成语句的可疑性。其他的故障定位技术通过一些可选的方法来识别故障的位置，包括动态程序依赖分析[2,45]、堆栈跟踪分析[53,55]、条件表达式突变分析[58]、信息检索[59]和版本历史分析[23,44]。

### 2.2 Automated Patch Generation

Automated patch generation [11, 34, 50, 57] aims to automatically repair software systems by producing a fix that can be validated before it is fully accepted into the system. Automated patch generation techniques can be divided into two main categories: generate-andvalidate approaches and semantics-driven approaches. Generateand- validate approaches [7, 22, 32, 43, 52] iteratively execute two activities: patch generation, which produces candidate patch of the bug by making atomic changes or applying bug fix templates; patch validation, which checks the correctness of the generated solutions by running test cases. For example, GenProg [52] uses genetic programming to guide the generation and validation activities. At every iteration, it randomly determines the location to apply an atomic change, according to a probability distribution that matches the suspiciousness of the statements computed with fault localization algorithms. Every candidate solution is validated running the available test suite. It defines a fitness function that measures the fitness of each program variant based on the number of passing and failing test cases. Semantics-driven approaches [21, 30, 38] formally encode the problem, either as a formula whose solutions correspond to the possible fixes, or as an analytical procedure whose outcome is a fix. A solution found by such approaches is correct by construction, thus no validation is needed. For example, SemFix [38] uses fault localization to identify the statement that should be changed, then tries to synthesize a fix by modifying a branch predicate or changing the right hand side of an assignment.

自动补丁生成[11,34,50,57]的目的是通过生成一个补丁来自动修复软件系统，这个补丁可以在系统完全接受之前进行验证。自动补丁生成技术可以分为两大类:生成和验证方法和语义驱动方法。Generateand- validate方法[7,22,32,43,52]迭代地执行两个活动:补丁生成，通过进行原子性更改或应用bug修复模板生成bug的候选补丁;补丁验证，它通过运行测试用例来检查生成的解决方案的正确性。例如，GenProg[52]使用遗传编程来指导生成和验证活动。在每一次迭代中，它根据与故障定位算法计算的语句可疑性相匹配的概率分布，随机确定应用原子更改的位置。运行可用的测试套件对每个候选解决方案进行验证。它定义了一个适应度函数，该函数根据通过和失败测试用例的数量来度量每个程序变体的适应度。语义驱动的方法[21,30,38]将问题正式编码为一个公式，其解决方案对应于可能的修复，或者作为一个分析过程，其结果是修复。这种方法找到的解决方案通过构造是正确的，因此不需要验证。例如，SemFix[38]使用错误定位来识别应该更改的语句，然后尝试通过修改分支谓词或更改赋值的右侧来综合修复。

### 2.3 Patch Recommendation

Patch recommendation [3, 16, 20, 24, 36] suggests a few candidate changes which may repair a given fault. In some cases, the recommended patches are perfect fixes, while in other cases some efforts are required from the developers to produce the final fix. Although these techniques do not guarantee a working repair, their results are still useful in assisting developers in deriving the patch. A number of patch recommendation techniques have been proposed so far. For example, Getafix [3] from Facebook learns recurring fix patterns for static analysis warnings and suggests fixes for future occurrences of the same bug category. It firstly splits a given set of example fixes into AST-level edits, then it learns recurring fix patterns from these edits based on a clustering technique which produces a hierarchy of fix patterns. Finally, given a bug under fix, it finds suitable fix patterns, ranks candidate fixes, and suggests the top-most fixes to developers. As another example, CLEVER [36] from Ubisoft aims to intercept risky commits before they reach the central repository. It first builds a metric-based model to assess the risky level of incoming commits, then it uses clone detection to compare code blocks in risky commits with some known historical fault-introducing commits.

补丁建议[3,16,20,24,36]建议一些可能修复给定错误的候选更改。在某些情况下，推荐的补丁是完美的修复，而在其他情况下，则需要开发人员进行一些工作来生成最终的修复。虽然这些技术不能保证修复工作正常进行，但是它们的结果对于帮助开发人员获得补丁还是很有用的。目前已经提出了许多补丁推荐技术。例如，来自Facebook的Getafix[3]学习了针对静态分析警告的重复修复模式，并为将来出现的相同bug类别提供修复建议。它首先将给定的一组示例修复划分为最小层次的编辑，然后根据生成修复模式层次结构的集群技术从这些编辑中学习重复的修复模式。最后，对于fix下的bug，它会找到合适的修复模式，对候选修复进行排序，并向开发人员建议最重要的修复。另一个例子是，Ubisoft的聪明的[36]旨在在危险的提交到达中央存储库之前拦截它们。它首先构建一个基于度量的模型来评估传入提交的风险级别，然后使用克隆检测来比较风险提交中的代码块与一些已知的历史错误引入提交。

All these aforementioned techniques depend on existing patches or already-known bug patterns. In contrast, we do not assume enough debugging reports, and we extract templates of defectpatch pairs through data mining. Open-source dataset such as Defect4J [19] contains labeled defects and the corresponding patches, which have been examined and analyzed by many researchers. Yet, recent studies [56] indicate that many state-of-the-art patch generation techniques has the problem of over-fitting to specific benchmark set. Therefore, many of the existing techniques cannot be directly applied on the industrial codebase, which is quite different from the open-source dataset in many ways.

所有这些前面提到的技术都依赖于现有的补丁或已经知道的bug模式。相反，我们没有假设足够的调试报告，而是通过数据挖掘提取缺陷补丁对的模板。像defect这样的开源数据集包含了标记的defect和相应的patch，这些都是很多研究者研究和分析过的。然而，最近的研究[56]表明，许多最先进的补丁生成技术都存在对特定基准集过度拟合的问题。因此，许多现有的技术不能直接应用于工业代码库，这在许多方面与开源数据集有很大的不同。

## 3 PRELIMINARY STUDY

To better understand the codebase at Alibaba and the challenges in applying existing techniques in the industrial development environment, we conducted a preliminary study of the usage scenarios of patch recommendation techniques within the company and empirically analyzed the characteristics of the company codebase.

为了更好地了解阿里巴巴的代码库以及在行业发展环境中应用现有技术所面临的挑战，我们对公司内部补丁推荐技术的使用场景进行了初步研究，并对公司代码库的特点进行了实证分析。

### 3.1 Challenges for Existing Techniques

Through manual inspection, interviews with developers, and empirical studies, we identified three key challenges for existing fault localization and automated patch generation techniques to be successfully applied on our benchmark.

通过人工检查、与开发人员的访谈和实证研究，我们确定了现有故障定位和自动生成补丁技术在我们的基准上成功应用的三个关键挑战。

- Insufficient Labeled Data. 
A lot of fault localization and automated patch generation techniques require labeled defect and patch samples to be able to extract patterns of typical bug fixes. Yet, this is a substantial obstacle in our case, since there exists very few labeled defects or patches in the company codebase. Moreover, due to the widespread legacy code in the codebase, a large number of software projects only have partial debugging reports and very limited test cases. The commit messages may be succinct and do not follow any standard template either. Therefore, it is challenging to label defects and the associated fixes manually, given the size and complexity of the codebase. The business logic and bug fix patterns of the internal company codebase are quite different from that of open source projects [42, 47]. Thus, we decide not to directly use the labeled data from open-source projects.

许多错误定位和自动补丁生成技术都需要标记的缺陷和补丁样本来提取典型错误修复的模式。然而，在我们的案例中，这是一个实质性的障碍，因为在公司的代码库中存在很少标记的缺陷或补丁。此外，由于代码库中广泛存在遗留代码，大量软件项目只有部分调试报告和非常有限的测试用例。提交消息可能很简洁，也不遵循任何标准模板。因此，考虑到代码库的大小和复杂性，手工标记缺陷和相关的修复是很有挑战性的。内部公司代码库的业务逻辑和bug修复模式与开源项目的模式有很大的不同[42,47]。因此，我们决定不直接使用来自开源项目的标记数据。

- High Responsive Standard. 
The application scenario of patch recommendation in Alibaba is highly interactive. Patch recommendation needs to be run whenever new commits are submitted by developers for code review. The recommended patches are then checked by developers, who may decide to incorporate the suggestions into the commits. On average, a developer submits three to four commits per week, and both the submitter and reviewer expect prompt patch recommendations to avoid delays during the review process. Therefore, the responding time for patch recommendation is supposed to be reasonably low in order to be integrated into the development routine. This renders some automated patch generation approaches inappropriate, since they need to repeatedly compile and execute tests for each identified defect.

阿里巴巴的补丁推荐应用场景交互性强。每当开发人员提交新的提交以进行代码审查时，都需要运行补丁推荐。然后由开发人员检查推荐的补丁，他们可能决定将建议合并到提交中。平均而言，开发人员每周提交三到四次提交，提交者和审查者都希望得到及时的补丁建议，以避免审查过程中的延迟。因此，为了集成到开发例程中，补丁推荐的响应时间应该比较低。这使得一些自动生成补丁的方法变得不合适，因为它们需要重复地编译和执行每个已识别缺陷的测试。

- Generalizability Requirement. 
Our benchmark set consists of more than 10K projects supporting more than 100 software applications. These applications cover a variety of domains including e-commerce, finance, cloud computing, and artificial intelligence, many of which are used by millions of users on a daily basis. The patch recommendation techniques should be generalizable to cover all different projects and defect types.

我们的基准测试集包含支持100多个软件应用程序的10K多个项目。这些应用程序涵盖了电子商务、金融、云计算和人工智能等多个领域，其中许多应用程序每天都有数百万用户使用。补丁推荐技术应该是通用的，以覆盖所有不同的项目和缺陷类型。

### 3.2 Challenges in Defect-Patch Identification

There exists techniques [27–29, 48] which automatically identify changes of certain kinds, e.g., bugs and fixes, from commit histories. These techniques, such as the SZZ algorithm [48], can be useful for labeling defect-patch pairs in the absence of high-quality labeled data. The high-level idea is to first locate bug-fixing commits based on keywords in commit messages and debugging reports. For example, a commit is considered as bug-fixing commit if it appears in a bug report or its commit message contains keywords such as “bug” and “fix”. For each identified bug-fixing commit, one can trace each line of changed code back in history to locate the bug-inducing commits using version control information such as git-blame [12].

有一些技术[27 - 29,48]可以自动地从提交历史中识别某些类型的变更，例如，bug和修复。这些技术，例如SZZ算法[48]，可以在缺少高质量标记数据的情况下用于标记缺陷-补丁对。高级思想是首先根据提交消息和调试报告中的关键字定位修复错误的提交。例如，如果提交出现在bug报告中，或者其提交消息包含“bug”和“fix”之类的关键字，则将其视为bug修复提交。对于每个已确定的bug修复提交，可以使用版本控制信息(如git-blame[12])跟踪历史上更改的每行代码，以定位导致bug的提交。

Various optimizations have been introduced in the SZZ algorithm to reduce the false positives. For example, the timestamp of a candidate bug-inducing commit is compared with the time when the bug is reported to rule out unreasonable results. Yet, we found that even with these optimizations, the SZZ algorithm still does not perform well on our benchmark (25% true positive rate). To figure out the reason why the SZZ algorithm does not perform well on our codebase, we quantitatively analyzed the dataset and identified two stages in the algorithm where imprecision can be introduced: (1) the identification of bug-fixing commits, and (2) the location of bug-inducing commits via back-tracing in history.

SZZ算法中引入了各种优化以减少误报。例如，将引发bug的候选提交的时间戳与报告bug以排除不合理结果的时间进行比较。然而，我们发现即使使用这些优化，SZZ算法在我们的基准测试(25%的真实阳性率)上仍然表现不佳。找出为什么SZZ算法不执行我们的代码库,我们定量分析数据集和确认两个阶段在不精确的算法可以介绍:(1)确定的bug修复提交,和(2)的位置通过历史上back-tracing bug-inducing提交。

- Imprecision in Locating Bug-Fixing Commits. 
The first reason causing imprecision is that the keyword matching approach is not always reliable. For example, through manual inspection, we discovered that a commit with the commit message containing the keyword “fix”, does not change any program logic and only modifies the label of an Android UI button. Even if a commit does fix a bug, it can also make irrelevant changes to other parts of the code. Fig. 1 shows the distribution of the number of changed files in each commit matched by at least one keyword. The keywords we used include “fix”, “bug”, “repair”, “wrong”, “fail”, “problem”, and their corresponding Chinese translations. About 60% of the commits change more than one file, among which 40% touch over two files. Many such commits are multi-purpose, causing imprecision in locating bug-fixing commits.

造成不精确的第一个原因是关键字匹配方法并不总是可靠的。例如，通过手动检查，我们发现提交消息包含关键字“fix”，不改变任何程序逻辑，只修改Android UI按钮的标签。即使提交确实修复了错误，它也可能对代码的其他部分做出无关的更改。图1显示了至少一个关键字匹配的每次提交中的更改文件数量的分布。我们使用的关键词包括“fix”、“bug”、“repair”、“wrong”、“fail”、“problem”，以及对应的中文翻译。大约60%的提交更改不止一个文件，其中40%涉及两个文件。许多这样的提交是多用途的，导致定位错误修复提交的不精确性。

- Imprecision in Locating Bug-Inducing Commits. Additional imprecision can be introduced when there are multiple bug-inducing commits in the history. When back-tracing in history, it is possible to end up with more than one commits, each partially contributing to the defect. We randomly select four projects from the dataset. Fig. 2 plots the distribution of the number of bug-inducing commits identified by the SZZ algorithm for each bug-fixing commit. For example, for project 𝑃1, among all the defects, 78% are induced by one commit, 14% are induced by two commits, 4% are induced by three commits, and 3% are induced by more than three commits. We found that the commit-level back-tracing often introduces too much irrelevant changes for the similar reason discussed before.

当历史中有多个引发bug的提交时，可能会引入额外的不精确性。当回溯历史时，可能会出现多个提交，每个提交都会部分地导致缺陷。我们从数据集中随机选择四个项目。图2描绘了SZZ算法识别的每个bug修复提交的bug诱导提交数的分布。例如,对于项目𝑃1,在所有的缺陷,78%是由一个提交,14%是由两个提交,4%是由三个承诺,3%是由三个以上提交。我们发现，基于前面讨论过的类似原因，委员会级别的回溯经常引入太多不相关的更改。

## 4 OUR APPROACH

Fig. 3 overviews the workflow of Precfix. Precfix consists of an offline patch discovery component and an online patch recommendation component. The patch discovery component first extracts potential defect-patch pairs from commits in the version controlled history, clusters defect-patch pairs based on their similarity, and finally extracts generic patch templates to be stored in a database. The patch recommendation component recommends patch candidates to developers and collects their feedback. It removes patches rejected by developers, and includes manually crafted patch templates submitted by developers to improve the patch database.

图3概述了Precfix的工作流程。Precfix包含一个离线补丁发现组件和一个在线补丁推荐组件。patch discovery组件首先从版本控制历史记录中的提交中提取出潜在的缺陷-补丁对，然后根据缺陷-补丁对之间的相似性进行聚类，最后提取出要存储在数据库中的通用补丁模板。补丁推荐组件向开发人员推荐候选的补丁，并收集他们的反馈。它删除开发人员拒绝的补丁，并包括开发人员提交的手工制作的补丁模板，以改进补丁数据库。

### 4.1 Patch Discovery

The offline patch discovery component performs three steps to generate patch templates: extracting defect-patch pairs, clustering defect-patch pairs, and collecting generic patch templates.

离线补丁发现组件执行三个步骤来生成补丁模板:提取缺陷-补丁对、集群缺陷-补丁对和收集通用补丁模板。

#### 4.1.1 Extracting Defect-Patch Pairs.

The first step is to extract a large number of defect-patch pairs from the codebase. We make two improvements to the SZZ algorithm, to adapt to the needs of the industrial codebase.

第一步是从代码库中提取大量缺陷-补丁对。我们对SZZ算法做了两个改进，以适应工业代码库的需要。

- Constraining the Number of Changed Files. 
The first adjustment is to set a threshold on the number of files modified in a bug-fixing commit, filtering out any commit that exceed the threshold. In this way, we could reduce the false positives that are caused by multi-purpose commits (C.f. Sect. 3). We heuristically chose five as the threshold, which help reduce false positives without discarding too many candidate commits (22.7% as indicated in Fig. 1).

第一个调整是对修复bug提交中修改的文件数量设置阈值，过滤掉任何超过阈值的提交。这样，我们可以减少由多目的提交所导致的误报(c.f. . 3节)。我们启发式地选择5个作为阈值，这有助于减少误报，同时又不放弃太多的候选提交(22.7%，如图1所示)。

- Identifying Bug-Inducing Code Snippets. 
The second improvement aims to improve the precision of identifying bug-inducing changes. As studied in Sect. 3, commit-level back-tracing in history tends to be imprecise, especially when there are many multipurpose commits contributing to the defect.

第二个改进是为了提高识别引起bug的变化的精度。正如在第3节中所研究的，历史上的委员会级别的回溯往往是不精确的，特别是当有许多多用途的提交导致缺陷时。

Therefore, instead of tracing commits back in time and identifying bug-inducing commits, we directly identify bug-inducing code snippets. In fact, the differences before and after the introduction of a bug-fixing commit already contain information about both the defects and the patches. Now we describe the improved method-level defect-patch pair extraction in more details.

因此，我们直接识别导致bug的代码片段，而不是及时跟踪提交和识别导致bug的提交。实际上，在引入bug修复提交之前和之后的差异已经包含了关于缺陷和补丁的信息。现在，我们将更详细地描述改进的方法级缺陷-补丁对提取。

- Method-Level Defect-Patch Pair Extraction. 
To further reduce false positives, we constrain the defects and patches as the removed and inserted lines in a bug-fixing commit, respectively. We confine the scope of defects and patches to be within a single method. Of course, these constraints may rule out some genuine defect-patch pairs, e.g., patches only removing old lines or inserting new lines. Our current bug-fix model balances between the completeness and accuracy, attempting to be simple but applicable to the most common cases.We randomly sampled 90 bug fixes from the benchmark set and confirmed that 68 (76%) of them contain both removed and inserted lines, which fall into our simplified bug model. We found in practice that the compromises made in the recall (24%) help significantly improve the precision.

为了进一步减少误报，我们分别在bug修复提交中删除和插入的行时约束缺陷和补丁。我们将缺陷和补丁的范围限制在单一方法之内。当然，这些约束可能会排除一些真正的缺陷-补丁对，例如，补丁只删除旧行或插入新行。我们当前的bug修复模型在完整性和准确性之间保持平衡，试图简单但适用于最常见的情况。我们从基准测试集中随机抽样90个bug修复，并确认其中68个(76%)包含删除和插入的行，它们属于我们简化的bug模型。我们在实践中发现，召回中做出的妥协(24%)有助于显著提高准确性。

Given a bug-fixing commit and the source code of the project, the workflow of extracting defect-patch pairs is the following. 
(1) Check out the last-updated snapshot before the bug-fixing commit and record inserted and removed lines in the commit for later parsing. 
(2) Extract inserted lines in the commit, which are patch snippets. 
(3) Extract removed lines in the commit, which are defect snippets. 
(4) Associate each defect snippet with the corresponding patch snippet, forming defect-patch pairs. 
(5) Filter out the lines of defect snippets that are not in the same method scope as any patch lines, making defect-patch pairs more cohesive. 
(6) Filter out assertion lines, comments, and logging lines from the defect-patch pairs as they are generally irrelevant to bug fixes. At this point, we have obtained a set of defect-patch pairs, on which we perform the remaining steps of the patch discovery.

给定修复错误的提交和项目的源代码，提取缺陷-补丁对的工作流程如下。
(1)在提交修复bug之前检查最后更新的快照，并在提交中插入和删除记录，以便以后进行解析。
(2)提取提交中插入的行，这些行是补丁片段。
(3)提取提交中被删除的行，它们是缺陷片段。
(4)将每个缺陷片段与对应的补丁片段相关联，形成缺陷-补丁对。
(5)过滤掉与任何修补程序不在同一方法范围内的缺陷片段的行，使缺陷-修补程序对更具内聚性。
(6)从缺陷-补丁对中过滤掉断言行、注释和日志行，因为它们通常与bug修复不相关。此时，我们已经获得了一组缺陷-补丁对，我们将在其上执行补丁发现的其余步骤。

#### 4.1.2 Clustering Defect-Patch Pairs. 

To obtain common defects patterns in the codebase, we group all the extracted defect-patch pairs into a set of clusters. We use density-based spatial clustering of applications with noise (DBSCAN) [9] as our clustering algorithm. DBSCAN is a wellestablished clustering method without the need to determine the number of clusters (𝐾) in advance. Given a set of points in some vector space, DBSCAN groups points that are closely packed together (nearby neighbors), marking as outliers those points that lie in low-density regions. We choose DBSCAN instead of other clustering algorithms, such as KNN [8] and 𝐾-means, because in our case, the number 𝐾 could not be known in advance. A well known limitation of the vanilla DBSCAN algorithm is that it is computationally expensive when coping with large-scale dataset [37], since it considers all possible comparisons between every data point. In our application scenario, most of the comparisons are unnecessary, as many code snippets are very loosely related. Therefore, we make three customized improvements to DBSCAN to mitigate this issue.

为了获得代码库中常见的缺陷模式，我们将所有提取的缺陷-补丁对分组到一组集群中。我们使用基于密度的空间聚类应用程序与噪声(DBSCAN)[9]作为我们的聚类算法。DBSCAN是他聚类方法而不需要事先确定集群(𝐾)的数量。给定某个向量空间中的一组点，DBSCAN将紧挨在一起的点(邻近的点)分组，将位于低密度区域的点标记为离群点。我们选择DBSCAN代替其他聚类算法,如资讯[8]和𝐾-means,因为在我们的例子中,𝐾数量不能提前知道。普通DBSCAN算法的一个众所周知的局限性是，当处理大型数据集[37]时，它的计算开销非常大，因为它考虑了每个数据点之间所有可能的比较。在我们的应用程序场景中，大多数比较都是不必要的，因为许多代码片段都是松散相关的。因此，我们对DBSCAN做了三个定制的改进来缓解这个问题。

- Utilizing SimHash-KDTree Reducers. 
The first customization is to use SimHash-KDTree reducers to avoid the comparison of some irrelevant data points. SimHash [33] is an algorithm that generates a low-dimensional vector signature for any high-dimensional vector. The generated low-dimensional vector has the same expressiveness as the original high-dimensional vector, but is shorter in length, thus enables faster comparison. During preprocessing, we use the SimHash algorithm to map the contents of code snippets to 16-bit hashcode sequences. Then we run the KDTree algorithm [4] to group all the sequences that have a Hamming distance less than 4 into the same reducer. During the clustering, we only compute similarity of code snippets in the same reducer.

第一个定制是使用simhashk - kdtree约简器来避免对一些不相关的数据点进行比较。SimHash[33]是一种为任何高维向量生成低维向量签名的算法。生成的低维向量具有与原始高维向量相同的表达能力，但是长度更短，因此可以更快地进行比较。在预处理期间，我们使用SimHash算法将代码片段的内容映射到16位hashcode序列。然后我们运行KDTree算法[4]将所有汉明距离小于4的序列分组到同一个减速机中。在聚类的过程中，我们只计算同一减速机中代码片段的相似度。

- Exploiting API Sequence Information. 
The second improvement is to exploit the information from API call sequences to avoid irrelevant comparisons. The intuition behind this is that if two code snippets contain two completely different API call sequences, then they are obviously not belonging to the same patch pattern, thus should not be compared during the clustering. Therefore, we parse the ASTs of defect-patch pairs and extract API call sequences for each code snippet in advance. During clustering, even if two code snippets are in the same reducer, as long as their API call sequences do not match, we skip the comparison of them.We name this filtering process as APISeq.

第二个改进是利用API调用序列中的信息来避免不相关的比较。这背后的直觉是，如果两个代码片段包含两个完全不同的API调用序列，那么它们显然不属于同一补丁模式，因此不应该在集群期间进行比较。因此，我们解析缺陷-补丁对的ast，并预先为每个代码片段提取API调用序列。在集群过程中，即使两个代码片段位于相同的减速器中，只要它们的API调用序列不匹配，我们就会跳过对它们的比较。我们将这个过滤过程命名为APISeq。

- Normalizing Code Snippets. 
We also improve the clustering accuracy of DBSCAN by normalizing code snippets. Some common coding patterns have multiple equivalent ways of expression. These ways, although semantically equivalent, are syntactically different, which decreases the accuracy of clustering. For instance, a string value could be expressed as either a single string literal or the concatenation of several string literals. When we compare code snippets, we want to consider these equivalent expressions as the same. Therefore, we normalize the code snippets to convert these semantically equivalent snippets into syntactically same format. Our normalization rules are as follows: (1) merge consecutive calls of the same API into one API call sequence, concatenated by the dot (“.”) operator; (2) change “.append(string)” to the concatenation of strings; and (3) merge the concatenation of strings into a single string.

我们还通过规范化代码片段来提高DBSCAN的聚类精度。一些常见的编码模式有多种等价的表达方式。这些方法虽然在语义上是等价的，但在语法上是不同的，这就降低了聚类的准确性。例如，一个字符串值可以表示为单个字符串文字，也可以表示为几个字符串文字的连接。在比较代码段时，我们希望将这些等效表达式视为相同的表达式。因此，我们对代码片段进行规范化，将这些语义上等价的代码片段转换为语法上相同的格式。我们的规范化规则如下:(1)将同一API的连续调用合并为一个API调用序列，并由点(" . ")操作符连接;(2)将“.append(string)”改为字符串的拼接;(3)将字符串串接合并成一个字符串。

DBSCAN computes the distance between two defect-patch pairs based on their similarity. The computation of similarity is described in Algorithm 1. Given two defect-patch pairs, the algorithm first checks whether they are in the same reducer. The reducers are obtained using the customized hashing algorithm (SimHash + KDTree). If they are not in the same reducer, then the similarity computation is skipped (Line 2). Otherwise, the algorithm invokes APISeq, additional filtering based on the API sequence used, on each defect-patch pair to extract API sequences (Lines 3-4). If there is no intersection between the two API sequences, the algorithm also skips the similarity computation (Line 6).

DBSCAN根据缺陷修补对之间的相似性计算它们之间的距离。算法1描述了相似度的计算。给定两个缺陷-补丁对，算法首先检查它们是否在同一个减速机中。使用定制的哈希算法(SimHash + KDTree)获得约简。如果它们不在同一个reduce中，则跳过相似度计算(第2行)，否则，算法调用APISeq，根据使用的API序列对每个缺陷-补丁对进行额外的过滤，提取API序列(第3-4行)。如果两个API序列之间没有交集，算法也会跳过相似度计算(第6行)。

The remaining defect-patch pairs after the filtering proceed to the similarity computation. The similarity function we used in Algorithm 1 is a weighted sum of two similarity metrics: the Jaccard similarity coefficient [39] and the Levenshtein distance [25] (Lines 7-9). We choose these two similarity metrics because the Jaccard similarity coefficient is able to capture the token overlapping rate while the Levenshtein distance has the ability to capture the ordering information of the token list.

过滤后的剩余缺陷-patch对进行相似度计算。我们在算法1中使用的相似度函数是两个相似度度量的加权和:Jaccard相似系数[39]和Levenshtein距离[25](第7-9行)。我们选择这两个相似度度量是因为Jaccard相似系数能够捕获令牌重叠率，而Levenshtein距离能够捕获令牌列表的排序信息。

#### 4.1.3 Collecting Generic Patch Templates. 

After the clustering step finishes, for each pattern cluster, Precfix tries to extract a generic patch template, which summarizes the common pattern of defectpatch pairs in that cluster. Each template encodes a pattern of defect and its corresponding patch.

在集群步骤完成之后，对于每个模式集群，Precfix尝试提取一个通用的补丁模板，该模板总结了该集群中缺陷补丁对的常见模式。每个模板编码一个缺陷模式及其相应的补丁。

The goal of template generalization is to make the patch generally applicable in different contexts and understandable to developers. Although all the defect-patch pairs from the same cluster are similar, they differ from each other in some context-specific way, e.g., variable names. We abstract away all context-specific contents and only preserve the generic templates.

模板泛化的目标是使补丁普遍适用于不同的环境，并使开发人员能够理解。尽管来自同一集群的所有缺陷-补丁对都是类似的，但它们在某些特定于上下文的方式(例如变量名)上是不同的。我们抽象了所有上下文相关的内容，只保留了通用模板。

The first step of template generalization is to convert code lines into a list of tokens, each token is either a symbol or a parameter. Symbols are context-independent, they together form the common patterns among all the defect-patch pairs in a cluster; parameters are context-specific that represent the abstraction of variable names. When a template is applied at a specific context, its parameters are supposed to be replaced by the actual variable names.

模板泛化的第一步是将代码行转换为令牌列表，每个令牌要么是一个符号，要么是一个参数。符号是与上下文无关的，它们共同构成一个簇中所有缺陷-补丁对的共同模式;参数是特定于上下文的，表示变量名称的抽象。当模板应用于特定的上下文时，它的参数应该由实际的变量名替换。

The second step is to extract templates from the token lists. We use the recursive longest common substring (RLCS) algorithm [15] to perform matching between token lists. RLCS recursively calls longest common substring matching until all the tokens are matched. The highlighted parts in Fig. 4 are the matched tokens after the RLCS process (Step 1). We adjust the vanilla RLCS so that different parameter names are not matched while the other symbols are. Finally, we collect all the parameter names that are matched in the same cluster to a list and store them in our self-defined parameter format. Step 2 in Fig. 4 shows that different parameter names are recognized and collected, and stored in the standard format: @𝑃𝑎𝑟𝑎{parameter name list}.

第二步是从令牌列表中提取模板。我们使用递归最长公共子串(RLCS)算法[15]来执行令牌列表之间的匹配。RLCS递归地调用最长公共子字符串匹配，直到所有标记都匹配为止。图4中突出显示的部分是经过RLCS处理(步骤1)后匹配的标记。最后，我们将同一集群中匹配的所有参数名称收集到一个列表中，并将它们存储为自定义参数格式。步骤2的图4显示了不同的参数名称识别和收集,并存储在标准格式:@𝑃𝑎𝑟𝑎}{参数名称列表。

Finally, for each defect-patch pair, we change the parameter names in the patch snippet to match the parameter names in the corresponding defect snippet. After this step, Precfix constructs a patch template database, which contains all the extracted templates.

最后，对于每一个缺陷-补丁对，我们改变补丁片段中的参数名称来匹配相应缺陷片段中的参数名称。在此步骤之后，Precfix构建一个补丁模板数据库，其中包含所有提取的模板。

### 4.2 Patch Recommendation

The online patch recommendation component is triggered whenever developers commit new code changes. During code review, Precfix matches each of developers’ newly committed code snippet with the templates database. If a code snippet matches with a template, Precfix replaces the generic parameter placeholders in the template with the actual parameter names used in the specific contexts. If multiple templates are matched, they are ranked based on the frequency of the corresponding defect-patch pairs before clustering. By default, Precfix recommends the most popular patch candidate. The patch candidate is then sent to the code submitters and reviewers for feedback.We collect feedback from developers to validate the patch candidates and improve the template database of Precfix. More specifically, the defects and the corresponding recommended patches are presented to the developers. As developers inspect each recommended patch, we also collect their reactions on the patch, i.e., accept or reject. If patches from a certain template are frequently rejected, we will manually inspect the template and decide whether to remove it.

只要开发人员提交新的代码更改，在线补丁推荐组件就会被触发。在代码审查期间，Precfix将每个开发人员新提交的代码片段与模板数据库进行匹配。如果代码段与模板匹配，Precfix将模板中的通用参数占位符替换为在特定上下文中使用的实际参数名。如果匹配多个模板，则在聚类之前根据相应缺陷-补丁对的频率对它们进行排序。默认情况下，Precfix推荐最流行的补丁候选。然后将候选补丁发送给代码提交者和评审者以获得反馈。我们收集开发人员的反馈来验证候选补丁并改进Precfix的模板数据库。更具体地说，缺陷和相应的推荐补丁被呈现给开发人员。当开发人员检查每个推荐的补丁时，我们也收集他们对补丁的反应，即，接受或拒绝。如果某个模板的补丁经常被拒绝，我们将手动检查该模板并决定是否删除它。

During the patch validation process, in addition to feedback on the recommended patches, Precfix also accepts patch templates created by developers and is able to integrate them into Precfix’s template database. We encourage developers to make custom adjustments to the existing patches or build their own patch templates based on their experience. We believe this contribution mechanism can enrich the template database in the long run.

在补丁验证过程中，除了对推荐的补丁提供反馈外，Precfix还接受开发人员创建的补丁模板，并能够将它们集成到Precfix的模板数据库中。我们鼓励开发人员对现有补丁进行自定义调整，或者根据他们的经验构建自己的补丁模板。我们相信，从长远来看，这种贡献机制可以丰富模板数据库。

Fig. 5 shows the Precfix user interface for patch recommendation. Developers interact with it during code reviews. Fig. 5(a) shows the inline view of an identified defect and the corresponding patch recommended. Fig. 5(b) shows the detailed view of the defect-patch pairs (left) and the developer template suggestion form (right). The left side shows the list of defect-patch pairs from which the recommended template was extracted. The details of each defect-patch pair can be expanded, viewed, and voted for or against. Specifically, the two records shown in Fig. 5(b) are from
the files “.../Rec***dler.java” and “.../Trip***Impl.java”, respectively. The right side is a form allowing developers to devise their own patches and submit to the database.

图5为patch推荐的Precfix用户界面。开发人员在代码评审期间与它交互。图5(a)显示了识别缺陷的内联视图和推荐的相应补丁。图5(b)显示了缺陷-补丁对的详细视图(左侧)和开发人员模板建议表单(右侧)。左边显示了缺陷-补丁对的列表，从其中提取了推荐的模板。每个缺陷补丁对的详细信息可以展开、查看并投票支持或反对。具体来说，图5(b)中显示的两条记录来自文件“…/Rec***dler”。java”和“* * * Impl…/旅行。java”,分别。右侧是一个表单，允许开发人员设计自己的补丁并提交到数据库。

## 5 IMPLEMENTATION AND EVALUATION

In this section, we describe the implementation details of Precfix and evaluate its effectiveness and efficiency on our benchmark set.

Precfix is implemented on top of the cloud-based data processing platform, MaxCompute [14], developed by Alibaba. The commit history data is preprocessed and stored in data tables on the Alibaba Cloud [13] storage first. The defect-patch pair extraction is implemented as a set of SQL scripts and user-defined functions (about 900 LOC). The clustering of defect-patch pairs is highly parallelized (taking about 1 KLOC Java code) and handled by the MapReduce engine of MaxCompute. The online patch recommendation component is integrated with the internal code review process and is triggered whenever a new commit is submitted.

在本节中，我们将描述Precfix的实现细节，并评估它在基准集上的有效性和效率。

Precfix是在阿里巴巴开发的基于云的数据处理平台MaxCompute[14]上实现的。提交的历史数据先被预处理并存储在阿里巴巴云[13]存储的数据表中。缺陷-补丁对提取是作为一组SQL脚本和用户定义函数(大约900 LOC)实现的。缺陷-补丁对的集群是高度并行的(大约需要1个KLOC Java代码)，由MaxCompute的MapReduce引擎处理。在线补丁推荐组件与内部代码审查流程集成，并在提交新提交时触发。

The goal of our empirical evaluation is to answer the following research questions. RQ1: How effective is Precfix in locating defects and discovering patches? RQ2: How efficient is Precfix in recommending patches? RQ3: How much do our improvements of DBSCAN increase the efficiency of clustering? RQ4: What kind of patches does Precfix recommend? RQ5: What are the users’ opinions on the usability of Precfix?

我们实证评估的目的是回答以下研究问题。RQ1: Precfix在定位缺陷和发现补丁方面有多有效?RQ2: Precfix在推荐补丁时的效率如何?RQ3:我们对DBSCAN的改进在多大程度上提高了集群的效率?RQ4: Precfix推荐什么样的补丁?RQ5:用户对Precfix的可用性有什么看法?

We run the offline patch discovery component of Precfix on the randomly extracted sample dataset described in Sec 3. The sample dataset includes 10K projects, 15M commits, and 30M files. On this dataset, Precfix extracted 3K bug fix templates, forming the template database. During the clustering stage, 4,098 MB memory was allocated for each reducer.

我们在sec3中描述的随机抽取的样本数据集上运行Precfix的离线补丁发现组件。示例数据集包括10K项目、15M提交和30M文件。在这个数据集上，Precfix提取了3K bug修复模板，形成了模板数据库。在集群阶段，为每个减速器分配了4,098 MB内存。

## 6 CONCLUSION

In this paper, we present Precfix, a patch recommendation technique designed for large-scale industrial codebase. For patch discovery, Precfix extracts defect-patch pairs from version controlled histories, clusters these pairs with common patterns, and extracts bug fix templates from clusters. For patch recommendation, Precfix relies on the feedback from developers during code review. Moreover, Precfix integrates custom patches created by developers to improve its template database. We run Precfix’s patch generation on a subset (10K projects) of the internal codebase at Alibaba and extract 3K patches. Our manual inspection confirms that Precfix has low false positive rate (22%) while achieves a high extractability score (93%). In addition, our user study on the usability of Precfix’s patch recommendation functionality shows that the majority (10/12) of the interviewed developers appreciated Precfix and would like to see it widely adopted.

在本文中，我们提出了Precfix，一种针对大规模工业代码库的补丁推荐技术。对于补丁发现，Precfix从版本控制的历史记录中提取缺陷-补丁对，将这些缺陷-补丁对与常见模式进行集群，并从集群中提取缺陷修复模板。对于补丁推荐，Precfix依赖于代码评审期间开发人员的反馈。此外，Precfix集成了开发人员创建的定制补丁，以改进其模板数据库。我们在阿里巴巴内部代码库的一个子集(10K项目)上运行Precfix的补丁生成，并提取3K个补丁。我们的人工检查证实Precfix的假阳性率低(22%)，而提取性评分高(93%)。此外，我们对Precfix的补丁推荐功能的可用性的用户研究表明，大多数(10/12)受访的开发人员都很欣赏Precfix，并希望看到它被广泛采用。

The current implementation of Precfix still has some incompleteness in terms of the type of defects and patches handled. In the future, we would like to lift these restrictions while maintaining the high accuracy of patch recommendation. We would also like to apply machine learning-based approaches to automate the patch template improvement as much as possible.

就所处理的缺陷和补丁的类型而言，Precfix的当前实现仍然存在一些不完整性。在未来，我们希望在保持补丁推荐的高准确性的同时，解除这些限制。我们还希望应用基于机器学习的方法来尽可能多地自动化补丁模板改进。
