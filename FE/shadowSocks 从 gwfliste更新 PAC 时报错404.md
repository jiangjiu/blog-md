# shadowSocks 从 gwflist更新 PAC 时404

问题：MacOS下ShadowsocksX点击“从GFWList更新PAC”报“Request failed ： not found（404）”错误。

查看了 github 的 issue,发现是新问题,但是已经有大神解决了,解决方案如下。
下面是大神的 shell 代码，果断观摩之：

[大神的 github 地址](https://gist.github.com/VincentSit/b5b112d273513f153caf23a9da112b3a)

```shell
#!/bin/bash
# update_gfwlist.sh
# Author : VincentSit
# Copyright (c) http://xuexuefeng.com
#
# Example usage
#
# ./whatever-you-name-this.sh
#
# Task Scheduling (Optional)
#
#	crontab -e
#
# add:
# 30 9 * * * sh /path/whatever-you-name-this.sh
#
# Now it will update the PAC at 9:30 every day.
#
# Remember to chmod +x the script.


GFWLIST="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
PROXY="127.0.0.1:1080"
USER_RULE_NAME="user-rule.txt"

check_module_installed()
{
	pip list | grep gfwlist2pac &> /dev/null

	if [ $? -eq 1 ]; then
		echo "Installing gfwlist2pac."

		pip install gfwlist2pac
	fi
}

update_gfwlist()
{
	echo "Downloading gfwlist."

	curl -s "$GFWLIST" --fail --socks5-hostname "$PROXY" --output /tmp/gfwlist.txt

	if [[ $? -ne 0 ]]; then
		echo "abort due to error occurred."
    exit 1
	fi

	cd ~/.ShadowsocksX || exit 1

	if [ -f "gfwlist.js" ]; then
		mv gfwlist.js ~/.Trash
	fi

	if [ ! -f $USER_RULE_NAME ]; then
		touch $USER_RULE_NAME
	fi

	/usr/local/bin/gfwlist2pac \
    --input /tmp/gfwlist.txt \
    --file gfwlist.js \
    --proxy "SOCKS5 $PROXY; SOCKS $PROXY; DIRECT" \
    --user-rule $USER_RULE_NAME \
    --precise

  rm -f /tmp/gfwlist.txt

  echo "Updated."
}

check_module_installed
update_gfwlist
```

运行这个 sh 文件只需要拖进终端，可能还需要sudo 权限。
这个脚本取代了*从 gwfList 更新 PAC*这个按钮的功能，以后新规则写好运行一次就 ok 了~

**osX 测试通过**

## 参考资料
[github 上关于这个问题的 issue](https://github.com/shadowsocks/shadowsocks-iOS/issues/212#issuecomment-220008369)

