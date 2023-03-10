---
author: oneAPI Community Blog
author_tag: oneapi
blog_subtitle: "The latest articles on DEV Community \U0001F469‍\U0001F4BB\U0001F468‍\U0001F4BB
  by oneAPI Community (@oneapi)."
blog_title: "DEV Community \U0001F469‍\U0001F4BB\U0001F468‍\U0001F4BB: oneAPI Community"
blog_url: https://dev.to/oneapi
category: oneapi
date: '2023-01-10 23:08:36'
layout: post
original_url: https://dev.to/oneapi/modern-software-development-tools-and-oneapi-part-2-4bjp
slug: modern-software-development-tools-and-oneapi-part-2
title: Modern Software Development Tools and oneAPI Part 2
---

<h1>
  
  
  Modern Software Development Tools and oneAPI Part 2
</h1>

<p>This is part 2 of using a modern open source toolchain to build oneAPI based applications. If this is the first time you're seeing this post, you can read part one <a href="https://dev.to/oneapi/modern-software-development-tools-and-oneapi-part-1-40km">here</a></p>


<p>In part one, we talked about how easy it is to put a container together that will allow you to have everything you need to compile and build an oneAPI application. However, the sample code was compiled simply with:</p>


<p><code>clang++ -fsycl simple-oneapi.cpp -o simple-sycl</code></p>


<p>Not particularly interesting, is it?</p>


<p>This post will focus on building this same code using <a href="https://mesonbuild.com/">Meson</a>. Meson focuses on simplicity, is a build system generator, but has a concept of back-ends where it can generate whatever the back-end defines. Together with the backend creates a featureful build system. Currently, the default back-end is the fabulous ninja on the Linux platform. Ninja is a command runner that is extremely fast compared to something like Make. Meson also supports xcode and vscode as back-ends allowing to easily use Meson on MacOS and Windows.</p>


<p>Meson just recently hit 1.0 after ten years of development. A wonderful milestone. You can read about it in this <a href="https://nibblestew.blogspot.com/2022/12/after-exactly-10-years-meson-100-is-out.html">blog post</a>.</p>


<h2>
  
  
  Installing Meson and Ninja
</h2>

<p>To get Meson working, we first need to install its prerequisites. This is fairly easy to do, assuming that you are not in the oneAPI container.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ distrobox enter oneapi
$ pip3 install --user meson
$ pip3 install --user ninja
</code></pre>

</div>




<p>You are, of course, welcome to install them site-wide so you don't clutter up your home directory and also, have a clear delineation between toolchain in the container and in your regular host. Make sure that you set your PATH to include /usr/local/bin.</p>


<p>Now we have Meson and the ninja build system ready to go!</p>


<h2>
  
  
  Building a simple sycl app with Meson
</h2>

<p>Let's start with a fresh directory. The sample sycl code is simple and easy enough to turn into a Meson-based project. Please keep in mind that this is not meant to be a complete tutorial on Meson. If you have questions, please respond to the blog post and I will do my best to answer.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ mkdir -p $HOME/src/simple-oneapi
$ cd $HOME/src/simple-oneapi
</code></pre>

</div>




<p>The first thing to do is to write a meson.build file in the top level directory. The meson.build file will contain the following:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>project('simple-oneapi', ['cpp', 'c'],
        version: '0.1.0',
    meson_version: '&gt;= 0.59.0',
  default_options: [ 'warning_level=2', 'werror=false', 'cpp_std=gnu++2a', ],
)
subdir('src')
</code></pre>

</div>




<p>This file will define the project and its prerequisites. From here, you can see that we are defining a project that will use the C++ language version and requires a Meson version above 0.59.0. You can also define what the default compiler options are for the project. Finally, it defines that there are other sub directories with code.</p>


<p>For those who use make, it should be familiar to have each sub-directory have its own meson.build files to define how the code in that directory will be built. This will be no different.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ mkdir src
$ cd src
</code></pre>

</div>




<p>Since we defined a src sub-directory, we're going to go ahead and create one and then add our simple oneapi source code and a meson file.</p>


