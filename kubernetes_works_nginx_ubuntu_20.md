# 2023-05 nginx on Ubuntu 20

-   [2023-05 nginx on Ubuntu 20](#nginx-on-ubuntu-20)
    -   [Server Node (antec)](#server-node-antec)
        -   [taint nodes](#taint-nodes)
        -   [install containerd](#install-containerd)
        -   [taint nodes](#taint-nodes-1)
        -   [Create cluster](#create-cluster)
        -   [apply flannel](#apply-flannel)
        -   [Check \_\_\_](#check-___)
    -   [nuc2020](#nuc2020)
        -   [~~install kubeadm, kubectl
            etc.~~](#install-kubeadm-kubectl-etc.)
        -   [join cluster](#join-cluster)
    -   [antec](#antec)
        -   [Check node joined
            successfully](#check-node-joined-successfully)
        -   [Testing](#testing)
    -   [Client Node](#client-node)
    
## Server Node (antec)


	antec Sun 07 May 2023  7:53PM> kubectl get all --all-namespaces                                                                     /home/sarnobat

		E0507 19:58:34.124367  648188 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 19:58:34.125132  648188 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 19:58:34.126834  648188 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 19:58:34.128708  648188 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 19:58:34.130525  648188 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		The connection to the server 192.168.1.4:6443 was refused - did you specify the right host or port?

	antec Sun 07 May 2023  7:58PM> sudo kubeadm reset                                                                                   /home/sarnobat

	W0507 19:58:42.526781  648199 preflight.go:56] [reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
	[reset] Are you sure you want to proceed? [y/N]: y
	[preflight] Running pre-flight checks
	W0507 19:58:45.186816  648199 removeetcdmember.go:106] [reset] No kubeadm config, using etcd pod spec to get data directory
	[reset] Stopping the kubelet service
	[reset] Unmounting mounted directories in "/var/lib/kubelet"
	[reset] Deleting contents of directories: [/etc/kubernetes/manifests /var/lib/kubelet /etc/kubernetes/pki]
	[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

	The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

	The reset process does not reset or clean up iptables rules or IPVS tables.
	If you wish to reset iptables, you must do so manually by using the "iptables" command.

	If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
	to reset your system's IPVS tables.

	The reset process does not clean your kubeconfig files and you must remove them manually.
	Please, check the contents of the $HOME/.kube/config file.

### ~~taint nodes~~
TODO: Explain what this means

~~
	antec Sun 07 May 2023  7:58PM> kubectl taint nodes --all                                                                            /home/sarnobat

		error: at least one taint update is required
~~

	antec Sun 07 May 2023  7:59PM> kubectl taint nodes --all node-role.kubernetes.io/control-plane-                                     /home/sarnobat

		E0507 19:59:52.834337  648465 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 19:59:52.837840  648465 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		The connection to the server 192.168.1.4:6443 was refused - did you specify the right host or port?

	antec Sun 07 May 2023  7:59PM> ps aux | grep containerd                                                                             /home/sarnobat

		root        2180  0.2  0.0 2321040 23028 ?       Ssl  Apr06 123:04 /usr/bin/containerd
		sarnobat  648481  0.0  0.0  17864  2276 pts/19   S+   20:00   0:00 grep containerd
		root     1560181  0.0  0.0 1891168 23920 ?       Ssl  Apr24  12:43 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

	antec Sun 07 May 2023  8:00PM> sudo systemctl status containerd                                                                     /home/sarnobat

	● containerd.service - containerd container runtime
		 Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
		 Active: active (running) since Thu 2023-04-06 17:46:58 PDT; 1 month 0 days ago
		   Docs: https://containerd.io
	   Main PID: 2180 (containerd)
		  Tasks: 15
		 Memory: 33.5M
			CPU: 2h 42min 29.350s
		 CGroup: /system.slice/containerd.service
				 └─2180 /usr/bin/containerd

	Notice: journal has been rotated since unit was started, output may be incomplete.
	antec Sun 07 May 2023  8:00PM> sudo mkdir -p /etc/containerd/; containerd config default | sudo tee /etc/containerd/config.toml     /home/sarnobat

		disabled_plugins = []
		imports = []
		oom_score = 0
		plugin_dir = ""
		required_plugins = []
		root = "/var/lib/containerd"
		state = "/run/containerd"
		temp = ""
		version = 2

		[cgroup]
		  path = ""

		[debug]
		  address = ""
		  format = ""
		  gid = 0
		  level = ""
		  uid = 0

		[grpc]
		  address = "/run/containerd/containerd.sock"
		  gid = 0
		  max_recv_message_size = 16777216
		  max_send_message_size = 16777216
		  tcp_address = ""
		  tcp_tls_ca = ""
		  tcp_tls_cert = ""
		  tcp_tls_key = ""
		  uid = 0

		[metrics]
		  address = ""
		  grpc_histogram = false

		[plugins]

		  [plugins."io.containerd.gc.v1.scheduler"]
			deletion_threshold = 0
			mutation_threshold = 100
			pause_threshold = 0.02
			schedule_delay = "0s"
			startup_delay = "100ms"

		  [plugins."io.containerd.grpc.v1.cri"]
			device_ownership_from_security_context = false
			disable_apparmor = false
			disable_cgroup = false
			disable_hugetlb_controller = true
			disable_proc_mount = false
			disable_tcp_service = true
			enable_selinux = false
			enable_tls_streaming = false
			enable_unprivileged_icmp = false
			enable_unprivileged_ports = false
			ignore_image_defined_volumes = false
			max_concurrent_downloads = 3
			max_container_log_line_size = 16384
			netns_mounts_under_state_dir = false
			restrict_oom_score_adj = false
			sandbox_image = "registry.k8s.io/pause:3.6"
			selinux_category_range = 1024
			stats_collect_period = 10
			stream_idle_timeout = "4h0m0s"
			stream_server_address = "127.0.0.1"
			stream_server_port = "0"
			systemd_cgroup = false
			tolerate_missing_hugetlb_controller = true
			unset_seccomp_profile = ""

			[plugins."io.containerd.grpc.v1.cri".cni]
			  bin_dir = "/opt/cni/bin"
			  conf_dir = "/etc/cni/net.d"
			  conf_template = ""
			  ip_pref = ""
			  max_conf_num = 1

			[plugins."io.containerd.grpc.v1.cri".containerd]
			  default_runtime_name = "runc"
			  disable_snapshot_annotations = true
			  discard_unpacked_layers = false
			  ignore_rdt_not_enabled_errors = false
			  no_pivot = false
			  snapshotter = "overlayfs"

			  [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
				base_runtime_spec = ""
				cni_conf_dir = ""
				cni_max_conf_num = 0
				container_annotations = []
				pod_annotations = []
				privileged_without_host_devices = false
				runtime_engine = ""
				runtime_path = ""
				runtime_root = ""
				runtime_type = ""

				[plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

			  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

				[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
				  base_runtime_spec = ""
				  cni_conf_dir = ""
				  cni_max_conf_num = 0
				  container_annotations = []
				  pod_annotations = []
				  privileged_without_host_devices = false
				  runtime_engine = ""
				  runtime_path = ""
				  runtime_root = ""
				  runtime_type = "io.containerd.runc.v2"

				  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
					BinaryName = ""
					CriuImagePath = ""
					CriuPath = ""
					CriuWorkPath = ""
					IoGid = 0
					IoUid = 0
					NoNewKeyring = false
					NoPivotRoot = false
					Root = ""
					ShimCgroup = ""
					SystemdCgroup = false

			  [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
				base_runtime_spec = ""
				cni_conf_dir = ""
				cni_max_conf_num = 0
				container_annotations = []
				pod_annotations = []
				privileged_without_host_devices = false
				runtime_engine = ""
				runtime_path = ""
				runtime_root = ""
				runtime_type = ""

				[plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

			[plugins."io.containerd.grpc.v1.cri".image_decryption]
			  key_model = "node"

			[plugins."io.containerd.grpc.v1.cri".registry]
			  config_path = ""

			  [plugins."io.containerd.grpc.v1.cri".registry.auths]

			  [plugins."io.containerd.grpc.v1.cri".registry.configs]

			  [plugins."io.containerd.grpc.v1.cri".registry.headers]

			  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

			[plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
			  tls_cert_file = ""
			  tls_key_file = ""

		  [plugins."io.containerd.internal.v1.opt"]
			path = "/opt/containerd"

		  [plugins."io.containerd.internal.v1.restart"]
			interval = "10s"

		  [plugins."io.containerd.internal.v1.tracing"]
			sampling_ratio = 1.0
			service_name = "containerd"

		  [plugins."io.containerd.metadata.v1.bolt"]
			content_sharing_policy = "shared"

		  [plugins."io.containerd.monitor.v1.cgroups"]
			no_prometheus = false

		  [plugins."io.containerd.runtime.v1.linux"]
			no_shim = false
			runtime = "runc"
			runtime_root = ""
			shim = "containerd-shim"
			shim_debug = false

		  [plugins."io.containerd.runtime.v2.task"]
			platforms = ["linux/amd64"]
			sched_core = false

		  [plugins."io.containerd.service.v1.diff-service"]
			default = ["walking"]

		  [plugins."io.containerd.service.v1.tasks-service"]
			rdt_config_file = ""

		  [plugins."io.containerd.snapshotter.v1.aufs"]
			root_path = ""

		  [plugins."io.containerd.snapshotter.v1.btrfs"]
			root_path = ""

		  [plugins."io.containerd.snapshotter.v1.devmapper"]
			async_remove = false
			base_image_size = ""
			discard_blocks = false
			fs_options = ""
			fs_type = ""
			pool_name = ""
			root_path = ""

		  [plugins."io.containerd.snapshotter.v1.native"]
			root_path = ""

		  [plugins."io.containerd.snapshotter.v1.overlayfs"]
			root_path = ""
			upperdir_label = false

		  [plugins."io.containerd.snapshotter.v1.zfs"]
			root_path = ""

		  [plugins."io.containerd.tracing.processor.v1.otlp"]
			endpoint = ""
			insecure = false
			protocol = ""

		[proxy_plugins]

		[stream_processors]

		  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
			accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
			args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
			env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
			path = "ctd-decoder"
			returns = "application/vnd.oci.image.layer.v1.tar"

		  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
			accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
			args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
			env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
			path = "ctd-decoder"
			returns = "application/vnd.oci.image.layer.v1.tar+gzip"

		[timeouts]
		  "io.containerd.timeout.bolt.open" = "0s"
		  "io.containerd.timeout.shim.cleanup" = "5s"
		  "io.containerd.timeout.shim.load" = "5s"
		  "io.containerd.timeout.shim.shutdown" = "3s"
		  "io.containerd.timeout.task.state" = "2s"

		[ttrpc]
		  address = ""
		  gid = 0
		  uid = 0

### install containerd

	antec Sun 07 May 2023  8:00PM> sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml           /home/sarnobat

	antec Sun 07 May 2023  8:00PM> sudo systemctl restart containerd                                                                    /home/sarnobat

	antec Sun 07 May 2023  8:00PM> kubectl taint nodes --all node-role.kubernetes.io/control-plane-                                     /home/sarnobat

		E0507 20:00:44.968912  648545 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 20:00:44.969496  648545 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		The connection to the server 192.168.1.4:6443 was refused - did you specify the right host or port?

### taint nodes

	antec Sun 07 May 2023  8:00PM> swapoff -a                                                                                           /home/sarnobat
	antec Sun 07 May 2023  8:01PM> sudo systemctl restart kubelet                                                                       /home/sarnobat
	antec Sun 07 May 2023  8:01PM> kubectl taint nodes --all node-role.kubernetes.io/control-plane-                                     /home/sarnobat

		E0507 20:01:33.624612  648951 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 20:01:33.625571  648951 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		The connection to the server 192.168.1.4:6443 was refused - did you specify the right host or port?

	antec Sun 07 May 2023  8:01PM> sudo swapoff -a                                                                                      /home/sarnobat

	antec Sun 07 May 2023  8:02PM> sudo hostnamectl set-hostname kubernetes-master                                                      /home/sarnobat
	antec Sun 07 May 2023  8:02PM> sudo hostnamectl set-hostname kubernetes-worker                                                      /home/sarnobat

	antec Sun 07 May 2023  8:02PM> lsmod | grep br_netfilter                                                                            /home/sarnobat

		br_netfilter           32768  0
		bridge                331776  1 br_netfilter

	antec Sun 07 May 2023  8:02PM> sudo sysctl net.bridge.bridge-nf-call-iptables=1                                                     /home/sarnobat

		net.bridge.bridge-nf-call-iptables = 1

### Create cluster

	antec Sun 07 May 2023  8:02PM> sudo kubeadm init --pod-network-cidr=10.244.0.0/16                                                   /home/sarnobat

		I0507 20:03:07.447819  649218 version.go:256] remote version is much newer: v1.27.1; falling back to: stable-1.26
		[init] Using Kubernetes version: v1.26.4
		[preflight] Running pre-flight checks
		[preflight] Pulling images required for setting up a Kubernetes cluster
		[preflight] This might take a minute or two, depending on the speed of your internet connection
		[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
		[certs] Using certificateDir folder "/etc/kubernetes/pki"
		[certs] Generating "ca" certificate and key
		[certs] Generating "apiserver" certificate and key
		[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes-worker kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.4]
		[certs] Generating "apiserver-kubelet-client" certificate and key
		[certs] Generating "front-proxy-ca" certificate and key
		[certs] Generating "front-proxy-client" certificate and key
		[certs] Generating "etcd/ca" certificate and key
		[certs] Generating "etcd/server" certificate and key
		[certs] etcd/server serving cert is signed for DNS names [kubernetes-worker localhost] and IPs [192.168.1.4 127.0.0.1 ::1]
		[certs] Generating "etcd/peer" certificate and key
		[certs] etcd/peer serving cert is signed for DNS names [kubernetes-worker localhost] and IPs [192.168.1.4 127.0.0.1 ::1]
		[certs] Generating "etcd/healthcheck-client" certificate and key
		[certs] Generating "apiserver-etcd-client" certificate and key
		[certs] Generating "sa" key and public key
		[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
		[kubeconfig] Writing "admin.conf" kubeconfig file
		[kubeconfig] Writing "kubelet.conf" kubeconfig file
		[kubeconfig] Writing "controller-manager.conf" kubeconfig file
		[kubeconfig] Writing "scheduler.conf" kubeconfig file
		[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
		[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
		[kubelet-start] Starting the kubelet
		[control-plane] Using manifest folder "/etc/kubernetes/manifests"
		[control-plane] Creating static Pod manifest for "kube-apiserver"
		[control-plane] Creating static Pod manifest for "kube-controller-manager"
		[control-plane] Creating static Pod manifest for "kube-scheduler"
		[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
		[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
		[apiclient] All control plane components are healthy after 24.513865 seconds
		[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
		[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
		[upload-certs] Skipping phase. Please see --upload-certs
		[mark-control-plane] Marking the node kubernetes-worker as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
		[mark-control-plane] Marking the node kubernetes-worker as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
		[bootstrap-token] Using token: 33hpb0.ps5p55au5vdualfr
		[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
		[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
		[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
		[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
		[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
		[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
		[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
		[addons] Applied essential addon: CoreDNS
		[addons] Applied essential addon: kube-proxy

		Your Kubernetes control-plane has initialized successfully!

		To start using your cluster, you need to run the following as a regular user:

		  mkdir -p $HOME/.kube
		  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		  sudo chown $(id -u):$(id -g) $HOME/.kube/config

		Alternatively, if you are the root user, you can run:

		  export KUBECONFIG=/etc/kubernetes/admin.conf

		You should now deploy a pod network to the cluster.
		Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
		  https://kubernetes.io/docs/concepts/cluster-administration/addons/

		Then you can join any number of worker nodes by running the following on each as root:

		kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr \
			--discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78

	antec Sun 07 May 2023  8:04PM> sudo kubeadm init --pod-network-cidr=10.244.0.0/16                                                   /home/sarnobat
	antec Sun 07 May 2023  8:08PM> mkdir -p $HOME/.kube                                                                                 /home/sarnobat

		sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		sudo chown $(id -u):$(id -g) $HOME/.kube/config
		cp: overwrite '/home/sarnobat/.kube/config'? y

	antec Sun 07 May 2023  8:08PM> sudo ufw allow 6443                                                                                  /home/sarnobat

		sudo ufw allow 6443/tcp
		Rules updated
		Rules updated (v6)
		Rules updated
		Rules updated (v6)

### apply flannel
TODO: explain why

	antec Sun 07 May 2023  8:08PM> kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

		kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
		namespace/kube-flannel created
		clusterrole.rbac.authorization.k8s.io/flannel created
		clusterrolebinding.rbac.authorization.k8s.io/flannel created
		serviceaccount/flannel created
		configmap/kube-flannel-cfg created
		daemonset.apps/kube-flannel-ds created
		error: unable to read URL "https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml", server reported 404 Not Found, status code=404

	antec Sun 07 May 2023  8:08PM> kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

		namespace/kube-flannel unchanged
		clusterrole.rbac.authorization.k8s.io/flannel unchanged
		clusterrolebinding.rbac.authorization.k8s.io/flannel unchanged
		serviceaccount/flannel unchanged
		configmap/kube-flannel-cfg unchanged
		daemonset.apps/kube-flannel-ds unchanged

	antec Sun 07 May 2023  8:08PM> kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml

		error: unable to read URL "https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml", server reported 404 Not Found, status code=404

	antec Sun 07 May 2023  8:08PM> kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml

		resource mapping not found for name: "flannel" namespace: "" from "https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1"
		ensure CRDs are installed first
		resource mapping not found for name: "flannel" namespace: "" from "https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
		ensure CRDs are installed first

	antec Sun 07 May 2023  8:09PM> kubectl get pods --all-namespaces                                                                    /home/sarnobat

		NAMESPACE      NAME                                        READY   STATUS    RESTARTS   AGE
		  namespace: kube-system
		spec:
		  containers:
		  - command:
			- kube-scheduler
			- --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
			- --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
			- --bind-address=127.0.0.1
			- --kubeconfig=/etc/kubernetes/scheduler.conf
			- --leader-elect=true
			image: registry.k8s.io/kube-scheduler:v1.26.4
			imagePullPolicy: IfNotPresent
			livenessProbe:
			  failureThreshold: 8
			  httpGet:
				host: 127.0.0.1
				path: /healthz
				port: 10259
				scheme: HTTPS
			  initialDelaySeconds: 10
			  periodSeconds: 10
			  timeoutSeconds: 15
			name: kube-scheduler
		"/etc/kubernetes/manifests/kube-scheduler.yaml" 59L, 1440B                                                                          26,9          22%
		kube-flannel   kube-flannel-ds-wdrzc                       1/1     Running   0          69s
		kube-system    coredns-787d4945fb-224pv                    1/1     Running   0          4m51s
		kube-system    coredns-787d4945fb-zmzf6                    1/1     Running   0          4m51s
		kube-system    etcd-kubernetes-worker                      1/1     Running   0          4m58s
		kube-system    kube-apiserver-kubernetes-worker            1/1     Running   0          4m58s
		kube-system    kube-controller-manager-kubernetes-worker   1/1     Running   0          4m58s
		kube-system    kube-proxy-x97gx                            1/1     Running   0          4m51s
		kube-system    kube-scheduler-kubernetes-worker            1/1     Running   0          4m58s

### Check ___
	antec Sun 07 May 2023  8:09PM> kubectl get componentstatus                                                                          /home/sarnobat

		Warning: v1 ComponentStatus is deprecated in v1.19+
		NAME                 STATUS    MESSAGE                         ERROR
		controller-manager   Healthy   ok
		scheduler            Healthy   ok
		etcd-0               Healthy   {"health":"true","reason":""}

	antec Sun 07 May 2023  8:09PM> kubectl get cs                                                                                       /home/sarnobat

		Warning: v1 ComponentStatus is deprecated in v1.19+
		NAME                 STATUS    MESSAGE                         ERROR
		scheduler            Healthy   ok
		controller-manager   Healthy   ok
		etcd-0               Healthy   {"health":"true","reason":""}

	antec Sun 07 May 2023  8:09PM> sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml +/port                                         /home/sarnobat
	antec Sun 07 May 2023  8:10PM> sudo systemctl restart kubelet.service                                                               /home/sarnobat

	antec Sun 07 May 2023  8:10PM> kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr    --discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78

		[preflight] Running pre-flight checks
		error execution phase preflight: [preflight] Some fatal errors occurred:
			[ERROR IsPrivilegedUser]: user is not running as root
		[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
		To see the stack trace of this error execute with --v=5 or higher

	antec Sun 07 May 2023  8:11PM> sudo kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr       --discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78

		[preflight] Running pre-flight checks
		error execution phase preflight: [preflight] Some fatal errors occurred:
			[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
			[ERROR Port-10250]: Port 10250 is in use
			[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
		[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
		To see the stack trace of this error execute with --v=5 or higher

	antec Sun 07 May 2023  8:11PM> kubectl get nodes                                                                                    /home/sarnobat

		NAME                STATUS   ROLES           AGE   VERSION
		kubernetes-worker   Ready    control-plane   23m   v1.26.3

	antec Sun 07 May 2023  8:27PM> set                                                                                                  /home/sarnobat
	antec Sun 07 May 2023  8:28PM> history                                                                                              /home/sarnobat

		27334  sudo sysctl net.bridge.bridge-nf-call-iptables=1
		27335  sudo kubeadm init --pod-network-cidr=10.244.0.0/16
		27336  mkdir -p $HOME/.kube\nsudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config\nsudo chown $(id -u):$(id -g) $HOME/.kube/config
		27337  sudo ufw allow 6443\nsudo ufw allow 6443/tcp
		27338  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml\nkubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
		27339  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
		27340  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
		27341  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml
		27342  kubectl get pods --all-namespaces\n
		27343  kubectl get componentstatus\n
		27344  kubectl get cs\n
		27345  sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml +/port
		27346  sudo systemctl restart kubelet.service
		27347  kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr\t--discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78
		27348  sudo kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr\t--discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78
		27349  kubectl get nodes

	antec Sun 07 May 2023  8:28PM> history -100                                                                                         /home/sarnobat

		27251  cd /media/sarnobat/_30_olive/ && sh /tmp/rsync_relative.sh old_computers/100G /media/sarnobat/_44_rackley/
		27252  cd /media/sarnobat/_30_olive/ && bash /tmp/rsync_relative.sh old_computers/100G /media/sarnobat/_44_rackley/
		27253  sh ~/bin/rsync_videos_stripe_job.sh
		27254  ls /media/sarnobat/_30_olive/
		27255  find /media/sarnobat/_30_olive/ -type d -iname "10G"
		27256  find /media/sarnobat/_30_olive/ -type d -iname "100G"
		27257  tmux new rsync_striped_reorganize
		27258  tmux new -s rsync_striped_reorganize
		27259  ls_striped.sh
		27260  df
		27261  df | grep scar
		27262  ~/bin/df.sh
		27263  ~/bin/df.sh | grep scarl
		27264  ~/bin/df.sh | head
		27265  ~/bin/df.sh | head -20
		27266  ls_striped.sh | grep 100M
		27267  ls_striped.sh | grep -v navajo | grep -v sandy | grep -v backup |  grep 100M
		27268  ls_striped.sh | grep -v navajo | grep -v sandy | grep -v backup |  grep -v cache | grep 100M
		27269  crontab -e
		27270  sh ~/bin/rsync_videos_stripe_job.sh
		27271  sh ~/bin/rsync_videos_stripe_job.sh
		27272  sh ~/bin/rsync_videos_stripe_job.sh
		27273  ~/bin/df.sh | head -30
		27274  sh ~/bin/rsync_videos_stripe_job.sh
		27275  sh ~/bin/rsync_videos_stripe_job.sh
		27276  cat /tmp/du_videos.txt | grep 100G
		27277  sh ~/bin/rmdir_recursive.sh /media/sarnobat/_27_moonstone/old_computers/2022-04-09_kingston_240G.nuc2017.overheating/Videos/100G
		27278  find /media/sarnobat/_27_moonstone/old_computers/2022-04-09_kingston_240G.nuc2017.overheating/Videos/100G
		27279  find /media/sarnobat/_27_moonstone/old_computers/
		27280  cat /tmp/du_videos.txt | grep 100G
		27281  ~/bin/df.sh
		27282  ~/bin/df.sh | head -20
		27283  ~/bin/df.sh | head -30
		27284  ls /media/sarnobat/_42_purpureus
		27285  cat /tmp/du_videos.txt | grep 10G
		27286  cat /tmp/du_videos.txt | grep 10G$
		27287  cat /tmp/du_videos.txt | grep 10G$ | grep -v 3TB
		27288  cat /tmp/du_videos.txt | grep 10G$ | grep -v 3TB | perl -pe 's{.*?/media}{/media}g'
		27289  cat /tmp/du_videos.txt | grep 10G$ | grep -v 3TB | perl -pe 's{.*?/media}{/media}g' | sort
		27290  ~/bin/df.sh
		27291  ~/bin/df.sh | head -30
		27292  ~/bin/df.sh | head -40
		27293  ~/bin/df.sh | grep _40
		27294  ~/bin/df.sh | grep _4
		27295  ~/bin/df.sh | grep _2
		27296  ls_striped.sh | grep 1G
		27297  cat /tmp/du_videos.txt | grep 1G$ | grep -v 3TB | perl -pe 's{.*?/media}{/media}g' | sort
		27298  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort
		27299  ls /media/sarnobat/_7_unmellow/videos/1k/1G
		27300  find /media/sarnobat/_7_unmellow/videos/1k/1G
		27301  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort
		27302  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)}{cd $1}g'
		27303  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)}{cd $1}'
		27304  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)}{cd $1\t\t}'
		27305  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)(videos.*)}{cd $1\t\tsh /tmp/rsync_relative.sh }'
		27306  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)(videos.*)}{cd $1\t\tsh /tmp/rsync_relative.sh $2}'
		27307  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)(videos.*)}{cd $1\t\tsh /tmp/rsync_relative.sh $2}' | grep -v _2
		27308  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)(videos.*)}{cd $1\t\tsh /tmp/rsync_relative.sh $2}' | grep -v _2 | grep -v _3
		27309  cat /tmp/du_videos.txt | grep 1G$ | grep -v -e 3TB -e unmirr | perl -pe 's{.*?/media}{/media}g' | sort | perl -pe 's{(.*)(videos.*)}{cd $1\t\tsh /tmp/rsync_relative.sh $2\t}' | grep -v _2 | grep -v _3
		27310  ~/bin/df.sh
		27311  ~/bin/df.sh | grep _2
		27312  ~/bin/df.sh | grep -e _2 -e _3
		27313  ~/bin/df.sh | grep -e _2 -e _3 | head
		27314  tmux attach-session -t rsync_striped_reorganize
		27315  source ~/.zshrc
		27316  ~/bin/df.sh
		27317  kubectl get all --all-namespaces
		27318  sudo kubeadm reset\n
		27319  kubectl taint nodes --all
		27320  kubectl taint nodes --all node-role.kubernetes.io/control-plane-\n
		27321  ps aux | grep containerd
		27322  sudo systemctl status containerd\n
		27323  sudo mkdir -p /etc/containerd/; containerd config default | sudo tee /etc/containerd/config.toml\n
		27324  sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml\n
		27325  sudo systemctl restart containerd\n
		27326  kubectl taint nodes --all node-role.kubernetes.io/control-plane-\n
		27327  swapoff -a
		27328  sudo systemctl restart kubelet
		27329  kubectl taint nodes --all node-role.kubernetes.io/control-plane-\n
		27330  sudo swapoff -a\n
		27331  sudo hostnamectl set-hostname kubernetes-master
		27332  sudo hostnamectl set-hostname kubernetes-worker\n
		27333  lsmod | grep br_netfilter\n
		27334  sudo sysctl net.bridge.bridge-nf-call-iptables=1
		27335  sudo kubeadm init --pod-network-cidr=10.244.0.0/16
		27336  mkdir -p $HOME/.kube\nsudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config\nsudo chown $(id -u):$(id -g) $HOME/.kube/config
		27337  sudo ufw allow 6443\nsudo ufw allow 6443/tcp
		27338  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml\nkubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
		27339  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
		27340  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
		27341  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml
		27342  kubectl get pods --all-namespaces\n
		27343  kubectl get componentstatus\n
		27344  kubectl get cs\n
		27345  sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml +/port
		27346  sudo systemctl restart kubelet.service
		27347  kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr\t--discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78
		27348  sudo kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr\t--discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78
		27349  kubectl get nodes
		27350  history
		antec Sun 07 May 2023  8:28PM> history -100 | grep -i swap                                                                          /home/sarnobat
		27327  swapoff -a
		27330  sudo swapoff -a\n

## nuc2020

### ~~install kubeadm, kubectl etc.~~

	  ipfs pin update <from-path> <to-path> - Update a recursive pin.
	  ipfs pin verify                       - Verify that recursive pins are complete.

	  For more information about each command, use:
	  'ipfs pin <subcmd> --help'


	appserver Sun 07 May 2023 12:44AM> ipfs pin ls QmUZUmSBL4TGGqw2eNjKPpyPdvGpJUHBBoihradVWrSsEm                                 /home/sarnobat/trash
	QmUZUmSBL4TGGqw2eNjKPpyPdvGpJUHBBoihradVWrSsEm recursive
	appserver Sun 07 May 2023 12:44AM> ipfs pin ls *                                                                              /home/sarnobat/trash
	Error: invalid path "2022-09-09": selected encoding not supported
	appserver Sun 07 May 2023  1:20AM> ipfs pin ls .                                                                              /home/sarnobat/trash
	Error: invalid path ".": cid too short
	appserver Sun 07 May 2023  1:20AM> ipfs pin ls                                                                                /home/sarnobat/trash
	QmQGiYLVAdSHJQKYFRTJZMG4BXBHqKperaZtyKGmCRLmsF indirect
	QmQy6xmJhrcC5QLboAcGFcAE1tC8CrwDVkrHdEYJkLscrQ indirect
	QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
	QmUZUmSBL4TGGqw2eNjKPpyPdvGpJUHBBoihradVWrSsEm recursive
	QmPZ9gcCEpqKTo6aq61g2nXGUhM4iCL3ewB6LDXZCtioEB indirect
	QmQ5vhrL7uv6tuoN9KeVBwd4PwfQkXdVVmDLUZuTNxqgvm indirect
	QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc recursive
	QmU5k7ter3RdjZXu3sHghsga1UQtrztnQxmTL22nPnsu3g indirect
	QmYCvbfNbCwFR45HiNP45rwJgvatpiW38D961L5qAhUM5Y indirect
	QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y indirect
	appserver Sun 07 May 2023  1:20AM> ipfs                                                                                       /home/sarnobat/trash
	USAGE
	  ipfs  - Global p2p merkle-dag filesystem.

	  ipfs [--config=<config> | -c] [--debug | -D] [--help] [-h] [--api=<api>] [--offline] [--cid-base=<base>] [--upgrade-cidv0-in-output] [--encoding=<encoding> | --enc] [--timeout=<timeout>] <command> ...

	SUBCOMMANDS
	  BASIC COMMANDS
		init          Initialize local IPFS configuration
		add <path>    Add a file to IPFS
		cat <ref>     Show IPFS object data
		get <ref>     Download IPFS objects
		ls <ref>      List links from an object
		refs <ref>    List hashes of links from an object

	  DATA STRUCTURE COMMANDS
		dag           Interact with IPLD DAG nodes
		files         Interact with files as if they were a unix filesystem
		block         Interact with raw blocks in the datastore

	  TEXT ENCODING COMMANDS
		cid           Convert and discover properties of CIDs
		multibase     Encode and decode data with Multibase format

	  ADVANCED COMMANDS
		daemon        Start a long-running daemon process
		shutdown      Shut down the daemon process
		resolve       Resolve any type of content path
		name          Publish and resolve IPNS names
		key           Create and list IPNS name keypairs
		pin           Pin objects to local storage
		repo          Manipulate the IPFS repository
		stats         Various operational stats
		p2p           Libp2p stream mounting (experimental)
		filestore     Manage the filestore (experimental)
		mount         Mount an IPFS read-only mount point (experimental)

	  NETWORK COMMANDS
		id            Show info about IPFS peers
		bootstrap     Add or remove bootstrap peers
		swarm         Manage connections to the p2p network
		dht           Query the DHT for values or peers
		routing       Issue routing commands
		ping          Measure the latency of a connection
		bitswap       Inspect bitswap state
		pubsub        Send and receive messages via pubsub

	  TOOL COMMANDS
		config        Manage configuration
		version       Show IPFS version information
		diag          Generate diagnostic reports
		update        Download and apply go-ipfs updates
		commands      List all available commands
		log           Manage and show logs of running daemon

	  Use 'ipfs <command> --help' to learn more about each command.

	  ipfs uses a repository in the local file system. By default, the repo is
	  located at ~/.ipfs. To change the repo location, set the $IPFS_PATH
	  environment variable:

		export IPFS_PATH=/path/to/ipfsrepo

	  EXIT STATUS

	  The CLI will exit with one of the following values:

	  0     Successful execution.
	  1     Failed executions.

	appserver Sun 07 May 2023  1:20AM> ipfs pin ls                                                                                /home/sarnobat/trash
	QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
	QmYCvbfNbCwFR45HiNP45rwJgvatpiW38D961L5qAhUM5Y indirect
	QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y indirect
	QmPZ9gcCEpqKTo6aq61g2nXGUhM4iCL3ewB6LDXZCtioEB indirect
	QmQy6xmJhrcC5QLboAcGFcAE1tC8CrwDVkrHdEYJkLscrQ indirect
	QmU5k7ter3RdjZXu3sHghsga1UQtrztnQxmTL22nPnsu3g indirect
	QmUZUmSBL4TGGqw2eNjKPpyPdvGpJUHBBoihradVWrSsEm recursive
	QmQ5vhrL7uv6tuoN9KeVBwd4PwfQkXdVVmDLUZuTNxqgvm indirect
	QmQGiYLVAdSHJQKYFRTJZMG4BXBHqKperaZtyKGmCRLmsF indirect
	QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc recursive
	appserver Sun 07 May 2023  1:20AM> ipfs resolve QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y                                /home/sarnobat/trash
	/ipfs/QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y
	appserver Sun 07 May 2023  1:20AM> ipfs name QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y                                   /home/sarnobat/trash
	Error: expected 0 argument(s), got 1

	USAGE
	  ipfs name - Publish and resolve IPNS names.

	  ipfs name

	  IPNS is a PKI namespace, where names are the hashes of public keys, and
	  the private key enables publishing new (signed) values. In both publish
	  and resolve, the default name used is the node's own PeerID,
	  which is the hash of its public key.

	SUBCOMMANDS
	  ipfs name publish <ipfs-path> - Publish IPNS names.
	  ipfs name resolve [<name>]    - Resolve IPNS names.

	  For more information about each command, use:
	  'ipfs name <subcmd> --help'

	EXPERIMENTAL SUBCOMMANDS
	  ipfs name inspect <record> - Inspects an IPNS Record
	  ipfs name pubsub           - IPNS pubsub management


	appserver Sun 07 May 2023  1:20AM> ipfs name inspect QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y                           /home/sarnobat/trash

	Error: lstat QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y: no such file or directory

	WARNING:   EXPERIMENTAL, command may change in future releases

	USAGE
	  ipfs name inspect <record> - Inspects an IPNS Record

	  ipfs name inspect [--verify=<verify>] [--] <record>

	  Prints values inside of IPNS Record protobuf and its DAG-CBOR Data field.
	  Passing --verify will verify signature against provided public key.

	  For more information about each command, use:
	  'ipfs name inspect <subcmd> --help'


	appserver Sun 07 May 2023  1:20AM> ipfs                                                                                       /home/sarnobat/trash
	USAGE
	  ipfs  - Global p2p merkle-dag filesystem.

	  ipfs [--config=<config> | -c] [--debug | -D] [--help] [-h] [--api=<api>] [--offline] [--cid-base=<base>] [--upgrade-cidv0-in-output] [--encoding=<encoding> | --enc] [--timeout=<timeout>] <command> ...

	SUBCOMMANDS
	  BASIC COMMANDS
		init          Initialize local IPFS configuration
		add <path>    Add a file to IPFS
		cat <ref>     Show IPFS object data
		get <ref>     Download IPFS objects
		ls <ref>      List links from an object
		refs <ref>    List hashes of links from an object

	  DATA STRUCTURE COMMANDS
		dag           Interact with IPLD DAG nodes
		files         Interact with files as if they were a unix filesystem
		block         Interact with raw blocks in the datastore

	  TEXT ENCODING COMMANDS
		cid           Convert and discover properties of CIDs
		multibase     Encode and decode data with Multibase format

	  ADVANCED COMMANDS
		daemon        Start a long-running daemon process
		shutdown      Shut down the daemon process
		resolve       Resolve any type of content path
		name          Publish and resolve IPNS names
		key           Create and list IPNS name keypairs
		pin           Pin objects to local storage
		repo          Manipulate the IPFS repository
		stats         Various operational stats
		p2p           Libp2p stream mounting (experimental)
		filestore     Manage the filestore (experimental)
		mount         Mount an IPFS read-only mount point (experimental)

	  NETWORK COMMANDS
		id            Show info about IPFS peers
		bootstrap     Add or remove bootstrap peers
		swarm         Manage connections to the p2p network
		dht           Query the DHT for values or peers
		routing       Issue routing commands
		ping          Measure the latency of a connection
		bitswap       Inspect bitswap state
		pubsub        Send and receive messages via pubsub

	  TOOL COMMANDS
		config        Manage configuration
		version       Show IPFS version information
		diag          Generate diagnostic reports
		update        Download and apply go-ipfs updates
		commands      List all available commands
		log           Manage and show logs of running daemon

	  Use 'ipfs <command> --help' to learn more about each command.

	  ipfs uses a repository in the local file system. By default, the repo is
	  located at ~/.ipfs. To change the repo location, set the $IPFS_PATH
	  environment variable:

		export IPFS_PATH=/path/to/ipfsrepo

	  EXIT STATUS

	  The CLI will exit with one of the following values:

	  0     Successful execution.
	  1     Failed executions.

	appserver Sun 07 May 2023  1:21AM> ls ~/.ipfs.                                                                                /home/sarnobat/trash
	ls: cannot access '/home/sarnobat/.ipfs.': No such file or directory
	appserver Sun 07 May 2023  1:21AM> ls ~/.ipfs                                                                                 /home/sarnobat/trash
	api  blocks  config  datastore	datastore_spec	gateway  keystore  repo.lock  version
	appserver Sun 07 May 2023  1:21AM> find ~/.ipfs                                                                               /home/sarnobat/trash
	/home/sarnobat/.ipfs
	/home/sarnobat/.ipfs/repo.lock
	/home/sarnobat/.ipfs/gateway
	/home/sarnobat/.ipfs/keystore
	/home/sarnobat/.ipfs/version
	/home/sarnobat/.ipfs/blocks
	/home/sarnobat/.ipfs/blocks/UC
	/home/sarnobat/.ipfs/blocks/UC/CIQFKVEG2CPWTPRG5KNRUAWMOABRSTYUFHFK3QF6KN3M67G5E3ILUCY.data
	/home/sarnobat/.ipfs/blocks/U2
	/home/sarnobat/.ipfs/blocks/U2/CIQHFTCY7XL57YWLVDQ6UAXUOND3ADYQYJKYXA6G7A5IMD7SMO22U2A.data
	/home/sarnobat/.ipfs/blocks/V3
	/home/sarnobat/.ipfs/blocks/V3/CIQAPZYJAKUKALYI4YTB5PUMEN5BZYZHUQZWGFL4Q3HZUV26SYX2V3Q.data
	/home/sarnobat/.ipfs/blocks/R3
	/home/sarnobat/.ipfs/blocks/R3/CIQBED3K6YA5I3QQWLJOCHWXDRK5EXZQILBCKAPEDUJENZ5B5HJ5R3A.data
	/home/sarnobat/.ipfs/blocks/BE
	/home/sarnobat/.ipfs/blocks/BE/CIQCXBHBZAHEHBHU6P7PEA72E7UZQRJALHH7OH2FCWSWMTU7DMWVBEA.data
	/home/sarnobat/.ipfs/blocks/JN
	/home/sarnobat/.ipfs/blocks/JN/CIQPHMHGQLLZXC32FQQW2YVM4KGFORVFJAQYY55VK3WJGLZ2MS4RJNQ.data
	/home/sarnobat/.ipfs/blocks/RO
	/home/sarnobat/.ipfs/blocks/RO/CIQDRD2UT66U4EATJW53PSVWMFFPGNAN42PVWMDLHJD6FA5EVNNZROI.data
	/home/sarnobat/.ipfs/blocks/QD
	/home/sarnobat/.ipfs/blocks/QD/CIQL4QZR6XGWMPEV5Q2FCTDFD7MF3G5OOC5CMEDUHNA5VXYZVDLFQDA.data
	/home/sarnobat/.ipfs/blocks/TP
	/home/sarnobat/.ipfs/blocks/TP/CIQCODPXR5G237BYM7E5JF4A624CLH2TQDLC4QI6HEZK7FUWZQESTPI.data
	/home/sarnobat/.ipfs/blocks/CP
	/home/sarnobat/.ipfs/blocks/CP/CIQFY33OB6EGMICDWK2FOPJL4YSG4CR56BVB5LHELSANW36BYAUSCPQ.data
	/home/sarnobat/.ipfs/blocks/N6
	/home/sarnobat/.ipfs/blocks/N6/CIQGFYPT5OBMRC7ZMUFC2R3ZQPKOGBMHJEDDFEVS5ALYBKIZCXPTN6Y.data
	/home/sarnobat/.ipfs/blocks/75
	/home/sarnobat/.ipfs/blocks/75/CIQBEM7N2AM5YRAMJY7WDI6TJ4MGYIWVBA7POWSBPYKENY5IKK2I75Y.data
	/home/sarnobat/.ipfs/blocks/IL
	/home/sarnobat/.ipfs/blocks/IL/CIQJFGRQHQ45VCQLM7AJNF2GF5UHUAGGHC6LLAH6VYDEKLQMD4QLILY.data
	/home/sarnobat/.ipfs/blocks/OO
	/home/sarnobat/.ipfs/blocks/OO/CIQBT4N7PS5IZ5IG2ZOUGKFK27IE33WKGJNDW2TY3LSBNQ34R6OVOOQ.data
	/home/sarnobat/.ipfs/blocks/X3
	/home/sarnobat/.ipfs/blocks/X3/CIQFTFEEHEDF6KLBT32BFAGLXEZL4UWFNWM4LFTLMXQBCERZ6CMLX3Y.data
	/home/sarnobat/.ipfs/blocks/HB
	/home/sarnobat/.ipfs/blocks/HB/CIQMDQRK7B5DSZKBYOX4353TGN5J3JXS5VS6YNSAEJBOXBG26R76HBY.data
	/home/sarnobat/.ipfs/blocks/KE
	/home/sarnobat/.ipfs/blocks/KE/CIQD44K6LTXM6PHWK2RHB3G2VCYFPMVBTALE572GSMETJGBJTELFKEI.data
	/home/sarnobat/.ipfs/blocks/IG
	/home/sarnobat/.ipfs/blocks/IG/CIQF37MJMQBSCNICEBHHBUMQEDRU6KD2HTZFNXV7EOXTUQABMDG3IGQ.data
	/home/sarnobat/.ipfs/blocks/RR
	/home/sarnobat/.ipfs/blocks/RR/CIQODOBNMTM4MFM7QMMC27SGXAOVQFBJRPAG3AAOV67F7ESYL7R7RRQ.data
	/home/sarnobat/.ipfs/blocks/I2
	/home/sarnobat/.ipfs/blocks/I2/CIQBZNLCBI3U2I5F7O636DRBO552SCMSK2X2WYVCQ6BMYJN4MJTRI2Q.data
	/home/sarnobat/.ipfs/blocks/LB
	/home/sarnobat/.ipfs/blocks/LB/CIQEBIDOXSDDZEMFXGLMAUK36MA3EVOWNOZPV26HCFE4SEONFXO6LBA.data
	/home/sarnobat/.ipfs/blocks/MJ
	/home/sarnobat/.ipfs/blocks/MJ/CIQHQFRJK4MU2CVNFR3QG6KZB3FZG6OG7EBI4SUNB5K4S4T5UVECMJA.data
	/home/sarnobat/.ipfs/blocks/6Y
	/home/sarnobat/.ipfs/blocks/6Y/CIQA4T3TD3BP3C2M3GXCGRCRTCCHV7XSGAZPZJOAOHLPOI6IQR3H6YQ.data
	/home/sarnobat/.ipfs/blocks/ZR
	/home/sarnobat/.ipfs/blocks/ZR/CIQGQKGRCCLACORJVCOCZ54B6WYNYEF56DWYOR2YHFQYH4YHL3RYZRY.data
	/home/sarnobat/.ipfs/blocks/SHARDING
	/home/sarnobat/.ipfs/blocks/XV
	/home/sarnobat/.ipfs/blocks/XV/CIQGAS6MQJCEC37C2IIH5ZFYJCSTT7TCKJP3F7SLGNVSDVZSMACCXVA.data
	/home/sarnobat/.ipfs/blocks/VN
	/home/sarnobat/.ipfs/blocks/VN/CIQPEOA2TS3RMLOBOF55ZOEZE3TNBQG3HCNFOYC3BATAIJBOIE5FVNY.data
	/home/sarnobat/.ipfs/blocks/.temp
	/home/sarnobat/.ipfs/blocks/IY
	/home/sarnobat/.ipfs/blocks/IY/CIQB4655YD5GLBB7WWEUAHCO6QONU5ICBONAA5JEPBIOEIVZ5RXTIYY.data
	/home/sarnobat/.ipfs/blocks/diskUsage.cache
	/home/sarnobat/.ipfs/blocks/_README
	/home/sarnobat/.ipfs/api
	/home/sarnobat/.ipfs/datastore_spec
	/home/sarnobat/.ipfs/config
	/home/sarnobat/.ipfs/datastore
	/home/sarnobat/.ipfs/datastore/MANIFEST-000011
	/home/sarnobat/.ipfs/datastore/000028.ldb
	/home/sarnobat/.ipfs/datastore/CURRENT
	/home/sarnobat/.ipfs/datastore/000033.ldb
	/home/sarnobat/.ipfs/datastore/LOCK
	/home/sarnobat/.ipfs/datastore/000032.ldb
	/home/sarnobat/.ipfs/datastore/000035.ldb
	/home/sarnobat/.ipfs/datastore/000034.ldb
	/home/sarnobat/.ipfs/datastore/000031.ldb
	/home/sarnobat/.ipfs/datastore/000038.ldb
	/home/sarnobat/.ipfs/datastore/000043.log
	/home/sarnobat/.ipfs/datastore/000029.ldb
	/home/sarnobat/.ipfs/datastore/000042.ldb
	/home/sarnobat/.ipfs/datastore/000037.ldb
	/home/sarnobat/.ipfs/datastore/000044.ldb
	/home/sarnobat/.ipfs/datastore/CURRENT.bak
	/home/sarnobat/.ipfs/datastore/LOG
	/home/sarnobat/.ipfs/datastore/000036.ldb
	/home/sarnobat/.ipfs/datastore/000030.ldb
	/home/sarnobat/.ipfs/datastore/000040.ldb
	appserver Sun 07 May 2023  1:21AM> sudo reboot now                                                                            /home/sarnobat/trash
	Connection to netgear.rohidekar.com closed by remote host.
	Connection to netgear.rohidekar.com closed.
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	ssh: connect to host netgear.rohidekar.com port 22222: Connection refused
	Warning: No xauth data; using fake authentication data for X11 forwarding.
	cd /net/antec/media/sarnobat
	Do not cd to an nfs mount, you'll get locked out of ssh
	autofs_restart
	umount /media/sarnobat/ubuntu_valuables1; sudo mount /dev/disk/by-label/ubuntu_valuables /media/sarnobat/ubuntu_valuables
	appserver Sun 07 May 2023  6:18PM> umount /media/sarnobat/ubuntu_valuables1; sudo mount /dev/disk/by-label/ubuntu_valuables /media/sarnobat/ubuntu_valuables
	appserver Sun 07 May 2023  6:21PM> ~/bin/df.sh                                                                               /home/sarnobat/db.git
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/sda10                         83G   36G   43G  46% /home
	antec:/media/sarnobat/sandy       293G  235G   43G  85% /media/sarnobat/sandy
	/dev/sdb1                          60G   27G   34G  44% /media/sarnobat/ubuntu_valuables
	antec:/media/sarnobat/unmirrored  2.7T  2.6T   25G 100% /media/sarnobat/unmirrored
	antec:/media/sarnobat/e           292G  255G   22G  93% /media/sarnobat/e
	/dev/sda9                          19G   78M   18G   1% /usr/local
	/dev/sda7                          19G   24K   18G   1% /srv
	/dev/sda8                          19G  265M   17G   2% /opt
	/dev/sda4                          19G  897M   17G   6% /
	/dev/sda5                          19G  6.5G   11G  38% /usr
	/dev/sda6                          19G  7.7G  9.7G  45% /var
	antec:/media/sarnobat/3TB         2.7T  2.6T     0 100% /media/sarnobat/3TB
	appserver Sun 07 May 2023  6:21PM> cd ..                                                                                     /home/sarnobat/db.git
	appserver Sun 07 May 2023  6:24PM> ls --color -lS --reverse --human-readable --almost-all                                           /home/sarnobat

		total 2.3G
		-rw-r--r--  1 sarnobat sarnobat    0 Apr  9  2022  .sudo_as_admin_successful
		-rw-rw-r--  1 sarnobat sarnobat    0 May  2 08:00 'Sofia Vergara'\''s Son Knows׃ “Mom Knows Best“ ¦ Head & Shoulders Shampoo [AxjKUb6zfko].f137.mp4.part'
		-rw-rw-r--  1 sarnobat sarnobat    0 Dec 30 04:30  Sofia_Vergara_s_Son_Knows_Mom_Knows_Best_Head_and_Shoulders_Shampoo-AxjKUb6zfko-.f137.mp4.part
		-rw-rw-r--  1 sarnobat sarnobat    0 May  7 18:19  .hushlogin
		lrwxrwxrwx  1 sarnobat sarnobat   24 Mar 19  2021  .zshrc -> computers.git/nuc/.zshrc
		lrwxrwxrwx  1 sarnobat sarnobat   24 Apr 26  2022  3TB -> ../../media/sarnobat/3TB
		lrwxrwxrwx  1 sarnobat sarnobat   25 Jan  8 19:46  .vim -> computers.git/common/.vim
		lrwxrwxrwx  1 sarnobat sarnobat   25 Mar 19  2021  bin -> computers.git/ubuntu/bin/
		lrwxrwxrwx  1 sarnobat sarnobat   30 Apr 28  2022  www -> computers.git/nuc/var/www/html
		lrwxrwxrwx  1 sarnobat sarnobat   31 Apr 26  2022  .groovy -> github/groovy_libraries/.groovy
		lrwxrwxrwx  1 sarnobat sarnobat   32 Jan  7 15:11  .zshrc-misc -> computers.git/common/.zshrc-misc
		lrwxrwxrwx  1 sarnobat sarnobat   32 Aug 27  2022  ubuntu_valuables -> /media/sarnobat/ubuntu_valuables
		lrwxrwxrwx  1 sarnobat sarnobat   32 Jan  8 19:58  .dir_colors -> computers.git/common/.dir_colors
		lrwxrwxrwx  1 sarnobat sarnobat   37 Jan  7 15:11  .zshrc-git -> computers.git/2022/mac/zsh/.zshrc-git
		lrwxrwxrwx  1 sarnobat sarnobat   37 Dec 25 02:15  favorites -> ../../media/sarnobat/e/2021/favorites
		lrwxrwxrwx  1 sarnobat sarnobat   40 Jan  7 15:11  .zshrc-key-bindings -> computers.git/common/.zshrc-key-bindings
		lrwxrwxrwx  1 sarnobat sarnobat   43 Jan  7 15:11  .zshrc-tab-completions -> computers.git/common/.zshrc-tab-completions
		lrwxrwxrwx  1 sarnobat sarnobat   44 Jan  7 15:11  .zshrc.mac.common -> computers.git/2022/mac/zsh/.zshrc.mac.common
		lrwxrwxrwx  1 sarnobat sarnobat   45 Apr 27  2022  videos -> /media/sarnobat/3TB/2021/disks/thistle/videos
		lrwxrwxrwx  1 sarnobat sarnobat   48 Jan  9 17:10  .aliases -> computers.git/2022/dotfiles_and_scripts/.aliases
		lrwxrwxrwx  1 sarnobat sarnobat   49 Feb 13 10:07  sarnobat.git -> ../../media/sarnobat/unmirrored/2021/sarnobat.git
		lrwxrwxrwx  1 sarnobat sarnobat   49 Jan  7 14:41  conf.tail -> computers.git/2022/dotfiles_and_scripts/conf.tail
		-rw-rw-r--  1 sarnobat sarnobat   50 May  2 08:00 'Sofia Vergara'\''s Son Knows׃ “Mom Knows Best“ ¦ Head & Shoulders Shampoo [AxjKUb6zfko].f137.mp4.ytdl'
		-rw-rw-r--  1 sarnobat sarnobat   50 Dec 30 04:30  Sofia_Vergara_s_Son_Knows_Mom_Knows_Best_Head_and_Shoulders_Shampoo-AxjKUb6zfko-.f137.mp4.ytdl
		lrwxrwxrwx  1 sarnobat sarnobat   55 Jan  8 19:43  .vimrc -> computers.git/2022/dotfiles_and_scripts/2021/vim/.vimrc
		lrwxrwxrwx  1 sarnobat sarnobat   61 Jan 17 00:11  youtube_download_no_record -> computers.git/2022/ubuntu/bin/2021/youtube_download_no_record
		-rw-------  1 sarnobat sarnobat  100 Apr 20 21:20  .lesshst
		-rw-rw-r--  1 sarnobat sarnobat  171 May  7 18:19  .gitconfig
		-rw-r--r--  1 sarnobat sarnobat  220 Apr  9  2022  .bash_logout
		-rw-r--r--  1 sarnobat sarnobat  807 Apr  9  2022  .profile
		-rw-------  1 sarnobat sarnobat 1.1K Aug 20  2022  .bash_history
		-rw-------  1 sarnobat sarnobat 1.2K May  7 18:18  .Xauthority
		-rw-rw-r--  1 sarnobat sarnobat 3.0K May  6 23:17  .wget-hsts
		-rw-r--r--  1 sarnobat sarnobat 3.7K Apr  9  2022  .bashrc
		drwxr-xr-x 10 sarnobat sarnobat 4.0K May  6 23:51  trash
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Apr 22  2022  .tldr
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Templates
		drwx------  2 sarnobat sarnobat 4.0K Mar 28 20:40  .ssh
		drwxrwxr-x 31 sarnobat sarnobat 4.0K May  1  2022  src.git
		drwx------  3 sarnobat sarnobat 4.0K May  6 23:10  snap
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Apr 28  2022  sarnobat.git.old
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Public
		drwx------  3 sarnobat sarnobat 4.0K Apr 26  2022  .pki
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Pictures
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Oct 10  2022  News
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Music
		drwx------  4 sarnobat sarnobat 4.0K Jan 18 20:23  .mozilla
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Sep  9  2022  .m2
		drwx------  3 sarnobat sarnobat 4.0K Apr  9  2022  .local
		drwxrwxr-x  5 sarnobat sarnobat 4.0K May  6 23:49  .ipfs
		drwxrwxr-x  7 sarnobat sarnobat 4.0K Dec 26 00:14  .gradle
		drwx------  3 sarnobat sarnobat 4.0K Jan 26 18:52  .gnupg
		drwxrwxr-x 22 sarnobat sarnobat 4.0K Mar 28 20:40  github
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Downloads
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Documents
		drwxr-xr-x  2 sarnobat sarnobat 4.0K May  6 23:50  Desktop
		drwxrwxr-x  6 sarnobat sarnobat 4.0K Feb 13 10:08  db.git.small_alternative
		drwxrwxr-x  6 sarnobat sarnobat 4.0K May  6 23:51  db.git
		drwx------ 15 sarnobat sarnobat 4.0K Feb 18 00:13  .config
		drwxrwxr-x  9 sarnobat sarnobat 4.0K Feb 18 17:40  computers.git
		drwx------ 18 sarnobat sarnobat 4.0K May  7 18:00  .cache
		drwxrwxr-x  3 sarnobat sarnobat  12K May  2 17:56  new
		-rw-------  1 sarnobat sarnobat  30K May  6 23:53  .viminfo
		-rw-rw-r--  1 sarnobat sarnobat  49K May  7 18:18  .zcompdump
		-rw-rw-r--  1 sarnobat sarnobat  78K May  3  2020 'Teen Pregnancy Rates by State [hjmbpqpbnkw41].png'
		-rw-------  1 sarnobat sarnobat 244K Jan  8 19:35  mbox
		-rw-------  1 sarnobat sarnobat 640K May  7 18:24  .zsh-history
		-rw-rw-r--  1 sarnobat sarnobat  55M May  7 18:00 'Hot Flexing Boobs Biceps and Back [ph5f8357baac2d9].mp4'
		-rw-rw-r--  1 sarnobat sarnobat 211M Apr 13  2019 'How the Internet Crossed the Sea ｜ Nostalgia Nerd [A8q7Ayvw5kA].webm'
		-rw-rw-r--  1 sarnobat sarnobat 219M Apr 16 02:37  ccia0aUKXIk-.f313.webm.part
		-rw-rw-r--  1 sarnobat sarnobat 604M Jan 10  2021 'The Origins Of Mussolini'\''s Italy ｜ Secrets Of War ｜ Timeline [tAqkl-hCW0U].webm'
		-rw-rw-r--  1 sarnobat sarnobat 1.2G Apr 16 23:58  Why_Chicagoland_s_Oasis_is_Disappearing_The_Rise_and_Fall_of_The_Illinois_Tollway_Oasis-gWH0BBuSpaU.f313.webm.part

	appserver Sun 07 May 2023  6:24PM> ls --color -lS --reverse --human-readable --almost-all                                           /home/sarnobat

		total 3.9G
		-rw-r--r--  1 sarnobat sarnobat    0 Apr  9  2022  .sudo_as_admin_successful
		-rw-rw-r--  1 sarnobat sarnobat    0 May  2 08:00 'Sofia Vergara'\''s Son Knows׃ “Mom Knows Best“ ¦ Head & Shoulders Shampoo [AxjKUb6zfko].f137.mp4.part'
		-rw-rw-r--  1 sarnobat sarnobat    0 Dec 30 04:30  Sofia_Vergara_s_Son_Knows_Mom_Knows_Best_Head_and_Shoulders_Shampoo-AxjKUb6zfko-.f137.mp4.part
		-rw-rw-r--  1 sarnobat sarnobat    0 May  7 18:19  .hushlogin
		lrwxrwxrwx  1 sarnobat sarnobat   24 Mar 19  2021  .zshrc -> computers.git/nuc/.zshrc
		lrwxrwxrwx  1 sarnobat sarnobat   24 Apr 26  2022  3TB -> ../../media/sarnobat/3TB
		lrwxrwxrwx  1 sarnobat sarnobat   25 Jan  8 19:46  .vim -> computers.git/common/.vim
		lrwxrwxrwx  1 sarnobat sarnobat   25 Mar 19  2021  bin -> computers.git/ubuntu/bin/
		lrwxrwxrwx  1 sarnobat sarnobat   30 Apr 28  2022  www -> computers.git/nuc/var/www/html
		lrwxrwxrwx  1 sarnobat sarnobat   31 Apr 26  2022  .groovy -> github/groovy_libraries/.groovy
		lrwxrwxrwx  1 sarnobat sarnobat   32 Jan  7 15:11  .zshrc-misc -> computers.git/common/.zshrc-misc
		lrwxrwxrwx  1 sarnobat sarnobat   32 Aug 27  2022  ubuntu_valuables -> /media/sarnobat/ubuntu_valuables
		lrwxrwxrwx  1 sarnobat sarnobat   32 Jan  8 19:58  .dir_colors -> computers.git/common/.dir_colors
		lrwxrwxrwx  1 sarnobat sarnobat   37 Jan  7 15:11  .zshrc-git -> computers.git/2022/mac/zsh/.zshrc-git
		lrwxrwxrwx  1 sarnobat sarnobat   37 Dec 25 02:15  favorites -> ../../media/sarnobat/e/2021/favorites
		lrwxrwxrwx  1 sarnobat sarnobat   40 Jan  7 15:11  .zshrc-key-bindings -> computers.git/common/.zshrc-key-bindings
		lrwxrwxrwx  1 sarnobat sarnobat   43 Jan  7 15:11  .zshrc-tab-completions -> computers.git/common/.zshrc-tab-completions
		lrwxrwxrwx  1 sarnobat sarnobat   44 Jan  7 15:11  .zshrc.mac.common -> computers.git/2022/mac/zsh/.zshrc.mac.common
		lrwxrwxrwx  1 sarnobat sarnobat   45 Apr 27  2022  videos -> /media/sarnobat/3TB/2021/disks/thistle/videos
		lrwxrwxrwx  1 sarnobat sarnobat   48 Jan  9 17:10  .aliases -> computers.git/2022/dotfiles_and_scripts/.aliases
		lrwxrwxrwx  1 sarnobat sarnobat   49 Feb 13 10:07  sarnobat.git -> ../../media/sarnobat/unmirrored/2021/sarnobat.git
		lrwxrwxrwx  1 sarnobat sarnobat   49 Jan  7 14:41  conf.tail -> computers.git/2022/dotfiles_and_scripts/conf.tail
		-rw-rw-r--  1 sarnobat sarnobat   50 May  2 08:00 'Sofia Vergara'\''s Son Knows׃ “Mom Knows Best“ ¦ Head & Shoulders Shampoo [AxjKUb6zfko].f137.mp4.ytdl'
		-rw-rw-r--  1 sarnobat sarnobat   50 Dec 30 04:30  Sofia_Vergara_s_Son_Knows_Mom_Knows_Best_Head_and_Shoulders_Shampoo-AxjKUb6zfko-.f137.mp4.ytdl
		lrwxrwxrwx  1 sarnobat sarnobat   55 Jan  8 19:43  .vimrc -> computers.git/2022/dotfiles_and_scripts/2021/vim/.vimrc
		lrwxrwxrwx  1 sarnobat sarnobat   61 Jan 17 00:11  youtube_download_no_record -> computers.git/2022/ubuntu/bin/2021/youtube_download_no_record
		-rw-------  1 sarnobat sarnobat  100 Apr 20 21:20  .lesshst
		-rw-rw-r--  1 sarnobat sarnobat  171 May  7 18:19  .gitconfig
		-rw-r--r--  1 sarnobat sarnobat  220 Apr  9  2022  .bash_logout
		-rw-r--r--  1 sarnobat sarnobat  807 Apr  9  2022  .profile
		-rw-------  1 sarnobat sarnobat 1.1K Aug 20  2022  .bash_history
		-rw-------  1 sarnobat sarnobat 1.2K May  7 18:18  .Xauthority
		-rw-rw-r--  1 sarnobat sarnobat 3.0K May  6 23:17  .wget-hsts
		-rw-r--r--  1 sarnobat sarnobat 3.7K Apr  9  2022  .bashrc
		drwxr-xr-x 10 sarnobat sarnobat 4.0K May  6 23:51  trash
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Apr 22  2022  .tldr
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Templates
		drwx------  2 sarnobat sarnobat 4.0K Mar 28 20:40  .ssh
		drwxrwxr-x 31 sarnobat sarnobat 4.0K May  1  2022  src.git
		drwx------  3 sarnobat sarnobat 4.0K May  6 23:10  snap
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Apr 28  2022  sarnobat.git.old
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Public
		drwx------  3 sarnobat sarnobat 4.0K Apr 26  2022  .pki
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Pictures
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Oct 10  2022  News
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Music
		drwx------  4 sarnobat sarnobat 4.0K Jan 18 20:23  .mozilla
		drwxrwxr-x  3 sarnobat sarnobat 4.0K Sep  9  2022  .m2
		drwx------  3 sarnobat sarnobat 4.0K Apr  9  2022  .local
		drwxrwxr-x  5 sarnobat sarnobat 4.0K May  6 23:49  .ipfs
		drwxrwxr-x  7 sarnobat sarnobat 4.0K Dec 26 00:14  .gradle
		drwx------  3 sarnobat sarnobat 4.0K Jan 26 18:52  .gnupg
		drwxrwxr-x 22 sarnobat sarnobat 4.0K Mar 28 20:40  github
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Downloads
		drwxr-xr-x  2 sarnobat sarnobat 4.0K Apr  9  2022  Documents
		drwxr-xr-x  2 sarnobat sarnobat 4.0K May  6 23:50  Desktop
		drwxrwxr-x  6 sarnobat sarnobat 4.0K Feb 13 10:08  db.git.small_alternative
		drwxrwxr-x  6 sarnobat sarnobat 4.0K May  6 23:51  db.git
		drwx------ 15 sarnobat sarnobat 4.0K Feb 18 00:13  .config
		drwxrwxr-x  9 sarnobat sarnobat 4.0K Feb 18 17:40  computers.git
		drwx------ 18 sarnobat sarnobat 4.0K May  7 18:00  .cache
		drwxrwxr-x  3 sarnobat sarnobat  12K May  2 17:56  new
		-rw-------  1 sarnobat sarnobat  30K May  6 23:53  .viminfo
		-rw-rw-r--  1 sarnobat sarnobat  49K May  7 18:18  .zcompdump
		-rw-rw-r--  1 sarnobat sarnobat  78K May  3  2020 'Teen Pregnancy Rates by State [hjmbpqpbnkw41].png'
		-rw-------  1 sarnobat sarnobat 244K Jan  8 19:35  mbox
		-rw-------  1 sarnobat sarnobat 640K May  7 18:52  .zsh-history
		-rw-rw-r--  1 sarnobat sarnobat 1.2M Oct  4  2019 'Serie A 1999-2000, day 05 Fiorentina - Roma 1-3 (2 Cafu, Tommasi, Batistuta) [mtBX6Z8ae0M].webm'
		-rw-rw-r--  1 sarnobat sarnobat 8.4M Mar 19  2018 'Hardcore Pawn - Lex Luger Visits the Store [woui4O25Z18].webm'
		-rw-rw-r--  1 sarnobat sarnobat  16M Apr 20 14:11 'The Mountie on REAL REASON Why He Beat Bret Hart for WWE Intercontinetal Title [XwVy9ljEmTw].webm'
		-rw-rw-r--  1 sarnobat sarnobat  30M May  7 18:52 'Goals Galore VHS ｜ 1988-89 ｜ 110 Great Goals from the Season [VgeqR5O2zYw].f251.webm.part'
		-rw-rw-r--  1 sarnobat sarnobat  43M Apr 18 12:45 'Sycho Sid REVEALS Who Was Supposed To Win At Survivor Series 1996 [cRmYs4goLss].webm'
		-rw-rw-r--  1 sarnobat sarnobat  55M May  7 18:00 'Hot Flexing Boobs Biceps and Back [ph5f8357baac2d9].mp4'
		-rw-rw-r--  1 sarnobat sarnobat  97M Apr 12  2016 'Ariana Grande： My Way (Trailer) [KcoIDeULdbA].mp4'
		-rw-rw-r--  1 sarnobat sarnobat 150M Nov 10  2021 'The Reign of Terror ｜ Wrestling With Wregret [bao2JhbHPQw].webm'
		-rw-rw-r--  1 sarnobat sarnobat 211M Apr 13  2019 'How the Internet Crossed the Sea ｜ Nostalgia Nerd [A8q7Ayvw5kA].webm'
		-rw-rw-r--  1 sarnobat sarnobat 219M Apr 16 02:37  ccia0aUKXIk-.f313.webm.part
		-rw-rw-r--  1 sarnobat sarnobat 449M May  7 18:52 'Goals Galore VHS ｜ 1988-89 ｜ 110 Great Goals from the Season [VgeqR5O2zYw].f247.webm'
		-rw-rw-r--  1 sarnobat sarnobat 604M Jan 10  2021 'The Origins Of Mussolini'\''s Italy ｜ Secrets Of War ｜ Timeline [tAqkl-hCW0U].webm'
		-rw-rw-r--  1 sarnobat sarnobat 856M Jan  7  2022 'Goals Galore VHS ｜ 1989-90 ｜ 110 Great Goals from the Season [ArvyL0uze00].mkv'
		-rw-rw-r--  1 sarnobat sarnobat 1.2G Apr 16 23:58  Why_Chicagoland_s_Oasis_is_Disappearing_The_Rise_and_Fall_of_The_Illinois_Tollway_Oasis-gWH0BBuSpaU.f313.webm.part

	appserver Sun 07 May 2023  6:52PM> yt-dlp --trim-filenames 100 --restrict-filename "https://twitter.com/WWFOldSchoolcom/status/1094945886529089538?lang=en"

		[twitter] Extracting URL: https://twitter.com/WWFOldSchoolcom/status/1094945886529089538?lang=en
		[twitter] 1094945886529089538: Downloading guest token
		[twitter] 1094945886529089538: Downloading JSON metadata
		[twitter] 1094945886529089538: Downloading m3u8 information
		[info] 1094945420609024002: Downloading 1 format(s): http-832
		[download] Destination: WWFOldSchool.com_-_Rare_footage_of_@HulkHogan_celebrating_with_Vince_McMahon_Randy_Savage_@brutusbeefcake__and_@RealJimmyHart_after_WrestleMania_IX_went_off_the_air_HulkHogan_WrestleMania-[1094945420609024002].mp4
		[download] 100% of    4.04MiB in 00:00:02 at 1.82MiB/s

	appserver Sun 07 May 2023  7:25PM> kubectl get all --all-namespaces                                                                 /home/sarnobat
	zsh: command not found: kubectl
	appserver Sun 07 May 2023  7:58PM> kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr        --discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78
	zsh: command not found: kubeadm
	appserver Sun 07 May 2023  8:11PM> sudo apt install -y kubeadm                                                                      /home/sarnobat

		Reading package lists... Done
		Building dependency tree
		Reading state information... Done

		No apt package "kubeadm", but there is a snap with that name.
		Try "snap install kubeadm"

		E: Unable to locate package kubeadm

	appserver Sun 07 May 2023  8:11PM> snap install kubeadm                                                                             /home/sarnobat
	error: access denied (try with sudo)

	appserver Sun 07 May 2023  8:11PM> sudo snap install kubeadm                                                                        /home/sarnobat
	error: This revision of snap "kubeadm" was published using classic confinement and thus may perform
		   arbitrary system changes outside of the security sandbox that snaps are usually confined to,
		   which may put your system at risk.

		   If you understand and want to proceed repeat the command including --classic.

	appserver Sun 07 May 2023  8:11PM> sudo apt install kubeadm                                                                         /home/sarnobat
	Reading package lists... Done
	Building dependency tree
	Reading state information... Done

	No apt package "kubeadm", but there is a snap with that name.
	Try "snap install kubeadm"

	E: Unable to locate package kubeadm

	appserver Sun 07 May 2023  8:12PM> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add                 /home/sarnobat

	OK

	appserver Sun 07 May 2023  8:14PM> echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list                /home/sarnobat
	sudo mv ~/kubernetes.list /etc/apt/sources.list.d

	appserver Sun 07 May 2023  8:14PM> sudo apt update && sudo apt install kubeadm                                                      /home/sarnobat
	Hit:1 https://dl.google.com/linux/chrome/deb stable InRelease
	Hit:2 http://us.archive.ubuntu.com/ubuntu focal InRelease
	Get:4 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
	Get:5 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
	Ign:6 http://ppa.launchpad.net/b-stolk/ppa/ubuntu focal InRelease
	Get:7 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
	Hit:8 http://ppa.launchpad.net/videolan/master-daily/ubuntu focal InRelease
	Get:3 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [8,993 B]
	Get:9 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 DEP-11 Metadata [275 kB]
	Err:10 http://ppa.launchpad.net/b-stolk/ppa/ubuntu focal Release
	  404  Not Found [IP: 185.125.190.52 80]
	Get:11 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 DEP-11 Metadata [410 kB]
	Get:12 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [65.7 kB]
	Get:13 http://us.archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 DEP-11 Metadata [944 B]
	Get:14 http://us.archive.ubuntu.com/ubuntu focal-backports/main amd64 DEP-11 Metadata [7,968 B]
	Get:15 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 DEP-11 Metadata [30.5 kB]
	Get:16 http://security.ubuntu.com/ubuntu focal-security/main amd64 DEP-11 Metadata [59.9 kB]
	Get:17 http://security.ubuntu.com/ubuntu focal-security/universe amd64 DEP-11 Metadata [95.6 kB]
	Get:18 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 DEP-11 Metadata [940 B]
	Reading package lists... Done
	E: The repository 'http://ppa.launchpad.net/b-stolk/ppa/ubuntu focal Release' does not have a Release file.
	N: Updating from such a repository can't be done securely, and is therefore disabled by default.
	N: See apt-secure(8) manpage for repository creation and user configuration details.

	appserver Sun 07 May 2023  8:14PM> sudo apt install kubeadm                                                                         /home/sarnobat

	Reading package lists... Done
	Building dependency tree
	Reading state information... Done
	The following packages were automatically installed and are no longer required:
	Unpacking socat (1.7.3.3-2) ...
	Selecting previously unselected package kubelet.
	Preparing to unpack .../6-kubelet_1.27.1-00_amd64.deb ...
	Unpacking kubelet (1.27.1-00) ...
	Selecting previously unselected package kubectl.
	Preparing to unpack .../7-kubectl_1.27.1-00_amd64.deb ...
	Unpacking kubectl (1.27.1-00) ...
	Selecting previously unselected package kubeadm.
	Preparing to unpack .../8-kubeadm_1.27.1-00_amd64.deb ...
	Unpacking kubeadm (1.27.1-00) ...
	Setting up conntrack (1:1.4.5-2) ...
	Setting up kubectl (1.27.1-00) ...
	Setting up ebtables (2.0.11-3build1) ...
	Setting up socat (1.7.3.3-2) ...
	Setting up cri-tools (1.26.0-00) ...
	Setting up kubernetes-cni (1.2.0-00) ...
	Setting up ethtool (1:5.4-1) ...
	Setting up kubelet (1.27.1-00) ...
	Created symlink /etc/systemd/system/multi-user.target.wants/kubelet.service → /lib/systemd/system/kubelet.service.
	Setting up kubeadm (1.27.1-00) ...
	Processing triggers for man-db (2.9.1-1) ...

	appserver Sun 07 May 2023  8:15PM> kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr        --discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78

	[preflight] Running pre-flight checks
	error execution phase preflight: [preflight] Some fatal errors occurred:
		[ERROR IsPrivilegedUser]: user is not running as root
	[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
	To see the stack trace of this error execute with --v=5 or higher

	appserver Sun 07 May 2023  8:15PM> sudo kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr   --discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78

	[preflight] Running pre-flight checks
		[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
	[preflight] Reading configuration from the cluster...
	[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
	[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
	[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
	[kubelet-start] Starting the kubelet
	[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
	[kubelet-check] Initial timeout of 40s passed.
	[kubelet-check] It seems like the kubelet isn't running or healthy.
	[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
	[kubelet-check] It seems like the kubelet isn't running or healthy.
	[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
	[kubelet-check] It seems like the kubelet isn't running or healthy.
	[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
	[kubelet-check] It seems like the kubelet isn't running or healthy.
	[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
	[kubelet-check] It seems like the kubelet isn't running or healthy.
	[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.

	Unfortunately, an error has occurred:
		timed out waiting for the condition

	This error is likely caused by:
		- The kubelet is not running
		- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

	If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
		- 'systemctl status kubelet'
		- 'journalctl -xeu kubelet'
	error execution phase kubelet-start: timed out waiting for the condition
	To see the stack trace of this error execute with --v=5 or higher

	appserver Sun 07 May 2023  8:17PM> journalctl -u kubelet -f                                                                         /home/sarnobat

		-- Logs begin at Wed 2023-05-03 13:00:27 PDT. --

		May 07 20:17:45 nuc2020 kubelet[30340]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
		May 07 20:17:45 nuc2020 kubelet[30340]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
		May 07 20:17:45 nuc2020 kubelet[30340]: I0507 20:17:45.599893   30340 server.go:199] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
		May 07 20:17:45 nuc2020 kubelet[30340]: I0507 20:17:45.602446   30340 server.go:415] "Kubelet version" kubeletVersion="v1.27.1"
		May 07 20:17:45 nuc2020 kubelet[30340]: I0507 20:17:45.602465   30340 server.go:417] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
		May 07 20:17:45 nuc2020 kubelet[30340]: I0507 20:17:45.602664   30340 server.go:837] "Client rotation is on, will bootstrap in background"
		May 07 20:17:45 nuc2020 kubelet[30340]: E0507 20:17:45.603126   30340 bootstrap.go:241] unable to read existing bootstrap client config from /etc/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
		May 07 20:17:45 nuc2020 kubelet[30340]: E0507 20:17:45.603161   30340 run.go:74] "command failed" err="failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
		May 07 20:17:45 nuc2020 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
		May 07 20:17:45 nuc2020 systemd[1]: kubelet.service: Failed with result 'exit-code'.
		qMay 07 20:17:55 nuc2020 systemd[1]: kubelet.service: Scheduled restart job, restart counter is at 14.
		May 07 20:17:55 nuc2020 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
		May 07 20:17:55 nuc2020 systemd[1]: Started kubelet: The Kubernetes Node Agent.
		May 07 20:17:55 nuc2020 kubelet[30357]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
		May 07 20:17:55 nuc2020 kubelet[30357]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
		May 07 20:17:55 nuc2020 kubelet[30357]: I0507 20:17:55.848662   30357 server.go:199] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
		May 07 20:17:55 nuc2020 kubelet[30357]: I0507 20:17:55.851348   30357 server.go:415] "Kubelet version" kubeletVersion="v1.27.1"
		May 07 20:17:55 nuc2020 kubelet[30357]: I0507 20:17:55.851364   30357 server.go:417] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
		May 07 20:17:55 nuc2020 kubelet[30357]: I0507 20:17:55.851528   30357 server.go:837] "Client rotation is on, will bootstrap in background"
		May 07 20:17:55 nuc2020 kubelet[30357]: E0507 20:17:55.851939   30357 bootstrap.go:241] unable to read existing bootstrap client config from /etc/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
		May 07 20:17:55 nuc2020 kubelet[30357]: E0507 20:17:55.851964   30357 run.go:74] "command failed" err="failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
		May 07 20:17:55 nuc2020 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
		May 07 20:17:55 nuc2020 systemd[1]: kubelet.service: Failed with result 'exit-code'.
		^C

	appserver Sun 07 May 2023  8:17PM> sudo apt update && sudo apt install kubelet                                                      /home/sarnobat

		Hit:1 http://us.archive.ubuntu.com/ubuntu focal InRelease
		Hit:2 https://dl.google.com/linux/chrome/deb stable InRelease
		Hit:4 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease
		Ign:5 http://ppa.launchpad.net/b-stolk/ppa/ubuntu focal InRelease
		Hit:6 http://security.ubuntu.com/ubuntu focal-security InRelease
		Hit:7 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease
		Hit:3 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
		Hit:8 http://ppa.launchpad.net/videolan/master-daily/ubuntu focal InRelease
		Err:9 http://ppa.launchpad.net/b-stolk/ppa/ubuntu focal Release
		  404  Not Found [IP: 185.125.190.52 80]
		Sorting... Done
		Full Text Search... Done
		kubelet/kubernetes-xenial,now 1.27.1-00 amd64 [installed,automatic]
		  Kubernetes Node Agent

	appserver Sun 07 May 2023  8:23PM> sudo apt reinstall kubelet                                                                       /home/sarnobat

		Reading package lists... Done
		Building dependency tree
		Reading state information... Done
		The following packages were automatically installed and are no longer required:
		  libbasicusageenvironment1 libgroupsock8 liblivemedia77 libopenmpt-modplug1 libplacebo7 libsdl-image1.2 libsrt1 libusageenvironment3
		Use 'sudo apt autoremove' to remove them.
		0 upgraded, 0 newly installed, 1 reinstalled, 0 to remove and 310 not upgraded.
		Need to get 18.7 MB of archives.
		After this operation, 0 B of additional disk space will be used.
		Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.27.1-00 [18.7 MB]
		Fetched 18.7 MB in 4s (5,317 kB/s)
		(Reading database ... 214195 files and directories currently installed.)
		Preparing to unpack .../kubelet_1.27.1-00_amd64.deb ...
		Unpacking kubelet (1.27.1-00) over (1.27.1-00) ...
		Setting up kubelet (1.27.1-00) ...

	appserver Sun 07 May 2023  8:24PM> journalctl -u kubelet -f                                                                         /home/sarnobat

		-- Logs begin at Wed 2023-05-03 13:00:27 PDT. --
		May 07 20:24:07 nuc2020 kubelet[32998]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
		May 07 20:24:07 nuc2020 kubelet[32998]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
		May 07 20:24:07 nuc2020 kubelet[32998]: I0507 20:24:07.102878   32998 server.go:199] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
		May 07 20:24:07 nuc2020 kubelet[32998]: I0507 20:24:07.105167   32998 server.go:415] "Kubelet version" kubeletVersion="v1.27.1"
		May 07 20:24:07 nuc2020 kubelet[32998]: I0507 20:24:07.105181   32998 server.go:417] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
		May 07 20:24:07 nuc2020 kubelet[32998]: I0507 20:24:07.105346   32998 server.go:837] "Client rotation is on, will bootstrap in background"
		May 07 20:24:07 nuc2020 kubelet[32998]: E0507 20:24:07.105728   32998 bootstrap.go:241] unable to read existing bootstrap client config from /etc/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
		May 07 20:24:07 nuc2020 kubelet[32998]: E0507 20:24:07.105756   32998 run.go:74] "command failed" err="failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
		May 07 20:24:07 nuc2020 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
		May 07 20:24:07 nuc2020 systemd[1]: kubelet.service: Failed with result 'exit-code'.
		May 07 20:24:17 nuc2020 systemd[1]: kubelet.service: Scheduled restart job, restart counter is at 51.
		May 07 20:24:17 nuc2020 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
		May 07 20:24:17 nuc2020 systemd[1]: Started kubelet: The Kubernetes Node Agent.
		May 07 20:24:17 nuc2020 kubelet[33205]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
		May 07 20:24:17 nuc2020 kubelet[33205]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
		May 07 20:24:17 nuc2020 kubelet[33205]: I0507 20:24:17.350374   33205 server.go:199] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
		May 07 20:24:17 nuc2020 kubelet[33205]: I0507 20:24:17.352943   33205 server.go:415] "Kubelet version" kubeletVersion="v1.27.1"
		May 07 20:24:17 nuc2020 kubelet[33205]: I0507 20:24:17.352958   33205 server.go:417] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
		May 07 20:24:17 nuc2020 kubelet[33205]: I0507 20:24:17.353108   33205 server.go:837] "Client rotation is on, will bootstrap in background"
		May 07 20:24:17 nuc2020 kubelet[33205]: E0507 20:24:17.354147   33205 bootstrap.go:241] unable to read existing bootstrap client config from /etc/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
		May 07 20:24:17 nuc2020 kubelet[33205]: E0507 20:24:17.354186   33205 run.go:74] "command failed" err="failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
		May 07 20:24:17 nuc2020 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
		May 07 20:24:17 nuc2020 systemd[1]: kubelet.service: Failed with result 'exit-code'.
		qMay 07 20:24:27 nuc2020 systemd[1]: kubelet.service: Scheduled restart job, restart counter is at 52.
		May 07 20:24:27 nuc2020 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
		May 07 20:24:27 nuc2020 systemd[1]: Started kubelet: The Kubernetes Node Agent.
		May 07 20:24:27 nuc2020 kubelet[33246]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
		May 07 20:24:27 nuc2020 kubelet[33246]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
		May 07 20:24:27 nuc2020 kubelet[33246]: I0507 20:24:27.599843   33246 server.go:199] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
		May 07 20:24:27 nuc2020 kubelet[33246]: I0507 20:24:27.602471   33246 server.go:415] "Kubelet version" kubeletVersion="v1.27.1"
		May 07 20:24:27 nuc2020 kubelet[33246]: I0507 20:24:27.602486   33246 server.go:417] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
		May 07 20:24:27 nuc2020 kubelet[33246]: I0507 20:24:27.602638   33246 server.go:837] "Client rotation is on, will bootstrap in background"
		May 07 20:24:27 nuc2020 kubelet[33246]: E0507 20:24:27.603036   33246 bootstrap.go:241] unable to read existing bootstrap client config from /etc/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
		May 07 20:24:27 nuc2020 kubelet[33246]: E0507 20:24:27.603066   33246 run.go:74] "command failed" err="failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
		May 07 20:24:27 nuc2020 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
		May 07 20:24:27 nuc2020 systemd[1]: kubelet.service: Failed with result 'exit-code'.






		^C

	appserver Sun 07 May 2023  8:24PM> ls --color -lS --reverse --human-readable --almost-all                                           /home/sarnobat
	appserver Sun 07 May 2023  8:24PM> sudo service kubelet start                                                                       /home/sarnobat
	appserver Sun 07 May 2023  8:25PM> journalctl -u kubelet -f                                                                         /home/sarnobat

		-- Logs begin at Wed 2023-05-03 13:00:27 PDT. --
		May 07 20:25:39 nuc2020 kubelet[33704]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
		May 07 20:25:39 nuc2020 kubelet[33704]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
		May 07 20:25:39 nuc2020 kubelet[33704]: I0507 20:25:39.347252   33704 server.go:199] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
		May 07 20:25:39 nuc2020 kubelet[33704]: I0507 20:25:39.349758   33704 server.go:415] "Kubelet version" kubeletVersion="v1.27.1"
		May 07 20:25:39 nuc2020 kubelet[33704]: I0507 20:25:39.349772   33704 server.go:417] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
		May 07 20:25:39 nuc2020 kubelet[33704]: I0507 20:25:39.349909   33704 server.go:837] "Client rotation is on, will bootstrap in background"
		May 07 20:25:39 nuc2020 kubelet[33704]: E0507 20:25:39.350321   33704 bootstrap.go:241] unable to read existing bootstrap client config from /etc/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
		May 07 20:25:39 nuc2020 kubelet[33704]: E0507 20:25:39.350350   33704 run.go:74] "command failed" err="failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
		May 07 20:25:39 nuc2020 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
		May 07 20:25:39 nuc2020 systemd[1]: kubelet.service: Failed with result 'exit-code'.
		q
		May 07 20:25:49 nuc2020 systemd[1]: kubelet.service: Scheduled restart job, restart counter is at 60.
		May 07 20:25:49 nuc2020 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
		May 07 20:25:49 nuc2020 systemd[1]: Started kubelet: The Kubernetes Node Agent.
		May 07 20:25:49 nuc2020 kubelet[33749]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
		May 07 20:25:49 nuc2020 kubelet[33749]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
		May 07 20:25:49 nuc2020 kubelet[33749]: I0507 20:25:49.599935   33749 server.go:199] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
		May 07 20:25:49 nuc2020 kubelet[33749]: I0507 20:25:49.602531   33749 server.go:415] "Kubelet version" kubeletVersion="v1.27.1"
		May 07 20:25:49 nuc2020 kubelet[33749]: I0507 20:25:49.602545   33749 server.go:417] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
		May 07 20:25:49 nuc2020 kubelet[33749]: I0507 20:25:49.602667   33749 server.go:837] "Client rotation is on, will bootstrap in background"
		May 07 20:25:49 nuc2020 kubelet[33749]: E0507 20:25:49.603067   33749 bootstrap.go:241] unable to read existing bootstrap client config from /etc/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
		May 07 20:25:49 nuc2020 kubelet[33749]: E0507 20:25:49.603090   33749 run.go:74] "command failed" err="failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
		May 07 20:25:49 nuc2020 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
		May 07 20:25:49 nuc2020 systemd[1]: kubelet.service: Failed with result 'exit-code'.


		^C

	appserver Sun 07 May 2023  8:25PM> sudo kubeadm join 127.0.0.188:6443 --token u81y02.91gqwkxx6rnhnnly --discovery-token-ca-cert-hash sha256:4482ab1c66bf17992ea02c1ba580f4af9f3ad4cc37b24f189db34d6e3fe95c2d

		[preflight] Running pre-flight checks
			[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
		error execution phase preflight: [preflight] Some fatal errors occurred:
			[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
			[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
		[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
		To see the stack trace of this error execute with --v=5 or higher

	appserver Sun 07 May 2023  8:26PM> kubectl get nodes                                                                                /home/sarnobat

		E0507 20:27:13.942157   34151 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
		E0507 20:27:13.942477   34151 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
		E0507 20:27:13.943794   34151 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
		E0507 20:27:13.945169   34151 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
		E0507 20:27:13.946310   34151 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
		The connection to the server localhost:8080 was refused - did you specify the right host or port?

	appserver Sun 07 May 2023  8:27PM> sudo kubeadm join 127.0.0.188:6443 --token u81y02.91gqwkxx6rnhnnly --discovery-token-ca-cert-hash sha256:4482ab1c66bf17992ea02c1ba580f4af9f3ad4cc37b24f189db34d6e3fe95c2d

		[preflight] Running pre-flight checks
			[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
		error execution phase preflight: [preflight] Some fatal errors occurred:
			[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
			[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
		[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
		To see the stack trace of this error execute with --v=5 or higher

	appserver Sun 07 May 2023  8:28PM> set                                                                                              /home/sarnobat
	appserver Sun 07 May 2023  8:28PM> swa                                                                                              /home/sarnobat
	appserver Sun 07 May 2023  8:28PM> sudo swapoff -a                                                                                  /home/sarnobat
	appserver Sun 07 May 2023  8:28PM> sudo kubeadm join 127.0.0.188:6443 --token u81y02.91gqwkxx6rnhnnly --discovery-token-ca-cert-hash sha256:4482ab1c66bf17992ea02c1ba580f4af9f3ad4cc37b24f189db34d6e3fe95c2d

	[preflight] Running pre-flight checks
	error execution phase preflight: [preflight] Some fatal errors occurred:
		[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
		[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
	[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
	To see the stack trace of this error execute with --v=5 or higher

	appserver Sun 07 May 2023  8:28PM> sudo mv /etc/kubernetes/kubelet.conf /etc/kubernetes/kubelet.conf.disabled                       /home/sarnobat
	appserver Sun 07 May 2023  8:29PM> sudo mv /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/ca.crt.disabled                           /home/sarnobat
	appserver Sun 07 May 2023  8:29PM> sudo kubeadm join 127.0.0.188:6443 --token u81y02.91gqwkxx6rnhnnly --discovery-token-ca-cert-hash sha256:4482ab1c66bf17992ea02c1ba580f4af9f3ad4cc37b24f189db34d6e3fe95c2d

	[preflight] Running pre-flight checks
	^C

	appserver Sun 07 May 2023  8:31PM> sudo kubeadm join 127.0.0.188:6443 --token u81y02.91gqwkxx6rnhnnly --discovery-token-ca-cert-hash sha256:4482ab1c66bf17992ea02c1ba580f4af9f3ad4cc37b24f189db34d6e3fe95c2d -v10
		I0507 20:31:25.875536   35388 join.go:412] [preflight] found NodeName empty; using OS hostname as NodeName
		I0507 20:31:25.875666   35388 initconfiguration.go:117] detected and using CRI socket: unix:///var/run/containerd/containerd.sock
		[preflight] Running pre-flight checks
		I0507 20:31:25.875727   35388 preflight.go:93] [preflight] Running general checks
		I0507 20:31:25.875764   35388 checks.go:280] validating the existence of file /etc/kubernetes/kubelet.conf
		I0507 20:31:25.875774   35388 checks.go:280] validating the existence of file /etc/kubernetes/bootstrap-kubelet.conf
		I0507 20:31:25.875781   35388 checks.go:104] validating the container runtime
		I0507 20:31:25.892955   35388 checks.go:639] validating whether swap is enabled or not
		I0507 20:31:25.892995   35388 checks.go:370] validating the presence of executable crictl
		I0507 20:31:25.893014   35388 checks.go:370] validating the presence of executable conntrack
		I0507 20:31:25.893024   35388 checks.go:370] validating the presence of executable ip
		I0507 20:31:25.893036   35388 checks.go:370] validating the presence of executable iptables
		I0507 20:31:25.893051   35388 checks.go:370] validating the presence of executable mount
		I0507 20:31:25.893062   35388 checks.go:370] validating the presence of executable nsenter
		I0507 20:31:25.893070   35388 checks.go:370] validating the presence of executable ebtables
		I0507 20:31:25.893079   35388 checks.go:370] validating the presence of executable ethtool
		I0507 20:31:25.893086   35388 checks.go:370] validating the presence of executable socat
		I0507 20:31:25.893094   35388 checks.go:370] validating the presence of executable tc
		I0507 20:31:25.893103   35388 checks.go:370] validating the presence of executable touch
		I0507 20:31:25.893114   35388 checks.go:516] running all checks
		I0507 20:31:25.901221   35388 checks.go:401] checking whether the given node name is valid and reachable using net.LookupHost
		I0507 20:31:25.901343   35388 checks.go:605] validating kubelet version
		I0507 20:31:25.931455   35388 checks.go:130] validating if the "kubelet" service is enabled and active
		I0507 20:31:25.939012   35388 checks.go:203] validating availability of port 10250
		I0507 20:31:25.939118   35388 checks.go:280] validating the existence of file /etc/kubernetes/pki/ca.crt
		I0507 20:31:25.939127   35388 checks.go:430] validating if the connectivity type is via proxy or direct
		I0507 20:31:25.939144   35388 checks.go:329] validating the contents of file /proc/sys/net/bridge/bridge-nf-call-iptables
		I0507 20:31:25.939168   35388 checks.go:329] validating the contents of file /proc/sys/net/ipv4/ip_forward
		I0507 20:31:25.939183   35388 join.go:529] [preflight] Discovering cluster-info
		I0507 20:31:25.939196   35388 token.go:80] [discovery] Created cluster-info discovery client, requesting info from "127.0.0.188:6443"
		I0507 20:31:25.939519   35388 round_trippers.go:466] curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: kubeadm/v1.27.1 (linux/amd64) kubernetes/4c94112" 'https://127.0.0.188:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s'
		I0507 20:31:25.939652   35388 round_trippers.go:508] HTTP Trace: Dial to tcp:127.0.0.188:6443 failed: dial tcp 127.0.0.188:6443: connect: connection refused
		I0507 20:31:25.939668   35388 round_trippers.go:553] GET https://127.0.0.188:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s  in 0 milliseconds
		I0507 20:31:25.939675   35388 round_trippers.go:570] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 0 ms Duration 0 ms
		I0507 20:31:25.939681   35388 round_trippers.go:577] Response Headers:
		I0507 20:31:25.939706   35388 token.go:217] [discovery] Failed to request cluster-info, will try again: Get "https://127.0.0.188:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s": dial tcp 127.0.0.188:6443: connect: connection refused
		^C

