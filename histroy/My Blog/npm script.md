正如阮一峰老师在他的 《npm script使用指南》一文中最开始说的一句话：
> Node 开发离不开 npm，而脚本功能是 npm 最强大、最常用的功能之一。

现在大大小小的项目大都都需要使用相应的 npm script 来进行相应的命令串联或者操作，从而提升开发者的开发体验以及开发效率。

## npm script是什么？<br />
Node.js 在设计 npm 之初，便允许在开发者在 package.json 中，使用 script 字段来定义脚本命令，下图所展示的script字段，就是由我们使用npm init 自动生成的 package.json 文件所包含的：<br /> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1611391376482-3fde32f5-6db6-4393-ab35-e8db08aa92b2.png#align=left&display=inline&height=125&name=image.png&originHeight=250&originWidth=1036&size=45164&status=done&style=none&width=518)<br />有了这个，我们就可以在终端执行：<br /> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1611392757913-6718b76a-4b8e-437c-8094-6782c0410a24.png#align=left&display=inline&height=109&name=image.png&originHeight=218&originWidth=792&size=31554&status=done&style=none&width=396)<br />当然，我们也可以将 `npm run test` 简写为 `npm test`，或者更为简单的 `npm t`。这里的 `npm run` 实际上是 `npm run-script `命令的简写，当我们执行对应的` npm run test `时，其背后做了以下一系列的操作：

- 从 package.json 中读取 `script` 对象里面的全部配置
- 将传给 `npm run` 的第一个参数作为 key，也就是我们上面所输入的 test，然后在第一步生成的 script 对象中获取对应的值来作为接下来要执行的命令，如果没有找到，就直接抛出错误。
- 在系统默认的 shell 中执行所找到的命令。



需要注意的是，`npm run` 的命令，会将当前目录的 `node_modules/.bin` 子目录加入 `PATH` 变量，执行结束后，再将 PATH 变量恢复原样。因此，我们可以在命令中直接用脚本名进行调用，而不需要加入 `PATH` 变量。因此`node_modules/.bin`子目录中的所有脚本都可以直接以脚本名的形式调用，而不必写出完整路径。

通过这样的设计，我们就可以非常方便的统计和维护相互工程化或基建相关的脚本或者命令了。
