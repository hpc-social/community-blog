---
author: oneAPI Community Blog
author_tag: oneapi
blog_subtitle: The latest articles on DEV Community by oneAPI Community (@oneapi).
blog_title: 'DEV Community: oneAPI Community'
blog_url: https://dev.to/oneapi
category: oneapi
date: '2023-08-10 22:30:31'
layout: post
original_url: https://dev.to/oneapi/migrating-reduction-operations-to-sycl-in-a-molecular-docking-application-5fkp
slug: migrating-reduction-operations-to-sycl-in-a-molecular-docking-application
title: Migrating reduction operations to SYCL in a molecular docking application
---

<p>I completed porting of a molecular docking application from CUDA to SYCL using the Intel® DPC++ Compatibility Tool (Compatibility Tool) in June 2021. Let me share selected techniques that I used without delving into the details of the docking application. If you want to learn how to use this tool to migrate CUDA applications to SYCL, please refer to [1].</p>


<p>The Compatibility Tool adds comments in the code where manual migration may be required. Typically, the manual changes required fall into two categories. First, changes are required for the code to compile and make the code functionally correct. Other changes are necessary to get better performance. Here, I will cover code that uses the operation of 'reduction'. Reductions are frequently used in High Performance Computing and scientific applications and can be performance hotspots. The first example finds the sum of integers and the second finds the minimum of floats and the identifier of the run that corresponds to the minimum.</p>


<h2>
  
  
  Integer Reductions to find the number of evaluations
</h2>

<p>The  docking application performs integer reductions to keep a running count of the number of score evaluations. This reduction is  implemented as a multi-line macro in CUDA as shown below.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight cuda"><code><span class="cp">#define REDUCEINTEGERSUM(value, pAccumulator)     
</span>    <span class="k">if</span> <span class="p">(</span><span class="n">threadIdx</span><span class="p">.</span><span class="n">x</span> <span class="o">==</span> <span class="mi">0</span><span class="p">)</span>     
    <span class="p">{</span>     
        <span class="o">*</span><span class="n">pAccumulator</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>     
    <span class="p">}</span>     
    <span class="n">__threadfence</span><span class="p">();</span>     
    <span class="n">__syncthreads</span><span class="p">();</span>     
    <span class="k">if</span> <span class="p">(</span><span class="n">__any_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">value</span> <span class="o">!=</span> <span class="mi">0</span><span class="p">))</span>     
    <span class="p">{</span>     
        <span class="kt">uint32_t</span> <span class="n">tgx</span>            <span class="o">=</span> <span class="n">threadIdx</span><span class="p">.</span><span class="n">x</span> <span class="o">&amp;</span> <span class="n">cData</span><span class="p">.</span><span class="n">warpmask</span><span class="p">;</span>     
        <span class="n">value</span>                  <span class="o">+=</span> <span class="n">__shfl_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tgx</span> <span class="o">^</span> <span class="mi">1</span><span class="p">);</span>     
        <span class="n">value</span>                  <span class="o">+=</span> <span class="n">__shfl_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tgx</span> <span class="o">^</span> <span class="mi">2</span><span class="p">);</span>     
        <span class="n">value</span>                  <span class="o">+=</span> <span class="n">__shfl_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tgx</span> <span class="o">^</span> <span class="mi">4</span><span class="p">);</span>     
        <span class="n">value</span>                  <span class="o">+=</span> <span class="n">__shfl_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tgx</span> <span class="o">^</span> <span class="mi">8</span><span class="p">);</span>     
        <span class="n">value</span>                  <span class="o">+=</span> <span class="n">__shfl_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tgx</span> <span class="o">^</span> <span class="mi">16</span><span class="p">);</span>     
        <span class="k">if</span> <span class="p">(</span><span class="n">tgx</span> <span class="o">==</span> <span class="mi">0</span><span class="p">)</span>     
        <span class="p">{</span>     
            <span class="n">atomicAdd</span><span class="p">(</span><span class="n">pAccumulator</span><span class="p">,</span> <span class="n">value</span><span class="p">);</span>     
        <span class="p">}</span>     
    <span class="p">}</span>     
    <span class="n">__threadfence</span><span class="p">();</span>     
    <span class="n">__syncthreads</span><span class="p">();</span>     
    <span class="n">value</span> <span class="o">=</span> <span class="o">*</span><span class="n">pAccumulator</span><span class="p">;</span>     
    <span class="n">__syncthreads</span><span class="p">();</span>
