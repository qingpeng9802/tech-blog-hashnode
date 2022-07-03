## Github Copilot 评估

# Github Copilot 评估  

## 优缺点和讨论  

### 优点  

- 大量框架式代码、重复性轮子代码  

    例如，构造复杂对象的方式、简单的带类型的循环体，公开 API 的调用约定。  

    它能迅速提供一些符合语法并且是最接近最佳实践的代码框架，这能有效避免一些潜在的低质量再创作。
    这里往往是最令人惊喜的部分。它能提示一些 API 的最佳实践用的技巧，有助于防止未来潜在的错误。  

    + 显然，其使用巨大语料库带来了这个优势。  

- 提供代码对应的（英语）注释方面非常优秀  

    它能够提供符合业界广泛使用的词和易于理解的句子，并且对正在注释的代码本身的大意理解得很好。  

    例如，它能提供一些正在注释的函数内的参数及其简单的含义。
    但是一旦注释涉及到较复杂的逻辑或提及别的函数和变量（不是正在注释的代码段），提示的正确性就会变差。  

    + 这可能是由于 [1] 2.Evaluation Framework 和 5.Docstring Generation 所导致的，见
    "Each problem includes a function signature, docstring, body, and several unit tests."
    他们使用的评估策略应该能够使得 `docstring` 的（反向）生成效果非常优秀，
    Codex（Copilot 的底层技术）的演示视频也着重演示了这个令人惊喜的特性，即从描述生成代码。

- 调试目的的格式字符串  

    这些提示可以节省很多修改字符串的时间，且准确度很高。  

    + 同上，很可能是注重 `docstring` 和 `unit tests` 的结果。  

### 缺点  

- 缺失对 “代码的” 上下文的理解  

    它很多时候生成的代码对于英语文本表现出了很高的理解程度，能够有效的 refer 上下文的词义，
    但它对于代码本身的逻辑表现出了很差的理解。  

    比如，对于在良好命名的变量数组进行的操作，它能生成很好的变量命名和正确的代码相对位置，即正确的抽象语法树 AST，但是对数组的操作是完全错误的，包括但不仅限于越界访问和不适当的 `push()/pop()`。  
    
    类似的情况也出现在了使用对象上面，它对引用成员函数/变量非常准确，但是不了解如何正确的操作变量和调用合适的方法，
    包括但不仅限于木讷的传参（根据函数名盲目的传参，类型正确但意义错误）和对调用结果进行匪夷所思的操作。  

    + 这有一点像是 [1] 6.Limitations 提到的 "have difficulty with binding attributes to objects ... when the number of operations and variables in the docstring is large."  
  

- 对于上下文的理解集中在靠近代码的位置，有一种传统 NLP 模型的 “就近原则” 的风味（在使用 Self-Attention 结构的情况下很奇怪），而实质上在代码语义上靠近的变量/函数无法被正确理解和引用。  

    + [1] 6.Limitations 提到了类似的问题 "To concretely illustrate model performance degradation as docstring length increases"
    比如，对于多次使用的类、类型，它不提供 “方法相似的” 关于如何进行初始化和调用的方式，而是倾向于提供 “位置相似的” （目前代码位置旁边使用过几次的）方式。  

        这有时候非常的令人困扰，特别是因为它容易造成低级错误，混淆程序员的想法 ([1] 7.2 Misalignment). 并且，不支持跨文件的语义使得这个问题雪上加霜。

- 不支持跨文件。  

    + 实现所限  

- 对于需要高度优化的代码、高度创新性的代码、无能为力。  

    + 对于带有技巧性细节的代码不是 Copilot 的期望工作范围，可以理解。  


## 语言体验  

当我写 Python 的大多数情况下是关闭它的，因为 Python 的代码是信息密度比较大的，错误的提示很容易打断思路。
特别是拥有动态类型意味着 Python 有一定的便利性取舍，Copilot 倾向于提供类型较窄的代码段，额外的修改成本（例如修改类型注释）过高。  

