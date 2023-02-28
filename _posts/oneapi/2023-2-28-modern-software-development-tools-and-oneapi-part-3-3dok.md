---
author: oneAPI Community Blog
author_tag: oneapi
blog_subtitle: The latest articles on DEV Community by oneAPI Community (@oneapi).
blog_title: 'DEV Community: oneAPI Community'
blog_url: https://dev.to/oneapi
category: oneapi
date: '2023-02-28 02:19:25'
layout: post
original_url: https://dev.to/oneapi/modern-software-development-tools-and-oneapi-part-3-3dok
slug: modern-software-development-tools-and-oneapi-part-3
title: Modern Software Development Tools and oneAPI Part 3
---

<p>This is the third part in the series. Part 1 is <a href="https://dev.to/oneapi/modern-software-development-tools-and-oneapi-part-1-40km">here</a> and Part 2 is <a href="https://dev.to/oneapi/modern-software-development-tools-and-oneapi-part-2-4bjp">here</a>.</p>


<p>Welcome to the third, and likely the final, post in this blog series! To recap, in the last blog post we talked about build systems particularly <a href="https://mesonbuild.com/">meson</a> Before that, we talked about building a container that contains the pure open source elements of the oneAPI developer environment and use it to build a simple oneAPI SYCL program.</p>


<p>In this post, we're going to take our key learnings from the last two blog posts and use them to build a true user-friendly experience where you can write code using a modern IDE and compile and run them inside a container like you might be used to on other platforms like Windows and MacOS.</p>


<p>First, allow us to introduce you to this modern IDE - 'GNOME Builder'. <a href="https://apps.gnome.org/app/org.gnome.Builder/">GNOME Builder</a> is an IDE developed for <a href="https://www.gnome.org/">GNOME</a> desktop. It is integrated to be able to write GNOME and GTK applications easily with all the modern features one would expect from a IDE, and then some.</p>