</code></pre>

</div>




<p>Let us review what this code is doing:</p>


<ul>
<li>The code is called for each work item (thread) in a work group (warp)</li>
<li>
<em>*pAccumulator</em> is where the final sum is stored summing across all work items</li>
<li>The combination of <em>__threadfence()</em> and <em>__syncthreads()</em> guarantees memory consistency and synchronizes threads in the warp at the point of the call.</li>
<li>The <em>__any_sync()</em> call executes the block for those non-exited threads for which <em>'value != 0'</em>
</li>
<li>The following <em>__shfl_sync</em> calls do a tree-wise summing with the final sum available in the first thread in the warp in variable <em>value</em>
</li>
<li>The <em>value</em> is then added to the Accumulator atomically with <em>atomicAdd</em> and finally all threads assign the sum to the <em>value</em> variable.</li>
</ul>

<p>For more details about these CUDA calls please refer to [2].</p>


<p>The Compatibility tool was not able to automatically migrate this code with the following comments.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/*
DPCT1023:40: The DPC++ sub-group does not support mask options for sycl::ext::oneapi::any_of.

DPCT1023:41: The DPC++ sub-group does not support mask options for shuffle.

DPCT1007:39: Migration of this CUDA API is not supported by the Intel(R) DPC++ Compatibility Tool.
*/
</code></pre>

</div>




<p>However, SYCL supports a rich set of functions for performing reductions. In this case, the reduce_over_group() function in SYCL can be used to create the same functionality as the above code as follows.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
#define REDUCEINTEGERSUM(value, pAccumulator)     
        int val = sycl::reduce_over_group(item_ct1.get_group(), value, std::plus&lt;&gt;());      
        *pAccumulator = val;     
        item_ct1.barrier(sycl::access::fence_space::local_space);
</code></pre>

</div>




<p>The <em>sycl::reduce_over_group</em> is a collective function. The usage of this function simplifies the macro. The function takes the group, the value to be reduced, and the reduction operation which in this case is plus or summation. The function can adapt to varied sizes of work groups in SYCL and will use the best available optimizations available per the compiler and run-time.</p>


<h2>
  
  
  Finding the minimum energy
</h2>

<p>In another part of the application, a block of CUDA threads perform shuffles to find the minimum of scores <em>v0</em> and the corresponding identifier <em>k0</em> of the run in the simulation that is the minimum score. The CUDA code calls a macro WARPMINIMUM2 (not shown) which in turn calls another macro WARPMINIMUMEXCHANGE (shown) with <em>mask</em> set to 1, 2, 4, 8, and 16.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight cuda"><code><span class="cp">#define WARPMINIMUMEXCHANGE(tgx, v0, k0, mask)     
</span>    <span class="p">{</span>     
        <span class="kt">float</span> <span class="n">v1</span>    <span class="o">=</span> <span class="n">v0</span><span class="p">;</span>     
        <span class="kt">int</span> <span class="n">k1</span>      <span class="o">=</span> <span class="n">k0</span><span class="p">;</span>     
        <span class="kt">int</span> <span class="n">otgx</span>    <span class="o">=</span> <span class="n">tgx</span> <span class="o">^</span> <span class="n">mask</span><span class="p">;</span>     
        <span class="kt">float</span> <span class="n">v2</span>    <span class="o">=</span> <span class="n">__shfl_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">v0</span><span class="p">,</span> <span class="n">otgx</span><span class="p">);</span>     
        <span class="kt">int</span> <span class="n">k2</span>      <span class="o">=</span> <span class="n">__shfl_sync</span><span class="p">(</span><span class="mh">0xffffffff</span><span class="p">,</span> <span class="n">k0</span><span class="p">,</span> <span class="n">otgx</span><span class="p">);</span>     
        <span class="kt">int</span> <span class="n">flag</span>    <span class="o">=</span> <span class="p">((</span><span class="n">v1</span> <span class="o">&lt;</span> <span class="n">v2</span><span class="p">)</span> <span class="o">^</span> <span class="p">(</span><span class="n">tgx</span> <span class="o">&gt;</span> <span class="n">otgx</span><span class="p">))</span> <span class="o">&amp;&amp;</span> <span class="p">(</span><span class="n">v1</span> <span class="o">!=</span> <span class="n">v2</span><span class="p">);</span>     
        <span class="n">k0</span>          <span class="o">=</span> <span class="n">flag</span> <span class="o">?</span> <span class="n">k1</span> <span class="o">:</span> <span class="n">k2</span><span class="p">;</span>     
        <span class="n">v0</span>          <span class="o">=</span> <span class="n">flag</span> <span class="o">?</span> <span class="n">v1</span> <span class="o">:</span> <span class="n">v2</span><span class="p">;</span>     
    <span class="p">}</span>
