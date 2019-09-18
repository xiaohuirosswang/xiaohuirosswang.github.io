---
layout: post
title: "一个使用Git workflow 管理软件生命周期的成熟方案"
description: ""
category: [Testing, Git]
---
# Software life cycle and a practical git workflow


### [最近看到一篇非常好的关于实际软件开发团队中可行的Git workflow的文章][1]，很有感触，结合之前关于软件生命周期的相关经验，梳理一下自己的思路。

### 软件生命周期
大型软件，比如Windows，Office，BizTalk Server等，代码规模很大，开发人员众多，并且经常位于全球各地，这对代码管理和流程控制提出了很高的要求。像微软、IBM等大型软件公司内部都有完善的代码控制、测试和发布流程，本文以微软内部软件生命周期为例。

#### CTP - Beta - RC - RTM
* CTP - customer technology preview, 向潜在目标客户介绍新产品或者已有产品的新版本中的主打功能或者功能更新。收集客户反馈，相应调整产品功能点。_这里省略了之前的market research, 确定产品需求文档等。
* Beta - 在CTP版本的基础上，结合试用客户反馈和业内评测结果以及行业发展，调整软件需求文档，完整的产品测试以及bug修复后，公开发布的试用版。
* RC - Release candidate, 公测版发布后，结合客户反馈，对使用中和测试中出现的bug进行triage，prioritize并修复，功能点开发完全完成并稳定的版本。
* RTM - Release to manufacture, 产品最终的发布版本。
* 售后技术支持 - 针对产品发布后出现的bug提供技术支持，hotfix补丁包以及Service pack。
* 结合客户反馈和行业发展，确定下个版本的技术平台（通常涉及对新技术或者协议的支持、和其他软件产品的配合等）

大家都知道，微软是一家非常重视软件测试的公司，内部的测试流程非常严格，这其实是大规模软件产品所必须的。

从CTP开始，每个产品会有一个由产品、开发和测试相关人员组成的triage meeting小组，这个小组定期review新发现的bug，确定产品功能点调整需求，给现有bug确定优先级，review test plan进行质量控制，拿BizTalk Server举例：

1. 每天针对现有已经check in的代码打包，有专门的team维护相应的打包脚本。
2. 在新版本的基础上，运行已有的自动化测试用例，产品规模和复杂大越大，测试用例越多。但是根据测试用例的测试点和复杂度，基本可以分为BVT（Basic verification test）suites，IDW（it does work？）suites，Functional（end to end的复杂测试用例集合）suites，BVT每个版本都会跑，确保及时发现任何regression bug；IDW和Functional的test suites根据确定的test plan进行测试并有test team对测试结果进行调查，code defect分配给开发，test issue自己解决，迭代推进。
3. Beta和RTM等对外发布的版本，会进行media validation，对软件的下载包或者光盘媒介进行最后的测试。

### 和上面的流程无缝结合的git workflow
在上面介绍的基础上，我们看看这个被广泛推荐的git workflow

<img src="/images/git-branch-1.png" alt="Sanjose" class="img-center" />

### 说明
1. master分支和develop分支永远存在。
2. master分支上面的是对外发布的版本。
3. hotfix只从master分支，也就是已发布的版本代码中checkout，并在此基础上修复相应的bug，并最终merge回develop和master分支。
4. 新功能模块分支从develop branch checkout，开发完成后merge会develog分支。
5. release分支上面是各个milestone的版本，只从develop分支checkout，发布后merge到develop和master分支。

#### 结合上面提到的软件生命周期和顺便提到的测试流程，很容易把软件开发各个阶段的代码控制和这个workflow结合起来。

[1]: http://www.juvenxu.com/2010/11/28/a-successful-git-branching-model/

