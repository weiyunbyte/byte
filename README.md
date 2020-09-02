# [vue](http://weiyunbyte.com)
###vue移动端UI框架
* [Vant UI](https://github.com/youzan/vant.git) 有赞开发的轻量、可靠的小程序 UI 组件库
* [Nut UI](https://github.com/jdf2e/nutui) 京东开发的一套移动端轻量级组件库
* [Cube UI](https://github.com/didi/cube-ui) 滴滴开发的Vue提供的出色的移动ui lib工具
* [Mint UI](https://mint-ui.github.io/#!/zh-cn) 饿了么开源的移动端UI组件库（很久没更新了）
###letsencrypt利用acme.sh生成通配符的https证书
* 安装:  
```
curl  https://get.acme.sh | sh
```
* 并创建 一个 bash 的 alias, 方便你的使用: 
```
alias acme.sh=~/.acme.sh/acme.sh
```
* 生成证书:（阿里云服务器）
```
acme.sh  --issue -d yoururl.com  -d '*.yoururl.com'  --dns dns_ali
```
* copy/安装 证书：  
```
acme.sh --install-cert -d yoururl.com -d *.yoururl.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```
* 自动升级acme.sh:  
```
acme.sh  --upgrade  --auto-upgrade
```
* 修改nginx 443配置
###自己的脚手架工具
* 创建脚手架项目文件ceshi-cli，再创建一个index.js文件
```
#!/usr/bin/env node
function start() {
	console.log('hello world')
}
start()
```
* 通过命令 npm init -y初始化package.json文件，配置bin选项  
-y 表示接受npm的一些默认参数
```
"bin": {
    "ceshi": "index.js"
  }
```
* 使用commander.js解析命令行参数
安装：
```
npm install commander
```
再index.js中引入
```
const {program} = require('commander')
```
调用program的一些api  
版本号：
```
program.version(require('./package.json').version) // 输出版对应的版本号
program
    .command('create <projectName>')
    .description('用于创建一个项目模板')
    .option("-T, --template [template]", "输入使用的模板名字")
    .action(function(projectName, options){
        let template = options.template || "vue-default-template";
        projectName = projectName || 'untitled';
        console.log(`成功创建项目：${projectName}`)
        console.log(`所使用的模板：${template}`);
    });
// 打印所以模板
program
    .command('checkAll')
    .description('查看所有的模板')
    .action(function(){
        const templateList = [
            'vue-default-template',
            'vue-default-template-ts'
        ]
        templateList.forEach((temp,index) => {
            console.log(`(${index+1})  ${temp}`)
        })
    })
program.parse(process.argv);
```
命令行中输入：
```
ceshi create demoProject -T vue-default-template
ceshi checkAll
```
* 使用inquirer.js设计命令行交互  
安装：  
```
npm install inquirer
```
我们创建一个inquirer.js文件，来写交互的逻辑  
index.js文件  
```
program
    .command('create <projectName>')
    .description('用于创建一个项目模板')
    .option("-T, --template [template]", "输入使用的模板名字")
    .action(async function(projectName, options){
        let template = options.template;
        projectName = projectName || 'untitled';

        if(!template){
            template = await chooseTemplate() // 注意这里是一个异步方法
        }


        console.log(`成功创建项目：${projectName}`)
        console.log(`所使用的模板：${template}`);
    });
```
inquirer.js文件
```
const inquirer = require('inquirer')

async function chooseTemplate(){
    const promptList = [
        {
            type: "list", // type决定交互的方式，比如当值为input的时候就是输入的形式，list就是单选，checkbox是多选...
            name: "template",
            message: "选择一个需要创建的工程化模板",
            choices: [
                {
                    name: "vue-default (js版本的vue全家桶工程化模板)",
                    value: "vue-template-default",
                }
            ],
        },
    ];
    const answers = await inquirer.prompt(promptList);  // 执行命令行交互，并将交互的结果返回
    const {template} = answers
    console.log(`你选择的模板是：${template}`)
    return template  // 返回我们选择的模板
}

module.exports = {
    chooseTemplate
}
```
命令行执行:  
```
ceshi create demoProject
```
* 创建工程化模板push到github
```
git init
git add .
git commit -m "first commit"
git branch -M master
git remote add origin https://github.com/ceshi-cli/ceshi.git
git push -u origin master
```
* 使用download-git-repo下载我的工程模板
我们先做模板名字的映射   
创建templateMap.js
```
const templateMap = new Map()

templateMap.set('vue-template-default',"https://github.com:ceshi/vue-template-default#master")

module.exports = templateMap
```
需要注意的是这里downloadUrl的格式，可以理解为仓库地址 + #分支名，但需要将仓库地址中https://github.com后面的 '/ '换成 ':'

否则你就会发现一直报128的错误  
安装：  
```
npm install download-git-repo
```
引入使用：  
```
const download = require('download-git-repo')
const templateMap = require('./templateMap')
program
    .command('create <projectName>')
    .description('用于创建一个项目模板')
    .option("-T, --template [template]", "输入使用的模板名字")
    .action(async function(projectName, options){
        let template = options.template;
        projectName = projectName || 'untitled';

        if(!template){
            template = await chooseTemplate() // 注意这里是一个异步方法
        }

        const downloadUrl = templateMap.get(template) // templateMap是一个引入的自定义Map

        download(downloadUrl, projectName,{clone: true} , error => {
            if(error){
                console.log(`创建项目失败：${projectName}`)
                console.log('失败原因：',error)
            }else {
                console.log(`成功创建项目：${projectName}`)
            }
        })

    });
```
命令行里执行：  
```
ceshi create demoProject
```
到此就完成了定制工程化项目功能  
* 美化命令行
使用ora美化loading效果  
安装：  
```
npm install ora
// 下载前提示loading
    const spinner = ora({
        text: '正在下载模板...',
        color: "yellow",
        spinner: {
            interval: 80,
            frames: ["⠋", "⠙", "⠹", "⠸", "⠼", "⠴", "⠦", "⠧", "⠇", "⠏"],
        },
    });
    spinner.start();
    const downloadUrl = templateMap.get(template)
        download(downloadUrl, projectName,{clone: true} , error => {
            if(error){
                spinner.fail(`创建项目失败：${projectName}`)
                console.log('失败原因：',error.message)
            }else {
                spinner.succeed(`成功创建项目：${projectName}`)
            }
        })
```
使用chalk改变命令行颜色  
安装：  
```
npm install chalk
const chalk = require('chalk');
console.log(chalk.rgb(216, 27, 96)('hello world'))
console.log(chalk.cyanBright('ceshi-cli命令行工具...\n'))
```
最后：  
```
#! /usr/bin/env node

const { program } = require('commander') // 引入
const download = require('download-git-repo')
const templateMap = require('./templateMap')
const ora = require('ora')
const chalk = require('chalk');


const {chooseTemplate} = require('./inquirers')

 function start() {
    console.log(chalk.rgb(216, 27, 96)('hello world'))
    console.log(chalk.cyanBright('ceshi-cli命令行工具...\n'))

    program.version(require('./package.json').version) // 输出版对应的版本号

    program
        .command('create <projectName>')
        .description('用于创建一个项目模板')
        .option("-T, --template [template]", "输入使用的模板名字")
        .action(async function(projectName, options){
            let template = options.template;
            projectName = projectName || 'untitled';

            if(!template){
                template = await chooseTemplate() // 注意这里是一个异步方法
            }

            console.log(chalk.rgb(69, 39, 160)('你选择的模板是 👉'),chalk.bgRgb(69, 39, 160)(template))

            // 下载前提示loading
            const spinner = ora({
                text: '正在下载模板...',
                color: "yellow",
                spinner: {
                    interval: 80,
                    frames: ["⠋", "⠙", "⠹", "⠸", "⠼", "⠴", "⠦", "⠧", "⠇", "⠏"],
                },
            });
            spinner.start();


            /**
             * @downloadUrl   注意所需要的格式，不要直接复制粘贴仓库地址
             *
             * @project       项目名称
             *
             */
            const downloadUrl = templateMap.get(template)
            download(downloadUrl, projectName,{clone: true} , error => {
                if(error){
                    spinner.fail(`下载失败 😭😭😭`)
                    console.log(chalk.bgRgb(220,0,8)(`  创建项目失败：${projectName} `),'😭😭😭')
                    console.log('🧐🧐🧐 失败原因：',chalk.bgRgb(220,0,8)(error.message))
                }else {
                    spinner.succeed(`下载完成：${projectName}`)
                    console.log('✌✌✌',chalk.rgb(69, 39, 160)('成功创建项目  👉  '),chalk.bgRgb(69, 39, 160)(projectName))
                }
            })

        });

    program
        .command('checkAll')
        .description('查看所有的模板')
        .action(function(){
            const templateList = [
                'vue-default-template'
            ]
            templateList.forEach((temp,index) => {
                console.log(chalk.rgb(69, 39, 160)(`(${index+1})  ${temp}`))
            })
        })

    program.parse(process.argv);
}


start()
```