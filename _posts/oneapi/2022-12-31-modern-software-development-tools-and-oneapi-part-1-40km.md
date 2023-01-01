---
author: oneAPI Community Blog
author_tag: oneapi
blog_subtitle: "The latest articles on DEV Community \U0001F469‍\U0001F4BB\U0001F468‍\U0001F4BB
  by oneAPI Community (@oneapi)."
blog_title: "DEV Community \U0001F469‍\U0001F4BB\U0001F468‍\U0001F4BB: oneAPI Community"
blog_url: https://dev.to/oneapi
category: oneapi
date: '2022-12-31 23:50:26'
layout: post
original_url: https://dev.to/oneapi/modern-software-development-tools-and-oneapi-part-1-40km
slug: modern-software-development-tools-and-oneapi-part-1
title: Modern Software Development Tools and oneAPI Part 1
---

<p>This will be the last blog post for this year (unless, I manage to get a second one in by the stroke of midnight tomorrow!).<br />
I wanted to end 2022 with a departure from the last two blog posts. In this one, we're going to be looking at oneAPI toolchain from a different perspective.</p>


<p>I wanted to build a pure oneAPI environment that uses two things:</p>


<ul>
<li>A different build system than your usual CMake <em>and</em>
</li>
<li>an opportunity to use a Linux based IDE to write code.</li>
</ul>

<p>Most HPC code runs on Linux in Data center. Linux is known for using arcane tools like VIM or Emacs and a build system. Sure, you can use VScode on Linux and that can be a viable option. However, I am an open-source person; I live and breathe the ideology and I am a true believer. oneAPI is an open platform, and we should use it on an open platform.</p>


<p>The two pieces of software I want to introduce you all is GNOME Builder and Meson. </p>

<h2>
  
  
  GNOME Builder
</h2>

<p>GNOME Builder is an IDE that is primarily targeted at building GNOME-based applications. Unlike your typical IDE, GNOME Builder uses containers for building software internally. Containers in these case are flatpak SDKs - containers that contain everything you need to build an application. With GNOME Builder, you can get started writing code without the tedium of installing the entire software development toolchain you'd need to build said applications. That means, you aren't going to need to install a compiler, linker, libraries and anything else. Everything is all-inclusive - much like a vacation resort in Cancun, say! :-)</p>


<p>Back to Builder! oneAPI is not one of the options for building software inside GNOME Builder. GNOME Builder includes containers to build against GNOME software.</p>


<p>GNOME Builder is the brainchild of Christian Hergert, a long time Free Software programmer who was frustrated with the current state of tools to build software within the application community and started the Builder project about 7 years ago and was initially funded by a kickstarter. Through some great luck, Christian is now paid by Red Hat to help build GNOME Builder as well as improving the application building story on Linux and other platforms.</p>


<p>There is something wonderfully intriguing about Builder which is why I'm highlighting it here and why I picked it as the IDE of choice to write oneAPI-related code.</p>

<h3>
  
  
  This one special trick
</h3>

<p>Builder will allow you to use a podman or docker container to run everything. So in this case, we're going to create that all-inclusive experience! Once you've done the work of building a container with all the oneAPI tools into it, others can then re-use the container as a fixed environment and other folx can clone the project you are working on and then easily collaborate.</p>

<h2>
  
  
  Meson
</h2>

<p>The next piece of software I wanted to highlight is Meson. Meson is a build system that is written in python and <em>tries</em> to be an intuitive system that <em>tries</em> to do the right thing through easily understandable syntax. For those of you who have ever used autoconf - this was the application community's response to autoconf.</p>


<p>Autoconf during its heyday was a massive boon to those who were writing code that could work on many different UNIX and Linux distributions. However, autoconf was difficult to figure out and most folx simply copy from another project and then move on. Writing anything sophisticated required extensive, expended effort.</p>


<p>In my personal opinion, build systems are really hard to get right and there are so many oddities in how we build software that a build system has to get right, in order to be effective.</p>


<p>Meson is written by Finnish programmer, Jussi Pakkanen, who was frustrated with the current state of build systems and their arcane configuration syntax and sometimes rather unexpected behaviors!</p>


