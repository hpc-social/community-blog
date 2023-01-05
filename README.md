# hpc.social Community Syndicated Blog

![assets/images/blog.png](assets/images/blog.png)

This is the repository for the [syndicated community blog](https://hpc.social/projects/community-blog/) for the hpc.social community!
Note that we have two flavors of blogs - a community blog (here) along with a collection
of personal blogs served from  [syndicated community blog](https://hpc.social/projects/blog/).
The criteria for adding content feeds here is the following:

>  This is content from people who represent projects, ecosystems, or governance boards that talk about specific community interested content around the work they represent. The content range from release notes, tricks and tips, discussion around tooling and infrastructure, and other things that are neutrally branded. Discussion of branded topics like CUDA, SYCL, and oneAPI are ok - discussions about hardware are ok. Product announcements are not ok especially.

You can see the background for this discussion in [this thread](https://github.com/hpc-social/blog/pull/13).
Contribution steps are the equivalent across our community blogs, and you can
read about them [here](https://github.com/hpc-social/blog).

## Development

Note that we develop with a [shared theme](https://github.com/hpc-social/hpc-social-blog-theme)
you can generally update here via:

```bash
$ bundle install
$ bundle update
``` 

And any changes to the theme should bed one there (and thus consistent across the sites).
