ansible-role-nfvbench Release Notes
===================================


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
