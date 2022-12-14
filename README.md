# Comcat: A Quality Analysis of Source Code Comments via Comment Categorization
By Skyler Grandel

## Overview
This project aims to automatically provide a qualitative analysis of software comments through a novel classification schema. This schema will make use of recent advancements in natural language processing to systematically classify source code comments by purpose and the information they provide.

### The Problem
It is very difficult to qualitatively assess comments (or really anything) in a systematic or objective way. Therefore, we need criteria by which we might determine if a comment is useful or not. A common approach to this problem is to classify our data and then experimentally determine which classes are the most useful. Therefore, we need a classification schema and a tool to automatically classify comments according to this schema.

### The Solution
To develop a classification schema, we classified comments by the information they provide about the associated code until we reached saturation. This means that we determined what information the comment contained and placed it in the appropriate category; if no appropriate category existed, we created a new category until we were reasonably sure that no more categories existed (saturation in this case was defined as classifying at least 100 comments without discovering a new category). The resulting categories are listed below:

0: Function (A comment that describes a function. Typically a comment is in this cat iff it's javadoc style and it's not a copyright comment)

1: Variable (Describe Variable/Constant) 

2: Functionality (Describe block functionality; usually just summarizes or describes what the following lines of code do; specifically inline)

3: Branch (Describe possible branches and/or preconditions)

4: Reasoning (Describe reasoning behind implementation decisions)

5: Quirk (Random quirk of the code, authors trying to be funny, niche problems the code can run into, etc)

6: Use guidelines (Comments that tell readers how to use a function/container/variable but dont describe their function)

7: Source (Describe Code Source; comments like /\* This code was stolen from \[some stack overflow link\]\*/)

8: Copyright (Copyright and author info)

9: Section (comments like, /\* The following tests are to test foo() \*/ or /\* Setter and Getter methods \*/)

10: Code (commented out code)

11: Task (TODOs, FIXMEs, etc.)

To automatically classify these comments, we fine tuned a roBERTa based classification transformer on a dataset of manually labeled comments. This utilizes the transformer as a feature selector to vectorize the comment and a NN based classifier head.

### Improving the Model
Various different models were evaluated on this task and many different methods were used to improve performance. Multiple classification heads were tested and an SVM with an RBF kernel actually worked the best for a long time (likely due our limited training data). Multiple transformer models were tested as well, but roBERTa achieved the best results. We also performed domain adaptation by fine-tuning the language model on a dataset of over 1 million unlabeled comments, but this failed to improve our results. We also tried various data preprocessing steps, changing the comments to the following forms:

\"/\*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This is the original form \*/    /\*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;of a multiline comment, and it has special characters like +-@ and stars and slashes \*/&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\"

\"&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This is the form&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;of a multiline comment, and it still has special characters like +-@ but no stars or slashes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\"

\"&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This is the form&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;of a multiline comment and it no longer has special characters or stars or slashes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\"

\"This is the form of a multiline comment and it no longer has special characters like or stars or slashes and extra spaces are removed\"

Each of these forms performed about the same with the SVM head, but removing all special characters actually caused the NN head to achieve similar results to the SVM head. However, removing extra spaces tanked the performance for all heads, leading us to the conclusion that the model utilizes the spaces rather than the special characters to interpret the format of the comment. With this change, we attempted domain adaptation again, resulting in a significant improvement in the NN head's performance over that of the SVM head.

The final model (at time of writing) achieves an accuracy of ~85% and an F1 score of ~0.86. Furthermore, if we assume that categories 0 through 3 are "good" comments and categories 4 through 11 are "bad" comments and we thus reduce the problem to a binary classification problem, we achieve an accuracy of ~95%.

## Critical Analysis
### Broader Impact
Readability and comprehension are necessary for the lifelong maintainability of software. Software comments are listed as an important tool for improving this comprehension in many coding guidelines. Yet research has sometimes shown that comments do not necessarily make code more comprehensible, and often experienced developers only skim or even ignore most comments. This disconnect implies that many of the kinds of comments that developers write are largely unhelpful and a waste of time to read and write. In fact, assuming that our dataset is representative, this would indicate that about 40% of comments are in the categories that we assume are not useful. This research attempts to find the comments that are helpful and eliminate those that are not to guide future documentation efforts. Furthermore, researchers today are often concerned with automatic code commenting and summarization tools. These tools should be guided by research that maximizes code comprehension and maintainability, generating the kinds of comments that software developers find to be the most helpful. Therefore, the model produced by this research would be useful in filtering unhelpful comments from training data to produce documentation which this experimentation indicates improves code comprehension. In total, this research will improve guidelines on manual and automatic comment generation for the benefit of the maintainability of all future software.

### Next Steps
The most important next step for improving this model is expanding the training dataset. The current model was only trained on about 500 classified comments, which is nowhere near enough data for a NN effectively learn a classification schema. Furthermore, some of our less common categories have very few training examples, so more data should help round out our predictions. To that end, we are setting up a crowd sourcing task to label as much data as possible. Furthermore, only limited error analysis has been performed, so it might be helpful to generate and analyze a confusion matrix, among other things.

Outside of the model itself, the usefulness of each category of comments still needs to be experimentally determined, so we are in the process of setting up a human study to discover which kinds of comments help developers with different programming tasks.

## Resource Links
The model can be found on [this huggingface page](https://huggingface.co/skylergrandel/Comcat).

Here are a few papers for some background on comments, comment categorization, and automatic comment generation:

Steidl et al. classify comments into seven categories using older ML techniques in [???Quality analysis of source code comments???](https://ieeexplore.ieee.org/document/6613836)

Song et al. identify quality evaluation criteria for comments and survey general methods of comment generation in [???A Survey of Automatic Generation of Source Code Comments: Algorithms and Techniques???](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=8778714)

Khamis et al. automatically evaluate comments on readability, consistency, and completeness of documentation in [???Automatic quality assessment of source code comments: The javadocminer???](https://www.researchgate.net/profile/Juergen-Rilling/publication/221474509_Automatic_Quality_Assessment_of_Source_Code_Comments_The_JavadocMiner/links/0046352d5ad855f900000000/Automatic-Quality-Assessment-of-Source-Code-Comments-The-JavadocMiner.pdf)

This model is based on roBERTa, which you can read about [here](https://arxiv.org/pdf/1907.11692.pdf%5C)