<p>It has an impressive set of features - the author, Christian Hergert, wrote it because he was [frustrated (<a href="https://foundation.gnome.org/2015/01/09/interview-with-christian-hergert-about-builder-an-ide-for-gnome-2/">https://foundation.gnome.org/2015/01/09/interview-with-christian-hergert-about-builder-an-ide-for-gnome-2/</a>)  with the state of IDEs on the Linux platform. GNOME Builder is not just an IDE, but a complete showcase of what a non-trivial application written in GNOME can do.</p>


<p>This blog post is about oneAPI - why use an IDE that is optimized for using GNOME to build applications?</p>


<p>Great question. The desktop ecosystem (GNOME and <a href="https://kde.org/">KDE</a> has been focused on distribution of apps through a container technology called <a href="https://flatpak.org">flatpak</a>. Flatpak allows you to have an runtime that contains everything to run a GNOME (or KDE) application. There is an associated SDK that contains all the tools needed to build the application. GNOME Builder is the first IDE that integrates this idea of containerized applications into the user experience. With Builder, you only need the application - you don't need a compiler, profiler, or development libraries - it integrates all that inside a container. This means that you don't need to think about how to setup a developer environment for any GNOME application.</p>


<p>The containers in the past have been flatpak based containers. But it turns out that you can leverage GNOME Builder to use any container created by podman, toolbox, or distrobox.</p>


<p>In essence, the first blog post in this series mimicked what flatpak already does: which is a container that contains everything you need to build an oneAPI application/program instead of a GNOME one.</p>


<p>In a bit of circularity that you might find amusing - we will use flatpak to get the application and then use another comtainer to build our sample application that uses Meson.</p>


<p>If you have not read the first two blog posts, this might be a good time to stop and read those first because we'll be using the container we created in the first blog post and the build system we used in the second blog post. It's also important  you  use a distro like Fedora or openSUSE that supports flatpak out of the box.</p>


<p>With the pre-requisites out of the way, let's first start by installing GNOME Builder. You can use any desktop you want, but I will be using GNOME here as it is what I usually run, please translate accordingly.</p>


<p>Here are the steps:</p>


<ul>
<li>First make sure you add the flathub flatpak respository:
</li>
</ul>

<p><code>$  flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo</code><br />
</p>


<ul>
<li>Install GNOME Builder:
</li>
</ul>

<p><code>$ flatpak install flathub org.gnome.Builder</code><br />
</p>


<ul>
<li>Run GNOME builder either through your desktop launch options. For GNOME, hit the meta key (usually Windows key) and then type in "Builder" - GNOME Builder should be your first,  and likely only, option. You can also run it from the command line:,
</li>
</ul>

<p><code>$ flatpak run org.gnome.Builder</code><br />
</p>


<p>You should now have GNOME Builder running on your machine!</p>


<h2>
  
  
  Creating a Project
</h2>

<p>The first step is to create a project.</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--AbJZDlQG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pqle4qghdklomilmrz04.png"><img alt="Image description" height="723" src="https://res.cloudinary.com/practicaldev/image/fetch/s--AbJZDlQG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pqle4qghdklomilmrz04.png" width="880" /></a></p>


<p>Select "Create New Project..." </p>


<p>You will be presented with a new screen where you put in the details for the project. Let's call our project "oneapi-simple".</p>


<p>Next we need to select the application-id. Application-ids are generally a reverse DNS type of string usually based on a hostname. I have my own domain, so I usually use that. But you can use whateer you like. In this case, I am going to use me.ramkrishna.oneapisimple.</p>


<p>We want to use C++, so under Language change it to C++. Note that the Template section has now changed to 'Command Line Tool' Which is exactly what we want.</p>


<p>Here is a filled-out screenshot of the window from Builder:</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--X-xgaUOr--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dx974pybl1jmypllto76.png"><img alt="Image description" height="728" src="https://res.cloudinary.com/practicaldev/image/fetch/s--X-xgaUOr--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dx974pybl1jmypllto76.png" width="880" /></a></p>


<p>We now create the project! Selected the "Create Project" and we are now ready to continue.</p>


<p>GNOME Builder has two sections - the sidebar and the main editor window. The side bar will have our files and so click on "src" and you should see two files - main.cpp and meson.buid.</p>


<h2>
  
  
  Setup the build system
</h2>

<p>You will notice that the project is already set up to use meson by default. Meson is the preferred build system for GNOME. Meson was created by someone from the GNOME community and thus is already well trusted. <br />
In the application space, meson has proven to be quite popular replacement for autotools.</p>


<p>Let's leave main.cpp alone for now, and focus on meson.build. If you click on meson.build, you'll see that it looks like this:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>oneapi_simple_sources = [
  'main.cpp',
]

oneapi_simple_deps = [
]

executable('oneapi-simple', oneapi_simple_sources,
  dependencies: oneapi_simple_deps,
  install: true,
)
</code></pre>

</div>




<p>This meson.build is set up to compile a generic project with the g++ compiler. So that's not going to work. If you read the previous blog post, we went through what we would need to make it work with the SYCL compiler.</p>


<p>Replace the contents of meson.build with this:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>simple_oneapi_sources = files('main.cpp')

simple_oneapi_deps = [
]

executable('simple-oneapi', simple_oneapi_sources,
  link_args:'-fsycl',
  cpp_args:'-fsycl',
  dependencies: simple_oneapi_deps,
  install: true, install_dir: '/var/home/sri/Projects/oneapi-simple/bin'
)
</code></pre>

</div>




<p>For the SYCL compiler, we need some extra linker flags. We're actually missing something even more important and that's the setup for the compiler itself!</p>


<p>Click on the 'meson.build' file in the top level - which should be right next to the 'COPYING' file. You'll notice that every time you open a new file, it creates a new tab in the editor view. You can easily switch to each file by clicking on the tab.</p>


<p>Let's take a look at it. It should look like this.<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>project('oneapi-simple', ['cpp', 'c'],
          version: '0.1.0',
    meson_version: '&gt;= 0.59.0',
  default_options: [ 'warning_level=2', 'werror=false', 'cpp_std=gnu++2a', ],
)

subdir('src')
</code></pre>

</div>




<p>The important part here is that we are identifying that this project is C++. All of this is correct and there is nothing more to be done.</p>


<h2>
  
  
  Set up our source
</h2>

<p>Now, that we have the build set up. It's time to replace the code in main.cpp. Currently, the code looks like:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>#include &lt;iostream&gt;

int main() {
    std::cout &lt;&lt; "Hello World\n";

    return 0;
}
</code></pre>

</div>




<p>We are going to replace it with:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>#include &lt;sycl/sycl.hpp&gt;

int main() {
  // Creating buffer of 4 ints to be used inside the kernel code
  sycl::buffer&lt;sycl::cl_int, 1&gt; Buffer(4);

  // Creating SYCL queue
  sycl::queue Queue;

  // Size of index space for kernel
  sycl::range&lt;1&gt; NumOfWorkItems{Buffer.size()};

  // Submitting command group(work) to queue
  Queue.submit([&amp;](sycl::handler &amp;cgh) {
    // Getting write only access to the buffer on a device
    auto Accessor = Buffer.get_access&lt;sycl::access::mode::write&gt;(cgh);
    // Executing kernel
    cgh.parallel_for&lt;class FillBuffer&gt;(
        NumOfWorkItems, [=](sycl::id&lt;1&gt; WIid) {
          // Fill buffer with indexes
          Accessor[WIid] = (sycl::cl_int)WIid.get(0);
        });
  });

  // Getting read only access to the buffer on the host.
  // Implicit barrier waiting for queue to complete the work.
  const auto HostAccessor = Buffer.get_access&lt;sycl::access::mode::read&gt;();

  // Check the results
  bool MismatchFound = false;
  for (size_t I = 0; I &lt; Buffer.size(); ++I) {
    if (HostAccessor[I] != I) {
      std::cout &lt;&lt; "The result is incorrect for element: " &lt;&lt; I
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
</code></pre>

</div>




<p>OK - now we have everything. But we can't quite compile yet. Right now, if you tried to compile this - it won't work. The reason is, the build is currently set up for native build'. Which means it will try to use the toolchain on the host system. On the host system, we don't have any of the oneAPI libraries or the SYCL compiler. So it won't find anything. Everything we wanted is encapsulated in a container.</p>


<p>This is why GNOME Builder is especially suited to do this exercise on Linux because you set the run and build environment to any podman (or docker) container.</p>


<h2>
  
  
  Set the build and run environment to our SYCL container.
</h2>

<p>Refer to the<br />
<a href="https://dev.to/oneapi/modern-software-development-tools-and-oneapi-part-1-40km">first</a><br />
blog post on how to setup the build and run container.</p>


<p>In that blog post, we named our container - 'oneapi'. It should container the SYCL compiler that we<br />
built and all the accompanying libraries to build our simple SYCL program.</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--f-dAEe_z--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/clukhlkv1bb36bmbet59.png"><img alt="Image description" height="616" src="https://res.cloudinary.com/practicaldev/image/fetch/s--f-dAEe_z--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/clukhlkv1bb36bmbet59.png" width="880" /></a></p>


<p>To set the build type - we need to move our cursor to the widget at the top in the center next to the<br />
hammer icon. Click on the down arrow, and then select 'Configure Project', there is a keyboard shortcut<br />
"alt+," (hold alt and then comma) and the window should pop up.</p>


<p>Select "Default" at the bottom of the dialog box.</p>


<p>Under Build Environment, you want to change that from 'Host Operating System' to 'oneapi'. If 'oneapi',<br />
does not appear on your list of choices then you have not created the container using distrobox. You<br />
should refer to the first blog post in the series for testing.</p>


<p>At this point, we have our build system using our container - but we aren't done yet. The problem now is<br />
that the build system will explicitly use c++ instead of the SYCL compiler. To override using the native toolchain, we generally use an environmental variable. This is generally not recommended but for sake of simplicity, we will use it for now. In another blog post, we can revisit the issue. For the impatient, it requires that you use the native file feature of meson - see <a href="https://mesonbuild.com/Machine-files.html">https://mesonbuild.com/Machine-files.html</a>.</p>


<p>For now, we will use Builder's ability to set shell environment variables to set the CXX and other<br />
critical environment variables.</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--IN7cA_Nq--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uv0t38nizjris20yn15n.png"><img alt="Image description" height="261" src="https://res.cloudinary.com/practicaldev/image/fetch/s--IN7cA_Nq--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uv0t38nizjris20yn15n.png" width="880" /></a></p>


<p>Click on 'Add Variables' and set the following key value pairs (be sure to replace the paths to the<br />
correct paths):<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>DPCPP_HOME=/var/home/your_login/src/dpcplusplus
PATH=/usr/bin:/usr/sbin:/usr/local/bin:/home/yourlogin/bin:/home/yourlogin/.local/bin:/var/home/yourlogin/src/dpcplusplus/llvm/build/bin
LD_LIBRARY_PATH=/var/home/yourlogin/src/dpcplusplus/llvm/build/lib
CC=clang
CXX=clang++
</code></pre>

</div>




<p>Your config should look like this:</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--WEj27cVk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hu8gd5dcfeqwplid9bfs.png"><img alt="Image description" height="559" src="https://res.cloudinary.com/practicaldev/image/fetch/s--WEj27cVk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hu8gd5dcfeqwplid9bfs.png" width="880" /></a></p>


<p>The environment variables that are set are mirrored from the environment variables we had to set when we set up a simple oneapi codebase inside the container in the first blog post. We are merely recreating it.</p>


<p>At this point, you can click on the "hammer" icon and GNOME Builder should proceed to properly build the source code. It will give two warnings that you can safely ignore at this point.</p>


<p>To execute the program, you need to hit the right pointing triangle(it looks like a "play" button) and it will try to execute it.</p>


<p>You'll note that it was not able to execute. </p>


<p>That's becasue when it is running it doesn't set the LD_LIBRARY_PATH<br />
inside the container. Since build environment is using non-standard paths we have to do a trick to set everything up so that it can find the libraries it needs.</p>


<p>So, to mitigate that we need to create a wrapper script that will set the LD_LIBRARY_PATH before executing. In another blog post, we will work on something a litte more clever. This will do for now.</p>


<p>Let's call the script 'run-oneapi.sh'. Here is the very simple code for it:<br />
</p>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/sh</span>
<span class="nb">export </span><span class="nv">LD_LIBRARY_PATH</span><span class="o">=</span><span class="s2">"/var/home/sri/src/dpcplusplus/llvm/build/lib"</span>
<span class="nb">exec</span> /var/home/sri/Projects/oneapi-simple/bin/oneapi-simple
</code></pre>

</div>




<p>Install it somewhere within your PATH environment. I have mine in ~/Projects/oneapi-simple/bin where the<br />
run time binary gets built and installed.</p>


<p>Once you have that, you need to let builder know how to run it.</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--eHBmLO5f--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6zi26imjcw56opdh30cl.png"><img alt="Image description" height="616" src="https://res.cloudinary.com/practicaldev/image/fetch/s--eHBmLO5f--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6zi26imjcw56opdh30cl.png" width="880" /></a></p>


<p>The first step is to go back to the build configuration menu, use the keyboard shortcut ALT-, and then select "Command" on the far left column.</p>


<p>Select "Create Command" and then fill in the dialog box like this:</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--tPf9uj8X--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dobm6erppuvh8pw03s1n.png"><img alt="Image description" height="880" src="https://res.cloudinary.com/practicaldev/image/fetch/s--tPf9uj8X--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dobm6erppuvh8pw03s1n.png" width="880" /></a></p>


<p>Once you've added that, you are ready to configure the run command to use this script.</p>


<p><a class="article-body-image-wrapper" href="https://res.cloudinary.com/practicaldev/image/fetch/s--NmukZYYg--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v4b84d8z97wb2ni19yec.png"><img alt="Image description" height="616" src="https://res.cloudinary.com/practicaldev/image/fetch/s--NmukZYYg--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v4b84d8z97wb2ni19yec.png" width="880" /></a></p>


<p>Click on 'Applications'</p>


<p>On the first line you'll see "Run Command" which will be set to "Automatically Discover". Use the drop down list to select "Run-oneapi".</p>


<p>Close the dialog box, and you will now have setup Builder to build and run. Since, everything is already cached. You will need to re-run the build.</p>


<p>Select the drop down list next to the hammer icon and select "Rebuild". This will rebuild the source from scratch and clear out all the cache.</p>


<p>You will now be setup to run.</p>


<p>Click on the play icon next to the hammer icon and it should now properly build and run.</p>


<p>Congratulations - you have now succesfully set up building an oneAPI build on GNOME Builder.</p>


<p>There are a lot of ways to go from here. I would love to hear if anybody actually set this up and give some feedback on whether you were able to make this work and what further plans you have. </p>


<p>There are definitely some improvements that need to be done. Since this set up doesn't actually work to ship an application.</p>


<p>This ends third in the series. I might revisit. I would love to get feedback, improvements and whether you all are hacking code using GNOME Builder!</p>