<p>Create a file called simple-oneapi.cpp with this source code:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="cp">#include</span> <span class="cpf">&lt;sycl/sycl.hpp&gt;</span><span class="cp">
</span>
<span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>
  <span class="c1">// Creating buffer of 4 ints to be used inside the kernel code</span>
  <span class="n">sycl</span><span class="o">::</span><span class="n">buffer</span><span class="o">&lt;</span><span class="n">sycl</span><span class="o">::</span><span class="n">cl_int</span><span class="p">,</span> <span class="mi">1</span><span class="o">&gt;</span> <span class="n">Buffer</span><span class="p">(</span><span class="mi">4</span><span class="p">);</span>

  <span class="c1">// Creating SYCL queue</span>
  <span class="n">sycl</span><span class="o">::</span><span class="n">queue</span> <span class="n">Queue</span><span class="p">;</span>

  <span class="c1">// Size of index space for kernel</span>
  <span class="n">sycl</span><span class="o">::</span><span class="n">range</span><span class="o">&lt;</span><span class="mi">1</span><span class="o">&gt;</span> <span class="n">NumOfWorkItems</span><span class="p">{</span><span class="n">Buffer</span><span class="p">.</span><span class="n">size</span><span class="p">()};</span>

  <span class="c1">// Submitting command group(work) to queue</span>
  <span class="n">Queue</span><span class="p">.</span><span class="n">submit</span><span class="p">([</span><span class="o">&amp;</span><span class="p">](</span><span class="n">sycl</span><span class="o">::</span><span class="n">handler</span> <span class="o">&amp;</span><span class="n">cgh</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Getting write only access to the buffer on a device</span>
    <span class="k">auto</span> <span class="n">Accessor</span> <span class="o">=</span> <span class="n">Buffer</span><span class="p">.</span><span class="n">get_access</span><span class="o">&lt;</span><span class="n">sycl</span><span class="o">::</span><span class="n">access</span><span class="o">::</span><span class="n">mode</span><span class="o">::</span><span class="n">write</span><span class="o">&gt;</span><span class="p">(</span><span class="n">cgh</span><span class="p">);</span>
    <span class="c1">// Executing kernel</span>
    <span class="n">cgh</span><span class="p">.</span><span class="n">parallel_for</span><span class="o">&lt;</span><span class="k">class</span> <span class="nc">FillBuffer</span><span class="o">&gt;</span><span class="p">(</span>
        <span class="n">NumOfWorkItems</span><span class="p">,</span> <span class="p">[</span><span class="o">=</span><span class="p">](</span><span class="n">sycl</span><span class="o">::</span><span class="n">id</span><span class="o">&lt;</span><span class="mi">1</span><span class="o">&gt;</span> <span class="n">WIid</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Fill buffer with indexes</span>
        <span class="n">Accessor</span><span class="p">[</span><span class="n">WIid</span><span class="p">]</span> <span class="o">=</span> <span class="p">(</span><span class="n">sycl</span><span class="o">::</span><span class="n">cl_int</span><span class="p">)</span><span class="n">WIid</span><span class="p">.</span><span class="n">get</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
        <span class="p">});</span>
  <span class="p">});</span>

  <span class="c1">// Getting read only access to the buffer on the host.</span>
  <span class="c1">// Implicit barrier waiting for queue to complete the work.</span>
  <span class="k">const</span> <span class="k">auto</span> <span class="n">HostAccessor</span> <span class="o">=</span> <span class="n">Buffer</span><span class="p">.</span><span class="n">get_access</span><span class="o">&lt;</span><span class="n">sycl</span><span class="o">::</span><span class="n">access</span><span class="o">::</span><span class="n">mode</span><span class="o">::</span><span class="n">read</span><span class="o">&gt;</span><span class="p">();</span>

  <span class="c1">// Check the results</span>
  <span class="kt">bool</span> <span class="n">MismatchFound</span> <span class="o">=</span> <span class="nb">false</span><span class="p">;</span>
  <span class="k">for</span> <span class="p">(</span><span class="kt">size_t</span> <span class="n">I</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">I</span> <span class="o">&lt;</span> <span class="n">Buffer</span><span class="p">.</span><span class="n">size</span><span class="p">();</span> <span class="o">++</span><span class="n">I</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">HostAccessor</span><span class="p">[</span><span class="n">I</span><span class="p">]</span> <span class="o">!=</span> <span class="n">I</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"The result is incorrect for element: "</span> <span class="o">&lt;&lt;</span> <span class="n">I</span>
                <span class="o">&lt;&lt;</span> <span class="s">" , expected: "</span> <span class="o">&lt;&lt;</span> <span class="n">I</span> <span class="o">&lt;&lt;</span> <span class="s">" , got: "</span> <span class="o">&lt;&lt;</span> <span class="n">HostAccessor</span><span class="p">[</span><span class="n">I</span><span class="p">]</span>
                <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
    <span class="n">MismatchFound</span> <span class="o">=</span> <span class="nb">true</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="n">MismatchFound</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"The results are correct!"</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="n">MismatchFound</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>




<p>Your src directory should look like:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ ls
simple-oneapi.cpp
</code></pre>

</div>




<p>Now, we are going to create a meson.build file to handle compiling simple-oneapi.cpp.</p>


<p>Recall that we compiled this source code using clang++ -fsycl - so we're going to have to duplicate that behavior.</p>


<p>Create a meson.build file with this content:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">simple_oneapi_sources</span> <span class="o">=</span> <span class="n">files</span><span class="p">(</span><span class="s">'simple-oneapi.cpp'</span><span class="p">)</span>

<span class="n">simple_oneapi_deps</span> <span class="o">=</span> <span class="p">[</span>
<span class="p">]</span>
<span class="n">executable</span><span class="p">(</span><span class="s">'simple-oneapi'</span><span class="p">,</span> <span class="n">simple_oneapi_sources</span><span class="p">,</span>
  <span class="n">link_args</span><span class="p">:</span><span class="s">'-fsycl'</span><span class="p">,</span>
  <span class="n">cpp_args</span><span class="p">:</span><span class="s">'-fsycl'</span><span class="p">,</span>
  <span class="n">dependencies</span><span class="p">:</span> <span class="n">simple_oneapi_deps</span><span class="p">,</span>
  <span class="n">install</span><span class="p">:</span> <span class="n">true</span><span class="p">,</span> <span class="n">install_dir</span><span class="p">:</span> <span class="s">'/var/home/sri/Projects/simple-oneapi/bin'</span>
<span class="p">)</span>
</code></pre>

</div>




<p>This demonstrates how easy to understand the syntax of Meson is, and it contributes quite a bit to maintainability, especially if your build system gets more complex. </p>


<h3>
  
  
  Define our source code
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">simple_oneapi_sources</span> <span class="o">=</span> <span class="n">files</span><span class="p">(</span><span class="s">'main.cpp'</span><span class="p">)</span>
</code></pre>

</div>




<p>This tells Meson what source files you have. It'll be a comma delineated list of sources.</p>


<p>You can define more source code files like so:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">simple_oneapi_sources</span> <span class="o">=</span> <span class="n">files</span><span class="p">(</span><span class="s">'simple-oneapi.cpp'</span><span class="p">,</span> <span class="s">'aux1.cpp'</span><span class="p">)</span>
</code></pre>

</div>




<h3>
  
  
  Define our dependencies
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">simple_oneapi_deps</span> <span class="o">=</span> <span class="p">[</span>
<span class="p">]</span>
</code></pre>

