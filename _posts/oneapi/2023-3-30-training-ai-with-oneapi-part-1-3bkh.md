---
author: oneAPI Community Blog
author_tag: oneapi
blog_subtitle: The latest articles on DEV Community by oneAPI Community (@oneapi).
blog_title: 'DEV Community: oneAPI Community'
blog_url: https://dev.to/oneapi
category: oneapi
date: '2023-03-30 05:54:13'
layout: post
original_url: https://dev.to/oneapi/training-ai-with-oneapi-part-1-3bkh
slug: training-ai-with-oneapi-part-1
title: Training AI with oneAPI part 1
---

<h2>
  
  
  oneAPI and AI
</h2>

<p>My last few blog posts were pretty fun to write. I'm going to change subjects today and talk about AI. We'll learn about what resources are around for those of you who know the particulars about AI and want to start a project, but are interested in delving deeper into the capabilities. </p>


<h2>
  
  
  Accenture and Intel
</h2>

<p>It should be no surprise that Intel is the largest stakeholder in oneAPI, if you've done any kind of research on oneAPI at all. Intel's interest is creating a performative software stack that runs well, not just on Intel platforms, but on any platform. oneAPI's purpose is to be able to take advantage of all the hardware you have on your system and not just the GPU or CPU. </p>


<p>Let's talk about oneAPI and the AI toolkits that have been released over the past 8 months or so. Intel worked with Accenture to release some open source "recipes" for AI. If you've always wanted to delve into AI but have problems on starting a project - this is a great place to ponder what kind of a project you'd want to write based on what problems it solves. These AI kits are a one-stop shop of everything you'd need to get started, including the training data! </p>


<h2>
  
  
  oneAPI AI Toolkits
</h2>

<p>The Intel and Accenture tool kits are all released as open source and can be found on github. The tool kits span a number of industries from telecommunications to health &amp; life sciences to retail and more. They provide a great showcase on how AI is being used today in different industries and what problems they are solving. </p>


<p>The best part of these tool kits is how comprehensive they are. You'll have everything you need to get the toolkit working smoothly, including the source code and training data. </p>


<p>In this blog post, I'll focus on two reference kits and explain how they work. The github page for them is fairly self-explanatory but it's still worth going over them. </p>


<h2>
  
  
  Disease Prediction
</h2>

<p>The first reference kit is <a href="https://github.com/oneapi-src/disease-prediction">disease prediction</a>. This toolkit will look through patient records and look for a possible disease indication. The interesting part of this is the use of NLP (Natural Language Processing) to sort through unstructured data located in patient records. </p>


<p>NLP has been used by healthcare for quite some time - but only now has there been significant investment by healthcare payers. NLP can be used in other areas as well. For instance, understanding what kind of dosage of medication that a patient should take based on their particular genetics! By training on the data of millions of patients, one could really understand the unique properties of the health of individuals and act accordingly. </p>


<p>There is also an interesting social change - the ability to objectively look at patients and their needs, especially women's health needs, means that we can create prediction models based on symptoms that can be followed up on. It's possible to reduce bias in care through such a system - as long as the AI employed is not biased. </p>


<p>To actually pore through patient records, you'd need an NLP that is already pre-trained on reading words. Normally, you'd have to have an algorithm and then use a large number of data sets to get to a point where you'd be able to read natural language. This is why they use the <a href="https://en.wikipedia.org/wiki/BERT_(language_model)">BERT</a> language model. More accurately, a specialized form of BERT called clinicalBERT which includes clinical jargon and medical references. </p>


<p>If you've followed the text of the github repo - the process is pretty simple.  </p>


<ul>
<li><p>You have the clinicalBERT language model which you would use to train your AI using the data from clinical records. </p>
</li>
<li><p>The language model then creates an association with symptoms to predicted disease probabilities. </p>
</li>
<li><p>Now you have a model that you can apply to any set of symptoms with an output of predicted probabilities. </p>
</li>
<li><p>The process of applying data to a model is called <em>model inference</em> </p>
</li>
</ul>

<p>We won't go through the process of building the software and training it here. The <a href="https://github.com/oneapi-src/disease-prediction">repo</a> has some clear steps in how to train your model. </p>


<p>The prerequisites require Python and PyTorch v1.11. The repo also goes on to describe how to optionally use the Intel extensions for Python for better performance. I'd like to also add that these extensions exist temporarily while the process of upstreaming to main line python continues. Rather than wait, the community can enjoy the optimizations now, rather than at some future date. </p>


<p>There are some instructions to do some bench marking - if you're interested in seeing how well it works on various other platforms. </p>


<h2>
  
  
  Increase Mortgage Loan Default Risk Prediction Speed
</h2>

<p>Next, I'll focus on banking and loans. Banks use AI prediction models to determine risk. This is an interesting case study and I'd like to see some comments about this particular scenario because I expect that some of you will have opinions! </p>


<p>The problem statement here is that in Q4 2021, mortgage delinquencies were 4.65% and outstanding balances of unpaid principals was approximately $2.6 trillion dollars. The average time to complete a foreclosure process was 941 days, leading to a result of approximately $7.6 billion in foreclosure costs alone. </p>


<p>What this kit offers is the ability to manage default risk, handle larger data sets, and reduce the underwriting wait time. The kit will improve customer service quality and speed up loan processing. </p>


<p>So, let's take a look at the github repo for <a href="https://github.com/oneapi-src/loan-default-risk-prediction">Loan Default Risk Prediction using XGBoost</a>. </p>


<p>This reference solution follows a similar idea. <a href="https://en.wikipedia.org/wiki/XGBoost">XGBoost</a> XBoost is part of a family of machine learning algorithms that uses decision trees. Specifically, XGBoost uses <a href="https://en.wikipedia.org/wiki/Gradient_boosting">gradient boosting</a> which instead of using one decision tree, it uses an ensemble of <a href="https://en.wikipedia.org/wiki/Decision_tree_learning">decision trees</a>. A decision tree is nothing more than a model that tries to make a prediction basted on a selection of data. </p>


<p>With that background in mind, you can look at the intended data set that is being used with the kind of parameters. </p>


<p>To make it even more interesting, a modification was made to the data set by adding synthetic bias_variable. The idea is to add bias value for each loan - the value is generated randomly. The reason is to demonstrate bias between a protected class and a privileged class. It isn't used as part of training the model. </p>


<p>Another thing that's wonderful about this AI toolkit is how it demonstrates bias model. The section about "Fairness Evaluation" goes into some length about whether the algorithm is fair. This is an important consideration when training AI models and is an area of active research. While AI can be a powerful tool, it can also be a tool that can augment inequalities and inequities and, when used in decision making, can exacerbate and preserve these existing inequalities. </p>


<p>To use the toolkit, you'll need Python v3.9 and XGBoost v0.81 and clone the repo:<br />
</p>


<p><code>git clone https://www.github.com/oneapi-src/loan-default-risk-prediction</code><br />
</p>


<p>use the bash script to set up the environment. </p>


<p>Follow the instructions in the repo on how to get the model running. </p>


<p>I will leave it up to the reader to run this model. Specifically run it many times and observe the fairness metric and see how that changes. </p>


<h2>
  
  
  Call to Action
</h2>

<p>What do you think about the ease of using these tool kits so far? I would love to hear your thoughts and see if the results were intriguing to you. The bias factor in the last AI toolkit is something that intrigues me - is the model of fairness really fair? How would you change any of these models? </p>


<p>Hit me up on the comments, let's have a conversation! </p>