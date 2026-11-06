# Fire TV 改造提示词模板（AI Agent / cleanyourtv）

> 适用机型：Fire TV Stick 4K Max（2代）。把 `<换成你盒子的IP>` 替换成你自己盒子的 IP 后，整段发给电脑版千问办公即可。
> 预计耗时 30–60 分钟；全程只禁用不卸载，可一条命令还原。

---

帮我用 cleanyourtv 项目改造我的 Fire TV Stick 4K Max（2代）。按顺序做，每完成一步给一张电视截图，最后列「改了什么、怎么还原」：

1、克隆并学习 https://github.com/Gathin23/cleanyourtv ，重点读 PROMPT.md 和 examples/amazon-fire-tv-stick-4k-max-2023/ 清单的头注。全程只用项目里的 Python 版 adb（tools/tv.py），不要用电脑二进制 adb。

2、前置：盒子已开 ADB 调试，和电脑同一网络，IP 是 <换成你盒子的IP>。首次连接电视会弹授权框，我会点「允许」。构建桌面需要 JDK 17 和 Android SDK 34，缺什么你自己装或走仓库的 Docker 构建。

3、先汉化：settings put system system_locales zh-CN，然后提醒我是否重启。重启后设置详情页几分钟内会变中文；最外层设置主页的磁贴文案由亚马逊服务器异步补发，可能要等几十分钟——这不是失败，别急着返工。

4、桌面：Fire TV 拿不到默认 Home（原桌面受保护，也不要去禁它），用仓库 launcher 的 stock_home_redirect 弹回方案：改 config.xml 后构建安装 com.gathin.tvhome，绑 HomeKeyService 无障碍，am start 拉起桌面并验证弹回。桌面要包含应用库（app_library/library_button/library_ok_opens 三键）、CPU 占用率状态行（show_stats）、农历（lunar_date）。

5、debloat：用同板实测清单裁剪「遥测 + 广告」两组来禁用，但把 com.amazon.hedwig、com.amazon.franktvinput、com.amazon.tifobserver（直播免费频道承重包）留在名单外；com.amazon.imp、com.amazon.tv.launcher、Appstore 不要碰。若恢复记录写不进 ~/.cleanyourtv，用 HOME=<仓库>/work/fakehome 跑 disable.sh/restore.sh，并把 ~/.android/adbkey 复制到 work/fakehome/.android/。禁用或启用任何包之后等 10 秒再重启。

6、关自动更新：禁用 com.amazon.tv.forcedotaupdater.v2。

7、主题：把 themes/halloween 复制到仓库外做派生（上游 festival_until 已过期，改一个远期日期），推送后万圣节皮肤和壁纸都要在。

8、插件：只装 addons/remote-access 这一个（快速开关 ADB 用），装完在电视弹出的「新卡片」对话框选「显示卡片」；其他插件都不要装。

9、收尾：验收完成后提醒我关闭 ADB 调试（或用 remote-access 卡一键关），并确认 5555 端口已关。

---

## 开 ADB 调试的方法（发给朋友先做这步）

遥控器路径：**设置 → 我的 Fire TV → 关于 → 对着设备名连按确认键 7 次** → 返回 → **开发者选项 → ADB 调试** 打开。盒子的 IP 在「设置 → 网络」里能看到。

## 朋友常问的三件事

- **要 root 吗？** 不要。所有改动只禁用不卸载，随时可还原。
- **还原怎么做？** 让助手跑 `tools/restore.sh`（按它生成的恢复记录回放），或卸载新桌面应用。
- **为什么设置主页等了很久还是英文？** 磁贴文案由亚马逊服务器异步补发，实测有滞后几十分钟的先例；详情页会先变中文。超过 1 小时仍英文再让助手排查。
