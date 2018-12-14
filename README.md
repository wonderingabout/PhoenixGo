![PhoenixGo](images/logo.jpg?raw=true)

**PhoenixGo** is a Go AI program which implements the AlphaGo Zero paper
"[Mastering the game of Go without human knowledge](https://deepmind.com/documents/119/agz_unformatted_nature.pdf)".
It is also known as "BensonDarr" in FoxGo, "cronus" in CGOS,
and the champion of "World AI Go Tournament 2018" held in Fuzhou China.

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
* Bazel (**0.11.1 is known-good**)
* (Optional) CUDA (9.0 is known good) and cuDNN (7.1.4 is known good) for GPU support 
* (Optional) TensorRT (for accelerating computation on GPU, 3.0.4 is known-good)

Other versions may work, but they have not been tested (especially for bazel), try and see. 

Recommendation : the bazel building uses a lot of RAM, so it is recommended that you restart your computer before you run the below command, and also exit all running programs, to free as much RAM as possible.

If you encounter errors during bazel configure, bazel building, or during the run of `mcts_engine` or `start.sh` (mostly cuda and cudnn path errors), see [FAQ question : building and running errors](https://github.com/Tencent/PhoenixGo/#1-i-am-getting-errors-during-bazel-configure-bazel-building-andor-running-phoenixgo-engine)

You have 2 possibilities for building on linux :

#### Possibility A : if you want the easy way, Automatic all in command

This all in one command includes : 
- Download and install bazel
- Clone from Tencent github
- Building with bazel
- Download trained weights

It is easier to use and should work on most linux distributions (has been tested successfully on ubuntu 16.04 LTS for example)

Run the all-in one command below :

```
sudo apt-get -y install pkg-config zip g++ zlib1g-dev unzip python git && git clone https://github.com/Tencent/PhoenixGo.git && cd PhoenixGo && wget https://github.com/bazelbuild/bazel/releases/download/0.11.1/bazel-0.11.1-installer-linux-x86_64.sh && chmod +x bazel-0.11.1-installer-linux-x86_64.sh && ./bazel-0.11.1-installer-linux-x86_64.sh --user && ./configure && bazel build //mcts:mcts_main && wget https://github.com/Tencent/PhoenixGo/releases/download/trained-network-20b-v1/trained-network-20b-v1.tar.gz && tar xvzf trained-network-20b-v1.tar.gz
```

After the building is a success, continue reading at [Distribute mode](https://github.com/Tencent/PhoenixGo##optional--distribute-mode)

#### Possibility B : manual way for more advanced users

##### Building

Clone the repository and configure the building:

```
$ git clone https://github.com/Tencent/PhoenixGo.git
$ cd PhoenixGo
$ ./configure
```

`./configure` will ask where CUDA and TensorRT have been installed, specify them if need.

Then build with bazel:

```
$ bazel build //mcts:mcts_main
```

Dependices such as Tensorflow will be downloaded automatically. The building prosess may take a long time.

##### Running

Download and extract the trained network, then run:

```
$ wget https://github.com/Tencent/PhoenixGo/releases/download/trained-network-20b-v1/trained-network-20b-v1.tar.gz
$ tar xvzf trained-network-20b-v1.tar.gz
$ scripts/start.sh
```

`start.sh` will detect the number of GPUs, run `mcts_main` with proper config file, and write log files in directory `log`.
You could also use a customized config by running `scripts/start.sh {config_path}`.
See also [#configure-guide](#configure-guide).

Furthermore, if you want to fully control all the options of `mcts_main` (such as, changing log destination),
you could also run `bazel-bin/mcts/mcts_main` directly. See also [#command-line-options](#command-line-options).

The engine supports the GTP protocol, means it could be used with a GUI with GTP capability,
such as [Sabaki](http://sabaki.yichuanshen.de). It can also run on command-line GTP server tools like [gtp2ogs](https://github.com/online-go/gtp2ogs). For more details, see [FAQ question](https://github.com/Tencent/PhoenixGo/#8-gtp-command-error--invalid-command)

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

For windows, see [FAQ question syntax error](https://github.com/Tencent/PhoenixGo/#9-syntax-error-windows)

## FAQ

#### 1. I am getting errors during bazel configure, bazel building, and/or running PhoenixGo engine

If you built with bazel, see : [Most common path errors during cuda/cudnn install and bazel configure](https://github.com/Tencent/PhoenixGo/wiki/Install-cuda-and-do-bazel-configuration)

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

#### 7. GTP command `time_settings` doesn't work.

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

#### 8. GTP command error : "invalid command"

Some GTP commands are not supported by PhoenixGo, for example the `showboard` command. Using unsupported commands will most likely make the engine not work. Make sure your GTP tool does not communicate with PhoenixGo with unsupported GTP commands.

For example, for gtp2ogs server command line GTP tool, you need to edit the file gtp2ogs.js and manually remove the `showboard` line if it is not already done, [see](https://github.com/online-go/gtp2ogs/commit/d5ebdbbde259a97c5ae1aed0ec42a07c9fbb2dbf)

#### 9. Syntax error (Windows)

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

#### 10. '"ckpt/zero.ckpt-20b-v1.FP32.PLAN"' error: No such file or directory

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

#### 11. What is the speed of the engine ? How can i make the engine think faster ?

GPU is much faster to compute than CPU (but only nvidia GPU are supported)

TensorRT also increases significantly the speed of computation

Bigger batch size significantly increases the speed of the computation, but a bigger batch size puts a bigger burden on the computation device (in case it is the GPU, higher GPU load, higher VRAM usage), increase it only if your computation device can handle it

Some independent speed benchmarks have been run, they are available in the wiki :

- for GTX 1060 :  [benchmark testing batch size from 4 to 64, tree size up to 2000M, max children up to 512)(https://github.com/Tencent/PhoenixGo/wiki/Benchmark-computation-speed-increase-with-batch-size-tensorrt-gtx1060)


# Benchmark setup :

## setup
- hardware : gtx 1060 6gb (1gpu, power limit set to 75W), ryzen r7 1700, 16gb ram
- software : ubuntu 16.04 LTS, nvidia 384, cuda 9.0, cudnn 7.1.4, tensorrt 3.0.4, bazel 0.11.1
- engine settings: unlimited time per move, all time management settings disabled in config file

## methodology : 
- most moves come from the same game played using gtp2ogs, for few moves moves, copy paste stderr output
- tensorRT is only used with batch size 4, because batch size of 5 and more is not supported (engine fails to run)

## credits :
credit for doing this tests go to [wonderingabout](https://github.com/wonderingabout)

## BATCH SIZE 4 :

batch size 4
tensorrt : OFF
8 threads
children : 64
400M tree size 
4000 sims

```
stderr: 25th move(b): dm, winrate=63.730274%, N=2305, Q=0.274606, p=0.400499, v=0.184802, cost 30390.792969ms, sims=4008, height=22, avg_height=7.976195, global_step=639200
stderr: 27th move(b): dl, winrate=63.091705%, N=4244, Q=0.261834, p=0.566229, v=0.203109, cost 30544.488281ms, sims=4008, height=25, avg_height=8.751671, global_step=639200
```


batch size 4
tensorrt : ON
8 threads
children : 64
400M tree size 
4000 sims

```
stderr: 21th move(b): cc, winrate=51.277012%, N=3998, Q=0.025540, p=0.965418, v=0.010357, cost 26055.263672ms, sims=4008, height=25, avg_height=6.226337, global_step=639200
stderr: 23th move(b): de, winrate=61.373413%, N=2270, Q=0.227468, p=0.697885, v=0.241479, cost 25925.646484ms, sims=4008, height=19, avg_height=5.417722, global_step=639200
```


batch size 4
tensorrt : OFF
8 threads
children : 64
800M tree size 
4000 sims

```
stderr: 9th move(b): qj, winrate=48.316456%, N=1933, Q=-0.033671, p=0.323761, v=-0.057669, cost 30800.728516ms, sims=4007, height=44, avg_height=8.337065, global_step=639200
stderr: 11th move(b): fc, winrate=48.151749%, N=2050, Q=-0.036965, p=0.463283, v=-0.082917, cost 30561.476562ms, sims=4008, height=32, avg_height=11.828917, global_step=639200
```


batch size 4
tensorrt : ON
8 threads
children : 64
800M tree size 
4000 sims

```
stderr: 15th move(b): fq, winrate=50.587959%, N=832, Q=0.011759, p=0.111651, v=-0.014564, cost 25733.691406ms, sims=4008, height=24, avg_height=7.357091, global_step=639200
stderr: 17th move(b): fp, winrate=50.431515%, N=4279, Q=0.008630, p=0.878906, v=-0.032462, cost 26154.666016ms, sims=4008, height=23, avg_height=8.776929, global_step=639200
stderr: 19th move(b): fm, winrate=50.772697%, N=2403, Q=0.015454, p=0.320239, v=-0.025142, cost 26311.226562ms, sims=4008, height=18, avg_height=8.245352, global_step=639200
```


batch size 4
tensorrt : OFF
8 threads
children : 64
2000M tree size 
4000 sims

```
stderr: 13th move(b): hc, winrate=55.519367%, N=1975, Q=0.110387, p=0.293458, v=0.101614, cost 33722.300781ms, sims=4008, height=23, avg_height=8.870925, global_step=639200
stderr: 17th move(b): ce, winrate=73.472389%, N=2986, Q=0.469448, p=0.264501, v=0.330677, cost 33759.488281ms, sims=4008, height=46, avg_height=12.595410, global_step=639200
```



batch size 4
tensorrt : ON
8 threads
children : 64
2000M tree size 
4000 sims

```
stderr: 19th move(b): db, winrate=75.447426%, N=2739, Q=0.508949, p=0.501109, v=0.503591, cost 27647.033203ms, sims=4008, height=25, avg_height=5.634705, global_step=639200
stderr: 20th move(w): cb, winrate=-nan%, N=0, Q=-nan, p=0.000000, v=nan, cost 3434.220703ms, sims=524, height=20, avg_height=7.251240, global_step=639200
```


batch size 4
tensorrt : OFF
8 threads
children : 512
400M tree size 
4000 sims

```
stderr: 23th move(b): cq, winrate=88.760635%, N=2396, Q=0.775213, p=0.193044, v=0.747791, cost 30913.226562ms, sims=4008, height=57, avg_height=14.380897, global_step=639200
stderr: 25th move(b): dq, winrate=86.716339%, N=3968, Q=0.734327, p=0.901415, v=0.808912, cost 31271.306641ms, sims=4007, height=43, avg_height=21.491110, global_step=639200
```


batch size 4
tensorrt : ON
8 threads
children : 512
400M tree size 
4000 sims

```
stderr: 9th move(b): cf, winrate=46.220604%, N=2333, Q=-0.075588, p=0.419347, v=-0.101920, cost 25676.927734ms, sims=4008, height=36, avg_height=6.813369, global_step=639200
stderr: 11th move(b): pj, winrate=47.592823%, N=1813, Q=-0.048144, p=0.472589, v=-0.036348, cost 26014.980469ms, sims=4008, height=22, avg_height=6.213001, global_step=639200
```


batch size 4
tensorrt : OFF
8 threads
children : 512
2000M tree size 
4000 sims

```
stderr: 1th move(b): dd, winrate=44.132729%, N=349, Q=-0.117345, p=0.080226, v=-0.120119, cost 31625.798828ms, sims=4008, height=10, avg_height=5.106929, global_step=639200
stderr: 3th move(b): qd, winrate=44.120907%, N=759, Q=-0.117582, p=0.163193, v=-0.116186, cost 31196.808594ms, sims=4008, height=24, avg_height=8.116901, global_step=639200
```


batch size 4
tensorrt : ON
8 threads
children : 512
2000M tree size 
4000 sims

```
stderr: 5th move(b): qn, winrate=44.146938%, N=760, Q=-0.117061, p=0.164774, v=-0.116266, cost 26392.984375ms, sims=4008, height=31, avg_height=10.278131, global_step=639200
stderr: 7th move(b): nd, winrate=45.115017%, N=1026, Q=-0.097700, p=0.179594, v=-0.126492, cost 26259.271484ms, sims=4008, height=44, avg_height=12.790710, global_step=639200
```

## BATCH SIZE 8 :

batch size 8
tensorrt : OFF
16 threads
children : 64
400M tree size
4000 sims

```
stderr: 29th move(b): dj, winrate=67.039803%, N=3292, Q=0.340796, p=0.809432, v=0.419116, cost 27238.894531ms, sims=4016, height=23, avg_height=3.907765, global_step=639200
stderr: 31th move(b): ek, winrate=69.330391%, N=4033, Q=0.386608, p=0.956573, v=0.415518, cost 27617.912109ms, sims=4016, height=21, avg_height=9.358269, global_step=639200
```


## BATCH SIZE 16 :

batch size 16
tensorrt : OFF
24 threads
children : 64
400M tree size
4000 sims

```
stderr: 35th move(b): mp, winrate=82.146217%, N=1159, Q=0.642924, p=0.216112, v=0.637870, cost 17744.708984ms, sims=4021, height=18, avg_height=6.282155, global_step=639200
stderr: 3th move(b): cd, winrate=44.138699%, N=780, Q=-0.117226, p=0.158862, v=-0.126368, cost 19640.619141ms, sims=4024, height=25, avg_height=8.314819, global_step=639200
stderr: 5th move(b): cn, winrate=44.210926%, N=884, Q=-0.115781, p=0.150587, v=-0.136205, cost 19437.255859ms, sims=4020, height=32, avg_height=11.667562, global_step=639200
stderr: 11th move(b): cp, winrate=50.521744%, N=3180, Q=0.010435, p=0.558747, v=-0.024745, cost 20502.611328ms, sims=4020, height=29, avg_height=4.975909, global_step=639200
```


batch size 16
tensorrt : OFF
32 threads
children : 64
400M tree size
4000 sims

```
stderr: 9th move(b): cf, winrate=46.273342%, N=2667, Q=-0.074533, p=0.512861, v=-0.077746, cost 19605.994141ms, sims=4032, height=32, avg_height=4.803508, global_step=639200
stderr: 11th move(b): ec, winrate=47.041210%, N=3350, Q=-0.059176, p=0.739531, v=-0.068733, cost 19384.156250ms, sims=4032, height=27, avg_height=9.472223, global_step=639200
stderr: 13th move(b): db, winrate=51.065022%, N=3975, Q=0.021300, p=0.933134, v=-0.017093, cost 19393.525391ms, sims=4032, height=25, avg_height=11.328493, global_step=639200
stderr: 19th move(b): ef, winrate=81.741035%, N=828, Q=0.634821, p=0.166898, v=0.609234, cost 19537.847656ms, sims=4032, height=22, avg_height=4.672567, global_step=639200
```


batch size 16
tensorrt : OFF
32 threads
children : 64
2000M tree size
4000 sims

```
stderr: 21th move(b): be, winrate=71.451530%, N=3672, Q=0.429031, p=0.499344, v=0.430739, cost 17760.664062ms, sims=4032, height=20, avg_height=3.586761, global_step=639200
stderr: 23th move(b): dg, winrate=74.842880%, N=3231, Q=0.496858, p=0.476955, v=0.489479, cost 17722.525391ms, sims=4032, height=22, avg_height=10.813924, global_step=639200
stderr: 25th move(b): eh, winrate=74.921097%, N=5719, Q=0.498422, p=0.960995, v=0.484069, cost 17674.404297ms, sims=4032, height=22, avg_height=11.018259, global_step=639200
stderr: 1th move(b): dd, winrate=44.089737%, N=361, Q=-0.118205, p=0.079234, v=-0.114998, cost 19507.214844ms, sims=4032, height=10, avg_height=5.267999, global_step=639200
stderr: 3th move(b): qd, winrate=44.138203%, N=807, Q=-0.117236, p=0.167434, v=-0.118984, cost 19442.664062ms, sims=4032, height=25, avg_height=8.325870, global_step=639200
stderr: 5th move(b): qn, winrate=44.223949%, N=929, Q=-0.115521, p=0.156585, v=-0.130661, cost 19547.380859ms, sims=4032, height=32, avg_height=11.702785, global_step=639200
```


batch size 16
tensorrt : OFF
32 threads
children : 192
2000M tree size
4000 sims

```
stderr: 17th move(b): je, winrate=52.257484%, N=2176, Q=0.045150, p=0.606293, v=0.074896, cost 17796.113281ms, sims=4032, height=21, avg_height=3.128364, global_step=639200
stderr: 19th move(b): df, winrate=66.187737%, N=2995, Q=0.323755, p=0.720188, v=0.383549, cost 17827.675781ms, sims=4032, height=22, avg_height=2.973594, global_step=639200
stderr: 25th move(b): fg, winrate=87.648285%, N=3418, Q=0.752966, p=0.333620, v=0.725144, cost 19522.392578ms, sims=4032, height=22, avg_height=8.429089, global_step=639200
```


batch size 16
tensorrt : OFF
32 threads
children : 512
2000M tree size
4000 sims

```
stderr: 13th move(b): jc, winrate=50.131180%, N=3161, Q=0.002624, p=0.645318, v=0.021434, cost 17991.837891ms, sims=4032, height=21, avg_height=3.247312, global_step=639200
stderr: 15th move(b): ec, winrate=53.321224%, N=3895, Q=0.066424, p=0.904836, v=0.107396, cost 17907.687500ms, sims=4032, height=22, avg_height=4.943458, global_step=639200
stderr: 23th move(b): df, winrate=87.167152%, N=3351, Q=0.743343, p=0.684157, v=0.739293, cost 19460.703125ms, sims=4029, height=20, avg_height=3.316754, global_step=639200
```

## BATCH SIZE 24 :

batch size 24
tensorrt : OFF
32 threads
children : 64
400M tree size
4000 sims

```
stderr: 21th move(b): mp, winrate=52.999252%, N=5199, Q=0.059985, p=0.369109, v=0.111925, cost 20766.712891ms, sims=4032, height=23, avg_height=9.521194, global_step=639200
stderr: 15th move(b): dc, winrate=52.322102%, N=4015, Q=0.046442, p=0.979087, v=0.014373, cost 20751.615234ms, sims=4030, height=26, avg_height=5.288996, global_step=639200
stderr: 17th move(b): jc, winrate=63.067364%, N=3402, Q=0.261347, p=0.746740, v=0.267917, cost 20870.205078ms, sims=4047, height=21, avg_height=3.669026, global_step=639200
```


batch size 24
tensorrt : OFF
48 threads
children : 64
400M tree size
4000 sims

```
stderr: 25th move(b): kn, winrate=61.381878%, N=1679, Q=0.227638, p=0.331912, v=0.194674, cost 20769.664062ms, sims=4046, height=17, avg_height=2.412944, global_step=639200
stderr: 27th move(b): lo, winrate=64.118179%, N=1633, Q=0.282364, p=0.279228, v=0.274228, cost 20648.003906ms, sims=4048, height=17, avg_height=6.799204, global_step=639200
```


## BATCH SIZE 32 : 

batch size 32
tensorrt : OFF
32 threads
children : 64
400M tree size
4000 sims

```
stderr: 13th move(b): qq, winrate=57.209267%, N=2005, Q=0.144185, p=0.405177, v=0.122235, cost 17227.105469ms, sims=4030, height=34, avg_height=6.547671, global_step=639200
stderr: 17th move(b): rn, winrate=58.125191%, N=3115, Q=0.162504, p=0.632014, v=0.188797, cost 16386.800781ms, sims=4032, height=28, avg_height=13.034529, global_step=639200
```


batch size 32
tensorrt : OFF
48 threads
children : 64
400M tree size
4000 sims

```
stderr: 31th move(b): nn, winrate=86.338608%, N=2997, Q=0.726772, p=0.367060, v=0.754056, cost 17249.060547ms, sims=4048, height=17, avg_height=6.970535, global_step=639200
stderr: 33th move(b): oo, winrate=86.423180%, N=2203, Q=0.728464, p=0.472367, v=0.673604, cost 17288.750000ms, sims=4048, height=24, avg_height=10.562322, global_step=639200
stderr: 35th move(b): om, winrate=86.368988%, N=3464, Q=0.727380, p=0.532638, v=0.804181, cost 17485.921875ms, sims=4046, height=24, avg_height=10.913864, global_step=639200
stderr: 1th move(b): dp, winrate=44.136177%, N=358, Q=-0.117276, p=0.080226, v=-0.114998, cost 17265.882812ms, sims=4048, height=10, avg_height=5.229761, global_step=639200
stderr: 3th move(b): dc, winrate=44.124222%, N=799, Q=-0.117516, p=0.164091, v=-0.116186, cost 15783.393555ms, sims=4048, height=25, avg_height=8.300819, global_step=639200
stderr: 5th move(b): nc, winrate=44.210690%, N=889, Q=-0.115786, p=0.150895, v=-0.126572, cost 15871.321289ms, sims=4048, height=33, avg_height=11.712460, global_step=639200
stderr: 7th move(b): df, winrate=45.026649%, N=956, Q=-0.099467, p=0.173773, v=-0.115591, cost 16095.766602ms, sims=4046, height=41, avg_height=17.676241, global_step=639200
stderr: 9th move(b): jc, winrate=47.230457%, N=1498, Q=-0.055391, p=0.235233, v=-0.090901, cost 16061.685547ms, sims=4047, height=40, avg_height=9.619899, global_step=639200
```


batch size 32
tensorrt : OFF
64 threads
children : 64
400M tree size
4000 sims

```
stderr: 21th move(b): od, winrate=75.232864%, N=1357, Q=0.504657, p=0.258191, v=0.459986, cost 15832.726562ms, sims=4064, height=22, avg_height=6.120484, global_step=639200
stderr: 23th move(b): oe, winrate=77.275383%, N=5187, Q=0.545508, p=0.983199, v=0.479445, cost 15976.324219ms, sims=4060, height=50, avg_height=24.729546, global_step=639200
```

## BATCH SIZE 48 : 

batch size 48
tensorrt : OFF
64 threads
children : 64
400M tree size
4000 sims

```
stderr: 29th move(b): qd, winrate=95.835518%, N=4095, Q=0.916710, p=0.985609, v=0.935179, cost 15217.651367ms, sims=4062, height=14, avg_height=6.751593, global_step=639200
```

## BATCH SIZE 64 : 

batch size 64
tensorrt : OFF
80 threads
children : 64
400M tree size
4000 sims

```
stderr: 31th move(b): rc, winrate=96.647415%, N=241, Q=0.932948, p=0.078281, v=0.954536, cost 15016.847656ms, sims=4075, height=12, avg_height=2.383732, global_step=639200
stderr: 33th move(b): pg, winrate=97.439018%, N=288, Q=0.948780, p=0.087517, v=0.940238, cost 14916.583008ms, sims=4080, height=13, avg_height=4.588904, global_step=639200
stderr: 1th move(b): dd, winrate=44.123699%, N=346, Q=-0.117526, p=0.080226, v=-0.114998, cost 16223.052734ms, sims=4079, height=10, avg_height=2.058298, global_step=639200
stderr: 9th move(b): cf, winrate=49.737022%, N=2402, Q=-0.005260, p=0.512962, v=-0.009907, cost 16592.304688ms, sims=4079, height=31, avg_height=4.388403, global_step=639200
```
# PERCENTAGES EXPLANATION : 

The speed gain number below is the time less needed to calculate a move, for example speed gain +12% = 12% less time to calculate a move as compared to batch size 4 = 27.5 seconds vs 30.5 seconds per move = 4 seconds difference out of 30.5 seconds 

# CONCLUSIONS for GTX 1060 : 

- TensorRT can increase speed by arround 15%-20% on a GTX 1060 with batch size 4
- bigger batch size significantly increases speed of the engine on a GTX 1060 :
     -> for batch size 8 , gain = +12%
     -> for batch size 16 , gain = +33%
     -> for batch size 24 , gain = +31%
     -> for batch size 32 , gain = +47%
- Compute device (GPU or CPU) utilization is higher with higher batching
- gtx 1060 is too weak to benefit higher batch sizes than 32
- number of threads does not significantly change speed of the engine
- tree size does not significantly change the speed of the engine (arround 5% more time with max tree size)

# TO DO : 
- i will try higher batch sizes with a Tesla V100 on windows 10