</code></pre>

</div>




<p>The <em>__shfl_sync</em> provides a way of moving a value from one thread to other threads in the warp in one instruction. In this code snippet <em>__shfl_sync</em> gets the <em>v0</em> or <em>k0</em> value from the thread identified by the <em>otgx</em> mask and saves it in <em>v2</em>, <em>k2</em> variables. We then compare <em>v1</em> with <em>v2</em> to set <em>flag</em> and eventually store the minimum in <em>v0</em> and the run identifier for this minimum in <em>k0</em>.</p>


<p>Compatibility Tool could not completely migrate this code and included this comment as the reason it could not. However, Compatibility Tool correctly replaced the <em>__shfl_sync</em> call with SYCL <em>shuffle</em> call as shown in the below diff which shows the manual change.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/*
DPCT1023:57: The DPC++ sub-group does not support mask options for shuffle.
*/
</code></pre>

</div>




<p>This comment indicates that the <em>shuffle</em> call in SYCL does not use a mask as shown below.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight diff"><code><span class="err">#define</span> WARPMINIMUMEXCHANGE(tgx, v0, k0, mask)     
        {     
                float v1 = v0;     
                int k1 = k0;     
                int otgx = tgx ^ mask;     
<span class="gd">-               float v2 = item_ct1.get_sub_group().shuffle(energy, otgx);     
</span><span class="gi">+               float v2 = item_ct1.get_sub_group().shuffle(v0, otgx);  
</span><span class="gd">-               int k2 = item_ct1.get_sub_group().shuffle(bestID, otgx);     
</span><span class="gi">+               int k2 = item_ct1.get_sub_group().shuffle(k0, otgx);     
</span>                int flag = ((v1 &lt; v2) ^ (tgx &gt; otgx)) &amp;&amp; (v1 != v2);     
                k0 = flag ? k1 : k2;     
                v0 = flag ? v1 : v2;     
        }
</code></pre>

</div>




<p>In this case, Compatibility Tool performed incorrect variable substitution for <em>v0</em> and <em>k0</em> in the shuffle calls using <em>energy</em> and <em>bestID</em> variables from the caller function. We manually fixed this by replacing <em>energy</em> with <em>v0</em> and <em>bestID</em> with <em>k0</em>. This bug has been fixed in recent versions of the Compatibility Tool.</p>


<h2>
  
  
  Summary
</h2>

<p>In summary, reduction operations in CUDA applications may not be migrated correctly by the Compatibility Tool. Review the comments provided by the tool to understand if manual migration is necessary and what change might be required. A good understanding of the original CUDA code will then help to make manual changes to develop functionally correct code in SYCL.</p>


<p>[1] <a href="https://www.intel.com/content/www/us/en/docs/dpcpp-compatibility-tool/get-started-guide/2023-1/overview.html">https://www.intel.com/content/www/us/en/docs/dpcpp-compatibility-tool/get-started-guide/2023-1/overview.html</a></p>


<p>[2] <a href="https://developer.nvidia.com/blog/using-cuda-warp-level-primitives/">https://developer.nvidia.com/blog/using-cuda-warp-level-primitives/</a></p>