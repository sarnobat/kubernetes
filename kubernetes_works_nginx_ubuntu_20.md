# 2023-05 nginx on Ubuntu 20

-   [2023-05 nginx on Ubuntu 20](#nginx-on-ubuntu-20)
    -   [Server Node (antec)](#server-node-antec)
        -   [~~taint nodes~~](#taint-nodes)
        -   [Install containerd](#install-containerd)
        -   [start containerd](#start-containerd)
        -   [start kubelet](#start-kubelet)
        -   [taint nodes](#taint-nodes-1)
        -   [Create cluster](#create-cluster)
        -   [apply flannel](#apply-flannel)
        -   [Check cluster master node is
            up](#check-cluster-master-node-is-up)
        -   [Check control plane is up](#check-control-plane-is-up)
    -   [nuc2020](#nuc2020)
        -   [install kubeadm, kubectl
            etc.](#install-kubeadm-kubectl-etc.)
        -   [install kubelet](#install-kubelet)
        -   [join cluster](#join-cluster)
    -   [Server node (antec)](#server-node-antec-1)
        -   [Check node joined
            successfully](#check-node-joined-successfully)
        -   [Testing nginx](#testing-nginx)
    -   [Client Node](#client-node)



based on
* https://www.cloudsigma.com/how-to-install-and-use-kubernetes-on-ubuntu-20-04/
    
## Server Node (antec)



### kubeadm reset

	antec Sun 07 May 2023  7:58PM> sudo kubeadm reset

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

### Install containerd

	antec Sun 07 May 2023  7:59PM> ps aux | grep containerd

		root        2180  0.2  0.0 2321040 23028 ?       Ssl  Apr06 123:04 /usr/bin/containerd
		sarnobat  648481  0.0  0.0  17864  2276 pts/19   S+   20:00   0:00 grep containerd
		root     1560181  0.0  0.0 1891168 23920 ?       Ssl  Apr24  12:43 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

	antec Sun 07 May 2023  8:00PM> sudo systemctl status containerd

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


	antec Sun 07 May 2023  8:00PM> sudo mkdir -p /etc/containerd/; containerd config default | sudo tee /etc/containerd/config.toml

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



	antec Sun 07 May 2023  8:00PM> sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

### systemctl restart containerd

	antec Sun 07 May 2023  8:00PM> sudo systemctl restart containerd

### swapoff

	antec Sun 07 May 2023  8:00PM> swapoff -a    
### systemctl restart kubelet

	antec Sun 07 May 2023  8:01PM> sudo systemctl restart kubelet                

### taint nodes

	antec Sun 07 May 2023  8:01PM> kubectl taint nodes --all node-role.kubernetes.io/control-plane-

		E0507 20:01:33.624612  648951 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		E0507 20:01:33.625571  648951 memcache.go:265] couldn't get current server API group list: Get "https://192.168.1.4:6443/api?timeout=32s": dial tcp 192.168.1.4:6443: connect: connection refused
		The connection to the server 192.168.1.4:6443 was refused - did you specify the right host or port?

	antec Sun 07 May 2023  8:01PM> sudo swapoff -a

	antec Sun 07 May 2023  8:02PM> sudo hostnamectl set-hostname kubernetes-master
	antec Sun 07 May 2023  8:02PM> sudo hostnamectl set-hostname kubernetes-worker

	antec Sun 07 May 2023  8:02PM> lsmod | grep br_netfilter

		br_netfilter           32768  0
		bridge                331776  1 br_netfilter

	antec Sun 07 May 2023  8:02PM> sudo sysctl net.bridge.bridge-nf-call-iptables=1

		net.bridge.bridge-nf-call-iptables = 1

### Create cluster

	antec Sun 07 May 2023  8:02PM> sudo kubeadm init --pod-network-cidr=10.244.0.0/16

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

	antec Sun 07 May 2023  8:04PM> sudo kubeadm init --pod-network-cidr=10.244.0.0/16
	antec Sun 07 May 2023  8:08PM> mkdir -p $HOME/.kube

		sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		sudo chown $(id -u):$(id -g) $HOME/.kube/config
		cp: overwrite '/.kube/config'? y

	antec Sun 07 May 2023  8:08PM> sudo ufw allow 6443

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

### Check cluster master node is up

	antec Sun 07 May 2023  8:09PM> kubectl get pods --all-namespaces

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


	antec Sun 07 May 2023  8:09PM> kubectl get componentstatus

		Warning: v1 ComponentStatus is deprecated in v1.19+
		NAME                 STATUS    MESSAGE                         ERROR
		controller-manager   Healthy   ok
		scheduler            Healthy   ok
		etcd-0               Healthy   {"health":"true","reason":""}

	antec Sun 07 May 2023  8:09PM> kubectl get cs

		Warning: v1 ComponentStatus is deprecated in v1.19+
		NAME                 STATUS    MESSAGE                         ERROR
		scheduler            Healthy   ok
		controller-manager   Healthy   ok
		etcd-0               Healthy   {"health":"true","reason":""}


### Check control plane is up

	antec Sun 07 May 2023  8:11PM> kubectl get nodes

		NAME                STATUS   ROLES           AGE   VERSION
		kubernetes-worker   Ready    control-plane   23m   v1.26.3

## nuc2020

### install kubeadm, kubectl etc.

	appserver Sun 07 May 2023  8:12PM> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

	OK

	appserver Sun 07 May 2023  8:14PM> echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
	sudo mv ~/kubernetes.list /etc/apt/sources.list.d

	appserver Sun 07 May 2023  8:14PM> sudo apt install kubeadm

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

### install kubelet

	appserver Sun 07 May 2023  8:17PM> sudo apt update && sudo apt install kubelet

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

	appserver Sun 07 May 2023  8:23PM> sudo apt reinstall kubelet

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

### kubeadm join

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


## Server node (antec)

### Check node joined successfully

	antec Sun 07 May 2023  8:28PM> kubectl get nodes

		NAME                STATUS     ROLES           AGE   VERSION
		kubernetes-worker   Ready      control-plane   27m   v1.26.3
		nuc2020             NotReady   <none>          11s   v1.27.1

	antec Sun 07 May 2023  8:31PM> kubectl get nodes

		NAME                STATUS   ROLES           AGE   VERSION
		kubernetes-worker   Ready    control-plane   27m   v1.26.3
		nuc2020             Ready    <none>          27s   v1.27.1

	antec Sun 07 May 2023  8:32PM> kubectl create deployment nginx --image=nginx

		deployment.apps/nginx created

	antec Sun 07 May 2023  8:33PM> kubectl describe deployment nginx

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

	antec Sun 07 May 2023  8:33PM>

		kubectl create service nodeport nginx --tcp=80:80
		service/nginx created

	antec Sun 07 May 2023  8:33PM> kubectl get svc

		NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
		kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        29m
		nginx        NodePort    10.96.115.33   <none>        80:31833/TCP   12s

	antec Sun 07 May 2023  8:28PM> kubectl get all --all-namespaces                                      /media/sarnobat/3TB/2021/disks/thistle/videos

		NAMESPACE      NAME                                            READY   STATUS    RESTARTS   AGE
		default        pod/nginx-748c667d99-q865x                      1/1     Running   0          65m
		kube-flannel   pod/kube-flannel-ds-swkrf                       1/1     Running   0          67m
		kube-flannel   pod/kube-flannel-ds-wdrzc                       1/1     Running   0          90m
		kube-system    pod/coredns-787d4945fb-224pv                    1/1     Running   0          94m
		kube-system    pod/coredns-787d4945fb-zmzf6                    1/1     Running   0          94m
		kube-system    pod/etcd-kubernetes-worker                      1/1     Running   0          94m
		kube-system    pod/kube-apiserver-kubernetes-worker            1/1     Running   0          94m
		kube-system    pod/kube-controller-manager-kubernetes-worker   1/1     Running   0          94m
		kube-system    pod/kube-proxy-qxb2m                            1/1     Running   0          67m
		kube-system    pod/kube-proxy-x97gx                            1/1     Running   0          94m
		kube-system    pod/kube-scheduler-kubernetes-worker            1/1     Running   0          94m

		NAMESPACE     NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
		default       service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP                  94m
		default       service/nginx        NodePort    10.96.115.33   <none>        80:31833/TCP             65m
		kube-system   service/kube-dns     ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   94m

		NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
		kube-flannel   daemonset.apps/kube-flannel-ds   2         2         2       2            2           <none>                   90m
		kube-system    daemonset.apps/kube-proxy        2         2         2       2            2           kubernetes.io/os=linux   94m

		NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
		default       deployment.apps/nginx     1/1     1            1           65m
		kube-system   deployment.apps/coredns   2/2     2            2           94m

		NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
		default       replicaset.apps/nginx-748c667d99     1         1         1       65m
		kube-system   replicaset.apps/coredns-787d4945fb   2         2         2       94m

### Testing nginx

	antec Sun 07 May 2023  8:33PM> curl 10.96.115.33:31833
	^C
	antec Sun 07 May 2023  8:35PM> curl nuc2020:31833

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