</div>




<p>This will be a list of dependencies for this project. In this case, we don't have any dependencies as this is a fairly simple example.</p>


<p>The dependencies are generally discovered through <code>pkg-config</code>. If you wanted to add a dependency, it would look something like this:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">simple_oneapi_deps</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">dependency</span><span class="p">(</span><span class="s">'zlib'</span><span class="p">),</span>
    <span class="n">dependency</span><span class="p">(</span><span class="s">'cups'</span><span class="p">,</span> <span class="n">method</span><span class="p">:</span> <span class="s">'pkg-config'</span><span class="p">),</span>
<span class="p">]</span>
</code></pre>

</div>




<p>See, the Meson documentation on <a href="https://mesonbuild.com/Dependencies.html">dependencies</a> for more information on dependencies.</p>


<p>Getting back to our example:</p>


<h3>
  
  
  Defining the final compile
</h3>

<p>We now want to create a binary executable that will ultimately put our sources and the required dependencies and compile them all together.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">executable</span><span class="p">(</span><span class="s">'simple-oneapi'</span><span class="p">,</span> <span class="n">simple_oneapi_sources</span><span class="p">,</span>
  <span class="n">link_args</span><span class="p">:</span><span class="s">'-fsycl'</span><span class="p">,</span>
  <span class="n">cpp_args</span><span class="p">:</span><span class="s">'-fsycl'</span><span class="p">,</span>
  <span class="n">dependencies</span><span class="p">:</span> <span class="n">simple_oneapi_deps</span><span class="p">,</span>
  <span class="n">install</span><span class="p">:</span> <span class="n">true</span><span class="p">,</span> <span class="n">install_dir</span><span class="p">:</span> <span class="s">'/var/home/sri/Projects/simple-oneapi/bin'</span>