<p>Meson can be better described as a system that generates the configurations for build systems to use. It isn't a full-fledged build system like Make or CMake. In fact, you could easily re-use CMake configuration files in Meson. It has a concept of a backend and can generate config for Xcode on MacOS, VScode on Windows and Ninja on Linux systems. Meson easily integrates with profilers and debuggers and is designed not to build within source tree but in a designed build area.</p>


<p>For those not familiar with ninja, it is an extremely fast build system that has been shown to be effective in building software very quickly!!</p>

<h2>
  
  
  Build an oneAPI Container
</h2>

<p>In this first part, we will focus on building an oneAPI container based on the Ubuntu 20.04 LTS release since that is what oneAPI works optimally on.</p>


<p>I will be using Fedora 37 SilverBlue edition. I like SilverBlue as it is built with containerized environments in mind. It allows you to build different container environments that you can enter and exit from on the command line and still easily integrate with the desktop.</p>


<p>Let's start then with building our oneAPI environment so that it will be able to run a simple oneAPI sample program.</p>


<p>Staying true to the spirit of open-source, I will build this environment from source and only use what's available on GitHub.</p>


<p>To start, find and install the 'distrobox' tool on your distro.</p>


<p><code>dnf install distrobox -y</code></p>


<p>Distrobox allows you to create containerized environments from the command line.</p>


<p>I use podman as I live in the Fedora world. Podman is a command line compatible version of Docker. It's reasonable that the following could be done through a Dockerfile but the oneAPI libraries changes often enough that this blog post would become stale in short order.</p>


<p><code>$ distrobox create oneapi -i docker.io/library/ubuntu:20.04</code></p>


<p>This will create a container called oneapi with an Ubuntu 20.04 setup.</p>


<p><code>$ distrobox enter oneapi</code></p>


<p>Will let you enter the container.</p>


<p>The beauty of distrobox is that you are in this container, it has mounted your home directory and you have essentially inherited your desktop system but the container is Ubuntu - and so you can use the Ubuntu distro tools to install software. Pretty neat, huh?</p>


<p>The first step is to build the DPC++ oneAPI compiler from source. </p>


<p><code>$ mkdir -p ~/src/sycl_workspace</code></p>


<p>You can use whatever area you want. I'm following this <a href="https://intel.github.io/llvm-docs/GetStartedGuide.html#create-dpc-workspace">guide</a> to build the compiler.</p>

<h3>
  
  
  Prequisites
</h3>

<p>Let's first grab our pre-requisites for building.<br />
</p>


<p><code>$ apt install git python3 ninja gcc c++ libstdc++ libstdc++-9-dev python3-pip python3-distutils python-distutils-extra python3-psutil -y<br />
$ pip3 install meson<br />
$ pip3 install ninja</code><br />
</p>


<p>Now that we have our build environment.</p>


<h3>
  
  
  Build DPC++ Compiler
</h3>



<p><code>$ cd ~/src/sycl_workspace<br />
$ export DPCPP_HOME=`pwd`<br />
$ git clone https://github.com/intel/llvm -b sycl</code><br />
</p>


<p>We can start the actual build:<br />
</p>


<p><code>$ python $DPCPP_HOME/llvm/buildbot/configure.py<br />
$ python $DPCPP_HOME/llvm/buildbot/compile.py</code><br />
</p>


<p>At the end of this exercise, you should have a working oneAPI DPC++ compiler.</p>


<p>But we aren't done yet - we still need to add some of the oneAPI libraries and runtimes to make our simple oneAPI example work.</p>


<p>We first need to install our low level runtimes: the things that recognizes accelerators. For now, we'll use the ones that recognize the x86 Intel processors as that is what is on my laptop right now.</p>


