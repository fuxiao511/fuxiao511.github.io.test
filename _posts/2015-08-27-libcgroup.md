https://blogs.oracle.com/linux/post/cgroup-v2-checkpoint



Given the above requirements, we have embarked on the following roadmap:

- Add cgroup v2 support to libcgroup. Libcgroup was started in 2008 during the early days of cgroup v1 but has largely languished over the last few years. As maintainers of [libcgroup](https://github.com/libcgroup), Oracle's Dhaval Giani and I are defining, guiding, and implementing the library's transition to full cgroup v2 support. In 2019, we restarted development on libcgroup and have since added [automated unit tests](https://travis-ci.org/libcgroup/libcgroup), [automated functional tests](https://github.com/libcgroup/libcgroup-tests/tree/master/ftests), and [code coverage](https://coveralls.io/github/libcgroup/libcgroup?branch=master). We recently added an ["ignore" feature](https://github.com/libcgroup/libcgroup/commit/ae76f3a8c9c90ac05feb046714f4f6566b24cd3c) to cgrules for an internal customer, and currently have a [patchset](https://sourceforge.net/p/libcg/mailman/message/37007410/) out for review to add cgroup v2 support to cgget and cgset.
- Create an abstraction layer that can receive cgroup v1 (or v2) requests and translate them to the correct underlying system settings - be it v1 or v2. This layer should allow cgroup v1 users to continue to specify v1 settings even if the application is running on a v2 system, thus minimizing changes to the application.
- And finally create a usability layer to further remove the user from the intricacies and pitfalls of cgroup management. Not all users are cgroup experts and not all users want to be cgroup experts. A usability layer would give these users the ability to consistently and safely configure their systems every single time.



三个默认的 systemd slice 分别为
system.slice
默认情况下所有 sytemd 启动的服务都会放到这个 slice 下一个新的子组中（ child group ）。
user.slice
当每个用户登录系统的时候一个新的子 slice 会被创建在这个 user.slice 中，每个登录系统用户的会话会放入这个新创建的子 slice 中。
machine.slice
被 libvirt 管理的虚拟机会被自动分配到这个 slice 中的一个子 slice 中，需要留意只有当虚拟机被创建 machine.slice 才会被创建，默认是没有的。
