---
author: oneAPI Community Blog
author_tag: oneapi
blog_subtitle: "The latest articles on DEV Community \U0001F469‍\U0001F4BB\U0001F468‍\U0001F4BB
  by oneAPI Community (@oneapi)."
blog_title: "DEV Community \U0001F469‍\U0001F4BB\U0001F468‍\U0001F4BB: oneAPI Community"
blog_url: https://dev.to/oneapi
category: oneapi
date: '2022-11-18 21:59:08'
layout: post
original_url: https://dev.to/oneapi/how-the-sycl-spec-gets-updated-48ke
slug: how-the-sycl-spec-gets-updated
title: How the SYCL spec gets updated
---

<h2>
  
  
  What is SYCL?
</h2>

<p>From the KHRONOS website (<a href="https://www.khronos.org/sycl/">https://www.khronos.org/sycl/</a>) </p>


<blockquote>
<p>SYCL (pronounced 'sickle') is a royalty-free, cross-platform abstraction layer that enables code for heterogeneous processors to be written using standard ISO C++ with the host and kernel code for an application contained in the same source file. </p>

</blockquote>

<p>In short, SYCL is a C++ extension offered as an industry replacement to CUDA. SYCL allows you to run on many kinds of accelerators like FPGAs, GPUs, and CPUs'- you could scale it from a Raspberry Pi to the latest GPUs and CPUIs on the market. </p>


<p>We will not go too much into SYCL from a code perspective. That will be the focus of future blog posts. In this post, we focus on how SYCL gets updated as an industry standard. </p>


<h2>
  
  
  KHRONOS
</h2>

<p>The caretaker of the SYCL spec is <a href="https://www.khronos.org/">Khronos</a> a non-profit standards body that is home to several projects focused on graphics, machine learning, parallel computing, VR and visual computing. You will note that all these pieces work together to build frameworks. Some of the more visible ones that you might be familiar with are openGL and Vulkan. </p>


<p>Khronos consists of members who represent industry players from companies, non-profits, and individuals. Every project will have a set of members.   </p>


<h2>
  
  
  How a feature becomes part of the spec
</h2>

<p>Let us start with how a feature gets introduced into the spec. Usually the process of adding a feature starts when a member company identifies a problem or an idea internally through feedback, bug reports, or strategic direction. </p>


<p>Once that feature is fleshed out, it is introduced to the SYCL working group. The working group then discusses the proposal as to how the feature evolves, and the proposal gets iterated on until people are satisfied with the changes. The feature is then integrated into the spec. </p>


<p>Once the feature is in the SYCL spec, the working group will ratify the feature with a vote. The vote is important because it means that the output of that feature is now under the protection of the Khronos group, which means that if another member objects because of an IP issue later they will not have grounds to do so, and the implementation of the feature is protected.  </p>


<h3>
  
  
  What happens before a member goes to the working group?
</h3>

<p>Most of the time, the member company already implemented the feature they were looking to integrate into the spec to ensure that the desired feature solves the problem at hand. This also assures the working group that the feature can be implemented. </p>


<p>As a specific example, Intel, when they want to design a feature to extend the SYCL spec, they start a branch in the DPG++ GitHub area and then work on it publicly. Once the implementation is complete, it is considered ready for submission to the working group. </p>


<h2>
  
  
  What happens after approval?
</h2>

<p>After the approval of the preliminary draft of the new spec  which contains the new features, the draft is open for public comments. Usually, there is a 30-45 -day period where Khronos members can inspect the new version of the spec, file IP claims against it, or offer changes. An iterative process happens when public feedback is used to make more changes and clean up. Finally, the new changes are ratified, and the feature implementation is put into the spec. </p>


<p>Keep in mind that this is all happening in parallel as other member projects are also targeting the same spec with their own features. </p>


<h2>
  
  
  Getting the final stamp of approval
</h2>

<p>Once you have finished ratifying the spec and produced a working implementation of the feature you will need to do a performance run using an included conformance suite provided by Khronos. Once that passes, you will be able to put the Khronos logo on your implementation of the updated spec.</p>


<p>That is a summary of how SYCL is updated through a standards body like Khronos.</p>


<h2>
  
  
  Summary
</h2>

<p>I hope this gives you a birds eye view of the process of how the SYCL spec is updated. If you have questions - please ask in the comments! If anyone is interested in Khronos membership - happy to direct.</p>