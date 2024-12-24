
## Linux & Unix 

Step 01. Setting Image Hub for Rustup，modified Configuration file: `~/.zshrc` or `~/.bashrc`

```bash
export RUSTUP_DIST_SERVER="https://rsproxy.cn" 
export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
```

Step 02. Rust Installed 

<font color="#9932CC"><b>Tips: source the rc file or restart the terminal to take effect.</b></font>

```sh
curl --proto '=https' --tlsv1.2 -sSf https://rsproxy.cn/rustup-init.sh | sh
```

Step 03. Setting Image Hub for crates.io, modified Configuration file: `~/.cargo/config`, support git and sparse protocol, rust version >= 1.68 recommands use `sparse-index`, fast more

```bash
# sparse
[source.crates-io]
replace-with = 'rsproxy-sparse'
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"
[net]
git-fetch-with-cli = true
```

```bash
# rsproxy
[source.crates-io]
replace-with = 'rsproxy'
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"
[net]
git-fetch-with-cli = true
```

## Windows 10/11

Read the record：[Rust Install in D Disk](obsidian://open?vault=Obsidian_Notes&file=Languages%2FRust%2FRust%20Installtion%20in%20D%20Disk%20with%20VSCode%20(Win%2010))

## Rust Update to release version

```sh
rustc -V
rustc 1.75.0 (82e1608df 2023-12-21)
```

```sh
# Update 
rustup update
```

## Rust Version Switch 

```sh
# first use nightly 
rustup install nightly

rustup default nightly # switch nightly version
rustup default stable # swtich stable version

rustup -V # Check version
```