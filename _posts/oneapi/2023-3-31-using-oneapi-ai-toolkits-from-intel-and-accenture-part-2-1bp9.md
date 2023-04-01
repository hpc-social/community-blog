---
author: oneAPI Community Blog
author_tag: oneapi
blog_subtitle: The latest articles on DEV Community by oneAPI Community (@oneapi).
blog_title: 'DEV Community: oneAPI Community'
blog_url: https://dev.to/oneapi
category: oneapi
date: '2023-03-31 23:21:33'
layout: post
original_url: https://dev.to/oneapi/using-oneapi-ai-toolkits-from-intel-and-accenture-part-2-1bp9
slug: using-oneapi-ai-toolkits-from-intel-and-accenture-part-2
title: Using oneAPI AI Toolkits from Intel and Accenture Part 2
---

<h2>
  
  
  Introduction
</h2>

<p>This is part 2 of our blog series. These posts are really about inspiring others to think of cool projects to do with oneAPI. In the last blog post, we discussed several toolkits that I thought were interesting. If someone produced an idea that uses those toolkits – I would love to know!!<br />
In this blog post, I want to focus on two other AI toolkits and how to use them. <br />
There is one other aspect that we should consider when looking at these toolkits: what do you believe might be unintended consequences?? As an exercise, I am going to go over these toolkits, but I would love to hear what you might believe are unintended consequences that have the potential to be overlooked. </p>

<h2>
  
  
  Personalized Retail Experiences with Enhanced Customer Segmentation
</h2>

<p>Accenture has over 30 toolkits to showcase oneAPI, with more to come.  In this post, I am going to look at this idea of using AI to personalize your shopping experience. I think all of us who do any kind of online shopping know the importance of providing a personalized shopping experience. First, we must understand how one might implement a personal shopping experience. <br />
Today, retailers have an incredible amount of data at their disposal. The analytics market is worth up to $20 billion around the world and is growing at a 19.3 percent Compound Annual Growth Rate (CAGR). Retailers are eager to understand customer behavior so that they can provide a better shopping experience and thus drive brand loyalty. </p>


<p>To access the AI toolkit clone the github repository:<br />
</p>


<p><code>$ git clone https://github.com/oneapi-src/customer-segmentation</code><br />
</p>


<p>The reference kit will show how to analyze customer purchasing data and segment it into clusters based on customer behavior. It will also show you how to optimize the reference solution using the Intel Scikit-Learn extension.</p>


<p>The reference example uses an experimental dataset. The dataset is a set of 500k transactions covering 4000 customers from a UK multinational online retailer over a period of a year. The dataset is fed into <a href="https://en.wikipedia.org/wiki/K-means_clustering">KMeans</a> and <a href="https://en.wikipedia.org/wiki/DBSCAN">DBSCAN</a> algorithms to label cluster based on different customer behaviors.</p>


<p>Try it out and send me some feedback. Also, keep in mind the challenge of what could go wrong. (disclaimer: I don’t know myself - I’m curious to hear theories)</p>


<h2>
  
  
  Faster session Notes with Speech-to-Text AI for Healthcare Providers
</h2>

<p>My second example is going back to healthcare. Always a fun one. The same challenge as in the previous one.</p>


<p>The premise for this is that mental health providers are required to document their sessions using progress notes. These recorded sessions then need to be transcribed into written notes, and be stored for later reference.</p>


<p>Managing these notes can take quite a bit of time. The idea, then, is to take these recorded notes and feed them to a speech-to-text AI algorithm and provide a summary. This summary can then be used to coordinate care, creating a paper trail, compliance, and keeping track of the state of the client.</p>


<p>By reducing the book keeping, a therapist would have more time for their patients or the capacity to see more patients. Given the shortage of mental health professionals, being able to be more efficient and allowing more “human”contact  time will help mental health professionals provide their clients with better care.</p>


<p>You can find the code for this implementation at:<br />
</p>


<p><code>$ git clone  https://github.com/oneapi-src/ai-transcribe</code><br />
</p>


<p>The high level overview of this implementation is something like this. The conversion from speech to text is achieved by using a sequence-to-sequence framework called Fairseq. Sequence-to-sequence modeling is a type of machine learning that is commonly built to create summaries, text translations and so on. It was initially conceived by Google. <a href="https://github.com/facebookresearch/fairseq">Fairseq</a> is an open source  sequence-to-sequence framework from Facebook.</p>


<p>The process is described like this:</p>


<ul>
<li>Take your dataset of unstructured audio samples</li>
<li>Run it through a data preprocessing using Fairseq modeling</li>
<li>Using <a href="https://www.geeksforgeeks.org/generative-adversarial-network-gan/">GAN</a> you create a trained model using both the training data and the pre-processing data.</li>
<li>Apply <a href="https://www.datacamp.com/blog/what-is-machine-learning-inference">inference</a> to generate the text.</li>
</ul>

<p>I think one of the more interesting parts of this pipeline is the GAN - which is described as two algorithms: one as a generator and the other as a test. The two work against each other until they both end up with the same dataset that, ostensibly, is accurate.</p>


<p>One other piece that is missing as part of training the algorithm is a database of English text corpus data. This database contains speech audio files and text transcription. It is used to create a relationship between an audio signal and phonemes as part of speech recognition.</p>


<p>Where GAN comes in is that a neural network is trained to generate what it thinks are the representations of the phonemes as opposed to the real world data obtained from the corpus data. The other neural network is trained on the corpus data and acts as the validator - as the two neural networks work with each other - the generator portion of the neural network will finally produce the results as expected by the other neural network.</p>


<p>It is through this that we can validate that the output is correct.</p>


<p>The entire software to do this is all open source - I would love to hear from people who have tried it and share what their results were!</p>


<p>I think it would be interesting to train this with humans to determine how accurate it is so you can fully train the corpus and generator algorithm for better results.</p>


<h2>
  
  
  Summary
</h2>

<p>I’ve reviewed two Accenture toolkits that demonstrate how AI can be used practically with real examples. Being a newcomer in this area, there is so much that I don’t know. Ironically, I use chatGPT to help explain some of the salient bits about how GAN works vis-a-vis audio data to really understand what was happening, especially in regards to mapping with words and phonemes.</p>


<p>Looking forward to people’s responses to this post and enjoying a great conversation about AI, its potential uses and applications by using real world examples.</p>


<h2>
  
  
  Call to Action
</h2>

<p>Has this blog post inspired you to write something based on the oneAPI AI toolkits? Let me know - I would love to know how it works out for you!</p>