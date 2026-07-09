---
name: jp-garbled-rename
description: 修复 Windows 中文环境下日文(Shift-JIS)乱码文件/文件夹名，并翻译为中文后批量重命名。适用于原名被GBK误解码成乱码的场景。
---

# 日文乱码重命名工作流

适用场景：原环境为日文(Shift-JIS)，在中文(GBK)环境下出现乱码文件名，需要恢复并翻译为中文再重命名。

## 总流程
1. 列出目录结构，确认乱码范围。
2. 通过编码回转还原日文：`GBK -> Shift-JIS`。
3. 生成日文到中文的翻译规则表，先让用户确认关键术语。
4. 批量重命名，最后再做简体化/统一符号处理。
5. 验证结果，输出最终清单。

## 关键命令（PowerShell）

### 1) 还原日文名称（GBK -> Shift-JIS）
```powershell
$gbk=[System.Text.Encoding]::GetEncoding('GBK')
$sjis=[System.Text.Encoding]::GetEncoding(932)
function Fix-Name($s){ try { $sjis.GetString($gbk.GetBytes($s)) } catch { $null } }

Get-ChildItem -LiteralPath $target | ForEach-Object {
  "$($_.Name) -> $(Fix-Name $_.Name)"
}
```

### 2) 生成翻译规则并重命名
```powershell
function Trans-JpToCn($s){
  $t=$s
  $t=$t -replace 'カットイン','插图'
  $t=$t -replace 'マップ','地图'
  # 按需补充更多词条
  return $t
}

Get-ChildItem -LiteralPath $target -File | ForEach-Object {
  $fixed=Fix-Name $_.Name
  $new=Trans-JpToCn $fixed
  if($new -and $new -ne $_.Name){ Rename-Item -LiteralPath $_.FullName -NewName $new }
}
```

### 3) 目录名重命名
```powershell
Get-ChildItem -LiteralPath $root -Directory | ForEach-Object {
  $fixed=Fix-Name $_.Name
  $new=Trans-JpToCn $fixed
  if($new -and $new -ne $_.Name){ Rename-Item -LiteralPath $_.FullName -NewName $new }
}
```

## 注意事项
1. 如出现部分文件仍乱码，可能是“二次乱码”或混合编码，可通过文件大小/内容做手工映射。
2. 避免重名冲突：重命名前先检查目标名是否已存在。
3. 最后再统一简体化（如 `婦人会` -> `妇人会`）和符号规范（`・` 等）。
4. 先在小范围目录试跑，确认规则无误后再批量应用。
