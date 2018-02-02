---
title: commander 包使用说明
tags: ['node']
categories : Tools
---

## commander 包使用说明


### 选项解析

`commander`的选项是用`.option()`方法定义的，也可以作为选项的文档。下面的例子从` process.argv`中解析参数和选项，把剩下的参数作为没有被选项占用的`program.args`数组;

```js

    #!/usr/bin/env node
 
    /**
     * Module dependencies.
     */
    
    var program = require('commander');
    
    program
    .version('0.1.0')
    .option('-p, --peppers', 'Add peppers')
    .option('-P, --pineapple', 'Add pineapple')
    .option('-b, --bbq-sauce', 'Add bbq sauce')
    .option('-c, --cheese [type]', 'Add the specified type of cheese [marble]', 'marble')
    .parse(process.argv);
    
    console.log('you ordered a pizza with:');
    if (program.peppers) console.log('  - peppers');
    if (program.pineapple) console.log('  - pineapple');
    if (program.bbqSauce) console.log('  - bbq');
    console.log('  - %s cheese', program.cheese);

```
<!-- more -->

短标志可以作为单个参数传递，例如`-abc`相当于`-a -b -c`。Multi-word选项如`--template-engine`是骆驼式的，变成`program.templateengine`等；请注意，以`--no`前缀开头的Multi-word选项将其后的参数布尔值设置为`false`;

```js

    #!/usr/bin/env node
    /**
     * Module dependencies.
     */

    var program = require('commander');
    program
    .option('--no-sauce', 'Remove sauce')  // program.sauce 的布尔值为 false
    .parse(process.argv);
    
    console.log('you ordered a pizza');
    if (program.sauce) console.log('  with sauce');
    else console.log(' without sauce');

```

###  控制

```js

    function range(val) {
        return val.split('..').map(Number);
    }
    
    function list(val) {
        return val.split(',');
    }
    
    function collect(val, memo) {
        memo.push(val);
        return memo;
    }
    
    function increaseVerbosity(v, total) {
        return total + 1;
    }
    
    program
    .version('0.1.0')
    .usage('[options] <file ...>')
    .option('-i, --integer <n>', 'An integer argument', parseInt)
    .option('-f, --float <n>', 'A float argument', parseFloat)
    .option('-r, --range <a>..<b>', 'A range', range)
    .option('-l, --list <items>', 'A list', list)
    .option('-o, --optional [value]', 'An optional value')
    .option('-c, --collect [value]', 'A repeatable value', collect, [])
    .option('-v, --verbose', 'A value that can be increased', increaseVerbosity, 0)
    .parse(process.argv);
    
    console.log(' int: %j', program.integer);
    console.log(' float: %j', program.float);
    console.log(' optional: %j', program.optional);
    program.range = program.range || [];
    console.log(' range: %j..%j', program.range[0], program.range[1]);
    console.log(' list: %j', program.list);
    console.log(' collect: %j', program.collect);
    console.log(' verbosity: %j', program.verbose);
    console.log(' args: %j', program.args);

```

### 正则表达式

```js

    program
    .version('0.1.0')
    .option('-s --size <size>', 'Pizza size', /^(large|medium|small)$/i, 'medium')
    .option('-d --drink [drink]', 'Drink', /^(coke|pepsi|izze)$/i)
    .parse(process.argv);
    
    console.log(' size: %j', program.size);
    console.log(' drink: %j', program.drink);

```
### 可变参数

一个命令的最后一个参数可以是可变的，只有最后一个参数;使一个参数为可变参数，你必须在参数名称之后追加`...`；example：


```js
    #!/usr/bin/env node
    /**
     * Module dependencies.
     */
    
    var program = require('commander');
    
    program
    .version('0.1.0')
    .command('rmdir <dir> [otherDirs...]')
    .action(function (dir, otherDirs) {
        console.log('rmdir %s', dir);
        if (otherDirs) {
            otherDirs.forEach(function (oDir) {
                console.log('rmdir %s', oDir);
            });
        }
    });
    
    program.parse(process.argv);

```
一个数组存储一个可变参数的值，这适用于`program.args`以及如上所示传递给`action`的参数。

###  指定参数语法

```js

    #!/usr/bin/env node
 
    var program = require('commander');
    
    program
    .version('0.1.0')
    .arguments('<cmd> [env]')
    .action(function (cmd, env) {
        cmdValue = cmd;
        envValue = env;
    });
    
    program.parse(process.argv);
    
    if (typeof cmdValue === 'undefined') {
    console.error('no command given!');
    process.exit(1);
    }
    console.log('command:', cmdValue);
    console.log('environment:', envValue || "no environment given");

```
尖括号(`e.g. <cmd>`)显示需要输入,方括号(`e.g. [env]`)显示可选的输入。


###  git风格的子命令模式

