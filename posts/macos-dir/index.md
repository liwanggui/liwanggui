# macOS 英文目录显示为中文


> 以目录名为 `Code` 进行讲解

1. 在英文目录下新建 `.localized` 隐藏目录
2. 进入`localized` 目录创建 `zh_CN.strings` 文件, 输入 "english" = "中文";

```bash
mkdir -p Code/.localized
cat Code/.localized/zh_CN.strings
"Code" = "代码";
```

> 注意: 不能少了分号

3. 最后将目录名改为以 `.localized` 结尾即可

```bash
mv Code Code.localized
```
