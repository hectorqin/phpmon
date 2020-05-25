# phpmon

🚀 Hyperf Watch Hot Reload Scripts

😊 Just like nodemon

👉 监听文件变化自动重启Hyperf以及其它进程. 基于Swoole的Process/Timer/Event实现，定时扫描文件并监听文件变动重启服务

Author: hectorqin@163.com

Tips: 建议PHP>=7.2 && Swoole>=4.4，php.ini需要开启exec方法.只建议在开发环境中使用，如果对您有帮助，请给项目一个Star，谢谢！

## 下载安装

【Windows】

下载完成后请拷贝到项目根目录上：

[下载地址->请右键另存为，并删去扩展名.txt](https://raw.githubusercontent.com/hectorqin/phpmon/master/phpmon)

【MacOS or Linux】

在项目根目录下启动终端控制台：

```sh
wget -O phpmon https://raw.githubusercontent.com/hectorqin/phpmon/master/phpmon
```

## Hyperf 使用说明

### 启动监听

```sh
php phpmon
```

### 启动监听并删除代理类缓存(./runtime/container)

```sh
php phpmon -c
```

### 退出监听

使用 `Control + C` 终止即可

### 手动重启

输入rs回车即可

## 其它进程使用说明

### 启动PHP内置web server，并监控php文件

```sh
php phpmon -e php -- -S 0.0.0.0:8080
```

### 启动Python内置web server

```sh
# Python2
php phpmon -n -b `which python` -- -m SimpleHTTPServer 8080
# Python3
php phpmon -n -b `which python` -- -m http.server 8080
```

### 启动nodejs进程，并监控js,json等文件

```sh
php phpmon -n -e js,json -b `which node` -- index.js
```

## 命令帮助

```sh
$ php phpmon -h
Usage: php phpmon [options]

Options:
-n, --nohyperf  ........... not start hyperf server, then pid file and watch
                            extensions are empty.
-w, --watch=dir ........... watch directory "dir" or files. use once for
                            each directory or file to watch.
-d, --delay=second ........ restart delay seconds after file changed.
-e, --ext=extension ....... extensions to look for, ie. php,env.
-b, --bin=path ............ executable script or binary file.
-i, --ignore=path ......... ignore specific files or directories, default
                            is ["vendor"].
-s, --script=file ......... script file, default is "./bin/hyperf.php".
-p, --pid=file ............ pid file, will be used to kill master process,
                            default is "./runtime/hyperf.pid".
-k  --kill=match .......... force kill processes that match this partern
                            before start.
-c  --clear ............... for hyperf only, to delete runtime/container
                            folder before start, default false.

-V, --verbose ............. show detail on what is causing restarts.
-h  --help ................ print usage.

-- <your args> ............ to tell phpmon stop slurping arguments, and
                            pass the left args to your binary. Default
                            is ["script file", "start"]

Examples:

- Exec php ./bin/hyperf start, watch current folder
php phpmon

- Exec php ./bin/hyperf start, watch current folder, and use the specified PHP
php phpmon -b /usr/local/bin/php

- Exec php ./bin/hyperf start, watch app,config folders
php phpmon -w app --watch config

- Exec php ./bin/hyperf start, watch app,config folders, and restart delay
    3 seconds after file change.
php phpmon -w app --watch config -d 3

- Exec php ./bin/hyperf start, watch app,config folders, and restart delay
    3 seconds after file change, and remove runtime/container folder.
php phpmon -w app --watch config -d 3 -c

- Exec php ./bin/hyperf start, watch app,config folders, and restart delay
    3 seconds after file change, and remove runtime/container folder, and set the entry script.
php phpmon -w app --watch config -d 3 -c -s hyperf

- Exec php ./bin/hyperf start, watch app,config folders, and restart delay
    3 seconds after file change, and remove runtime/container folder, and set the entry script,
    and ignore some file.
php phpmon -w app --watch config -d 3 -c -s hyperf -i config/config.php --ignore config/routes.php

- Exec other scripts
php phpmon -n -b /usr/local/bin/node -- server.js
php phpmon -n -b /usr/local/bin/python -- -m http.server 19980
php phpmon -n -- -s 0.0.0.0:19980
```

## 默认配置

```php
$isHyperf = !(isset($options['n']) ?: isset($options['nohyperf']) ?: false);
$entryPointFile = $options['s'] ?? $options['script'] ?? ($isHyperf ? './bin/hyperf.php' : '');
$config         = [
    // BINARY File 程序所在路径（默认自动获取 PHP 程序路径）
    'BINARY_FILE'        => $options['b'] ?? $options['bin'] ?? 'which php',
    // Watch Dir 监听目录（默认监听脚本所在的根目录）
    'WATCH_DIR'          => (array) (array_merge($options['w'] ?? [], $options['watch'] ?? []) ?: [__DIR__ . '/']),
    // Watch Ext 监听扩展名（多个可用英文逗号隔开）
    'WATCH_EXT'          => $options['e'] ?? $options['ext'] ?? ($isHyperf ? 'php,env' : ''),
    // Exclude Dir 排除目录（不监听的目录，数组形式)
    'EXCLUDE_DIR'        => (array) (array_merge($options['i'] ?? [], $options['ignore'] ?? []) ?: ['vendor']),
    // Kill Match Partern 杀死进程的匹配字符串
    'KILL_MATCH_PARTERN' => $options['k'] ?? $options['kill'] ?? '',
    // Start Command 启动命令
    'START_COMMAND'      => $startCommand ?: ($entryPointFile ? [$entryPointFile, 'start'] : ['start']),
    // PID File Path PID文件路径
    'PID_FILE_PATH'      => $options['p'] ?? $options['pid'] ?? ($isHyperf ? './runtime/hyperf.pid' : ''),
    // Scan Interval 扫描间隔（毫秒，默认2000）
    'SCAN_INTERVAL'      => 2000,
    // Restart Delay 重启延迟
    'RESTART_DELAY'      => $options['d'] ?? $options['delay'] ?? 0,
    // Clear 是否清除 runtime
    'CLEAR'              => isset($options['c']) ?: isset($options['clear']) ?: false,
    // Verbose 显示变化文件
    'VERBOSE'            => isset($options['v']) ?: isset($options['verbose']) ?: false,
];
```