<p>We will first need to identify the latest versions of the runtimes we need to download. You need to look this up in the <a href="https://github.com/intel/llvm/blob/sycl/buildbot/dependency.conf">dependency.conf</a>.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span><span class="nb">sudo mkdir</span> <span class="nt">-p</span> /opt/intel 
<span class="nv">$ </span><span class="nb">sudo mkdir</span> <span class="nt">-p</span> /etc/OpenCL/vendors/intel_fpgaemu.icd
<span class="nv">$ </span><span class="nb">cd</span> /tmp
<span class="nv">$ </span>wget https://github.com/intel/llvm/releases/download/2022-WW50/oclcpuexp-2022.15.12.0.01_rel.tar.gz
<span class="nv">$ </span>wget https://github.com/intel/llvm/releases/download/2022-WW50/fpgaemu-2022.15.12.0.01_rel.tar.gz
<span class="nv">$ </span><span class="nb">sudo </span>bash
<span class="c"># cd /opt/intel</span>
<span class="c"># mkdir oclfpgaemu-&lt;fpga_version&gt;</span>
<span class="c"># cd oclfpgaemu-&lt;fpga_version&gt;</span>
<span class="c"># tar xvfpz /tmp/fpgaemu-2022.15.12.0.01_rel.tar.gz</span>
<span class="c"># cd ..</span>
<span class="c"># mkdir oclcpuexp_&lt;cpu_version&gt;</span>
<span class="c"># cd oclcpuexp-&lt;cpu_version&gt;</span>
<span class="c"># tar xvfpz /tmp/oclcpuexp-&lt;cpu_version&gt;</span>
<span class="c"># cd ..</span>
</code></pre>

</div>




<p>Now to create some configuration files.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># pwd</span>
/opt/intel
<span class="c"># echo  /opt/intel/oclfpgaemu_&lt;fpga_version&gt;/x64/libintelocl_emu.so &gt;</span>
  /etc/OpenCL/vendors/intel_fpgaemu.icd
<span class="c"># echo /opt/intel/oclcpuexp_&lt;cpu_version&gt;/x64/libintelocl.so &gt;</span>
  /etc/OpenCL/vendors/intel_expcpu.icd
</code></pre>

</div>




<p>We'll need to grab a release of oneTBB from <a href="https://github.com/oneapi-src/oneTBB/releases">github</a><br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span><span class="nb">cd</span> /tmp
<span class="nv">$ </span>wget https://github.com/oneapi-src/oneTBB/releases/download/v2021.7.0/oneapi-tbb-2021.7.0-lin.tgz
</code></pre>

</div>




<p>and now extract it.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span><span class="nb">cd</span> /opt/intel
<span class="nv">$ </span><span class="nb">sudo </span>bash
<span class="c"># tar xvfpz /tmp/oneapi-tbb-2021.7.0-lin.tgz</span>
</code></pre>

</div>




<p>We'll need to reference some of the libraries in the oneTBB directory in our build.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbb.so /opt/intel/oclfpgaemu_&lt;fpga_version&gt;/x64</span>
<span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbbmalloc.so /opt/intel/oclfpgaemu_&lt;fpga_version&gt;/x64</span>
<span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbb.so.12 /opt/intel/oclfpgaemu_&lt;fpga_version&gt;/x64</span>
<span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbbmalloc.so.2 /opt/intel/oclfpgaemu_&lt;fpga_version&gt;/x64</span>

<span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbb.so /opt/intel/oclcpuexp_&lt;cpu_version&gt;/x64</span>
<span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbbmalloc.so /opt/intel/oclcpuexp_&lt;cpu_version&gt;/x64</span>
<span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbb.so.12 /opt/intel/oclcpuexp_&lt;cpu_version&gt;/x64</span>
<span class="c"># ln -s /opt/intel/oneapi-tbb-&lt;tbb_version&gt;/lib/intel64/gcc4.8/libtbbmalloc.so.2 /opt/intel/oclcpuexp_&lt;cpu_version&gt;/x64</span>
</code></pre>

</div>




<p>Now we need to configure the library paths:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># echo /opt/intel/oclfpgaemu_&lt;fpga_version&gt;/x64 &gt; /etc/ld.so.conf.d/libintelopenclexp.conf</span>
<span class="c"># echo /opt/intel/oclcpuexp_&lt;cpu_version&gt;/x64 &gt;&gt; /etc/ld.so.conf.d/libintelopenclexp.conf</span>
<span class="c"># ldconfig -f /etc/ld.so.conf.d/libintelopenclexp.conf</span>
</code></pre>