<span class="p">)</span>
</code></pre>

</div>




<p>We define our executable to be the sources we have defined, with the linker and pre-processor flags required. Finally, a location of where to put the resulting binary when we want to install it.</p>


<p>That's pretty much it. What made this tricky is that SYCL is a define-your-own-environment and so, we were not able to take advantage of a lot of the built-ins that Meson has. Instead, we had to set everything up manually. For instance, the link_args was required in order to compile the object files in the final compile.</p>


<h2>
  
  
  Setting up our build system
</h2>

<p>We now have all the elements to put together our build system and getting our compile going. Let's see how we can do that.</p>


<p>Let's go back to the top directory of our project.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ cd ~/src/simple-oneapi
</code></pre>

</div>




<p>To create this build system, we need to make sure that we set the right compiler. In this case, we are using clang++. Most of you should already be familiar with using environment variables to set up the environments.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ CC=clang CXX=clang++ meson setup builddir
</code></pre>

</div>




<p>Meson does not support in-tree source code builds, so you must always define a build directory.</p>


<p>The result should look like this:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>The Meson build system
Version: 1.0.0
Source <span class="nb">dir</span>: /var/home/sri/Projects/simple-oneapi
Build <span class="nb">dir</span>: /var/home/sri/Projects/simple-oneapi/builddir
Build <span class="nb">type</span>: native build
Project name: simple-oneapi
Project version: 0.1.0
C compiler <span class="k">for </span>the host machine: clang <span class="o">(</span>clang 16.0.0 <span class="s2">"clang version 16.0.0 (https://github.com/intel/llvm 08be083e07b1fd6437267e26adb92f1b647d57dd)"</span><span class="o">)</span>
C linker <span class="k">for </span>the host machine: clang ld.bfd 2.34
C++ compiler <span class="k">for </span>the host machine: clang++ <span class="o">(</span>clang 16.0.0 <span class="s2">"clang version 16.0.0 (https://github.com/intel/llvm 08be083e07b1fd6437267e26adb92f1b647d57dd)"</span><span class="o">)</span>
C++ linker <span class="k">for </span>the host machine: clang++ ld.bfd 2.34
Host machine cpu family: x86_64
Host machine cpu: x86_64
Build targets <span class="k">in </span>project: 1

Found ninja-1.11.1.git.kitware.jobserver-1 at /var/home/sri/.local/bin/ninja
</code></pre>

</div>




<p>We are now ready to build this simple oneapi project.</p>


<p>To build our project, we simply do:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ cd builddir
$ ninja
</code></pre>

</div>




<p>The resultant binary will be built in the src/ directory. Alternatively, if you want to be consistent especially if you're using the same codebase to build on windows and let Meson figure out which backend to use.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ cd builddir
$ meson compile
</code></pre>

</div>




<p>You can find the results in the src/ directory.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ cd src
$ ./simple-oneapi
The results are correct!
</code></pre>

</div>




<p>So, now we have successfully built a simple oneapi binary using Meson!!</p>


<p>There are several possibilities to using Meson as a build system. Meson integrates well with CMake and other build systems, so you would not have to rebuild your system from scratch.</p>


<p>The greatest advantage of Meson is speed and simplicity on the Linux platform.</p>


<p>Interested in learning more about Meson and being part of the community? Find out more at <a href="https://mesonbuild.com/">https://mesonbuild.com/</a>. There is a Meson community on Matrix - <a href="https://matrix.to/#/#mesonbuild:matrix.org">https://matrix.to/#/#mesonbuild:matrix.org</a>. I also highly encourage you to read Jussi Pakkane’ blog at <a href="https://nibblestew.blogspot.com/">https://nibblestew.blogspot.com/</a>.</p>


<p>In the next and final blog post, we'll talk about how we can use GNOME Builder in conjunction with our container and Meson to finally put a user-friendly developer environment to write oneAPI applications on the Linux platform.</p>