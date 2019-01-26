![PhoenixGo](images/logo.jpg?raw=true)

**PhoenixGo** is a Go AI program which implements the AlphaGo Zero paper
"[Mastering the game of Go without human knowledge](https://deepmind.com/documents/119/agz_unformatted_nature.pdf)".
It is also known as "BensonDarr" and "金毛测试" in [FoxGo](http://weiqi.qq.com/), "cronus" in [CGOS](http://www.yss-aya.com/cgos/),
and the champion of [World AI Go Tournament 2018](http://weiqi.qq.com/special/109) held in Fuzhou China.

If you use PhoenixGo in your project, please consider mentioning in your README.

If you use PhoenixGo in your research, please consider citing the library as follows:

```
@misc{PhoenixGo2018,
  author = {Qinsong Zeng and Jianchang Zhang and Zhanpeng Zeng and Yongsheng Li and Ming Chen and Sifan Liu}
  title = {PhoenixGo},
  year = {2018},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/Tencent/PhoenixGo}}
}
```

## Building and Running

### On Linux

#### Requirements

* GCC with C++11 support
* [Bazel](https://docs.bazel.build/versions/master/install.html) (**0.11.1 is known-good**)
* (Optional) CUDA and cuDNN for GPU support 
* (Optional) TensorRT (for accelerating computation on GPU, 3.0.4 is known-good)

The following environments have also been tested by independent contributors : [here](/docs/tested-versions.md). Other versions may work, but they have not been tested (especially for bazel), try and see. 

Recommendation : the bazel building uses a lot of RAM, so it is recommended that you restart your computer before you run the below command, and also exit all running programs, to free as much RAM as possible.

You have 2 possibilities for building on linux :

#### Building - Possibility A : (easy) Automatic all in one command for ubuntu

This all in one command includes : 
- Download and install bazel and its dependencies (apt-get)
- Clone PhoenixGo from Tencent github
- Download and extract the trained network (ckpt) archive
- Cleanup : remove the trained network archive (.tar.gz) and the bazel installer (.sh file)
- Configure the build : during `./configure` , bazel will ask building options :  Press ENTER for default.
CUDA and cuDNN, and optionally TensorRT, are needed only if you want the GPU version (much faster to compute moves)

-> If you want to build the CPU-only version, disable these features (much slower to compute moves).
-> If you want to build the GPU version, enable these features,  

Then choose the other building options you want, and modify paths if needed 
(see [FAQ question](/README.md/#b1-i-am-getting-errors-during-bazel-configure-bazel-building-andor-running-phoenixgo-engine) if you need help)

To speed up building and also to reduce build size at the same time, it is recommended to use the minimalist bazel configure settings, see this example with the GPU version building : [minimalist bazel configure](/docs/minimalist-bazel-configure.md)
- Build PhoenixGo with bazel :  
Dependencies such as Tensorflow will be downloaded and installed automatically. 

The building process may take a long time (1 hour or more) : 

The command below has been tested successfully on ubuntu 16.04 and 18.04 LTS for example

Run the all-in one command below :

```
sudo apt-get -y install pkg-config zip g++ zlib1g-dev unzip python git && git clone https://github.com/Tencent/PhoenixGo.git && cd PhoenixGo && wget https://github.com/bazelbuild/bazel/releases/download/0.11.1/bazel-0.11.1-installer-linux-x86_64.sh && chmod +x bazel-0.11.1-installer-linux-x86_64.sh && ./bazel-0.11.1-installer-linux-x86_64.sh --user && echo 'export PATH="$PATH:$HOME/bin"' >> ~/.bashrc && source ~/.bashrc && sudo ldconfig && wget https://github.com/Tencent/PhoenixGo/releases/download/trained-network-20b-v1/trained-network-20b-v1.tar.gz && tar xvzf trained-network-20b-v1.tar.gz && sudo rm -r trained-network-20b-v1.tar.gz && sudo rm -r bazel-0.11.1-installer-linux-x86_64.sh && ./configure && bazel build //mcts:mcts_main && ls
```

After the building is a success, continue reading at [Running](#running)

#### Building - Possibility B : manual steps for other use

for other linux distributions than ubuntu, or for other specific use, see : [manual building](/docs/manual-building.md)

#### Running 

The PhoenixGo engine supports GTP [(Go Text Protocol)](https://senseis.xmp.net/?GoTextProtocol), which means it can be used with a GUI with GTP capability, such as [Sabaki](http://sabaki.yichuanshen.de).
It can also run on command-line GTP server tools like [gtp2ogs](https://github.com/online-go/gtp2ogs). 

But PhoenixGo does not support all GTP commands, see [FAQ question](#a11-gtp-command-error--invalid-command).

There are 2 ways to run PhoenixGo engine

##### 1) start.sh : easy use

Run the engine : `scripts/start.sh`

`start.sh` will automatically detect the number of GPUs, run `mcts_main` with proper config file, and write log files in directory `log`.

You could also use a customized config file (.conf) by running `scripts/start.sh {config_path}`. 
If you want to do that, see also [#configure-guide](#configure-guide).

##### 2) mcts_main : full control use

If you want to fully control all the options of `mcts_main` (such as, changing log destination, or if start.sh is not compatible for your specific use), you can run directly `bazel-bin/mcts/mcts_main` instead. 

For a typical GTP use (run `genmove b` successfully for example), minimum should be to add these needed command line options :
- `--gtp` to enable GTP mode
- `--config_path=replace/with/path/to/your/config/file` to specify the path to your config file
- it is also needed to edit your config file (.conf) and manually add the full path to ckpt, see [FAQ question](#a5-ckptzerockpt-20b-v1fp32plan-error-no-such-file-or-directory). You can also change all config file settings, see [#configure-guide](#configure-guide).
- for other extra commands , see also [#command-line-options](#command-line-options) for details, or run `./mcts_main --help` . A copy of the `--help` is provided for your convenience [here](/docs/mcts-main-help.md)
- for example, for the username amd2018, and PhoenixGo directory in home directory :

```
cd /home/amd2018/PhoenixGo/bazel-bin/mcts && ./mcts_main --gtp --config_path=/home/amd2018/PhoenixGo/etc/mcts_1gpu.conf --logtostderr --v=0
```

#### (Optional) : Distribute mode

PhoenixGo support running with distributed workers, if there are GPUs on different machine.

Build the distribute worker:

```
bazel build //dist:dist_zero_model_server
```

Run `dist_zero_model_server` on distributed worker, **one for each GPU**.

```
CUDA_VISIBLE_DEVICES={gpu} bazel-bin/dist/dist_zero_model_server --server_address="0.0.0.0:{port}" --logtostderr
```

Fill `ip:port` of workers in the config file (`etc/mcts_dist.conf` is an example config for 32 workers),
and run the distributed master:

```
scripts/start.sh etc/mcts_dist.conf
```

### On macOS

**Note: Tensorflow stop providing GPU support on macOS since 1.2.0, so you are only able to run on CPU.**

#### Use Pre-built Binary

Download and extract [CPU-only version (macOS)](https://github.com/Tencent/PhoenixGo/releases/download/mac-x64-cpuonly-v1/PhoenixGo-mac-x64-cpuonly-v1.tgz)

Follow the document included in the archive : using_phoenixgo_on_mac.pdf

#### Building from Source

Same as Linux.

### On Windows

For all windows versions, see correct syntax in [FAQ question](#a4-syntax-error-windows)

#### Use Pre-built Binary

##### GPU version :

The GPU version is much faster, but works only with compatible nvidia GPU.
It supports this environment : 
- CUDA 9.0 only
- cudnn 7.1.x (x is any number) or lower for CUDA 9.0
- no AVX, AVX2, AVX512 instructions supported in this release (so it is currently much slower than the linux version)
- there is no TensorRT support on Windows

Download and extract [GPU version (Windows)](https://github.com/Tencent/PhoenixGo/releases/download/win-x64-gpu-v1/PhoenixGo-win-x64-gpu-v1.zip)

Then follow the document included in the archive : how to install phoenixgo.pdf

note : to support special features like CUDA 10.0 or AVX512 for example, you can build your own build for windows, see [#79](https://github.com/Tencent/PhoenixGo/issues/79)

##### CPU-only version : 

If your GPU is not compatible, or if you don't want to use a GPU, you can download this [CPU-only version (Windows)](https://github.com/Tencent/PhoenixGo/releases/download/win-x64-cpuonly-v1/PhoenixGo-win-x64-cpuonly-v1.zip), 

Follow the document included in the archive : how to install phoenixgo.pdf

## Configure Guide

Here are some important options in the config file:

* `num_eval_threads`: should equal to the number of GPUs
* `num_search_threads`: should a bit larger than `num_eval_threads * eval_batch_size`
* `timeout_ms_per_step`: how many time will used for each move
* `max_simulations_per_step`: how many simulations will do for each move (playouts)
* `gpu_list`: use which GPUs, separated by comma
* `model_config -> train_dir`: directory where trained network stored
* `model_config -> checkpoint_path`: use which checkpoint, get from `train_dir/checkpoint` if not set
* `model_config -> enable_tensorrt`: use TensorRT or not
* `model_config -> tensorrt_model_path`: use which TensorRT model, if `enable_tensorrt`
* `max_search_tree_size`: the maximum number of tree nodes, change it depends on memory size
* `max_children_per_node`: the maximum children of each node, change it depends on memory size
* `enable_background_search`: pondering in opponent's time
* `early_stop`: genmove may return before `timeout_ms_per_step`, if the result would not change any more
* `unstable_overtime`: think `timeout_ms_per_step * time_factor` more if the result still unstable
* `behind_overtime`: think `timeout_ms_per_step * time_factor` more if winrate less than `act_threshold`

Options for distribute mode:

* `enable_dist`: enable distribute mode
* `dist_svr_addrs`: `ip:port` of distributed workers, multiple lines, one `ip:port` in each line
* `dist_config -> timeout_ms`: RPC timeout

Options for async distribute mode:

> Async mode is used when there are huge number of distributed workers (more than 200),
> which need too many eval threads and search threads in sync mode.
> `etc/mcts_async_dist.conf` is an example config for 256 workers.

* `enable_async`: enable async mode
* `enable_dist`: enable distribute mode
* `dist_svr_addrs`: multiple lines, comma sperated lists of `ip:port` for each line
* `num_eval_threads`: should equal to number of `dist_svr_addrs` lines
* `eval_task_queue_size`: tunning depend on number of distribute workers
* `num_search_threads`: tunning depend on number of distribute workers

Read `mcts/mcts_config.proto` for more config options.

## Command Line Options

`mcts_main` accept options from command line:

* `--config_path`: path of config file
* `--gtp`: run as a GTP engine, if disable, gen next move only
* `--init_moves`: initial moves on the go board, for example of use, see [#83](https://github.com/Tencent/PhoenixGo/issues/83)
* `--gpu_list`: override `gpu_list` in config file
* `--listen_port`: work with `--gtp`, run gtp engine on port in TCP protocol
* `--allow_ip`: work with `--listen_port`, list of client ip allowed to connect
* `--fork_per_request`: work with `--listen_port`, fork for each request or not

Glog options are also supported:

* `--logtostderr`: log message to stderr
* `--log_dir`: log to files in this directory
* `--minloglevel`: log level, 0 - INFO, 1 - WARNING, 2 - ERROR
* `--v`: verbose log, `--v=1` for turning on some debug log, `--v=0` to turning off

`mcts_main --help` for more command line options.
A copy of the `--help` is provided for your convenience [here](/docs/mcts-main-help.md)

## Analysis

It is possible to analyse .sgf files using analysis tools such as :
- [GoReviewPartner](https://github.com/pnprog/goreviewpartner) : an automated tool to analyse and/or review one or many .sgf files (saved as .rsgf file). It supports PhoenixGo and other bots. See [FAQ question](#a25-how-to-analyzereview-one-or-many-sgf-files-with-goreviewpartner) for details

For analysis purpose, an easy way to display the PV (variations for main move path) is `--logtostderr --v=1` which will display the main move path winrate and continuation of moves analyzed, see [FAQ question](#a2-where-is-the-pv-analysis-) for details

## FAQ

### General Questions (windows/mac/linux):

#### A1. Where is the win rate?

Print in the log, something like:

<pre>
I0514 12:51:32.724236 14467 mcts_engine.cc:157] 1th move(b): dp, <b>winrate=44.110905%</b>, N=654, Q=-0.117782, p=0.079232, v=-0.116534, cost 39042.679688ms, sims=7132, height=11, avg_height=5.782244, global_step=639200
</pre>

#### A2. Where is the PV (Analysis) ?

It is possible to display the PV (variation of move path with continuation of the moves)

An easy way to do that is for example to increase verbose level, for example `--logtostderr --v=1`

result is something like this (in this example there are 7000 simulations per move) :

```
[5489] stderr: I0116 15:55:09.910559  5489 mcts_debugger.cc:43] ========== debug info for 27th move(b) begin ==========
[5489] stderr: I0116 15:55:09.910596  5489 mcts_debugger.cc:44] main move path: mg(6482,-0.12,0.89,-0.09),lf(6349,0.12,0.88,0.08),lg(3006,-0.12,0.53,-0.12),kf(2654,0.14,0.63,-0.01),qh(941,-0.11,0.16,-0.11),qi(785,0.12,0.77,0.13),rh(784,-0.12,0.98,-0.13),pi(538,0.13,0.52,0.09),rg(445,-0.12,0.73,-0.18),og(365,0.13,0.76,0.15),pf(363,-0.13,0.98,-0.19),rd(159,0.10,0.52,0.14),ql(47,-0.06,0.21,-0.18),oq(19,0.04,0.36,-0.03),nq(12,-0.05,0.58,-0.07),pq(11,0.05,0.97,0.03),np(10,-0.05,0.79,-0.12),qn(9,0.04,0.84,0.03),ol(8,-0.05,0.82,-0.05),fc(6,0.07,0.60,0.06),kg(4,-0.06,0.42,-0.13),jf(2,-0.02,0.53,0.02),jg(1,0.05,0.45,0.05),cg(0,-nan,0.00,nan)
[5489] stderr: I0116 15:55:09.910604  5489 mcts_debugger.cc:139] mg: N=6482, W=-757.077, Q=-0.116797, p=0.887549, v=-0.0932888
[5489] stderr: I0116 15:55:09.910609  5489 mcts_debugger.cc:139] oh: N=262, W=-38.6935, Q=-0.147685, p=0.0710239, v=-0.137658
[5489] stderr: I0116 15:55:09.910614  5489 mcts_debugger.cc:139] qh: N=38, W=-6.34157, Q=-0.166883, p=0.0139782, v=-0.144544
[5489] stderr: I0116 15:55:09.910617  5489 mcts_debugger.cc:139] nb: N=1, W=-0.294998, Q=-0.294998, p=0.000126838, v=-0.295011
[5489] stderr: I0116 15:55:09.910621  5489 mcts_debugger.cc:139] lc: N=1, W=-0.354553, Q=-0.354553, p=0.000164667, v=-0.354567
[5489] stderr: I0116 15:55:09.910626  5489 mcts_debugger.cc:139] ob: N=1, W=-0.28775, Q=-0.28775, p=0.000373717, v=-0.287757
[5489] stderr: I0116 15:55:09.910630  5489 mcts_debugger.cc:139] oj: N=1, W=-0.359818, Q=-0.359818, p=0.000128213, v=-0.359819
[5489] stderr: I0116 15:55:09.910634  5489 mcts_debugger.cc:139] rc: N=1, W=-0.30574, Q=-0.30574, p=0.000222438, v=-0.305748
[5489] stderr: I0116 15:55:09.910639  5489 mcts_debugger.cc:139] oc: N=1, W=-0.281403, Q=-0.281403, p=0.000368299, v=-0.281408
[5489] stderr: I0116 15:55:09.910642  5489 mcts_debugger.cc:139] qd: N=1, W=-0.288712, Q=-0.288712, p=0.00012808, v=-0.288724
[5489] stderr: I0116 15:55:09.910646  5489 mcts_debugger.cc:48] model global step: 639200
[5489] stderr: I0116 15:55:09.910648  5489 mcts_debugger.cc:49] ========== debug info for 27th move(b) end   ==========
```

It is also possible to develop all the tree by adding these lines to your config file :

```
debugger {
    print_tree_depth: 6
    print_tree_width: 6
}
```

In this example, a tree of 6 depth 6 width moves will be printed

#### A2.5 How to analyze/review one or many sgf file(s) with GoReviewPartner

Support is still in testing, but so far these are the recommended settings to do use PhoenixGo with GoReviewPartner

2 config files are provided to you using optimized settings for grp (GoReviewPartner)
- with tensorrt (linux only) : [mcts_1gpu_grp.conf](/etc/mcts_1gpu_grp.conf)
- without tensorrt (linux, mac, windows, gpu or cpu version) :
 [mcts_1gpu_notensorrt_grp.conf](/etc/mcts_1gpu_notensorrt_grp.conf)
 
note : it is also possible with multiple GPU, only the most common examples were shown here

if you want to do the changes manually, you need to :

in phoenixgo .conf config file :

- disable all time settings in config file (set to `0`)
- set time to unlimited (set timeout to 0 ms)
- use simulations per move to have fixed computation per move (playouts), because it is not needed to play fast since this 
is an analysis, unlimited time
- for the same reason, set `enable background search` to 0 (disable pondering), because this is not a live game, 
time settings are not needed and can cause conflicts
- add debug width and height with `debugger` in config file, see previous FAQ question

in grp (GoReviewPartner) settings :
- it is easier to use one of the pre-made grp profile in config.ini (slightly modify them if needed)

Don't forget to add paths in the config file as explained in [FAQ question](#a5-ckptzerockpt-20b-v1fp32plan-error-no-such-file-or-directory)

If you're on windows, you need to also pay attention to the syntax too, see [FAQ question](#a4-syntax-error-windows)

and run the mcts_main (not start.sh) with the needed parameters, 
see an example [here](https://github.com/wonderingabout/goreviewpartner/blob/config-profiles-phoenixgo/config.ini#L100-L116)

also, see [#86](https://github.com/Tencent/PhoenixGo/issues/86) and [#99](https://github.com/pnprog/goreviewpartner/issues/99) for details

#### A3. There are too much log.

Passing `--v=0` to `mcts_main` will turn off many debug log.
Moreover, `--minloglevel=1` and `--minloglevel=2` could disable INFO log and WARNING log.

Or, if you just don't want to log to stderr, replace `--logtostderr` to `--log_dir={log_dir}`,
then you could read your log from `{log_dir}/mcts_main.INFO`.

#### A4. Syntax error (Windows)

For windows,
- in config file, 

you need to write path with `/` and not `\` in the config file .conf, for example : 

```
model_config {
      train_dir: "c:/users/amd2018/Downloads/PhoenixGo/ckpt"
```

- in cmd.exe,

Here you need to write paths with `\` and not `/`. Also command format on windows needs a space and not a `=`, for example : 

`mcts_main.exe --gtp --config_path C:\Users\amd2018\Downloads\PhoenixGo\etc\mcts_1gpu_notensorrt.conf`

See next point below :

#### A5. '"ckpt/zero.ckpt-20b-v1.FP32.PLAN"' error: No such file or directory

This fix works for all systems : Linux, Mac, Windows, only the name of the ckpt file changes. Modify your config file and write the full path of your ckpt directory, for example for linux : 

```
model_config {
    train_dir: "/home/amd2018/PhoenixGo/ckpt"
```
    
for example, for windows :

```
model_config {
    train_dir: "c:/users/amd2018/Downloads/PhoenixGo/ckpt"
```

if you use tensorRT (linux only, and compatible nvidia GPU only), also change path of tensorRT, for example :

```
model_config {
    train_dir: "/home/amd2018/PhoenixGo/ckpt/"
    enable_tensorrt: 1
    tensorrt_model_path: "/home/amd2018/test/PhoenixGo/ckpt/zero.ckpt-20b-v1.FP32.PLAN"
}
```

#### A6. How to run with Sabaki?

Setting GTP engine in Sabaki's menu: `Engines -> Manage Engines`, fill `Path` with path of `start.sh`.
Click `Engines -> Attach` to use the engine in your game.
See also [#22](https://github.com/Tencent/PhoenixGo/issues/22).

#### A7. How make PhoenixGo think with longer/shorter time?

Modify `timeout_ms_per_step` in your config file.

#### A8. How make PhoenixGo think with constant time per move?

Modify your config file. `early_stop`, `unstable_overtime`, `behind_overtime` and
`time_control` are options that affect the search time, remove them if exist then
each move will cost constant time/simulations.

#### A9. What is the speed of the engine ? How can i make the engine think faster ?

GPU is much faster to compute than CPU (but only nvidia GPU are supported)

TensorRT also increases significantly the speed of computation, but it is only available for linux with a compatible nvidia GPU

Bigger batch size significantly increases the speed of the computation, but a bigger batch size puts a bigger burden on the computation device (in case it is the GPU, higher GPU load, higher VRAM usage), increase it only if your computation device can handle it

Some independent speed benchmarks have been run, they are available in the docs :

- for GTX 1060 :  [benchmark testing batch size from 4 to 64, tree size up to 2000M, max children up to 512, with tensorRT ON and OFF](/docs/benchmark-gtx1060.md)

#### A10. GTP command `time_settings` doesn't work.

Add these lines in your config:

```
time_control {
    enable: 1
    c_denom: 20
    c_maxply: 40
    reserved_time: 1.0
}
```

`c_denom` and `c_maxply` are parameters for deciding how to use the "main time".
`reserved_time` is how many seconds should reserved (for network latency) in "byo-yomi time".

#### A11. GTP command error : `invalid command`

Some GTP commands are not supported by PhoenixGo, for example the `showboard` command. 

To know supported GTP commands, start phoenixgo in GTP mode and enter the GTP command `list_commands`

Result as of today is : 

```
version
protocol_version
list_commands
quit
clear_board
boardsize
komi
time_settings
time_left
place_free_handicap
set_free_handicap
play
genmove
final_score
get_debug_info
get_last_move_debug_info
```

If you use unsupported commands the engine will not work. Make sure your GTP tool does not communicate with PhoenixGo with unsupported GTP commands.

For example, for [gtp2ogs](https://github.com/online-go/gtp2ogs) server command line GTP tool, you need to edit the file gtp2ogs.js and manually remove the existing `showboard` line if it is not already done, [see](https://github.com/online-go/gtp2ogs/commit/d5ebdbbde259a97c5ae1aed0ec42a07c9fbb2dbf)

#### A12. The game does not start, error : `unacceptable komi`

With the default settings, the only komi value supported is only 7.5, with chinese rules only. If it is not automated, you need to manually set komi value to 7.5 with chinese rules.

If you want to implement PhoenixGo to a server where players play with different komi values (for example 6.5, 0.5, 85.5, 200.5,etc), you need to force komi to 7.5 for the players. 

If it is not possible, there is a workaround you can use : configure you GTP tool to tell PhoenixGo engine that the komi for the game is 7.5 even if it is not true

if you do that, the game will not be scored correctly because PhoenixGo will think that the komi is 7.5 while the real komi is different, but at least PhoenixGo will be able to play the game. An example for [gtp2ogs](https://github.com/online-go/gtp2ogs) is provided [here](https://github.com/wonderingabout/gtp2ogs-tutorial/blob/master/docs/3A4-linux-optional-edit-gtp2ogs-js-file.md) and [here](https://github.com/wonderingabout/gtp2ogs-tutorial/blob/master/docs/3B4-windows-optional-edit-gtp2ogs-js-file.md)

information : the BensonDarr on [FoxGo server](http://weiqi.qq.com/) is able to play with 6.5 komi because it has been modified, but this is not true for the PhoenixGo engine provided here.

#### A13. I have a nvidia RTX card (Turing) or Tesla V100/Titan V (Volta), is it compatible ?

##### RTX cards (Turing) :

- need CUDA 10.0 or higher (so currently, only linux is supported, or windows with your own building)
- If you compile PhoenixGo, it has been tested to work on linux [here](/docs/tested-versions.md) with CUDA 10.0, cudnn 7.4.2, ubuntu 18.04.
- However currently there is no tensorRT support for PhoenixGo (RTX cards require tensorRT 5.x or more (and this also requires tensorflow 1.9 or more), which is currently not supported by PhoenixGo)

##### Volta cards (Tesla V100 / Titan V and similar)

- are compatible with cuda 9.0 and higher (it is recommended to use latest version when possible), cudnn 7.1.x or higher (x is any number)
- has been tested to work successfully on windows
- However currently there is no tensorRT support for PhoenixGo (Tesla V100 has been tested not to work with PhoenixGo tensorrt 3.0.4)

### Specific questions : building with bazel questions (linux and mac)

#### B1. I am getting errors during bazel configure, bazel building, and/or running PhoenixGo engine

If you built with bazel, see : [Most common path errors during cuda/cudnn install and bazel configure](/docs/path-errors.md)

See also [minimalist bazel configure](/docs/minimalist-bazel-configure.md) for an example of build configure

If you are still getting errors, try using an older version of bazel. For example bazel 0.20.0 is known to cause issues, and **bazel 0.11.1 is known good**

#### B2. The PhoenixGo and bazel folders are too big

During the bazel building, there are many options that can are not required and can be disabled

This will reduce building time and will have smaller size after the building, see [minimalist bazel configure](/docs/minimalist-bazel-configure.md) and [#76](https://github.com/Tencent/PhoenixGo/issues/76)

#### B3. I cannot increase batch size to more than 4 with TensorRT (linux only) : errors

Increasing batch size in the config file makes the engine compute faster, as explained earlier in [FAQ question](#6-what-is-the-speed-of-the-engine--how-can-i-make-the-engine-think-faster-)

However, with default building, you cannot use batch size higher than 4 with tensorRT
To increase batch size for example to 32 with tensorRT enabled, you need to build tensorrt model with bazel, See : [#75](https://github.com/Tencent/PhoenixGo/issues/75)

#### B4. How to remove entirely all PhoenixGo and bazel files and folders ?

assuming you installed PhoenixGo in home directory (`~` or `/home/yourusername/`)

```
# clean with bazel
cd ~/PhoenixGo && bazel clean
# remove PhoenixGo local directory
sudo rm -rf ~/PhoenixGo
# remove any remaining bazel file
sudo rm -rf ~/.cache/bazel
```
This will free a few GB (arround 3-6 GB depending on your installation settings)