</div>




<p>and we're done! Now we need to make sure that this toolchain actually works. So run this test.</p>


<p><strong>Make sure you are not root.</strong><br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>python <span class="nv">$DPCPP_HOME</span>/llvm/buildbot/check.py
</code></pre>

</div>




<p>If you come back with no failure then, congratulations, you're in good shape!! Sometimes, there might be a few missing dependencies, especially when it comes to python.</p>


<p>We are now ready to create a simple SYCL application and test. I'm going to re-use the one that is located on <a href="https://intel.github.io/llvm-docs/GetStartedGuide.html#run-simple-dpc-application">Github</a>.</p>


<p>Let's create our workspace and build this sample project.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span><span class="nb">mkdir</span> <span class="nt">-p</span> ~/src/simple-oneapi/
<span class="nv">$ </span><span class="nb">cd</span> ~/src/simple-oneapi
<span class="nv">$ </span><span class="nb">export </span><span class="nv">PATH</span><span class="o">=</span><span class="nv">$DPCPP_HOME</span>/llvm/build/bin:<span class="nv">$PATH</span>
<span class="nv">$ </span><span class="nb">export </span><span class="nv">LD_LIBRARY_PATH</span><span class="o">=</span><span class="nv">$DPCPP_HOME</span>/llvm/build/lib:<span class="nv">$LD_LIBRARY_PATH</span>
<span class="nv">$ </span><span class="nb">cat</span> <span class="o">&gt;</span> simple-oneapi.cpp

<span class="c">#include &lt;sycl/sycl.hpp&gt;</span>

int main<span class="o">()</span> <span class="o">{</span>
  // Creating buffer of 4 ints to be used inside the kernel code
  sycl::buffer&lt;sycl::cl_int, 1&gt; Buffer<span class="o">(</span>4<span class="o">)</span><span class="p">;</span>

  // Creating SYCL queue
  sycl::queue Queue<span class="p">;</span>

  // Size of index space <span class="k">for </span>kernel
  sycl::range&lt;1&gt; NumOfWorkItems<span class="o">{</span>Buffer.size<span class="o">()}</span><span class="p">;</span>

  // Submitting <span class="nb">command </span>group<span class="o">(</span>work<span class="o">)</span> to queue
  Queue.submit<span class="o">([</span>&amp;]<span class="o">(</span>sycl::handler &amp;cgh<span class="o">)</span> <span class="o">{</span>
    // Getting write only access to the buffer on a device
    auto Accessor <span class="o">=</span> Buffer.get_access&lt;sycl::access::mode::write&gt;<span class="o">(</span>cgh<span class="o">)</span><span class="p">;</span>
    // Executing kernel
    cgh.parallel_for&lt;class FillBuffer&gt;<span class="o">(</span>
        NumOfWorkItems, <span class="o">[=](</span>sycl::id&lt;1&gt; WIid<span class="o">)</span> <span class="o">{</span>
          // Fill buffer with indexes
          Accessor[WIid] <span class="o">=</span> <span class="o">(</span>sycl::cl_int<span class="o">)</span>WIid.get<span class="o">(</span>0<span class="o">)</span><span class="p">;</span>
        <span class="o">})</span><span class="p">;</span>
  <span class="o">})</span><span class="p">;</span>

  // Getting <span class="nb">read </span>only access to the buffer on the host.
  // Implicit barrier waiting <span class="k">for </span>queue to <span class="nb">complete </span>the work.
  const auto HostAccessor <span class="o">=</span> Buffer.get_access&lt;sycl::access::mode::read&gt;<span class="o">()</span><span class="p">;</span>

  // Check the results
  bool MismatchFound <span class="o">=</span> <span class="nb">false</span><span class="p">;</span>
  <span class="k">for</span> <span class="o">(</span>size_t I <span class="o">=</span> 0<span class="p">;</span> I &lt; Buffer.size<span class="o">()</span><span class="p">;</span> ++I<span class="o">)</span> <span class="o">{</span>
    <span class="k">if</span> <span class="o">(</span>HostAccessor[I] <span class="o">!=</span> I<span class="o">)</span> <span class="o">{</span>
      std::cout &lt;&lt; <span class="s2">"The result is incorrect for element: "</span> <span class="o">&lt;&lt;</span> <span class="no">I</span><span class="sh">
                &lt;&lt; " , expected: " &lt;&lt; I &lt;&lt; " , got: " &lt;&lt; HostAccessor[I]
                &lt;&lt; std::endl;
      MismatchFound = true;
    }
  }

  if (!MismatchFound) {
    std::cout &lt;&lt; "The results are correct!" &lt;&lt; std::endl;
  }

  return MismatchFound;
}
</span></code></pre>

