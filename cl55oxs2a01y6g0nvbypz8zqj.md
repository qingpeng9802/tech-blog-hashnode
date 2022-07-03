## GitHub Copilot Evaluation

# GitHub Copilot Evaluation  

## Pros, Cons, and Discussion  

### Pros  

- A large amount of framework-like code, repetitive wheel code  

    For example, the way to construct complicated objects, a simple typed loop body, and the call convention of pubic APIs.  

    It can quickly provide some framework-like code that is correct in syntax and closest to best practices, which can effectively avoid some potential low-quality re-creation. Here is often the most surprising part. It can prompt some tricks for API best practices to help prevent potential errors in the future.  

    + Obviously, using a huge corpus brings this advantage.  

- It is very excellent in providing comments for corresponding code.  

    It can provide words that are widely used in the industry and sentences that are easy to understand, and it has a good understanding of the general meaning of the code is commented.  

    For example, it can provide the simple meaning of the parameters being commented in the function. However, once the comment involves more complicated logic or refers to other functions and variables (not the code segment being commented), the accuracy of the prompts will become worse.  
    
    + This might be due to [1] 2.Evaluation Framework and 5.Docstring Generation, see "Each problem includes a function signature, docstring, body, and several unit tests.". The evaluation strategy they used should be able to make the (reversed) generation of `docstring` very excellent. The demonstration video of Codex (the backbone of Copilot) also focuses on this amazing feature that is generating code from the description.  

- The format string for debugging purposes  

    These prompts can save a lot of time to modify the string, and the accuracy is very high.  

    + As above, this is probably due to the emphasis on `docstring` and `unit tests` in the evaluation strategy.  

### Cons

- Lack of understanding of the context of "code".   

    Most of the time, the generated code shows a high degree of understanding of the English text, and it can effectively refer to the word meaning of the context, however, it shows a poor understanding of the logic of the code itself.  

    For example, for the operations on a well-named variable array, it can generate good names for variables and correct code relative position, that is, a correct abstract syntax tree (AST), however, the operations on the array are completely wrong, including but not limited to out-of-bounds array accesses and improper `push()/pop()`.   

    A similar situation also occurs in the use of objects. It is very accurate to reference member functions/variables, but it does not know how to operate variables correctly and call appropriate methods, including but not limited to dull parameter passing (blindly passing parameters according to the function names with correct types but wrong meanings) and strange operations on the result of the call.  

    + This is a bit like [1] 6.Limitations mentioned, "have difficulty with binding attributes to objects ... when the number of operations and variables in the docstring is large."  
  
- The understanding of the context focuses on the position close to the code, which has the taste of the "proximity principle" of the traditional NLP model (it is strange when using the Self-Attention structure), and the variables/functions that are close to the code semantics in nature cannot be correctly understood and referenced.  

    + [1] 6.Limitations mentioned a similar problem "To concretely illustrate model performance degradation as docstring length increases". For example, for classes and types used many times, it does not provide "method-similar" ways about how to initialize and call, but it tends to provide "location-similar" ways (ways used several times next to the current code location).  

        This is sometimes very disturbing, especially because it is easy to cause low-level errors, and confuse programmers' ideas ([1] 7.2.Misalignment). Also, it does not support cross-file semantics, which makes this problem even worse.  

- Cross-file is not supported.  

    + Due to its limitation of implementation.  

- No working with highly optimized code and highly creative code.  

    + Understandably, Copilot would not be expected to work with the code with highly technical details.  

## Language Experience

I turn it off in most cases when I write Python since Python code has a high information density, and wrong prompts are easy to interrupt my idea. In particular, having dynamic types means that Python has certain convenience trade-offs. Copilot tends to provide code segments with narrow types, and the additional modification cost (such as modifying type annotations) is too high.  

The better case to start it is when I write TypeScript. Because the statements of TypeScript are redundant, Copilot's code framework here is very helpful. However, in some cases, the prompt result will bring some trouble, [1] 7.2 Misalignment discussed this behavior, recommending code that looks good but is wrong is actually wasting programmers' time. In many cases, I also need to perform a type check manually on the prompt result (in an even worse case, a manual semantic check is also needed), which is no less than the time cost of building the code directly.  

I participated in the Technical Preview, but actually, I didn't use it all the time in the Technical Preview, because my experience didn't bring me feelings of dependency. It is just better than nothing. Thus, if you are interested in purchasing it, please be sure to take its free-trial first since its performance may not be as good as you think.


## Technical Analysis

The behavior Copilot shows has a taste of data-based machine learning. In fact, it is not very good at strict causal inference. Its prompt logic is more like *This is how they do,* not *This is correct.* It is more like a widely-read average-level programmer instead of a computer science expert.  

One of the differences between programming languages and natural languages is that programming languages have highly structured grammatical and logical features. Learning with the NLP method essentially learns knowledge specialized in "language features" instead of knowledge containing "linguistic structure", which is harmful to code prompts that need high accuracy.  

[1] 8.Related work mentioned "Two popular approaches to neural program learning are *program induction* and *program synthesis*," which are "a model generates program outputs directly from a latent program representation" and 
"a model explicitly generates a program, usually from a natural language specification. One of the most popular classical approaches used a probabilistic context free grammar (PCFG) to generate a program’s abstract syntax tree (AST)." The method of Codex belongs to "synthesized without passing through an AST representation [by Transformers]" (actually I prefer to the method that includes AST).

### tree-sitter

Copilot didn't mention this in the advertisement, interestingly, it actually uses [tree-sitter](https://tree-sitter.github.io/tree-sitter/) internally, a parser that can generate AST for multiple languages. Copilot has grammar files of Go, JavaScript, TypeScript, Python, and Ruby in its build. Also, it shows the behavior that generating AST from code text by calling `wasm` file. It can be observed that in almost all cases, the AST structure of the code prompted by Copilot is correct, this credit belongs to tree-sitter.  

Observed that when changing the variable name within the prompt, the synonymous variable names in the AST structure will also be changed in the new prompt, which means that Copilot keeps the AST information when prompting. From this observation, we can infer some engineering structures of tree-sitter in Copilot. If it is assumed that Copilot does not use the lower-level or modified Codex API, the Codex API should produce plain-text output (Top ranking possible results) after obtaining plain-text input, that is, Codex API doesn't involve AST information. Then, Copilot might filter and weight the results through tree-sitter (since Copilot has the feature of displaying ranking results). This process can make the results from Copilot correct at the AST level, which is the key for Copilot can be used in the industry.   

However, the AST information generated by tree-sitter may also bring some negative effects to the results of Copilot. The generated AST only contains [@justinmk](https://www.reddit.com/r/neovim/comments/ecwvop/comment/fbmhlq4/?utm_source=share&utm_medium=web2x&context=3): "structured syntax tree", rather than semantic information (at least not fully). This might cause some wrong weight assignments and aggravate the problem mentioned in [1] 6.Limitations.  

If you are interested in this topic, you can also read [2] which is enlightening, the paper "naturalizes" the code with the tree-sitter and CodeT5 method, and they have achieved good results.  

## References  
[1] [Evaluating Large Language Models Trained on Code](https://arxiv.org/pdf/2107.03374.pdf)  
[2] [NatGen: Generative pre-training by "Naturalizing" source code](https://arxiv.org/pdf/2206.07585.pdf)  

### Conflict of Interest  
No. The author got the access to Technical Preview of Github Copilot.   

### License  
Copyright (C) 2022 Qingpeng Li  
This work is licensed under a [Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0) License](https://creativecommons.org/licenses/by-nc-nd/4.0/).  