比较良好的情况是在我写 TypeScript 的情况下开启它，因为本身 TypeScript 的语句较为冗余，Copilot 在这里的代码框架很有帮助。但是在一些情况下提示的代码会带来一些困扰，[1] 7.2 Misalignment 讨论了这种行为，推荐看似不错但错误的代码实际上是浪费了程序员的时间。
很多情况下，我还需要对提示的代码进行一遍人工类型检查（更糟糕的情况是还需要人工语义检查），这不亚于直接构建代码的时间成本。  

我参与了 Technical Preview, 但实际上并没有在整个 Technical Preview 期间都使用它，因为我的体验并没有给我带来依赖感。它只是聊胜于无。所以如果你有意向要购买它，请一定要先进行免费试用，因为它的表现很可能并没有你想象的这么好。

## 技术分析  

它表现出来的行为有一种基于数据的机器学习的风味，实质上不是很擅长严格的因果推断。它的提示逻辑更像是 *This is how they do,* 而不是 *This is correct.* 它更像是一个博览群书的平均水平程序员，而不是一个计算机科学专家。   

编程语言和自然语言的差别之一就是编程语言拥有高度结构化的语法和逻辑特征，用 NLP 的方法来学习，实质上学习了一个专精 “语言特征” 的知识，
而不是一个含有 “语言学结构” 的知识，这对于需要高准确度的代码提示是有危害的。  

[1] 8.Related Work 提到了 
"Two popular approaches to neural program learning are *program induction* and *program synthesis*," 分别是
"a model generates program outputs directly from a latent program representation" 和
"a model explicitly generates a program, usually from a natural language specification. One of the most popular classical approaches used a probabilistic context free grammar (PCFG) to generate a program’s abstract syntax tree (AST)." Codex 的方法属于 "synthesized without passing through an AST representation [by Transformers]"（实际上我更倾向于包含 AST 的方法）。  

### tree-sitter  

Copilot 没有在广告中提及的，有趣的是，它实际上在内部采用了 [tree-sitter](https://tree-sitter.github.io/tree-sitter/)，一种能为多语言生成 AST 的 parser. Copilot 内部有 Go, JavaScript, TypeScript, Python 和 Ruby 的语法文件并显示出了通过调用 `wasm` 文件来生成代码文本的 AST 的行为。可以观察到，在几乎所有情况下，Copilot 生成的代码的 AST 结构都是正确无误的，这要归功于 tree-sitter。  

观察到，在提示的代码内更改变量名时，新的提示中的 AST 结构内同义的变量名也会被更改，这意味着 Copilot 在提示时是保持了 AST 信息的。通过这个观察，可以推断出一些 Copilot 内 tree-sitter 的工程结构。如果假设 Copilot 没有使用较底层的或更改后的 Codex API, 那么 Codex API 应该是得到纯文本输入后产生纯文本输出（Top ranking 的可能结果），即 Codex API 不包含 AST 信息。然后，Copilot 可能通过 tree-sitter 来对结果进行过滤与加权（因为 Copilot 有显示 ranking results 的特性）。这个过程能使得来自 Copilot 的结果都是 AST 层面正确的，这正是 Copilot 能工业化的关键。  

然而，tree-sitter 生成的 AST 信息可能同时也给结果带来了一定的负面作用，生成的 AST 只包含 [@justinmk](https://www.reddit.com/r/neovim/comments/ecwvop/comment/fbmhlq4/?utm_source=share&utm_medium=web2x&context=3): "structured syntax tree", 而不是包含 semantic information （至少不是 fully）。这可能导致一些权重的错误分配，并加重 [1] 6.Limitations 中提到的问题。  

对这个话题感兴趣的话，还可以阅读 [2], 这篇文章用 tree-sitter 融合 CodeT5 方法来改写代码获得了较好的效果，具有一定的启发性。


## 参考文献   
[1] [Evaluating Large Language Models Trained on Code](https://arxiv.org/pdf/2107.03374.pdf)  
[2] [NatGen: Generative pre-training by "Naturalizing" source code](https://arxiv.org/pdf/2206.07585.pdf)  

### 利益冲突  
无。作者得到了 access to Technical Preview of Github Copilot.  

### 许可证  
版权所有 (C) 2022 Qingpeng Li  
本作品采用 [署名-非商业性使用-禁止演绎 4.0 国际 (CC BY-NC-ND 4.0) 许可证](https://creativecommons.org/licenses/by-nc-nd/4.0/) 进行许可。  