### join cluster

	appserver Sun 07 May 2023  8:31PM> sudo kubeadm join 192.168.1.4:6443 --token 33hpb0.ps5p55au5vdualfr   --discovery-token-ca-cert-hash sha256:a0a4f3ec96303b6cbb741259663df3a0101109d284c68329e10d35341866dc78

		[preflight] Running pre-flight checks
		[preflight] Reading configuration from the cluster...
		[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
		[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
		[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
		[kubelet-start] Starting the kubelet
		[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

		This node has joined the cluster:
		* Certificate signing request was sent to apiserver and a response was received.
		* The Kubelet was informed of the new secure connection details.

		Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

	appserver Sun 07 May 2023  8:31PM> ip addr                                                                                          /home/sarnobat

		1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
			link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
			inet 127.0.0.1/8 scope host lo
			   valid_lft forever preferred_lft forever
			inet6 ::1/128 scope host
			   valid_lft forever preferred_lft forever
		2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
			link/ether 1c:69:7a:6e:92:ef brd ff:ff:ff:ff:ff:ff
			altname enp0s31f6
			inet 192.168.1.3/24 brd 192.168.1.255 scope global dynamic noprefixroute eno1
			   valid_lft 78210sec preferred_lft 78210sec
			inet6 fe80::4f35:f886:c69d:b370/64 scope link noprefixroute
			   valid_lft forever preferred_lft forever
		3: wlp0s20f3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
			link/ether 34:c9:3d:e3:f5:a8 brd ff:ff:ff:ff:ff:ff
		4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
			link/ether 02:42:e3:07:cb:be brd ff:ff:ff:ff:ff:ff
			inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
			   valid_lft forever preferred_lft forever
		5: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default
			link/ether 12:8e:a1:0a:3a:eb brd ff:ff:ff:ff:ff:ff
			inet 10.244.1.0/32 scope global flannel.1
			   valid_lft forever preferred_lft forever
			inet6 fe80::108e:a1ff:fe0a:3aeb/64 scope link
			   valid_lft forever preferred_lft forever
		6: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
			link/ether 86:b7:96:ed:8b:43 brd ff:ff:ff:ff:ff:ff
			inet 10.244.1.1/24 brd 10.244.1.255 scope global cni0
			   valid_lft forever preferred_lft forever
			inet6 fe80::84b7:96ff:feed:8b43/64 scope link
			   valid_lft forever preferred_lft forever
		7: vethc96f33bf@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default
			link/ether 82:61:1e:6b:b6:f7 brd ff:ff:ff:ff:ff:ff link-netns cni-08b49b24-28b8-0b49-92bf-7416d58ddfa3
			inet6 fe80::8061:1eff:fe6b:b6f7/64 scope link
			   valid_lft forever preferred_lft forever

	appserver Sun 07 May 2023  8:34PM> ifconfig eno0                                                                                    /home/sarnobat
	zsh: command not found: ifconfig
	appserver Sun 07 May 2023  8:34PM> ip addr eno0                                                                                     /home/sarnobat
	Command "eno0" is unknown, try "ip address help".
	appserver Sun 07 May 2023  8:35PM> ip addr eth0                                                                                     /home/sarnobat
	Command "eth0" is unknown, try "ip address help".
	appserver Sun 07 May 2023  8:35PM> ifconfig                                                                                         /home/sarnobat
	zsh: command not found: ifconfig
	appserver Sun 07 May 2023  8:35PM> ip addr                                                                                          /home/sarnobat

		1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
			link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
			inet 127.0.0.1/8 scope host lo
			   valid_lft forever preferred_lft forever
			inet6 ::1/128 scope host
			   valid_lft forever preferred_lft forever
		2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
			link/ether 1c:69:7a:6e:92:ef brd ff:ff:ff:ff:ff:ff
			altname enp0s31f6
			inet 192.168.1.3/24 brd 192.168.1.255 scope global dynamic noprefixroute eno1
			   valid_lft 78193sec preferred_lft 78193sec
			inet6 fe80::4f35:f886:c69d:b370/64 scope link noprefixroute
			   valid_lft forever preferred_lft forever
		3: wlp0s20f3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
			link/ether 34:c9:3d:e3:f5:a8 brd ff:ff:ff:ff:ff:ff
		4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
			link/ether 02:42:e3:07:cb:be brd ff:ff:ff:ff:ff:ff
			inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
			   valid_lft forever preferred_lft forever
		5: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default
			link/ether 12:8e:a1:0a:3a:eb brd ff:ff:ff:ff:ff:ff
			inet 10.244.1.0/32 scope global flannel.1
			   valid_lft forever preferred_lft forever
			inet6 fe80::108e:a1ff:fe0a:3aeb/64 scope link
			   valid_lft forever preferred_lft forever
		6: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
			link/ether 86:b7:96:ed:8b:43 brd ff:ff:ff:ff:ff:ff
			inet 10.244.1.1/24 brd 10.244.1.255 scope global cni0
			   valid_lft forever preferred_lft forever
			inet6 fe80::84b7:96ff:feed:8b43/64 scope link
			   valid_lft forever preferred_lft forever
		7: vethc96f33bf@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default
			link/ether 82:61:1e:6b:b6:f7 brd ff:ff:ff:ff:ff:ff link-netns cni-08b49b24-28b8-0b49-92bf-7416d58ddfa3
			inet6 fe80::8061:1eff:fe6b:b6f7/64 scope link
			   valid_lft forever preferred_lft forever

	appserver Sun 07 May 2023  8:35PM>                                                                                                  /home/sarnobat

## antec

### Check node joined successfully

	antec Sun 07 May 2023  8:28PM> kubectl get nodes                                                                                    /home/sarnobat

		NAME                STATUS     ROLES           AGE   VERSION
		kubernetes-worker   Ready      control-plane   27m   v1.26.3
		nuc2020             NotReady   <none>          11s   v1.27.1

	antec Sun 07 May 2023  8:31PM> kubectl get nodes                                                                                    /home/sarnobat

		NAME                STATUS   ROLES           AGE   VERSION
		kubernetes-worker   Ready    control-plane   27m   v1.26.3
		nuc2020             Ready    <none>          27s   v1.27.1

	antec Sun 07 May 2023  8:32PM> kubectl create deployment nginx --image=nginx                                                        /home/sarnobat

		deployment.apps/nginx created

	antec Sun 07 May 2023  8:33PM> kubectl describe deployment nginx                                                                    /home/sarnobat

		Name:                   nginx
		Namespace:              default
		CreationTimestamp:      Sun, 07 May 2023 20:33:10 -0700
		Labels:                 app=nginx
		Annotations:            deployment.kubernetes.io/revision: 1
		Selector:               app=nginx
		Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
		StrategyType:           RollingUpdate
		MinReadySeconds:        0
		RollingUpdateStrategy:  25% max unavailable, 25% max surge
		Pod Template:
		  Labels:  app=nginx
		  Containers:
		   nginx:
			Image:        nginx
			Port:         <none>
			Host Port:    <none>
			Environment:  <none>
			Mounts:       <none>
		  Volumes:        <none>
		Conditions:
		  Type           Status  Reason
		  ----           ------  ------
		  Available      False   MinimumReplicasUnavailable
		  Progressing    True    ReplicaSetUpdated
		OldReplicaSets:  <none>
		NewReplicaSet:   nginx-748c667d99 (1/1 replicas created)
		Events:
		  Type    Reason             Age   From                   Message
		  ----    ------             ----  ----                   -------
		  Normal  ScalingReplicaSet  9s    deployment-controller  Scaled up replica set nginx-748c667d99 to 1

	antec Sun 07 May 2023  8:33PM>                                                                                                      /home/sarnobat

		kubectl create service nodeport nginx --tcp=80:80
		service/nginx created

	antec Sun 07 May 2023  8:33PM> kubectl get svc                                                                                      /home/sarnobat

		NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
		kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        29m
		nginx        NodePort    10.96.115.33   <none>        80:31833/TCP   12s

### Testing
	antec Sun 07 May 2023  8:33PM> curl 10.96.115.33:31833                                                                              /home/sarnobat
	^C
	antec Sun 07 May 2023  8:35PM> curl nuc2020:31833                                                                                   /home/sarnobat

		<!DOCTYPE html>
		<html>
		<head>
		<title>Welcome to nginx!</title>
		<style>
		html { color-scheme: light dark; }
		body { width: 35em; margin: 0 auto;
		font-family: Tahoma, Verdana, Arial, sans-serif; }
		</style>
		</head>
		<body>
		<h1>Welcome to nginx!</h1>
		<p>If you see this page, the nginx web server is successfully installed and
		working. Further configuration is required.</p>

		<p>For online documentation and support please refer to
		<a href="http://nginx.org/">nginx.org</a>.<br/>
		Commercial support is available at
		<a href="http://nginx.com/">nginx.com</a>.</p>

		<p><em>Thank you for using nginx.</em></p>
		</body>
		</html>

## Client Node
