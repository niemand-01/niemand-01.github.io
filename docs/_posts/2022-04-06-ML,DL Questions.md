---
title: "ML DL Questions"
excerpt_separator: "<!--more-->"
categories:
  - AI
tags:
  - Interview Questions
---

Some close-end preparation for interview tomorrow.

## Git Question

- Difference between git rebase and git merge
  - The difference can be demonstrated by these 2 graphs:
    ```
    git merge feature main

    or 

    git checkout feature
    git merge main
    ```
    ![merge commit](https://wac-cdn.atlassian.com/dam/jcr:4639eeb8-e417-434a-a3f8-a972277fc66a/02%20Merging%20main%20into%20the%20feature%20branh.svg?cdnVersion=303)
    ```
    git checkout feature
    git rebase main
    ```
    ![rebase commit](https://wac-cdn.atlassian.com/dam/jcr:3bafddf5-fd55-4320-9310-3d28f4fca3af/03%20Rebasing%20the%20feature%20branch%20into%20main.svg?cdnVersion=303)

    Verbally put, merge is combine the codes together from both main branch and feature branch, but branch itself doesnt collapse.

    Rebase option at the other side, destroys the branch by concatenating the branch together into single branch, where all the branches other than "base" branch will be rewritten based on the latest "base" commit. 
- head
  - A head is nothing but a reference to the last commit object of a branch.
- git add
  - this command adds files and changes to the index of the existing directory.
- git pull vs git fetch
  - git pull = git fetch + git merge。
- basic git commiting process
  ![basic](https://segmentfault.com/img/bVbtc0c?w=655&h=645)
- git stash
  - git stash apply command is used for bringing the works back to the working directory from the stack where the changes were stashed using git stash command.
  - This helps the developers to resume their work where they had last left their work before switching to other branches.
- git diff vs git status
  - git diff works in a similar fashion to git status with the only difference of showing the differences between commits and also between the working directory and index.


## Tensorflow Question
- Tensorboard start
  ```
  %load_ext tensorboard # (On Jupyter notebook or Colab)
  %tensorboard --logdir logs/fit
  ```
- Eager execution vs. Graph Execution
  - Since eager execution runs all operations one-by-one in Python, it cannot take advantage of potential acceleration opportunities. But Graph Execution can run in parallel, even in sub-operation level.
  - Best practice:
    - code in eager, run in graph(tf.function)
  - ref: [here](https://towardsdatascience.com/eager-execution-vs-graph-execution-which-is-better-38162ea4dbf6)


- type of tensor
  - Constant tensors
  - Variable tensors
  - Placeholder tensors

- overfitting
  - Batch normalization
  - Regularization technique
  - Dropouts
  - Gradient cutting

- application
  - Time series analysis
  - Image recognition
  - Voice recognition
  - Video upscaling
  - Test-based applications

- word embedding (word2vec)
  - The Continuous Bag of Words model
  - The Skip-Gram model

- Distribution 
  - Beta
  - Bernoulli
  - Chi2
  - Dirichlet
  - Gamma
  - Uniform
- type1,2 error
  -  Type 1 errors refer to the occurrence of a false positive outcome, and Type 2 errors denote the occurrence of a false negative value when performing complex computations.

- optimizers
  - AdaDelta
  - AdaGrad
  - Adam
  - Momentum
  - RMSprop
  - Stochastic Gradient Descent

- loss
  - Numerical loss functions:
    - L1 loss
    - L2 loss
    - Pseudo-Huber loss
  - Categorical loss functions:
    - Hinge loss
    - Cross-entropy loss
    - Sigmoid-entropy loss
    - Weighted cross-entropy loss