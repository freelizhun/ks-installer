该文件内容对应于/kubesphere/results/preinstall/preInstall/command文件
{
	"command": [
		"ansible-playbook",
		"-b",
		"-e",
		"@/kubesphere/config/ks-config.yaml",
		"-e",
		"@/kubesphere/results/env/extravars",
		"/kubesphere/playbooks/preinstall.yaml"
	],
	"cwd": "/kubesphere/results",
	"env": {
		"KUBERNETES_PORT_443_TCP": "tcp://10.96.0.1:443",
		"ANSIBLE_RETRY_FILES_ENABLED": "False",
		"BINDING_CONTEXT_PATH": "/tmp/shell-operator/hook-kubesphere-installRunner-py-binding-context.json",
		"HOME": "/root",
		"PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
		"KUBERNETES_SERVICE_PORT": "443",
		"ANSIBLE_CALLBACK_PLUGINS": "/usr/lib/python2.7/site-packages/ansible_runner/callbacks",
		"KUBERNETES_SERVICE_HOST": "10.96.0.1",
		"ANSIBLE_ROLES_PATH": "/kubesphere/installer/roles",
		"ANSIBLE_CACHE_PLUGIN": "jsonfile",
		"SHELL_OPERATOR_WORKING_DIR": "/hooks",
		"ANSIBLE_HOST_KEY_CHECKING": "False",
		"ANSIBLE_CACHE_PLUGIN_CONNECTION": "/kubesphere/results/preinstall/preInstall/fact_cache",
		"ANSIBLE_STDOUT_CALLBACK": "awx_display",
		"PYTHONPATH": "/usr/lib/python2.7/site-packages/ansible_runner/callbacks",
		"KUBERNETES_PORT_443_TCP_ADDR": "10.96.0.1",
		"KUBERNETES_PORT": "tcp://10.96.0.1:443",
		"KUBERNETES_SERVICE_PORT_HTTPS": "443",
		"KUBERNETES_PORT_443_TCP_PROTO": "tcp",
		"HOSTNAME": "ks-installer-544d9c7d95-g8vkn",
		"RUNNER_OMIT_EVENTS": "False",
		"WORKING_DIR": "/hooks",
		"AWX_ISOLATED_DATA_DIR": "/kubesphere/results/preinstall/preInstall",
		"KUBERNETES_PORT_443_TCP_PORT": "443",
		"RUNNER_ONLY_FAILED_EVENTS": "False"
	}
}