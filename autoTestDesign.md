# 论程序员的自我修养——自动化功能测试友好的设计

## 自动化功能测试对软件设计的影响

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;功能测试的目的是为了模拟用户操作，从而验证系统能按照预想的方式运行，因此自动化测试的脚本无可避免地需要访问软件的用户界面。相信很多放弃使用自动化功能测试的团队对于自动化功能测试的态度和我刚刚接触自动化测试时一样，UI的易变性和测试脚本与UI的紧密耦合，加上维护测试脚本的团队（测试）和UI开发的团队（开发）往往不是同一个团队（或同一个人），导致了维护测试脚本的成本非常高。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但自动化功能测试的确能提高测试质量，并且减少人的重复劳动，这点收益足以能让我们想办法去解决上述的问题。和单元测试一样，从系统设计的时候就考虑到适应自动化测试，是成本最低的解决自动化测试问题的方法。如果对一个本来就对自动化测试不友好的系统重构，改变就可能涉及到系统的基础架构了，这个时候投入的人力物力就会几何级数的上升。

## 瘦客户端的设计

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;既然UI的频繁变更是影响自动化测试最重要的原因，那么我们是否可以跳过UI来进行自动化测试呢？我是一个很直观的想法，而且在某些情况下是可以实现的。MVC模式告诉我们，View层是不应该包含业务处理逻辑的，因此我们可以使用系统为UI层暴露的接口（公用API）对系统进行功能测试，从而避开UI层频繁变更的问题。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种设计要求系统的UI层足够的简单，不能包含一丁点的业务逻辑，公用API的功能也需要足够的完备，能完成系统所有的业务逻辑处理。另外值得注意的一点，测试使用的API也必须是UI层调用的API，否则就没有办法很好地模拟真实的系统行为。我们已经对系统真实情况做了妥协，就不能无底线地一直妥协下去。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Web应用的情况可能相对好一点，由于Web架构的特性，我们只需要直接组装HTTP请求发送到Web服务器，便可以绕过UI进行功能测试。尽管这依然需要UI层足够的薄，但这已经是大部分Web开发者的共识了。

![tiny client](https://phospher.github.io/20160817153409260.jpeg)

## 客户端驱动模式

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正如上面说的，使用公用API测试是一种妥协的做法，对于那些有着极高用户体验要求的系统，就算是UI的显示逻辑，都可能是非常复杂的。这个时候，就算业务逻辑处理能被证明是没有bug的，但也不能说明系统的质量就足以达到交付的要求，毕竟用户是通过UI操作系统，而不是通过公用API与系统交互的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在软件设计界有一句很经典的话：“大部分的软件耦合问题，都可以通过在中间加一层解决”。所谓的客户端驱动模式，也是通过在中间加一层来解决测试脚本与系统UI高耦合的问题的，我们姑且把中间的这层成为客户端驱动层。下图是使用客户端驱动模式设计的自动化测试的架构图：

![client driven](https://phospher.github.io/20130720171826890.jpeg)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以看到，测试脚本并不直接与系统的UI交互，而是通过客户端驱动层交互。这样测试用例便可以从与UI的耦合中解放出来，使用跟业务更接近的方式编写，如下面的测试用例：

```python
def test_testSearchEngineInput(self):
    seTest = SearchEngineProxy()
    seTest.submitTestText('Test')
    result = seTest.getSearchEngineTitle()
    self.assertTrue('Test' in result)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以看到，上面的代码没有与UI交互相关的代码，而是从业务场景的角度，提交文本到搜索引擎（submitTestText），获取搜索结果的标题（getSearchEngineTile），最后验证标题的正确性。所有与UI层交互都被封装到了客户端驱动（SearchEngineProxy）中。顺便提一个，我们也可以使用单元测试框架编写自动化功能测试，比如著名的xUnit系列。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;客户端驱动层则是一个由测试人员与开发人员共同完成的模块。如果需要简单分工的话，可以由测试人员负责设计接口，由开发人员负责实现。这样由于UI变更造成的影响便全部落到了开发人员的工作范围内，从而减少各种沟通和变更的成本。在客户端驱动层，上一篇博客（[论程序员的自我修养——自动化功能测试](https://phospher.github.io/autoTest)）中提到的自动化测试工具就有用武之地了，这就是为什么程序员也需要学会选择自动化测试工具。

## 选择适当的模式

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我无法评价上面两种模式的优劣，对于不同的项目，更应该根据项目的情况加以选择。相信不少程序员一边看客户端驱动模式，一边会皱眉头。谁都不想多维护一个模块的代码，客户端驱动模式也是一种对耦合的妥协，如果系统UI相对简单，通过简单的手工测试也可以发现UI显示逻辑中的bug，就可以直接通过公用API进行自动化测试了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但总有复杂的UI，需要通过自动化测试把UI层也进行测试，这个时候也别为了方便，直接让测试脚本与UI进行交互。毕竟测试脚本不总是由开发人员维护，与UI紧密耦合的测试脚本可能会让测试用例经常失败，严重打击团队的积极性和自动化测试结果的权威性，让自动化测试无法有效地执行下去。