</div>




<p>Let's build our simple oneapi source code!<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>clang++ <span class="nt">-fsycl</span> simple-sycl-app.cpp <span class="nt">-o</span> simple-sycl-app
</code></pre>

</div>




<p>It should compile and run without any errors.</p>


<p>If all works as anticipated, you should have a working setup.</p>


<h3>
  
  
  Setting up the container to code SYCL when you enter
</h3>

<p>Now, the next step is to make this container useful when you enter and have it always ready to build a sycl app.</p>


<p>Exit out of the container using the 'exit' command and you should be back on the host operating system.</p>


<p>Type:<br />
<code>$ uname -a</code></p>


<p>On my system, I get:</p>


<p><code>Linux fedora 6.0.13-300.fc37.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 14 16:15:19 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux</code></p>


<p>Re-enter the container:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>distrobox enter oneapi
<span class="nv">$ </span><span class="nb">uname</span> <span class="nt">-a</span>
Linux oneapi.fedora 6.0.13-300.fc37.x86_64 <span class="c">#1 SMP PREEMPT_DYNAMIC Wed Dec 14 16:15:19 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux</span>
</code></pre>

</div>




<p>You will notice that after "Linux" when you are in the container, there is a "oneapi" prefix to fedora.</p>


<p>We can take advantage of that. Let's make sure that when we enter the container that we can set things up from the shell perspective to be ready to write SYCL code.</p>


<p>Add this bit to your .bashrc:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">oneapi</span><span class="o">=</span><span class="sb">``</span><span class="nb">uname</span> <span class="nt">-a</span> | <span class="nb">grep</span> <span class="nt">-c</span> oneapi<span class="sb">``</span>

<span class="k">if</span> <span class="o">[</span> <span class="nv">$oneapi</span> <span class="nt">-gt</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
   </span><span class="nb">echo</span> <span class="s2">"Initializing oneAPI"</span>
   <span class="nb">export </span><span class="nv">DPCPP_HOME</span><span class="o">=</span><span class="s2">"/var/home/sri/src/dpcplusplus"</span>
   <span class="nb">export </span><span class="nv">PATH</span><span class="o">=</span><span class="s2">"</span><span class="nv">$PATH</span><span class="s2">:/var/home/sri/.local/bin:</span><span class="nv">$DPCPP_HOME</span><span class="s2">/llvm/build/bin"</span>
   <span class="nb">export </span><span class="nv">LD_LIBRARY_PATH</span><span class="o">=</span><span class="s2">"</span><span class="nv">$DPCPP_HOME</span><span class="s2">/llvm/build/lib"</span>
<span class="k">fi</span>
</code></pre>

</div>




<p><strong>Make sure you replace 'sri' with your your login details</strong></p>


<p>Now when we enter the 'oneapi' container our environment will be properly initialized.</p>


<p>Let's stop here, and we'll pick it up in the next post. The next post will focus on creating a meson setup around this simple oneapi code. Part 3 will focus on taking our meson configured source code and using GNOME Builder to build it. Stay tuned!</p>


<p>Photo by <a href="https://unsplash.com/@imattsmart?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">iMattSmart</a> on <a href="https://unsplash.com/photos/sm0Bkoj5bnA?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></p>