ansible-role-nfvbench Release Notes
===================================

0.3.2 (2022-07-01)
------------------

### Release Summary

Bugfix and small improvements release.


### Changes

- Fix undetected test failure when Xtesting `run_tests` command crashes because
  of a buggy test scenario.
- Fix the cleanup phase so that it does not fail when some of the OpenStack
  resources are already deleted.
- Decrease timeout from 1h to 2mn during connectiviy check with nfvbench HTTP
  server.
- Revert the role default config of nfvbench `cache_size` and `no_flow_stats`
  parameters: use nfvbench default config again (`cache_size=0` and
  `no_flow_stats=false`)
- Improve Ansible log messages: add a description (name) to the tasks that
  include a file, update some existing descriptions to make them (hopfully) more
  explicit.
- Add an explicit and early connectivity check of OpenStack API to make it clear
  when the API is unavailable.
- Document the main flags that control the role execution.


0.3.1 (2022-03-17)
------------------

### Release Summary

Bugfix release.


### Changes

- Fix results archive not created when the `BUILD_TAG` environment variable
  contains a slash.
- Increase server create timeout from 3mn to 5m for the traffic generator VM.
- Stop using the deprecated Ansible plugin `openstack.cloud.os_floating_ip`, use
  `openstack.cloud.floating_ip` instead.


0.3.0 (2022-03-10)
------------------

### Release Summary

Several improvements including TestAPI integration, traffic generation
performance and documentation.


### Changes

- Set missing env variables for TestAPI (`PROJECT_NAME`, `INSTALLER_TYPE`,
  `DEPLOY_SCENARIO`).  When those variables are present in the environment
  during the invocation of `ansible-role-nfvbench`, they will be conveyed to
  xtesting inside the traffic generator VM and used to properly set the
  corresponding fields in TestAPI results.
- Increase server delete timeout from 3 to 5 minutes for both the traffic
  generator VM and the loop VM.
- Improve traffic generator performance in default config.  Users not using the
  default config should update their config and set `cache_size: 16` and
  `no_flow_stats: true` in the `nfvbench_config:` section of their extra_vars
  file.
- Add two new flavors in default config file for basic and cpu-intensive compute
  nodes.
- Rewrite installation instructions.
- Require explicit versions of major dependencies.


0.2.3 (2021-06-21)
----------------

### Release Summary

First public release.
