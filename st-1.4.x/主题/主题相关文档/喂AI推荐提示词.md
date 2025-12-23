自定义主题技术规范
1. 模板变量

HTML编译时替换：

{{TITLE}}：页面标题

{{DATE}}：导出日期

{{DATA}}：数据对象（JSON），需 const data = {{DATA}}。

2. 数据结构
code
JavaScript
download
content_copy
expand_less
{
  config: { showMF: bool, showLY: bool }, // 标签页开关
  customTabs: string[],                   // 自定义标签页名
  items: [{
    id, tab, name, ver, quality, note,    // tab默认'LX' ，note(备注,重要)
    links: [{ text, url }],               // 外部链接
    file: { content, size },              // content为Base64(data:url)
    dlLink,                               // 直链URL
    keyConfig: {
      enableUrl, url,                     // 在线Key: url + key
      enableVar, var                      // 本地Key: 替换变量 var
    }
  }]
}
3. 核心逻辑要求
A. 标签页聚合

必须按此顺序生成：['LX'] + showMF?['Music Free'] + showLY?['澜音'] + customTabs。
内容渲染需按 item.tab 筛选。

B. 四种共存下载方式

以下4个判断必须是独立的 if (禁止 else if)。若数据存在，必须可以同时显示对应的4个按钮。

文件下载 (if item.file):

动作：<a href="${item.file.content}" download="${item.name}">

直链下载 (if item.dlLink):

动作：window.open(item.dlLink)

在线Key导入 (if item.keyConfig?.enableUrl):

动作：获取输入 key -> window.open(item.keyConfig.url + key)

本地Key生成 (if item.keyConfig?.enableVar):

动作：获取输入 key -> 执行 Key替换逻辑。

C. 本地Key替换逻辑 (必背代码)

当执行 本地Key生成 时，必须运行以下流程：

code
JavaScript
download
content_copy
expand_less
// 1. 解码: 去掉data:前缀，atob解码
const raw = atob(item.file.content.split(',')[1]);
// 2. 替换: 使用正则匹配变量声明 (const/var/let varName = "...")
const varName = item.keyConfig.var;
const regex = new RegExp(`(const|var|let)\\s+${varName}\\s*=\\s*['"][^'"]*['"]`, 'g');
const newContent = raw.replace(regex, `$1 ${varName} = "${key}"`);
// 3. 下载: 生成Blob并触发下载
const url = URL.createObjectURL(new Blob([newContent]));
const a = document.createElement('a');
a.href = url; a.download = item.name; a.click();

4. 约束

单HTML文件，无外部依赖，CSS/JS内联。