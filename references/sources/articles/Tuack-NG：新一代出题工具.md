:::align{center}
![](https://docs.tuack-ng.ink/assets/icon/favicon-96x96.png)
# Tuack-NG
:::

Tuack-NG 项目是重构后的 Tuack 项目，旨在提供更加高效和轻量的出题体验。

这个项目首次提出于[项目 / 计划：tuack-ng](https://www.luogu.com.cn/article/m2sc8qhd)，历时四个月，这个项目进入了**发布预览**阶段（v0.4.0）。

## 特色

- [x] 使用 Rust 编写，更加高效与轻量
- [x] 迁移了**绝大部分**内容
- [x] 与 Tuack 几乎完全相同的格式
- [x] 使用 Markdown 编写题面，支持单元格合并
- [x] 使用 Typst 替换 Latex 渲染
- [x] 实现更加健壮、通用的数据生成器（Dmk）
- [x] 更加直观的配置文件语法
- [x] 跨平台适配（包括 Nix）

## 兼容性

目前已经测试了以下系统（测试可能不全面，但基本功能保证可用）：

- Windows 10/11
- Arch Linux
- Arch Linux + Nix
- Ubuntu 25.10
- NixOS Unstable

以下系统可以通过编译，但没有条件测试：

- MacOS Intel (Nix)
- MacOS Apple Silicon (Nix)

## 更多信息

仓库：<https://github.com/tuack-ng/tuack-ng>

文档：<https://docs.tuack-ng.ink>，包含安装与使用说明（持续完善中，v1.0 前完成）。

## 需求与 Bug 反馈

欢迎各位在仓库的 Issue 中以及私信提交 Bug 反馈与功能请求。（请不要在评论区反馈，不方便反馈与沟通）

## 许可证

以 [AGPL 3.0](https://www.gnu.org/licenses/agpl-3.0.html) 或更高版本发布。

## 致谢

感谢原 Tuack 项目的原始思路与部分模板。

感谢 CNOI 项目的模板支持。

🌟 如果这个项目对你有帮助，欢迎点亮 Star ~