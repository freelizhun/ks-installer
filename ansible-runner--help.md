```bash
/ # ansible-runner --help
usage: ansible-runner [-h] [--version] [-p PLAYBOOK | -m MODULE | -r ROLE]
                      [--debug] [--logfile LOGFILE] [-b BINARY] [-i IDENT]
                      [--rotate-artifacts ROTATE_ARTIFACTS]
                      [--artifact-dir ARTIFACT_DIR]
                      [--project-dir PROJECT_DIR] [--inventory INVENTORY] [-j]
                      [--omit-event-data] [--only-failed-event-data] [-q] [-v]
                      [--limit LIMIT] [--cmdline CMDLINE] [--hosts HOSTS]
                      [--forks FORKS] [--roles-path ROLES_PATH]
                      [--role-vars ROLE_VARS] [--role-skip-facts]
                      [-a MODULE_ARGS] [--process-isolation]
                      [--process-isolation-executable PROCESS_ISOLATION_EXECUTABLE]
                      [--process-isolation-path PROCESS_ISOLATION_PATH]
                      [--process-isolation-hide-paths PROCESS_ISOLATION_HIDE_PATHS]
                      [--process-isolation-show-paths PROCESS_ISOLATION_SHOW_PATHS]
                      [--process-isolation-ro-paths PROCESS_ISOLATION_RO_PATHS]
                      [--directory-isolation-base-path DIRECTORY_ISOLATION_BASE_PATH]
                      [--resource-profiling]
                      [--resource-profiling-base-cgroup RESOURCE_PROFILING_BASE_CGROUP]
                      [--resource-profiling-cpu-poll-interval RESOURCE_PROFILING_CPU_POLL_INTERVAL]
                      [--resource-profiling-memory-poll-interval RESOURCE_PROFILING_MEMORY_POLL_INTERVAL]
                      [--resource-profiling-pid-poll-interval RESOURCE_PROFILING_PID_POLL_INTERVAL]
                      [--resource-profiling-results-dir RESOURCE_PROFILING_RESULTS_DIR]
                      COMMAND private_data_dir

Use 'ansible-runner' (with no arguments) to see basic usage

positional arguments:
  COMMAND               command directive for controlling ansible-runner
                        execution (one of 'run', 'start', 'stop', 'is-alive')
  private_data_dir      base directory cotnaining the ansible-runner metadata
                        (project, inventory, env, etc)

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -p PLAYBOOK, --playbook PLAYBOOK
                        invoke an Ansible playbook from the ansible-runner
                        project (See Ansible Playbook Options below)
  -m MODULE, --module MODULE
                        invoke an Ansible module directly without a playbook
                        (See Ansible Module Options below)
  -r ROLE, --role ROLE  invoke an Ansible role directly without a playbook
                        (See Ansible Role Options below)

Ansible Runner Options:
  configuration options for controlling the ansible-runner runtime
  environment.

  --debug               enable ansible-runner debug output logging
                        (default=False)
  --logfile LOGFILE     log output messages to a file (default=None)
  -b BINARY, --binary BINARY
                        specifies the full path pointing to the Ansible
                        binaries (default=None)
  -i IDENT, --ident IDENT
                        an identifier that will be used when generating the
                        artifacts directory and can be used to uniquely
                        identify a playbook run (default=ca2e497e-
                        db91-4c39-8f24-dbfec96c9534)
  --rotate-artifacts ROTATE_ARTIFACTS
                        automatically clean up old artifact directories after
                        a given number have been created (default=0, disabled)
  --artifact-dir ARTIFACT_DIR
                        optional path for the artifact root directory
                        (default=<private_data_dir>/artifacts)
  --project-dir PROJECT_DIR
                        optional path for the location of the playbook content
                        directory (default=<private_data_dir/project)
  --inventory INVENTORY
                        optional path for the location of the inventory
                        content directory
                        (default=<private_data_dir>/inventory)
  -j, --json            output the JSON event structure to stdout instead of
                        Ansible output (default=False)
  --omit-event-data     Omits including extra event data in the callback
                        payloads or the Runner payload data files (status and
                        stdout still included)
  --only-failed-event-data
                        Only adds extra event data for failed tasks in the
                        callback payloads or the Runner payload data files
                        (status and stdout still included for other events)
  -q, --quiet           disable all messages sent to stdout/stderr
                        (default=False)
  -v                    increase the verbosity with multiple v's (up to 5) of
                        the ansible-playbook output (default=None)

Ansible Options:
  control the ansible[-playbook] execution environment

  --limit LIMIT         matches Ansible's ```--limit``` parameter to further
                        constrain the inventory to be used (default=None)
  --cmdline CMDLINE     command line options to pass to ansible-playbook at
                        execution time (default=None)
  --hosts HOSTS         define the set of hosts to execute against
                        (default=None) Note: this parameter only works with -m
                        or -r
  --forks FORKS         matches Ansible's ```--forks``` parameter to set the
                        number of conconurent processes (default=None)

Ansible Role Options:
  configuration options for directly executing Ansible roles

  --roles-path ROLES_PATH
                        path used to locate the role to be executed
                        (default=None)
  --role-vars ROLE_VARS
                        set of variables to be passed to the role at run time
                        in the form of 'key1=value1 key2=value2
                        keyN=valueN'(default=None)
  --role-skip-facts     disable fact collection when the role is executed
                        (default=False)

Ansible Module Options:
  configuration options for directly executing Ansible modules

  -a MODULE_ARGS, --args MODULE_ARGS
                        set of arguments to be passed to the module at run
                        time in the form of 'key1=value1 key2=value2
                        keyN=valueN'(default=None)

Ansible Playbook Options:
  configuation options for executing Ansible playbooks

  --process-isolation   limits what directories on the filesystem the playbook
                        run has access to, defaults to /tmp (default=False)
  --process-isolation-executable PROCESS_ISOLATION_EXECUTABLE
                        process isolation executable that will be used.
                        (default=bwrap)
  --process-isolation-path PROCESS_ISOLATION_PATH
                        path that an isolated playbook run will use for
                        staging. (default=/tmp)
  --process-isolation-hide-paths PROCESS_ISOLATION_HIDE_PATHS
                        list of paths on the system that should be hidden from
                        the playbook run (default=None)
  --process-isolation-show-paths PROCESS_ISOLATION_SHOW_PATHS
                        list of paths on the system that should be exposed to
                        the playbook run (default=None)
  --process-isolation-ro-paths PROCESS_ISOLATION_RO_PATHS
                        list of paths on the system that should be exposed to
                        the playbook run as read-only (default=None)
  --directory-isolation-base-path DIRECTORY_ISOLATION_BASE_PATH
                        copies the project directory to a location in this
                        directory to prevent multiple simultaneous executions
                        from conflicting (default=None)
  --resource-profiling  Records resource utilization during playbook execution
  --resource-profiling-base-cgroup RESOURCE_PROFILING_BASE_CGROUP
                        Top-level cgroup used to collect information on
                        resource utilization. Defaults to ansible-runner
  --resource-profiling-cpu-poll-interval RESOURCE_PROFILING_CPU_POLL_INTERVAL
                        Interval (in seconds) between CPU polling for
                        determining CPU usage. Defaults to 0.25
  --resource-profiling-memory-poll-interval RESOURCE_PROFILING_MEMORY_POLL_INTERVAL
                        Interval (in seconds) between memory polling for
                        determining memory usage. Defaults to 0.25
  --resource-profiling-pid-poll-interval RESOURCE_PROFILING_PID_POLL_INTERVAL
                        Interval (in seconds) between polling PID count for
                        determining number of processes used. Defaults to 0.25
  --resource-profiling-results-dir RESOURCE_PROFILING_RESULTS_DIR
                        Directory where profiling data files should be saved.
                        Defaults to None (profiling_data folder under private
                        data dir is used in this case).
```