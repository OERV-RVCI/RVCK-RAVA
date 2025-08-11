FROM debian:latest

RUN sed -i 's@deb.debian.org@mirrors.tuna.tsinghua.edu.cn@g' /etc/apt/sources.list.d/debian.sources

RUN (type -p wget >/dev/null || (apt update &&apt-get install wget -y)) \
	&& mkdir -p -m 755 /etc/apt/keyrings \
        && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        && cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null
	
RUN apt update

RUN apt install gh lavacli gcc-riscv64-linux-gnu gcc-riscv64-unknown-elf \
	make gcc g++ flex device-tree-compiler pahole dwarves libfuse-dev kmod  bzip2 xz-utils curl cpio bc libssl-dev libncurses-dev bison flex libelf-dev \
	qemu-system-riscv64 \
	python3-ply python3-git \
	git vim bash-completion tmux sudo rsync \
	golang qemu-system-misc build-essential g++-riscv64-linux-gnu rpm tree zstd yq \
	-y

# RUN wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/bin/yq && chmod +x /usr/bin/yq