```js

    // file: ./examples/pm
    var program = require('commander');
    
    program
    .version('0.1.0')
    .command('install [name]', 'install one or more packages')
    .command('search [query]', 'search with optional query')
    .command('list', 'list packages installed', {isDefault: true})
    .parse(process.argv);

```
当使用描述参数调用`.command()`时，应该调用 `.action(callback)`来处理子命令，否则会出现错误;这告诉commander，你将要为子命令使用单独的可执行文件，很像`git(1)`和其他流行的工具。

commander将尝试使用名称`program-command`（如`pm-install`，`pm-search`）在输入的脚本（如`./examples/pm`）目录中搜索可执行文件。

选项可以通过调用`.command()`来传递. 为`opts.nohelp`指定`true`会从生成的帮助输出中删除该选项.  如果没有指定其他子命令，则为将运行`opts.isdefault`指定为`true`的子命令。

如果该程序被设计为全局安装, 确保可执行文件具有合适的模式(执行权限)，如 755。


###  --harmony

您可以通过两种方式启用`--harmony`选项：

-  在子命令脚本中使用 `#! /usr/bin/env node --harmony`. 注意一些os版本不支持这种模式.
-  调用命令时使用`--harmony`选项,如执行 `node --harmony examples/pm publish` ;在产生子命令的过程中，`--harmony`选项将被保留。

###  自动生成的帮助说明 --help

帮助信息是基于`information commander`已经知道的程序说明自动生成的，所以输入`--help`得到的信息是不需要自己设定的：

```js

    ./examples/pizza --help

    Usage: pizza [options]

    An application for pizzas ordering

    Options:

        -h, --help           output usage information
        -V, --version        output the version number
        -p, --peppers        Add peppers
        -P, --pineapple      Add pineapple
        -b, --bbq            Add bbq sauce
        -c, --cheese <type>  Add the specified type of cheese [marble]
        -C, --no-cheese      You do not want any cheese

```

###  定制的帮助说明

您可以通过监听`--help`来显示任意`-h`，`--help`信息。一旦你这么做了，Commander 将自动退出，这样你的程序的其余部分不会执行导致后面代码错误；例如在下面的可执行的代码里，`stuff`在使用`--help`时不会输出。

```js

    #!/usr/bin/env node
 
    /**
     * Module dependencies.
     */
    
    var program = require('commander');
    
    program
    .version('0.1.0')
    .option('-f, --foo', 'enable some foo')
    .option('-b, --bar', 'enable some bar')
    .option('-B, --baz', 'enable some baz');
    
    // must be before .parse() since
    // node's emit() is immediate
    
    program.on('--help', function(){
    console.log('  Examples:');
    console.log('');
    console.log('    $ custom-help --help');
    console.log('    $ custom-help -h');
    console.log('');
    });
    
    program.parse(process.argv);
    
    console.log('stuff');

```

当运行`node script-name.js -h`或`node script-name.js --help`时，将生成以下帮助输出:

```js

    Usage: custom-help [options]

    Options:

        -h, --help     output usage information
        -V, --version  output the version number
        -f, --foo      enable some foo
        -b, --bar      enable some bar
        -B, --baz      enable some baz

    Examples:

        $ custom-help --help
        $ custom-help -h

```

###  .outputHelp(cb)

输出帮助信息而不退出；可选回调函数`cb`允许在显示帮助文本之前对其进行后处理。

如果你想默认显示帮助（例如，如果没有提供命令），你可以使用类似于：

```js

    var program = require('commander');
    var colors = require('colors');
    
    program
        .version('0.1.0')
        .command('getstream [url]', 'get stream URL')
        .parse(process.argv);
 
    if (!process.argv.slice(2).length) {
        program.outputHelp(make_red);
    }
    
    function make_red(txt) {
        return colors.red(txt); //display the help text in red on the console
    }


```

###  .help(cb)

输出帮助信息并立即退出。可选回调函数`cb`允许在显示帮助文本之前对其进行后处理。


### Examples

```js
    var program = require('commander');
 
    program
    .version('0.1.0')
    .option('-C, --chdir <path>', 'change the working directory')
    .option('-c, --config <path>', 'set config path. defaults to ./deploy.conf')
    .option('-T, --no-tests', 'ignore test hook');
    
    program
    .command('setup [env]')
    .description('run setup commands for all envs')
    .option("-s, --setup_mode [mode]", "Which setup mode to use")
    .action(function(env, options){
        var mode = options.setup_mode || "normal";
        env = env || 'all';
        console.log('setup for %s env(s) with %s mode', env, mode);
    });
    
    program
    .command('exec <cmd>')
    .alias('ex')
    .description('execute the given remote cmd')
    .option("-e, --exec_mode <mode>", "Which exec mode to use")
    .action(function(cmd, options){
        console.log('exec "%s" using %s mode', cmd, options.exec_mode);
    }).on('--help', function() {
        console.log('  Examples:');
        console.log();
        console.log('    $ deploy exec sequential');
        console.log('    $ deploy exec async');
        console.log();
    });
    
    program
    .command('*')
    .action(function(env){
        console.log('deploying "%s"', env);
    });
    
    program.parse(process.argv);

```
