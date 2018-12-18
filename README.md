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
* Bazel (**0.11.1 is known-good**, 0.17.2 has been tested by an independent contributor to also work)
* (Optional) CUDA (9.0 is known good) and cuDNN (7.1.4 is known good) for GPU support 
* (Optional) TensorRT (for accelerating computation on GPU, 3.0.4 is known-good)

Other versions may work, but they have not been tested (especially for bazel), try and see. 

Recommendation : the bazel building uses a lot of RAM, so it is recommended that you restart your computer before you run the below command, and also exit all running programs, to free as much RAM as possible.

If you encounter errors during bazel configure, bazel building, or during the run of `mcts_engine` or `start.sh` (mostly cuda and cudnn path errors), see [FAQ question : building and running errors](https://github.com/Tencent/PhoenixGo/#1-i-am-getting-errors-during-bazel-configure-bazel-building-andor-running-phoenixgo-engine)

You have 2 possibilities for building on linux :

#### Possibility A : if you want the easy way, Automatic all in command

This all in one command includes : 
- Download and install bazel and dependencies
- Clone PhoenixGo from Tencent github
- Configure the build : 
`./configure` will ask where CUDA, cuDNN, and TensorRT have been installed, specify them if needed.
- Building with bazel : this may take long time (1 hour or more)
- Download the trained network (ckpt)

It is easier to use and should work on most linux distributions (has been tested successfully on ubuntu 16.04 LTS for example)

Run the all-in one command below :


```
sudo apt-get -y install pkg-config zip g++ zlib1g-dev unzip python git && git clone https://github.com/Tencent/PhoenixGo.git && cd PhoenixGo && wget https://github.com/bazelbuild/bazel/releases/download/0.11.1/bazel-0.11.1-installer-linux-x86_64.sh && chmod +x bazel-0.11.1-installer-linux-x86_64.sh && ./bazel-0.11.1-installer-linux-x86_64.sh --user && echo 'export PATH="$PATH:$HOME/bin"' >> ~/.bashrc && source ~/.bashrc && sudo ldconfig && ./configure && bazel build //mcts:mcts_main && wget https://github.com/Tencent/PhoenixGo/releases/download/trained-network-20b-v1/trained-network-20b-v1.tar.gz && tar xvzf trained-network-20b-v1.tar.gz

```

Press ENTER for default settings, except the path that need to be modified. After the building is a success, continue reading at [Running](https://github.com/Tencent/PhoenixGo#running)

#### Possibility B : manual way for more advanced users

##### Building

Clone the repository and configure the building:

```
$ git clone https://github.com/Tencent/PhoenixGo.git
$ cd PhoenixGo
$ ./configure
```

`./configure` will ask where CUDA, cuDNN, and TensorRT have been installed, specify them if needed.

Then build with bazel:

```
$ bazel build //mcts:mcts_main
```

Dependices such as Tensorflow will be downloaded automatically. The building prosess may take a long time (1 hour or more).

##### Downloading the trained network (ckpt)

Download and extract the trained network :

```
$ wget https://github.com/Tencent/PhoenixGo/releases/download/trained-network-20b-v1/trained-network-20b-v1.tar.gz
$ tar xvzf trained-network-20b-v1.tar.gz
```

#### Running 

Run the engine : `scripts/start.sh`

`start.sh` will detect the number of GPUs, run `mcts_main` with proper config file, and write log files in directory `log`.
You could also use a customized config by running `scripts/start.sh {config_path}`.
See also [#configure-guide](#configure-guide).

Furthermore, if you want to fully control all the options of `mcts_main` (such as, changing log destination),
you could also run `bazel-bin/mcts/mcts_main` directly. See also [#command-line-options](#command-line-options). See also [path config file settings](https://github.com/Tencent/PhoenixGo/#10-ckptzerockpt-20b-v1fp32plan-error-no-such-file-or-directory)

The engine supports the GTP protocol, means it could be used with a GUI with GTP capability,
such as [Sabaki](http://sabaki.yichuanshen.de). It can also run on command-line GTP server tools like [gtp2ogs](https://github.com/online-go/gtp2ogs). For more details, see [FAQ question](https://github.com/Tencent/PhoenixGo/#10-gtp-command-error--invalid-command)

#### (Optional) : Distribute mode

PhoenixGo support running with distributed workers, if there are GPUs on different machine.

Build the distribute worker:

```
$ bazel build //dist:dist_zero_model_server
```

Run `dist_zero_model_server` on distributed worker, **one for each GPU**.

```
$ CUDA_VISIBLE_DEVICES={gpu} bazel-bin/dist/dist_zero_model_server --server_address="0.0.0.0:{port}" --logtostderr
```

Fill `ip:port` of workers in the config file (`etc/mcts_dist.conf` is an example config for 32 workers),
and run the distributed master:

```
$ scripts/start.sh etc/mcts_dist.conf
```

### On macOS

**Note: Tensorflow stop providing GPU support on macOS since 1.2.0, so you are only able to run on CPU.**

#### Use Pre-built Binary

Download and extract https://github.com/Tencent/PhoenixGo/releases/download/mac-x64-cpuonly-v1/PhoenixGo-mac-x64-cpuonly-v1.tgz

Follow the document: using_phoenixgo_on_mac.pdf

#### Building from Source

Same as Linux.

### On Windows

#### Use Pre-built Binary

Download and extract https://github.com/Tencent/PhoenixGo/releases/download/win-x64-gpu-v1/PhoenixGo-win-x64-gpu-v1.zip

Or CPU-only version https://github.com/Tencent/PhoenixGo/releases/download/win-x64-cpuonly-v1/PhoenixGo-win-x64-cpuonly-v1.zip

Follow the document: how to install phoenixgo.pdf

## Configure Guide

Here are some important options in the config file:

* `num_eval_threads`: should equal to the number of GPUs
* `num_search_threads`: should a bit larger than `num_eval_threads * eval_batch_size`
* `timeout_ms_per_step`: how many time will used for each move
* `max_simulations_per_step`: how many simulations will do for each move
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
* `--init_moves`: initial moves on the go board
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

For windows, see [FAQ question syntax error](https://github.com/Tencent/PhoenixGo/#12-syntax-error-windows)

## FAQ

#### 1. I am getting errors during bazel configure, bazel building, and/or running PhoenixGo engine

If you built with bazel, see : [Most common path errors during cuda/cudnn install and bazel configure](https://github.com/Tencent/PhoenixGo/blob/master/docs/path-errors.md)

If you are still getting erros, try using an older version of bazel. For example bazel 0.20.0 is known to cause issues, and **bazel 0.11.1 is known good**

#### 2. Where is the win rate?

Print in the log, something like:

<pre>
I0514 12:51:32.724236 14467 mcts_engine.cc:157] 1th move(b): dp, <b>winrate=44.110905%</b>, N=654, Q=-0.117782, p=0.079232, v=-0.116534, cost 39042.679688ms, sims=7132, height=11, avg_height=5.782244, global_step=639200
</pre>

#### 3. There are too much log.

Passing `--v=0` to `mcts_main` will turn off many debug log.
Moreover, `--minloglevel=1` and `--minloglevel=2` could disable INFO log and WARNING log.

Or, if you just don't want to log to stderr, replace `--logtostderr` to `--log_dir={log_dir}`,
then you could read your log from `{log_dir}/mcts_main.INFO`.

#### 4. How to run with Sabaki?

Setting GTP engine in Sabaki's menu: `Engines -> Manage Engines`, fill `Path` with path of `start.sh`.
Click `Engines -> Attach` to use the engine in your game.
See also [#22](https://github.com/Tencent/PhoenixGo/issues/22).

#### 5. How make PhoenixGo think with longer/shorter time?

Modify `timeout_ms_per_step` in your config file.

#### 6. How make PhoenixGo think with constant time per move?

Modify your config file. `early_stop`, `unstable_overtime`, `behind_overtime` and
`time_control` are options that affect the search time, remove them if exist then
each move will cost constant time/simulations.

#### 7. What is the speed of the engine ? How can i make the engine think faster ?

GPU is much faster to compute than CPU (but only nvidia GPU are supported)

TensorRT also increases significantly the speed of computation, but it is only available for linux with a compatible nvidia GPU

Bigger batch size significantly increases the speed of the computation, but a bigger batch size puts a bigger burden on the computation device (in case it is the GPU, higher GPU load, higher VRAM usage), increase it only if your computation device can handle it

Some independent speed benchmarks have been run, they are available in the docs :

- for GTX 1060 :  [benchmark testing batch size from 4 to 64, tree size up to 2000M, max children up to 512, with tensorRT ON and OFF](https://github.com/Tencent/PhoenixGo/blob/master/docs/benchmark-gtx1060.md)

#### 8. I cannot increase batch size to more than 4 with TensorRT : errors

Increasing batch size in the config file makes the engine compute faster, as explained earlier in [FAQ question](https://github.com/Tencent/PhoenixGo/#7-what-is-the-speed-of-the-engine--how-can-i-make-the-engine-think-faster-)

However, with default building, you cannot use batch size higher than 4 with tensorRT
To increase batch size for example to 32 with tensorRT enabled, you need to build tensorrt model with bazel, See : [#75](https://github.com/Tencent/PhoenixGo/issues/75)

#### 9. GTP command `time_settings` doesn't work.

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

#### 10. GTP command error : `invalid command`

Some GTP commands are not supported by PhoenixGo, for example the `showboard` command. Using unsupported commands will most likely make the engine not work. Make sure your GTP tool does not communicate with PhoenixGo with unsupported GTP commands.

For example, for [gtp2ogs](https://github.com/online-go/gtp2ogs) server command line GTP tool, you need to edit the file gtp2ogs.js and manually remove the `showboard` line if it is not already done, [see](https://github.com/online-go/gtp2ogs/commit/d5ebdbbde259a97c5ae1aed0ec42a07c9fbb2dbf)

#### 11. The game does not start, error : `unacceptable komi`

With the default settings, the only komi value supported is only 7.5, with chinese rules only. If it is not automated, you need to manually set komi value to 7.5 with chinese rules.

information : the BensonDarr on [FoxGo](http://weiqi.qq.com/) server is able to play with 6.5 komi because it has been modified, but this is not true for the PhoenixGo engine provided here.

#### 12. Syntax error (Windows)

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

#### 13. '"ckpt/zero.ckpt-20b-v1.FP32.PLAN"' error: No such file or directory

This fix works for all systems : Linux, Mac, Windows, only the name of the ckpt file changes. Modify your config file and write the full path of your ckpt folder, for example for linux : 

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

