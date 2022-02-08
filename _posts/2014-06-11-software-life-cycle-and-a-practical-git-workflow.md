---
layout: post
title: "一个使用Git workflow 管理软件生命周期的成熟方案"
description: ""
category: [Testing, Git]
---
# Software life cycle and a practical git workflow


### [最近看到一篇非常好的关于实际软件开发团队中可行的 Git workflow 的文章][1]，很有感触，结合之前关于软件生命周期的相关经验，梳理一下自己的思路。

### 软件生命周期
大型软件，比如 Windows，Office，BizTalk Server等，代码规模很大，开发人员众多，并且经常位于全球各地，这对代码管理和流程控制提出了很高的要求。像微软、IBM 等大型软件公司内部都有完善的代码控制、测试和发布流程，本文以微软内部软件生命周期为例。

#### CTP - Beta - RC - RTM
* CTP - Community Technology Preview, 向潜在目标客户介绍新产品或者已有产品的新版本中的主打功能或者功能更新。收集客户反馈，相应调整产品功能点。当然前期还有 market research, 确定产品需求文档等。
* Beta - 在CTP版本的基础上，结合试用客户反馈和业内评测结果以及行业发展，调整软件需求文档，完整的产品测试以及bug修复后，公开发布的试用版。
* RC - Release candidate, 公测版发布后，结合客户反馈，对使用中和测试中出现的 bug 进行 triage，prioritize 并修复，功能点开发完全完成并稳定的版本。
* RTM - Release to manufactor, 产品最终的发布版本。
* 售后技术支持 - 针对产品发布后出现的bug提供技术支持，hotfix 补丁包以及 Service pack。
* 结合客户反馈和行业发展，确定下个版本的技术平台（通常涉及对新技术或者协议的支持、和其他软件产品的配合等）

大家都知道，微软是一家非常重视软件测试的公司，内部的测试流程非常严格，这其实是大规模软件产品所必须的。

从CTP开始，每个产品会有一个由产品、开发和测试相关人员组成的 triage meeting 小组，这个小组定期review新发现的bug，确定产品功能点调整需求，给现有bug确定优先级，review test plan进行质量控制，拿BizTalk Server举例：

1. 每天针对现有已经 check in 的代码打包，有专门的 team 维护相应的打包脚本。
2. 在新版本的基础上，运行已有的自动化测试用例，产品规模和复杂大越大，测试用例越多。但是根据测试用例的测试点和复杂度，基本可以分为
   - BVT（Build verification test）suites，
   - IDW（it does work？）suites，
   - Functional（end to end的复杂测试用例集合）suites，
   BVT每个版本都会跑，确保及时发现任何 regression bug；IDW 和 Functional 的 test suites 根据确定的 test plan 进行测试并有 test team 对测试结果进行调查，code defect 分配给开发，test issue 自己解决，迭代推进。
4. Beta 和 RTM 等对外发布的版本，会进行 media validation，对软件的下载包或者光盘媒介进行最后的测试。

### 和上面的流程无缝结合的git workflow
在上面介绍的基础上，我们看看这个被广泛推荐 的git workflow

<img src="/images/git-branch-1.png" alt="Sanjose" class="img-center" />

### 说明
1. master 分支和 develop 分支永远存在。
2. master 分支上面的是对外发布的版本。
3. hotfix 只从 master 分支，也就是已发布的版本代码中 checkout，并在此基础上修复相应的 bug，并最终 merge 回 develop 和 master 分支。
4. 新功能模块分支从 develop branch checkout，开发完成后 merge 回 develog 分支。
5. release 分支上面是各个 milestone 的版本，只从 develop 分支 checkout，发布后merge 到 develop 和 master 分支。

#### 结合上面提到的软件生命周期和顺便提到的测试流程，很容易把软件开发各个阶段的代码控制和这个workflow结合起来。

[1]: http://www.juvenxu.com/2010/11/28/a-successful-git-branching-model/

